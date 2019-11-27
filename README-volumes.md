# `k3s` and volumes

`k3s` does not come with any pre-installed StorageClass'es for PersistentVolumes (used by PersistenVolumeClaims in Deployments).

You can install [local-path-provisioner](https://github.com/rancher/local-path-provisioner) to do the job for you - it will enable you to utilize the local disk that `k3s` is running on.

## Install `local-path-provisioner`

```
# Make the required directory for the local-path-provisioner to store its data in aka the PV's where the PVC's are stored
sudo mkdir /opt/local-path-provisioner/

# Install local-path-provisioner
kubectl create -f https://raw.githubusercontent.com/rancher/local-path-provisioner/master/deploy/local-path-storage.yaml
```

Wait a little for the `local-path-storage` system to be setup. You can monitor the `local-path-storage` namespace or the `rollout status`

```bash
kubectl -n local-path-storage rollout status deployment local-path-provisioner
```

After that you should be able to add local volumes (PVC's) like so (if you installed the nginx deployment in [README.md](README.md):

**Create a file named `nginx-pvc.yml` with the following content:
```yaml
apiVersion: v1
kind: PersistentVolumeClaim
metadata:
  name: nginx-html
  namespace: nginx
spec:
  accessModes:
    - ReadWriteOnce
  storageClassName: local-path
  resources:
    requests:
      storage: 2Gi
```
Apply the file:
```bash
kubectl -n nginx create -f pvc.yml
```
Check that the PVC is being created (it should be in **Pending** mode)
```bash
kubectl -n nginx get pv,pvc,pods
```

Update the nginx deployment's containers section with the last 7 lines like shown here:
```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: nginx
spec:
  selector:
    matchLabels:
      app: nginx
  replicas: 1
  template:
    metadata:
      labels:
        app: nginx
    spec:
      containers:
      - name: nginx
        image: nginx:1.7.9
        ports:
        - containerPort: 80
        volumeMounts:
        - name: html
          mountPath: /usr/share/nginx/html
      volumes:
      - name: html
        persistentVolumeClaim:
          claimName: nginx-html
```

Re-check the PV, PVC 
```bash
kubectl -n nginx get pv,pvc,pods
```
The output should now be something like this:
```bash
kubectl -n nginx get pv,pvc,pod
NAME                                                        CAPACITY   ACCESS MODES   RECLAIM POLICY   STATUS   CLAIM           STORAGECLASS   REASON   AGE
persistentvolume/pvc-1c1f0987-d4d3-11e9-b617-0800277d4863   2Gi        RWO            Delete           Bound    nginx/htmldir   local-path              6m46s

NAME                            STATUS   VOLUME                                     CAPACITY   ACCESS MODES   STORAGECLASS   AGE
persistentvolumeclaim/htmldir   Bound    pvc-1c1f0987-d4d3-11e9-b617-0800277d4863   2Gi        RWO            local-path     7m5s

NAME                         READY   STATUS    RESTARTS   AGE
pod/nginx-7cb66fc985-vpxrb   1/1     Running   0          6m30s
```

Under the directory `/opt/local-path-provisioner/` you should now have a directory named after the PVC aka `pvc-1c1f0987-d4d3-11e9-b617-0800277d4863`.
If you place an `index.html` file there (**as root**) then you'll be able to see it via Nginx on [http://nginx.example.com](http://nginx.example.com) or via curl: 
```bash
curl http://nginx.example.com
```


## Official installation documentation and examples on GitHub

You can find the install and usage guide on [https://github.com/rancher/local-path-provisioner](https://github.com/rancher/local-path-provisioner)

[Go back to main page](README-first-draft.md)
