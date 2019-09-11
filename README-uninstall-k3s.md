# How to uninstall k3s

If you get tired of `k3s` and want to uninstall (or reinstall because the upgrade didn't work properly) then this is the way to do it:

1. Stop the `k3s` service `sudo systemctl stop k3s`
2. Disable the `k3s` service `sudo systemctl disable k3s`
3. Remove the systemd file(s) `sudo find /etc/systemd/system -iname 'k3s*' -delete`
4. If you use Docker: stop all containers `sudo docker stop $(docker ps -a -q)`
5. Unmount any volumes `umount $(mount |grep rancher | cut -d" " -f3)`
6. Remove all `k3s` directories `sudo rm -rf /etc/rancher /var/lib/rancher`
7. Purge/prune all Docker images `sudo docker system prune -a -f`

And that is it