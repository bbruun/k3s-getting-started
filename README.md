# k3s Install Process step-by-step

**Notes:**
- This guide is spawned both because of a friend and [this Rancher k3s GitHub](https://github.com/rancher/k3s/issues/795#issuecomment-530440037) issue.
- This guide is made from a fresh install of a [Ubuntu 18.04 Server](https://ubuntu.com/download/server) in VirtualBox.
- The Ubuntu 18.04 Server (hence forth just 'server') only has OpenSSH pre-installed during installation.
    - Docker was not pre-installed as it is a [snap](https://snapcraft.io) version - **it does not work well with `k3s` - DO NOT PRE-INSTALL IT** 
    

## Requirements before you get started

* Before you start them make sure your computer is able to go on the Internet (aka `ping google.com -c1`)
* If needed: Install [Docker](https://docs.docker.com/v17.09/engine/installation/linux/docker-ce/ubuntu/) using the proper installation method and **DO NOT use the _snap_ version provided by default via `apt/apt-get` - it is not working properly with `k3s`**
(note only install Docker if you need it - it can be omitted)

## Installing `k3s`


* Open a terminal and run the following command as your own user with **sudo** rights: ```curl -sfL https://get.k3s.io | sudo sh -```
  * The installation will now procede and once finished you'll get your command prompt back.
* Test that `k3s` is operational using the command: ```sudo kubectl get nodes```   

## Communicating/working with `k3s`

The command line tool `kubectl` is the way to communicate with `k3s` or any Kubernetes installation if you do not have any other 3rd party tools to do so.

**`k3s`** comes with its own built in `kubectl` command that is by default symlinked to `/usr/local/bin/kubectl`.
This version of `kubectl` is built into `k3s` and uses the `/etc/rancher/k3s/k3s.yaml` file which is root owned.
To use that version run ```sudo chmod o+r /etc/rancher/k3s/k3s.yaml``` and you can run `kubectl <commands>` without the need for sudo. 

### Alternative: use the official `kubectl`

You can install the official `kubectl` and work without compromising the security of the above `k3s.yaml` file.
 
To use the official Kubernetes `kubectl` version then either go to [https://kubernetes.io/docs/tasks/tools/install-kubectl/](https://kubernetes.io/docs/tasks/tools/install-kubectl/) to see the installation procedure for your distribution.
Or do the following 

Once you've installed the proper `kubectl` for your distribution then run the following 3 command:
You should now be above to work with `k3s` using `kubectl` the way it is designed. If you did the *chmod* above of the `k3s.yaml` file then re-run it with `o-r` instead of `o+r` to reset the permissions on the file to 600 or read/write only for root.

### A quick offical `kubectl` install/upgrade procedure
If you are on a amd64bit Intel or AMD Debian/Ubuntu or RHEL/CentOS like distribution then this can be done for install and upgrade of the official `kubectl` command:  

```bash
cd /tmp 
curl -LO https://storage.googleapis.com/kubernetes-release/release/`curl -s https://storage.googleapis.com/kubernetes-release/release/stable.txt`/bin/linux/amd64/kubectl
sudo rm /usr/local/bin/kubectl
chmod +x kubectl 
sudo mv kubectl /usr/local/bin/kubectl
```
 
**Regardless of which official `kubectl` installation method you choose then do this once**
The office `kubectl` uses the `~/.kube/config` file to get its configuration to Kubernetes or `k3s', so make a copy of the `k3s.yaml` file in your HOME dir like this:
 
```bash
mkdir ~/.kube
sudo cat /etc/rancer/k3s/k3s.yaml > ~/.kube/config
sudo rm /usr/local/bin/kubectl
```

 

## Your first deployment

Remember the IP address from the **Requirements** section? Now you need the IP address
If you don't then you can always get it via the following command

```bash
sudo kubectl get services --namespace kube-system traefik --output jsonpath='{.status.loadBalancer.ingress[0].ip}' 
``` 

The IP address is needed for you to setup one or more DNS entries (A records or CNAME's) pointing to it or local /etc/hosts entries eg 
```bash
x.x.x.x     www.example.com nginx.example.com example.com
```
(replace x.x.x.x with the IP from the above command)

This will enable you to setup an Ingress object so you can have multiple services running on the same IP normal FQDN's like "nginx.example.com" and the like.

### Setting up a Nginx service

In this repo there is a directory named [nginx-deployment](nginx-deployment) where there are 3 files in it.

The 3 files are 
- nginx-deployment.yml
- nginx-service.yml
- nginx-ingress.yml

**You need to update the `nginx-ingress.yml` file** with the FQDN you setup on your DNS or in /etc/hosts unless you configured it for **nginx.example.com**. 

Download the 3 files and apply them to `k3s` in a new namespace (namespaces are virtual "rooms" that separate applications from each other like directories do for files)

**Create a new namespace: nginx**
```bash
kubectl create namespace nginx

# Shortform - some parameters to kubectl have shortform eg ns for namespace, svc for service etc.
kubectl create ns nginx
```

**Apply the 3 files to the `nginx` namespace**
```bash
kubectl create ns nginx
kubectl -n nginx create -f nginx-deployment.yml
kubectl -n nginx create -f nginx-service.yml 
kubectl -n nginx create -f nginx-ingress.yml 
```

For each of the commands you should get one line back from `kubectl ...` which is 
- `namespace/nginx created` when creating the namespace 
- `deployment.apps/nginx created` when creating the Deployment
- `service/nginx created` when creating the Service
- `ingress.extensions/nginx created` when creating the Ingress object

After a little while you'll be able to see the setup using the following commands:
```bash
kubectl --namespace nginx get deployments
kubectl --namespace nginx get pods
kubectl --namespace nginx get ingresses
kubectl --namespace nginx get services
```

Each of these will show you one or more lines. 
Each line explains the current state of the Deployment, Pod(s), Service and Ingress controller.


**Check if Nginx is working**

```bash
curl http://nginx.example.com
```
*replace nginx.example.com with whatever you setup*

**Congratulations**

You have now setup Kubernets on your desktop/server and have deployed **nginx** (though some files are missing... we'll get to that in [README-volumes.md](README-volumes.md))


## Kubectl tips and tricks


### Command completion for `kubectl` 

If you use the BASH shell (most likely you do) then run the following command and logout and then login again
```bash
kubectl completion bash | tee -a ~/.profile
source ~/.profile
```  
You can now use <TAB> completion for most commands and pod lookups just like on the file system eg
```bash
# get a list of namespaces
kubectl -n <tab><tab>

# get a list of what you can get from the namespace (it is a lot)
kubectl -n nginx get <tab><tab>

# get logs for a particular pod (if there are multiple they will be shown so you can chose the next alphanumeric character of the pods name you want to get logs from)
kubectl -n nginx lo<tab> <tab><tab>
```
If you use an alternate shell then run `kubectl completion -h` to get help to setup completion.

