**This directory contains the 3 files described in the main [README.md](../README-first-draft.md) file** and the nginx-all-with-pvc.yml to deploy it all with a volume mounted that you can use - see below.



**[nginx-deployment.yml](nginx-deployment.yml)**
```
apiVersion: apps/v1
kind: Deployment
metadata:
  name: nginx
spec:
  selector:
    matchLabels:
      app: nginx
  replicas: 2
  template:
    metadata:
      labels:
        app: nginx
    spec:
      containers:
      - name: nginx
        image: nginx:stable
        ports:
        - containerPort: 80
```

**[nginx-ingress.yml](nginx-ingress.yml)**
```
apiVersion: networking.k8s.io/v1
kind: Ingress
metadata:
  name: nginx
spec:
  rules:
  - host: nginx.example.com
    http:
      paths:
        - path: /
          pathType: Prefix
          backend:
            service:
              name: nginx
              port: 
                number: 80
```

**[nginx-service.yml](nginx-service.yml)**
```
apiVersion: v1
kind: Service
metadata:
  name: nginx
spec:
  ports:
  - port: 80
    protocol: TCP
    targetPort: 80
  type: NodePort
  selector:
    app: nginx
```

# Deploying it all in one

## With PVC (volume for static Nginx content)

Deploy `nginx-all-with-pvc.yml` with 

```bash
kubectl apply -f nginx-all-with-pvc.yml
```

## Accessing Nginx using curl (or your browser)

Since the file says it uses "nginx.example.com" then you need to either 
1. setup a /etc/hosts entry to point to your IP address
2. use the Host header when accessing the IP address


## Setting up /etc/hosts

Get the IP address of the Ingress Controller in k3s
```bash
kubectl -n kube-system get svc 
```
Look for and note down the IP address in the `EXTERNAL-IP` column that is your IP address.

Add the following to your /etc/hosts file
```bash
echo "<the IP address above>   nginx.example.com" | sudo tee -a /etc/hosts
```

After that you can run 
```bash
curl http://nginx.example.com
```

You should see a "401 Access Denied" or similar as there are no index.html file in the Nginx html directory

## Using Host header in curl

If your IP address changes "all the time" and you will only be working on the CLI using curl then you can just use the following command:
```bash
curl -H "Host: nginx.exmaple.com" http://localhost
```
You should see a "401 Access Denied" or similar as there are no index.html file in the Nginx html directory

The Ingress Controll sends incoming traffic to the correct Service using the Host header.

## Accessing the Nginx HTML directory (DocumentRoot in apache)

The docker image deployed in the Pod in the Deployment has a local path mounted to /usr/share/nginx/html which you can find using the following command

```bash
kubectl -n nginx get pv -o jsonpath='{.items[].spec.hostPath.path}'
```

The directoy is owned by `root` and your user cannot access it without `sudo`, so run the following:

```bash
export NGINX_HTML_PATH=$(k -n nginx get pv -o jsonpath='{.items[].spec.hostPath.path}')
sudo chown -R $USER ${NGINX_HTML_PATH}
```

Setup a local symbolik link to that path 
```bash
export NGINX_HTML_PATH=$(k -n nginx get pv -o jsonpath='{.items[].spec.hostPath.path}')
ln -s ${NGINX_HTML_PATH} ${HOME}/nginx-html-directory
cd 
cd nginx-html-directory
echo "<html><body>Welcome to Nginx on k3s</body></html>" > index.html
```

After adding the above index.html then you should be able to see your custom index.html file using curl (or your browser depending on your setup)

```bash
curl http://nginx.example.com
curl -H "Host: nginx.example.com" http://localhost
```

The output should be what is in your index.html file eg :
```html
<html><body>Welcome to Nginx on k3s</body></html>
```


[Go back to main page](README.md)
