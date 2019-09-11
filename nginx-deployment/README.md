**This directory contains the 3 files described in the main [README.md](../README-first-draft.md) file**



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
        image: nginx:1.7.9
        ports:
        - containerPort: 80
```

**[nginx-ingress.yml](nginx-ingress.yml)**
```
apiVersion: extensions/v1beta1
kind: Ingress
metadata:
  name: nginx
spec:
  rules:
  - host: nginx.example.com
    http:
      paths:
      - backend:
          serviceName: nginx
          servicePort: 80
        path: 
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

[Go back to main page](README.md)
