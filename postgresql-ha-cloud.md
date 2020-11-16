# Strategy for using dedicated servers for PG

## Criteria of Success

-   PG runs on localpv set on the two new PROD servers
-   Master and Slave workloads are never scheduled in the same node
-   Helper Pod used to do migration from Ceph to localPV can be scheduled so it can mounts successfully both, Ceph and localPV
-   We block the two new servers so no workloads, except PG, can run there
-   We eventually can allow other workloads to run in the new nodes in the future

## Points to consider

This looks pretty similar to  [Dedicated Nodes Use Case](https://kubernetes.io/docs/concepts/configuration/taint-and-toleration/#example-use-cases)

-   we mark the dedicated nodes by tainting them using something like:
    
    ```
    kubectl taint nodes newNode1 db-node=master:NoSchedule
    kubectl taint nodes newNode2 db-node=slave:NoSchedule
    
    ```
    
-   we add tolerations for PG master
    
    ```yaml
    tolerations:
    - key: "db-node"
      operator: "Equal"
      value: "master"
      effect: "NoSchedule"
    
    ```
    
-   we add tolerations for PG slave
    
    ```yaml
    tolerations:
    - key: "db-node"
      operator: "Equal"
      value: "slave"
      effect: "NoSchedule"
    
    ```
    
-   the pods with the tolerations will be allowed to use the tainted (dedicated) nodes  **as well as any other nodes in the cluster**.
-   however, local PV will force the PG workloads to be scheduled in the right nodes.
-   we want also to ensure that the db workloads for master and slave are not scheduled in the same node
-   although the above constraint is achieved through node anti-affinity, this is not supported by PG bitnami chart
-   this previous constraint can be fulfilled by  **adding a label**  to the tainted nodes (e.g.,  `db-node=master`  `db-node=slave`), and by  **adding node affinity**  configuration to enforce that the pods can only be scheduled onto nodes labeled accordingly. For doing so we’ll make use of parameters  `postgresql.master.affinity`  and  `postgresql.slave.affinity`
-   would failover still work (promote slave to master) ???

## Pseudo-MOP

0.  Pre-requisites

-   create a test cluster in GKE with 3 nodes
-   set kubectl context to use the test cluster

2.  Set env

```
ENVIRONMENT=test
NAMESPACE=mypg
RELEASE=mypg

```

3.  write the pg overrides

```yaml
cat << EOF > $ENVIRONMENT-pg10-overrides.yaml
fullnameOverride: integrity-db
postgresqlUsername: integrity
postgresqlPassword: integritypwd
postgresqlDatabase: integrity

replication:
  enabled: true
  user: repl_user
  password: repl_password
  slaveReplicas: 1

master:
  # allow pg master workload to use tainted pg dedicated node
  tolerations:
  - key: "db-node"
    operator: "Equal"
    value: "master"
    effect: "NoSchedule"
  # ensure pg master workload run only onto that node
  affinity:
    nodeAffinity:
      requiredDuringSchedulingIgnoredDuringExecution:
        nodeSelectorTerms:
        - matchExpressions:
          - key: db-node
            operator: In
            values:
            - master
slave:
  # allow pg slave workload to use tainted pg dedicated node
  tolerations:
  - key: "db-node"
    operator: "Equal"
    value: "slave"
    effect: "NoSchedule"
  # ensure pg slave workload run only onto that node
  affinity:
    nodeAffinity:
      requiredDuringSchedulingIgnoredDuringExecution:
        nodeSelectorTerms:
        - matchExpressions:
          - key: db-node
            operator: In
            values:
            - slave
EOF

```

> NOTE: for PROD, the overrides should include config parameters (DEV-111) and storage-class localpv-pg (DEV-100)

3.  taint the nodes

```shell
PG_MASTER_NODE=gke-sprint-test-default-pool-917f4d1e-gvsl
PG_SLAVE_NODE=gke-sprint-test-default-pool-917f4d1e-m8wd

kubectl taint nodes $PG_MASTER_NODE db-node=master:NoSchedule
kubectl taint nodes $PG_SLAVE_NODE db-node=slave:NoSchedule

# list the taints in the nodes
kubectl get nodes -o json | jq '.items[].spec'

```

> NOTE: for PROD, when adding the node to rancher append  `--taints key=value:effect`, that is,  `--taints db-node=master:NoSchedule`  and  `--taints db-node=slave:NoSchedule`  to master and slave node respectively

4.  set label to the master and slave  
    With a localPV configuration, the PV contains node affinity information that the system uses to schedule pods to the correct nodes. However, we require anti-affinity so that master and slave are not scheduled in the same node, but anti-affinity is not supported in the PG bitnami chart so the idea is to use enforce scheduling to separate servers using node label and affinity for master and slave.

```
kubectl label nodes $PG_MASTER_NODE db-node=master
kubectl label nodes $PG_SLAVE_NODE db-node=slave

# list the labels in the nodes
kubectl get nodes -o json | jq '.items[].metadata.labels'

```

> NOTE: for PROD, when adding the node to rancher, append  `--label key=value`, that is,  `--label db-node=master`  and  `--label db-node=master`  to master and slave nodes respectively

5.  create a non-db workload

```
cat << EOF > nondb-pod.yaml
apiVersion: v1
kind: Pod
metadata:
 name: nondb-pod
 namespace: $NAMESPACE
spec:
 containers:
 - name: helper
   image: busybox:latest
   command: ['sh', '-c', 'echo Helper pod, please exec into me and work... ; tail -f /dev/null']
EOF

kubectl apply -f nondb-pod.yaml

```

6.  install pg chart 3.9.2

```
helm install --name $RELEASE stable/postgresql \
  --namespace $NAMESPACE \
  --version 3.9.2 \
  -f myvalues.yaml \
  -f $ENVIRONMENT-pg10-overrides.yaml

```

7.  check if we meet CoS

```
kubectl get nodes -o json | jq '.items[].spec'
kubectl get pods -n $NAMESPACE -o wide

echo "master node expected: $PG_MASTER_NODE" && \
echo "master node actual..: `kubectl get pods -n $NAMESPACE -o wide | grep integrity-db-master-0 | awk '{print $7}'`" && \
echo "slave node expected.: $PG_SLAVE_NODE" && \
echo "slave node actual...: `kubectl get pods -n $NAMESPACE -o wide | grep integrity-db-slave-0 | awk '{print $7}'`"
```

-   integrity-db-master-0 is scheduled in node $PG_MASTER_NODE
-   integrity-db-slave-0 is scheduled in node $PG_SLAVE_NODE
-   new non-pg workload are not scheduled in tainted nodes
-   does PG failover still work?  
    _although it worked ok in a GKE cluster where PG storage is on ceph, PG failover MOP might not work with a localPV setup unless we switch labels in nodes such that, master become slave, slave become master and previous to step 11 in failover MOP, we delete the entire content of the PG data folder_

8.  cleanup

```
 kubectl delete -f nondb-pod.yaml
 helm delete $RELEASE --purge
```

## Next step
To excercise this on pantera. That is, add there a new node (pantera-k8s-6, pantera-k8s-7), setting there the localPv partitions, and make them dedicated PG server for master and slave.


# Hostpath and localpv
## Pre-requisites

1.  Backup integrity database. This db backup shall be used only in case of a contingency. In a normal migration scenario the data will be moved from running 10.6 instance to 12.1 directly.
    
2.  Make sure we have enough storage for having an extra PV with the db data
    
3.  Stop all mediation and discovery jobs through NiFi UI
    

## Preparation (RKE node)
1. Connect to the RKE node that will host the local pv
BF: ssh to RKE node 68.30.132.50
```
export ENVIRONMENT=breakfix
export RELEASE=integrity-test
export NAMESPACE_SRC=by-integrity-test
```
```
export HOST_PATH=/${ENVIRONMENT}/tcc/integrity/pg12_data
sudo mkdir -p $HOST_PATH
sudo chmod 777 $HOST_PATH
```
## Upgrade Steps (Rancher node)
1.  Initialize variables for upgrade
```
# manually set these variables according to the environment
export NAMESPACE_SRC=by-integrity-test
export NAMESPACE_TGT=by-integrity-test
export RELEASE=integrity-test
export DATABASE_NAME=integrity
export MASTER_CLAIM_BASENAME=pgdata-pv-claim
#export SLAVE_CLAIM_BASENAME=data-integrity-db-slave-0

export ENVIRONMENT=breakfix
export HOST_PATH=/${ENVIRONMENT}/tcc/integrity/pg12_data

# other variable set
export PG10_MASTER_PVC=${MASTER_CLAIM_BASENAME}-pg10
export PG10_MASTER_VOLUME=`kubectl get pv | grep ${NAMESPACE_SRC}/${MASTER_CLAIM_BASENAME} | awk '{print $1}'`
#export PG10_SLAVE_VOLUME=`kubectl get pv | grep #${NAMESPACE_SRC}/${SLAVE_CLAIM_BASENAME} | awk '{print $1}'`
export STORAGE_SIZE=`kubectl get pv | grep ${NAMESPACE_SRC}/${MASTER_CLAIM_BASENAME} | awk '{print $2}'`
export STORAGE_CLASS=`kubectl get pv | grep ${NAMESPACE_SRC}/${MASTER_CLAIM_BASENAME} | awk '{print $7}'`

echo "####" $(date) "####" >> pg-migration-env
echo export NAMESPACE_SRC"="$NAMESPACE_SRC >> pg-migration-env
echo export NAMESPACE_TGT"="$NAMESPACE_TGT >> pg-migration-env
echo export RELEASE"="$RELEASE >> pg-migration-env
echo export DATABASE_NAME"="$DATABASE_NAME >> pg-migration-env
echo export MASTER_CLAIM_BASENAME"="$MASTER_CLAIM_BASENAME >> pg-migration-env
#echo export SLAVE_CLAIM_BASENAME"="$SLAVE_CLAIM_BASENAME >> pg-migration-env
echo export PG10_MASTER_PVC"="$PG10_MASTER_PVC >> pg-migration-env
echo export PG10_MASTER_VOLUME"="$PG10_MASTER_VOLUME >> pg-migration-env
#echo export PG10_SLAVE_VOLUME"="$PG10_SLAVE_VOLUME >> pg-migration-env
echo export STORAGE_SIZE"="$STORAGE_SIZE >> pg-migration-env
echo export STORAGE_CLASS"="$STORAGE_CLASS >> pg-migration-env

# make sure all and every variable has a non blank value set
cat pg-migration-env
```
    
2.  Scale down PG 10 master and slave to 0
```
kubectl scale sts/integrity-db-master --replicas=0 -n ${NAMESPACE_SRC}
#kubectl scale sts/integrity-db-slave --replicas=0 -n ${NAMESPACE_SRC}
```
    
3.  Delete current v10 master and slave PVCs. Master volume is retained to be attached to new PVC
```
# unbound master pg10 pvc-pv
kubectl patch pv $PG10_MASTER_VOLUME -p '{"spec":{"persistentVolumeReclaimPolicy":"Retain"}}'
kubectl delete pvc ${MASTER_CLAIM_BASENAME} -n ${NAMESPACE_SRC}
kubectl patch pv $PG10_MASTER_VOLUME -p '{"spec":{"claimRef": null}}'

# delete slave pg10 pvc-pv
#kubectl delete pvc ${SLAVE_CLAIM_BASENAME} -n ${NAMESPACE_SRC}
# TODO: make sure the pv bound to the pvc is also deleted
```
4. Re-create the master pg10 pvc and bound it to old pg10 pv
> is this really needed? why not do not delete it in step 3 instead?
```yaml
# recreate master pg10 pvc bound to old pg10 pv
cat << EOF | kubectl apply -f -
apiVersion: v1
kind: PersistentVolumeClaim
metadata:
  finalizers:
  - kubernetes.io/pvc-protection
  labels:
    app: postgresql
    release: $RELEASE
    role: master
  name: $PG10_MASTER_PVC
  namespace: $NAMESPACE_SRC
spec:
  accessModes:
  - ReadWriteOnce
  resources:
    requests:
      storage: $STORAGE_SIZE
  storageClassName: $STORAGE_CLASS
  volumeName: $PG10_MASTER_VOLUME
EOF
```
5. Provision new PV (hostPath) and PVC for the new pg 12 master
```yaml
cat << EOF | kubectl apply -n ${NAMESPACE_SRC} -f -
apiVersion: v1
kind: PersistentVolume
metadata:
  name: pgdata-pv-volume-pg12
  labels:
    type: local
spec:
  storageClassName: manual
  capacity:
    storage: ${STORAGE_SIZE}
  accessModes:
    - ReadWriteOnce
  persistentVolumeReclaimPolicy: Retain
  hostPath:
    path: "${HOST_PATH}"
---
apiVersion: v1
kind: PersistentVolumeClaim
metadata:
  name: pgdata-pv-claim
spec:
  storageClassName: manual
  accessModes:
    - ReadWriteOnce
  resources:
    requests:
      storage: ${STORAGE_SIZE}
EOF
```

7.  Delete master and slave statefulsets
```
kubectl delete sts integrity-db-master -n $NAMESPACE_SRC
#kubectl delete sts integrity-db-slave -n $NAMESPACE_SRC
```

8.  Upgrade integrity to the desired version - this will upgrade integrity with new postgresql 12.1.0
> **Checkpoint:** before running the next commands, make sure you have **capacity-overrides.yaml** and **hostpath-values.yaml** under your home directory
```
cd ~/charts/2.1.0-pr887ariel
helm upgrade integrity-test ./integrity --install --wait --recreate-pods --force --timeout 1000 --namespace by-integrity-test --values ./integrity/break-fix-values.yaml --values $HOME/capacity-overrides.yaml --values $HOME/hostpath-values.yaml --values ./integrity/images.yaml
```    
9.  Set a password for postgres user in PG 12
```
export PGPASSWORD=<YOUR SECRET PASSWORD>
kubectl exec integrity-db-master-0 -n ${NAMESPACE_SRC} -- sh -c "psql -U postgres -c \"ALTER USER postgres with password '$PGPASSWORD'\""
```

10.  Create pghelper pod to fix permissions
```yaml
export NAMESPACE_TGT=by-integrity-test
export NODE_WHERE_SRC_DB_IS_RUNNING=`kubectl get pods -n $NAMESPACE_TGT -o wide | grep integrity-db-master-0 | awk '{print $7}'`
export IMAGE=docker.registry:5000/bitnami/minideb:latest
export PG10_MASTER_PVC=pgdata-pv-claim-pg10

# create helper pod mounting pg 10 data
cat << EOF | kubectl apply -n $NAMESPACE_SRC -f -
apiVersion: v1
kind: Pod
metadata:
  name: pghelper
spec:
  containers:
  - name: postgresql
    image: $IMAGE
    command:
      - sh
      - -c
      - |
        sleep 20m
    securityContext:
      procMount: Default
      runAsUser: 0
    volumeMounts:
    - name: data
      mountPath: /bitnami/postgresql
  securityContext:
    fsGroup: 931822
    runAsUser: 931822
  nodeSelector:
    kubernetes.io/hostname: $NODE_WHERE_SRC_DB_IS_RUNNING
  volumes:
  - name: data
    persistentVolumeClaim:
      claimName: $PG10_MASTER_PVC
EOF
```

11. Fix permissions
```
# connect to pghelper
export POD=$(kubectl get pods -n ${NAMESPACE_SRC} | grep pghelper | awk '{print $2}')
kubectl exec -it ${POD} -n ${NAMESPACE_SRC} -- bash -c "chown -R 931822:931822 /bitnami;chmod  0700 /bitnami/postgresql/data;"
```

12.  Re-create pghelper pod mounting PG 10 data for migration
```yaml
export NAMESPACE_TGT=by-integrity-test
export NODE_WHERE_SRC_DB_IS_RUNNING=`kubectl get pods -n $NAMESPACE_TGT -o wide | grep integrity-db-master-0 | awk '{print $7}'`
export IMAGE=docker.registry:5000/bitnami/postgresql:10.6.0
export PG10_MASTER_PVC=pgdata-pv-claim-pg10

# create helper pod mounting pg 10 data
cat << EOF | kubectl apply -n $NAMESPACE_SRC -f -
apiVersion: v1
kind: Pod
metadata:
  name: pghelper
spec:
  containers:
  - name: postgresql
    image: $IMAGE
    volumeMounts:
    - name: data
      mountPath: /bitnami/postgresql
  securityContext:
    fsGroup: 931822
    runAsUser: 931822
  nodeSelector:
    kubernetes.io/hostname: $NODE_WHERE_SRC_DB_IS_RUNNING
  volumes:
  - name: data
    persistentVolumeClaim:
      claimName: $PG10_MASTER_PVC
EOF
```

13.  Move data from PG 10 to 12
```
# connect to pghelper
export POD=$(kubectl get pods -n ${NAMESPACE_SRC} | grep pghelper | awk '{print $2}')
kubectl exec -it ${POD} -n ${NAMESPACE_SRC} -- bash -c "export DATABASE_NAME=${DATABASE_NAME};export PGPASSWORD=$PGPASSWORD;export NAMESPACE_TGT=$NAMESPACE_TGT;bash"

# make sure there are no connections to the database
PGPASSWORD=$PGPASSWORD psql -h integrity-db.$NAMESPACE_TGT.svc.cluster.local -U postgres -c "SELECT pg_terminate_backend(pg_stat_activity.pid) FROM pg_stat_activity WHERE pg_stat_activity.datname = '$DATABASE_NAME' AND pid <> pg_backend_pid();"

# drop PG 12 database
PGPASSWORD=$PGPASSWORD psql -h integrity-db.$NAMESPACE_TGT.svc.cluster.local -U postgres -c "drop database $DATABASE_NAME"

# move PG 10 database onto PG 12 instance!!!
time pg_dump -C -h localhost -U postgres $DATABASE_NAME | PGPASSWORD=$PGPASSWORD psql -h integrity-db.$NAMESPACE_TGT.svc.cluster.local -U postgres
```
14. Scale helper to zero
```
kubectl scale sts/pghelper --replicas=0 -n ${NAMESPACE_SRC}
```
    
15.  Analyze statistics
```
kubectl exec -ti -n ${NAMESPACE_SRC} integrity-db-master-0 -- bash -c "time vacuumdb -U postgres -d $DATABASE_NAME --analyze-only --verbose"
```

> File **pg-migration-env** file created in first step, contains names of PVCs, PVs, and other information that is going to be necessary for either permanently reclaiming space (by removing old PG 10 data) or for reverting back to PG 10.6. **Please do not delete this file** but keep it safe for some time after the migration.

## Testing

1.  Test UI
2.  Test parsers
3.  Remove pghelper pod
    ```
    kubectl delete pod pghelper -n ${NAMESPACE_SRC}
    ```