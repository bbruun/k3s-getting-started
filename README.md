# k3s-getting-started

Getting started with k3s - 5 minus k8s

This is a quick installation guide for [k3s](https://k3s.io) with the setup of `kubectl` to get started including a smalll `nginx` deployment.


# What is k3s

k3s is a stripped version of the official fullblown Kubernetes source where the implementations for Amazon, Google, Azure hosting centers have been pulled and some of the subsystems (drivers) for Volumes etc. have been pulled so you can run a full blown Kubernets cluster at home or on remote places on small devices like a Rasberry Pi and still use the tools you do for managing a Kubernetes cluster.

k3s is also a great alternative for [minikube](https://kubernetes.io/docs/setup/learning-environment/minikube/) ans similar that run the full blown Kubernets and hence the requirements.

Read more about `k3s` at [k3s](https://k3s.io)


This guide is split in 4 sections

- installing `k3s` making it ready for use
- setup k3s as a systemd service
- setup `kubectl` to interact with `k3s`
- deploy a simple `nginx` service


# Installation requirements (linux description only)

To install k3s make sure you have the following

1. A fully functioning x86_64 bit GNU/Linux installation (server or desktop will do)
2. Approx 500MB free RAM at your disposal
3. A few GB of free space (Docker images take up space - they are not free from reality)
4. Root access (you need root to run k3s)
5. Internet access

# Install k3s (linux description only)


## Step 1: Download and place binary the correct place
Go to [https://github.com/rancher/k3s/releases](https://github.com/rancher/k3s/releases) and download the latest stable version (aka not one ending in `rc1` or `beta` etc).
There are a few binaries but you only need the one named `k3s`.

Download it to your laptop eg for k3s version 0.8.1

```
wget https://github.com/rancher/k3s/releases/download/v0.8.1/k3s
```

Make it executable and place it in /usr/local/bin/ 

```
chmod +x k3s
sudo mv k3s /usr/local/bin/
```

## Step 2: Decide if you want to use `Docker` or `containerd`

If you alread have [Docker](https://www.docker.com) installed then go with `Step 3` below make k3s utilize your already existing [Docker](https://www.docker.com) installation.

If you don't have Docker installed and don't plan on it either then go with `Step 4` which will utilize `containerd` which is the core of [Docker](https://www.docker.com) but isn't [Docker](https://www.docker.com)


## Step 3: Setup k3s as a systemd service (using Docker)

Create a file named `/etc/systemd/system/k3s.service` and add the following content:


```
[Unit]
Description=Lightweight Kubernetes
Documentation=https://k3s.io
After=network.target

[Service]
Type=notify
EnvironmentFile=/etc/systemd/system/k3s.service.env
ExecStartPre=-/sbin/modprobe br_netfilter
ExecStartPre=-/sbin/modprobe overlay
ExecStart=/usr/local/bin/k3s server --docker
KillMode=process
Delegate=yes
LimitNOFILE=infinity
LimitNPROC=infinity
LimitCORE=infinity
TasksMax=infinity
TimeoutStartSec=0

[Install]
WantedBy=multi-user.target
```

Run the following command when you make changes to the systemd services files to make sure your changes are committed to systemd and that your system will boot again and won't hang because a systemd service file has been updated/changed:

```
sudo systemctl daemon-reload
```

## Step 4: Setup k3s as a systemd service (using containerd)

This is essentially the same as the above except for the `ExecStar=/usr/local/bin/k3s server --docker` line except it is without `--docker`.

Create a file named `/etc/systemd/system/k3s.service` and add the following content:


```
[Unit]
Description=Lightweight Kubernetes
Documentation=https://k3s.io
After=network.target

[Service]
Type=notify
EnvironmentFile=/etc/systemd/system/k3s.service.env
ExecStartPre=-/sbin/modprobe br_netfilter
ExecStartPre=-/sbin/modprobe overlay
ExecStart=/usr/local/bin/k3s server
KillMode=process
Delegate=yes
LimitNOFILE=infinity
LimitNPROC=infinity
LimitCORE=infinity
TasksMax=infinity
TimeoutStartSec=0

[Install]
WantedBy=multi-user.target
```

Run the following command when you make changes to the systemd services files to make sure your changes are committed to systemd and that your system will boot again and won't hang because a systemd service file has been updated/changed:

```
sudo systemctl daemon-reload
```

## Step 5: Using the k3s service

You can now start and stop the k3s service as you please.

### Start k3s

```
sudo systemctl start k3s
```

### Stop k3s

```
sudo systemctl stop k3s
```

## Running k3s as a boot service

If you've followd the above installation steps and is able to start/stop k3s using the above `systemctl` commands then you can enable (or disable) it as a boot service to it will start when you boot your server/desktop or your remote _Rasberry Pi_ out in the shed...

**To enable as boot service**
```
sudo systemctl enable k3s
```

**To disable as boot service**
```
sudo systemctl disable k3s
```

Note: you'll still be able to manually stop and start (and restart) the k3s service if it is running as a boot service using the `sudo systemctl [start|stop|restart] k3s` command(s).


# First run after installation

After the installation above as a systemd service then start k3s using the command

```
sudo systemctl start k3s
```

The first start takes a little while as k3s needs to 

* Unpack some files
* Create some certificates (this requires CPU)
* Initiate the core services

## Setting up the `kubectl` command

The k3s deployment comes with its own built in `kubectl` command which you can either 

* use directly on every command (cumbersome)
* setup as an alias (better)
* install the full `kubectl` client (best)

### Using the built in `kubectl` command (cumbersome):

This section will help you setup `kubectl` which will can be done in several ways. Chose one and later when referencing `kubectl` it will be either of these you use, but it will also be the `kubectl` you use when reading and using documentation from the Internet.


`k3s` comes with a built in `kubectl` command which can be used like this:

```
sudo /usr/local/bin/k3s kubectl <commands>
```

### Using the built in `kubectl` command as an alias (better):

You can setup a terminal alias for the above built in `k3s kubectl` command by updating your ~/.bashrc which will enalbe the command on every reboot hence forth.

```
alias kubectl="sudo /usr/local/bin/k3s kubectl "
```

Before you can use the alias you need to do either of the following:

* run the above alias command in your terminal (once per terminal you want to work on right now)
* source the ~/.bashrc file eg `source ~/.bashrc` to activate the alias
* logout and then log  back in again as this will source the alias for you automatically

You can run the command above in the terminal.

Please note that this method will require your `sudo` password on the first run and when ever your sudo-timout has run out.

### Using the official `kubectl` command (best):

Since you might be using a distro that is special or not then follow the guide at [https://kubernetes.io](https://kubernetes.io/docs/tasks/tools/install-kubectl/) to install the `kubectl` tool on your system.

After installation you need to setup a `~/.kube/config` file for your user (other wise you'll end up needing to use sudo and set the environment variable KUBECONFIG every time you want to use k3s - that isn't advised).

1. Create the directory `~/.kube` using the command ```mkdir ~/.kube```
2. Copy the `k3s` kube config file to your directory using the command ```sudo cat /etc/rancher/k3s/k3s.yaml > ~/.kube/config```

You should now be able to run the command `kubectl` and have `k3s` at your fingertips :-)


# Make sure everything is working as expected:

Once you've started `k3s`and setup/decided on a `kubectl` method then run the following commands to make sure `k3s` is up and running:

## Get the node - to see if it is running
```
kubectl get nodes
```
You should see 2 lines - a named column line and a line with the computer you are running `k3s` on.

## Get a list of all pods
```
kubectl get pods --all-namespaces
```
You should see column list of namespaces and pods names and some extra information for each pod.


# Your first deployoment


To deploy to `k3s` you need 3 things

1. a `kind: Deployment` 
2. a `kind: Service`
3. a `kind: Ingress`

Take a small `nginx` example:

## Preparing the `nginx` Deployment files (declarative YAML configuration)

Create a `nginx` directory for the YAML files needed to deploy `nginx` to `k3s` eg :

```
mkdir ~/nginx-deployment
cd ~/nginx-deployment
``` 

In this directory you'll create 3 files.

1. Create a file named `nginx-deployment.yml` and add the following to it:
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
2. Create a file named `nginx-service.yml` and add the following:
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
3. Create a file named `nginx-ingress.yml` and add the following: 
```
apiVersion: extensions/v1beta1
kind: Ingress
metadata:
  name: nginx
spec:
  rules:
  - host: localhost
    http:
      paths:
      - backend:
          serviceName: nginx
          servicePort: 80
        path: /*
```

## Setup a namespace for `nginx` to run in 

Run the following command to create a namespace for `nginx` to run in 
```
kubectl create namespace nginx
```

## Deploy `nginx` to `k3s`

Make sure you are in the directory you created the above 3 YAML files.

Run the following commands:

```
kubectl -n nginx create -f nginx-deployment.yml
kubectl -n nginx create -f nginx-service.yml
kubectl -n nginx create -f nginx-ingress.yml
```

You can view the status of the deployment using the command
```
kubectl -n nginx describe deployment nginx
```

Or just wait for it to finish with the command if you trust your Internet and YAML files (typos)
```
kubectl -n nginx rollout status deployment nginx
```
Src: [https://www.mankier.com/1/kubectl-rollout-status]

Once the 3 `kubectl` or `kubectl rollout status` commands are OK aka everything is running then you should be able to open [http://localhost](http://localhost) and see the default `nginx` webpage.