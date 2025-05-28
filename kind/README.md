# Create the K8s cluster

## Create the K8s cluster with kind

```console
❯ cd kind

❯ kind create cluster --config  config.yaml
Creating cluster "cloudnative-pg-poc" ...
 ✓ Ensuring node image (kindest/node:v1.33.1) 🖼
 ✓ Preparing nodes 📦 📦 📦 📦
 ✓ Writing configuration 📜
 ✓ Starting control-plane 🕹
 ✓ Installing CNI 🔌
 ✓ Installing StorageClass 💾
 ✓ Joining worker nodes 🚜
Set kubectl context to "kind-cloudnative-pg-poc"
You can now use your cluster with:

kubectl cluster-info --context kind-cloudnative-pg-poc

Thanks for using kind! 😊
```

## Check the K8s cluster is created and ready

```console
❯ kubectl version
Client Version: v1.32.0
Kustomize Version: v5.5.0
Server Version: v1.33.1

❯ kubectl get nodes
NAME                               STATUS   ROLES           AGE     VERSION
cloudnative-pg-poc-control-plane   Ready    control-plane   4m2s    v1.33.1
cloudnative-pg-poc-worker          Ready    <none>          3m48s   v1.33.1
cloudnative-pg-poc-worker2         Ready    <none>          3m48s   v1.33.1
cloudnative-pg-poc-worker3         Ready    <none>          3m48s   v1.33.1
```

## Delete the k8s cluster with kind

```console
❯ kind delete cluster --name cloudnative-pg-poc
Deleting cluster "cloudnative-pg-poc" ...
Deleted nodes: ["cloudnative-pg-poc-worker2" "cloudnative-pg-poc-worker3" "cloudnative-pg-poc-worker" "cloudnative-pg-poc-control-plane"]
```
