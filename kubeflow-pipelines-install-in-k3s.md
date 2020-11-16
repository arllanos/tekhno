# Installing Kubeflow Pipelines in K3s

## Deploy Kubeflow Pipelines

```bash
export PIPELINE_VERSION=1.0.4

kubectl apply -k "github.com/kubeflow/pipelines/manifests/kustomize/cluster-scoped-resources?ref=$PIPELINE_VERSION"

kubectl wait --for condition=established --timeout=60s crd/applications.app.k8s.io

kubectl apply -k "github.com/kubeflow/pipelines/manifests/kustomize/env/platform-agnostic-pns?ref=$PIPELINE_VERSION"

kubectl get pods -n kubeflow
```

## Verify that the Kubeflow Pipelines UI is accessible by port-forwarding
```bash
kubectl port-forward -n kubeflow svc/ml-pipeline-ui 8081:80
```
Then, open the Kubeflow Pipelines UI at http://localhost:8081/

## Accessing other services
```bash
# mysql
kubectl port-forward -n kubeflow svc/mysql 3306
# minio
kubectl port-forward -n kubeflow svc/minio-service 9000
```
Then, open the corresponding UI at http://localhost:<forwarded_port>/

## Uninstalling Kubeflow Pipelines 
```bash
export PIPELINE_VERSION=1.0.4

kubectl delete -k "github.com/kubeflow/pipelines/manifests/kustomize/env/platform-agnostic-pns?ref=$PIPELINE_VERSION"

kubectl delete -k "github.com/kubeflow/pipelines/manifests/kustomize/cluster-scoped-resources?ref=$PIPELINE_VERSION"
```
## Credits
Adapted from [Deploying Kubeflow Pipelines on a local cluster](https://www.kubeflow.org/docs/pipelines/installation/localcluster-deployment/#k3s-on-windows-subsystem-for-linux-wsl).
