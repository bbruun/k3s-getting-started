## Systemd integration

There are 3 ways to run `k3s`

1. Manually in a termainal using `sudo k3s server` (tedious)
2. Set it up as a service you start/stop manually (practical)
3. Set it up as a boot service so it is alwasy ready (practical)


`k3s` supports 2 types of server runtime
* with **Docker**
* with **containerd**

As a usage point when working with Kubernetes either will do just fine, but if you use or plan on using the computer that `k3s` runs on then choose the **Docker** variant as it will use Docker so you can "play around" as well.
If you won't ever "touch" the server then use the **containerd** version.



## The 2 systemd service files

**Docker**
Use this systemd service file if you have Docker installed or plan on using Docker on the same server/host as `k3s` will be running on
* [k3s.service-docker.service](k3s.service-docker)


**containerd**
Use this systemd service file if you don't have Docker installed (or plan on using Docker) eg completely remote `k3s` server eg a Rasberry Pi or similar.
* [k3s.service-containerd.service](k3s.service-containerd)


# Installing it

1. Download the above file that you want to use (eg the docker version)
2. Move it to `/etc/systemd/system/k3s.service` using the command `sudo mv k3s.service-docker.service /etc/systemd/system/k3s.service`
3. Reload systemd `sudo systemctl daemon-reload`


[Go back to main page](README-first-draft.md)