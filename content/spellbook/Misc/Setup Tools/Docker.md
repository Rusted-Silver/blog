# Docker

```sh
sudo apt update && sudo apt install docker.io containerd docker-compose -y
sudo usermod -aG docker $USER
newgrp docker
```

Test. No need privilege since we are in docker group. After this, should log out and log in again or just... restart

```
docker run hello-world
```

## Quick MySQL

```sh
docker run -p 3306:3306 -e MYSQL_USER='db' -e MYSQL_PASSWORD='db-password' -e MYSQL_DATABASE='db' -e MYSQL_ROOT_PASSWORD='db' --mount type=bind,source="$(pwd)/db.sql",target=/docker-entrypoint-initdb.d/db.sql mysql
```
