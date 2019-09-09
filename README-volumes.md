# `k3s` and volumes

`k3s` does not come with any pre-installed StorageClass'es for PersistentVolumes (used by PersistenVolumeClaims in Deployments).

You can install [local-path-provisioner](https://github.com/rancher/local-path-provisioner) to do the job for you - it will enable you to utilize the local disk that `k3s` is running on.

## Install `local-path-provisioner`

```
kubectl create -f https://raw.githubusercontent.com/rancher/local-path-provisioner/master/deploy/local-path-storage.yaml
```

Wait a little for the `local-path-storage` system to be setup. You can monitor the `local-path-storage` namespace or the `rollout status`

```bash
kubectl -n local-path-storage rollout status deployment local-path-provisioner
```

After that you should be able to add local volumes (PVC's) like so:

```yaml
apiVersion: v1
kind: PersistentVolumeClaim
metadata:
  name: local-path-pvc
  namespace: default
spec:
  accessModes:
    - ReadWriteOnce
  storageClassName: local-path
  resources:
    requests:
      storage: 2Gi
```

You can find the install and usage guide on [https://github.com/rancher/local-path-provisioner](https://github.com/rancher/local-path-provisioner)


[Go back to main page](README.md)