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

## Download `k3s` from GitHub and install the binary

From [https://github.com/rancher/k3s](https://github.com/rancher/k3s) open the [`releases`](https://github.com/rancher/k3s/releases) page and find the latest stable version (as of this writing it is `0.8.1` and that is what is used in the documentation below).

Download the appropriate binary for your architecture - for 64bit Intel/AMD based GNU/Linux servers/computers it is the binary simply named `k3s`.
If you happen to have another architecture then copy the link to the the appropriate binary and remember to correct for filenames along the documentation in examples. 



**Download it using `wget` or `curl`**
Note: Exchange the URL `https://github..../k3s` with the architecture version you need  

**Using wget**
```bash
wget https://github.com/rancher/k3s/releases/download/v0.8.1/k3s -O /tmp/k3s
```
**Using curl**
```bash
curl https://github.com/rancher/k3s/releases/download/v0.8.1/k3s -o /tmp/k3s
```


**Install the binary in /usr/local/bin/**

```bash
chmod +x /tmp/k3s
sudo mv /tmp/k3s /usr/local/bin/
```

## Running k3s

There are several ways to run `k3s`.

- manually as `root` (or using `sudo`) all the time
- as a service you can start and stop
- as a boot service that will start when you boot your computer/or your computer is rebooted 

### Setup as a systemd service (manual and boot enabled)

This documentation has Docker pre-installed on the server used for documentation but if you don't plan on working with Docker your self then you can choose to use the other systemd service file.

Download either of these 
- [k3s.service-docker](https://raw.githubusercontent.com/bbruun/k3s-getting-started/master/k3s.service-docker) **if you have Docker installed**
- [k3s.service-containerd](https://raw.githubusercontent.com/bbruun/k3s-getting-started/master/k3s.service-containerd) **if you don't use Docker**

The difference is that `k3s` can utilize the aleady installed Docker service (assuming it is running), but it can also work perfectly well without by using its own [containerd](https://containerd.io) runtime. 

The service file is to be installed in `/etc/systemd/system/` as `k3s.service`:

**Systemd service with using Docker** 
```bash
wget https://github.com/bbruun/k3s-getting-started/blob/master/k3s.service-docker -O /tmp/k3s.service
sudo mv /tmp/k3s.service /etc/systemd/system/
sudo touch /etc/systemd/system/k3s.service.env
sudo systemctl daeamon-reload
```

**Systemd service with using built-in containerd** 
```bash
wget https://github.com/bbruun/k3s-getting-started/blob/master/k3s.service-containerd -O /tmp/k3s.service
sudo mv /tmp/k3s.service /etc/systemd/system/
sudo touch /etc/systemd/system/k3s.service.env
sudo systemctl daeamon-reload
```

### Enable it as a boot service

If you are only planning on test running `k3s` then you can skip this step.

```bash
sudo systemctl enable k3s
``` 


## First run (manual)

After completing installation above you have 2 options to start `k3s`

When `k3s` starts up it needs to install a few files and generate some certificates.
All depending on the CPU(s) and memory on the server it can take a few minutes (especially on a Rasberry Pi).

- use systemd (recommended)
- manually (might be helpfull to debug start/runtime issues)

### Starting `k3s` manually

To start `k3s` manually run the following command and if it is the first time then wait a litte as described above.
**If you have Docker installed**
```bash
sudo k3s server --docker 
```

**If you don't have Docker installed**

```bash
sudo k3s server 
```
Notes: 
- for both of the above  `sudo k3s server [--docker]` commands you see a lot of output.
- the output never ends as `k3s` has been started in the foreground like any program (use Ctrl-c to stop it)
- the output should end with a lot of messages starting with `I0911 ....` or similar - that indicates it is ready.
- don't worry if you see some error messages, this is not a final version yet (aka 0.8.1 which this documentation is written for)

**After start**

Once you've started `k3s` and you can see a log entry in the `I0911 ...` lines similar to the following it is ready for usage.
```text
I0911 18:41:19.047381    6814 flannel.go:102] Using interface with name enp0s3 and address 192.168.64.143
```  

## Communicating with `k3s`

There are 3 ways to communicate with Kubernetes - be it `k3s` or any of the hosted versions including a bare metal installation using another guide is to use `kubectl`.

`k3s` comes with its own built in `kubectl` command that you can use like so 
```bash
sudo k3s kubectl <commands>
```

The first `<command>`  to run is `get nodes` and `get pods --all-namespaces` to verify the installation
```bash
sudo k3s kubectl get nodes
sudo k3s kubectl get pods --all-namespaces
```

You should see something similar to this:
```bash
$ sudo k3s kubectl get nodes
NAME     STATUS   ROLES    AGE   VERSION
k3sdoc   Ready    master   10m   v1.14.6-k3s.1
$ sudo k3s kubectl get pods --all-namespaces
NAMESPACE     NAME                         READY   STATUS    RESTARTS   AGE
kube-system   coredns-b7464766c-hxtfj      1/1     Running   0          11m
kube-system   helm-install-traefik-rfpcp   0/1     Completed   0          2m6s
kube-system   svclb-traefik-p8jkt          2/2     Running   0          10m
kube-system   traefik-5c79b789c5-fvb9z     1/1     Running   0          10m
```

The first command above shows that `k3s` has one registered worker node named `k3sdoc` which is the servers name (it will be different for your computer)
The second command shows 3 pods running and 1 completed.

The pod that has completed is a one-time pod used to setup `traefik` (the Ingress controller) and can be removed using the command
```bash
sudo k3s kubectl -n kube-system delete pod helm-install-traefik-rfpcp
pod "helm-install-traefik-rfpcp" deleted
``` 

## Your first deployment

Remember the IP address from the **Requirements** section? Now you need the IP address
If you don't then you can alwasys get it via the following command

```bash
sudo k3s kubectl get services --namespace kube-system traefik --output jsonpath='{.status.loadBalancer.ingress[0].ip}' 
``` 

The IP address is needed for you to setup one or more DNS entries pointing to it or local /etc/hosts entries eg 
```bash
x.x.x.x     www.example.com nginx.example.com example.com
```
(replace x.x.x.x with the IP from the above command)

This will enable you to setup an Ingress object so you can have multiple services running on the same IP - think of it as a load balancer.

### Setting up a Nginx service

In this repo there is a directory named [nginx-deployment](nginx-deployment) where there are 3 files in it.

The 3 files are 
- nginx-deployment.yml
- nginx-service.yml
- nginx-ingress.yml

You need to update the `nginx-ingress.yml` file with the FQDN you setup on your DNS or in /etc/hosts unless you setup **nginx.example.com**

Download the 3 files and apply them to `k3s` in a new namespace (namespaces are virtual "rooms" that separate applications from each other like directories do for files)

**Create a new namespace: nginx**
```bash
sudo k3s kubectl create namespace nginx
```

**Apply the 3 files to the `nginx` namespace**
```bash
sudo k3s kubectl create ns nginx
sudo k3s kubectl -n nginx create -f nginx-deployment.yml
sudo k3s kubectl -n nginx create -f nginx-service.yml 
sudo k3s kubectl -n nginx create -f nginx-ingress.yml 
```

For each of the commands you should get one line back from `k3s kubectl ...` which is 
- `namespace/nginx created` when creating the namespace 
- `deployment.apps/nginx created` when creating the Deployment
- `service/nginx created` when creating the Service
- `ingress.extensions/nginx created` when creating the Ingress object

After a little while you'll be able to see the setup using the following commands:
```bash
sudo k3s kubectl --namespace nginx get deployments
sudo k3s kubectl --namespace nginx get pods
sudo k3s kubectl --namespace nginx get services
sudo k3s kubectl --namespace nginx get ingresses
```

Each of these will show you one or more lines. Each line explains the current state of the Deployment, Pod(s), Service and Ingress controller.

**Check if Nginx is working**

```bash
curl http://nginx.example.com
```

**Congratulations**


## First run using the systemd service

If you followd the above systemd setup then you should be able to run the following commands to start/stop `k3s`

**Start `k3s`**
```bash
sudo systemctl start k3s
``` 

**Stop `k3s`**
```bash
sudo systemctl stop k3s
```

### Enable `k3s` as a boot service

If you get tired of manually starting and stopping `k3s` and just want it ready on boot then enable it as a boot service

```bash
sudo systemctl enable k3s
```

And on subsequent boots and reboots you'll have `k3s` ready as a service.


## Kubectl tips and tricks

### Use the official `kubectl` (recommended)

It is nice that `k3s` comes with its own built in `kubectl` command, but it can be cumbersome to work with in the long run.

You can install and use the official `kubectl` which can be installed using [this installation guide](https://kubernetes.io/docs/tasks/tools/install-kubectl/).

Once installed on your computer then setup a KUBECONFIG file for your user so you can avoid using `sudo k3s kubectl` all the time

```bash
mkdir ~/.kube
sudo cat /etc/rancher/k3s/k3s.yaml | tee ~/.kube/config
```

Now you can use `kubectl` without using `sudo`.

Note: If you delete `k3s` or upgrade it and something breaks the YAML file in /etc/rancher/ then you need to re-do the above command to make `kubectl` work again. 



### Use `sudo k3s kubectl` as an alias
Note: This does not apply if you installed the offical `kubectl`

Setup a shell alias for the command.   
Add the following to ~/.bashrc and source the file

(Use `vim ~/.bashrc`)
```bash
echo 'alias kubectl="sudo /usr/local/bin/k3s kubectl "' | tee -a ~/.bashrc
source ~/.bashrc
```

After that and all subsequent reboots you'll be able to run `kubectl <commands>` with out the `sudo k3s kubectl ` part.

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