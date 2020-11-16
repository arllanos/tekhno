# Kubeflow Pipelines development

## Provisioning kubernetes cluster
For development you need a local or remote cluster so Pipelines code can connect to services like MinIO, Mysql or even then Kubernetes API Server.
For a local Kubernetes cluster, [K3S](https://k3s.io/) is recommended. It is a lightweight and fully functional certified distribution of Kubernetes by Rancher.

- [Install k3s on Linux/Mac](https://k3s.io/).
- [Install k3s on WSL 2](https://github.com/arllanos/tekhno/blob/master/k3s-install.md).

## Setting up local dev environment for Kubeflow Pipelines Backend
1. Install Go 1.13.x

2. Install dependencies
```
apt-get update && apt-get install -y cmake clang musl-dev openssl
```

3. [Install Kubeflow pipelines](https://github.com/arllanos/tekhno/blob/master/kubeflow-pipelines-install-in-k3s.md).

4. Edit `backend/src/apiserver/config/config.json` to point to your dev Mysql and Minio instances.
The following config has been added
- `DBConfig.Host`
- `ObjectStoreConfig.Host`

Optionally change `DBName` and `BucketName` to use separate DB and Bucket for dev
```json
{
  "DBConfig": {
    "Host": "127.0.0.1",
    "DriverName": "mysql",
    "DataSourceName": "",
    "DBName": "devmlpipeline",
    "GroupConcatMaxLen": "4194304"
  },
  "ObjectStoreConfig": {
    "Host": "127.0.0.1",
    "AccessKey": "minio",
    "SecretAccessKey": "minio123",
    "BucketName": "devmlpipeline",
    "PipelinePath": "pipelines"
  },
  "InitConnectionTimeout": "6m",
  "DefaultPipelineRunnerServiceAccount": "pipeline-runner",
  "CacheEnabled": "true",
  "SharedPipelinesEnabled": "true"
}

```
5. Compile
```bash
GO111MODULE=on go build -o bin/apiserver backend/src/apiserver/*.go
```

6. Hack so local code run as in-cluster
> This need to be repeated after each computer and/or cluster restart
```bash
# copy in-cluster service account at /var/run/secrets/kubernetes.io/serviceaccount to local dev
sudo mkdir -p /var/run/secrets/kubernetes.io/serviceaccount

POD=$(kubectl get pods -n kubeflow -l app=ml-pipeline -o jsonpath='{.items[0].metadata.name}')

kubectl exec -ti $POD -n kubeflow -- cat /var/run/secrets/kubernetes.io/serviceaccount/ca.crt > $HOME/ca.crt
kubectl exec -ti $POD -n kubeflow -- cat /var/run/secrets/kubernetes.io/serviceaccount/token > $HOME/token

sudo mv $HOME/ca.crt /var/run/secrets/kubernetes.io/serviceaccount
sudo mv $HOME/token /var/run/secrets/kubernetes.io/serviceaccount

# copy samples to /samples in local dev
kubectl cp kubeflow/$POD:/samples/ $HOME/samples/
sudo mv  $HOME/samples /
```

7. Expose cluster services locally
```bash
# expose kubernetes API server on localhost
kubectl proxy --port=8080 &

# expose mysql
kubectl port-forward -n kubeflow svc/mysql 3306 &

# expose minio
kubectl port-forward -n kubeflow svc/minio-service 9000 &

# expose visualization server (note this will listen on 8889 locally)
kubectl port-forward -n kubeflow svc/ml-pipeline-visualizationserver 8889:8888 &
```

8. Configure `launch.json` to be able to debug in vscode.
```json
{
    "version": "0.2.0",
    "configurations": [
        {
            "name": "Launch",
            "type": "go",
            "request": "launch",
            "mode": "auto",
            "program": "${workspaceFolder}/backend/src/apiserver",
            "env": {
                "KUBERNETES_SERVICE_HOST":"127.0.0.1",
                "KUBERNETES_SERVICE_PORT": "8080",
                "ML_PIPELINE_VISUALIZATIONSERVER_SERVICE_HOST": "127.0.0.1",
                "ML_PIPELINE_VISUALIZATIONSERVER_SERVICE_PORT": "8889"
            },
            "args": [
                "--config=backend/src/apiserver/config",
                "--sampleconfig=config/sample_config.json",
                "-logtostderr=true"]
        }
    ]
}
```
9. You can now debug Pipelines apiserver locally in vscode.

## Build and push changed images
```bash
export DOCKER_USER=<myuser>
export DOCKER_PASSWORD=<mypassword>

# To build the API server image and upload it to GCR on x86_64 machines:
echo $DOCKER_PASSWORD |docker login $DOCKER_REGISTRY --username=$DOCKER_USER --password-stdin

# tag choose tagging for either docker hub or private registry
IMAGE_TAG=$DOCKER_REGISTRY/$DOCKER_USER
# IMAGE_TAG=ml-pipeline/api-server

if [ $MACHINE_ARCH == "aarch64" ]; then
    # build API server image oad it to GCR on x86_64 machines:
    docker build -t bazel:0.24.0 -f backend/Dockerfile.bazel .
    docker build -t "${IMAGE_TAG}" -f backend/Dockerfile --build-arg BAZEL_IMAGE=bazel:0.24.0 .
else
    docker build -t "${IMAGE_TAG}" -f backend/Dockerfile .
fi

docker push ${IMAGE_TAG}/api-server:latest
```

## Backend deployments / image

| NAME | SRC CODE PATH | IMAGE |
|---|---|---|
| ml-pipeline | backend/src/apiserver| api-server |
| ml-pipeline-scheduledworkflow | backend/src/crd/controller/scheduledworkflow | scheduledworkflow |
| ml-pipeline-viewer-crd | backend/src/crd/controller/viewer | viewer-crd-controller |
| ml-pipeline-persistenceagent | backend/src/agent/persistence | persistenceagent |
| ml-pipeline-visualizationserver | backend/src/apiserver/visualization | visualization-server|
| cache-deployer-deployment | backend/src/cache/deployer | cache-deployer |
| cache-server | backend/src/cache | cache-server |


## Frontend deployments / image
| NAME | SRC CODE PATH | IMAGE |
|---|---|---|
| ml-pipeline-ui | frontend | frontend |

## Other deployments / image
| NAME | SRC CODE PATH | IMAGE |
|---|---|---|
| metadata-writer | | |
| metadata-grpc-deployment | | |
| metadata-envoy-deployment | | |
| controller-manager | | |
| minio | | |
| mysql| | |
| workflow-controller | | |
