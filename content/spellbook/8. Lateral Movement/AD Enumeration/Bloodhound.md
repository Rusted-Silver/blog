---
weight: 1
---
# Bloodhound

Bloodhound has 2 components. I like to call them `collector` and `server`

- Collector is ran on a machine that is connected to AD. It collects every info available by querying `ldap` and more
- Server is a visualizer. It takes all the info enumerated and create a visual representation.

The workflow is, you run the collector, then feed what it produces to the server.

## Server setup

Download the docker compose file

```sh
wget https://github.com/SpecterOps/BloodHound/raw/refs/heads/main/examples/docker-compose/docker-compose.yml
```

> Edit the file as you like, you can also set admin user/pass with `bhe_default_admin_principal_name=` and `bhe_default_admin_password=`

```sh
docker-compose up -d
```

## Collector

### `bloodhound-python`

Install

```sh
sudo apt update && sudo apt install bloodhound-python -y
```

Run

```sh
bloodhound-python -c ALL -ns 172.16.8.3 -u "$user" -p "$pass" -d inlanefreight.local --zip
```

If `ldaps`, use `--use-ldaps`

```sh
bloodhound-python -c ALL -ns 172.16.8.3 -u "$user" -p "$pass" -d inlanefreight.local --use-ldaps --zip
```

If can't use `udp`, or behind a pivot box, or is using `proxychains`, use `--dns-tcp`

```sh
bloodhound-python -c ALL -ns 172.16.8.3 -u "$user" -p "$pass" -d inlanefreight.local --dns-tcp --zip
```

### `rusthound-ce`

install

```sh
git clone https://github.com/g0h4n/RustHound-CE.git
cd RustHound-CE
sudo apt update && sudo apt install cargo rustup -y
rustup default stable
make release
```

Run

```sh
./rusthound-ce -c All -i 172.16.8.3 -d inlalocalnefreight.local -u "$user" -p "$pass" -z
```

If `ldaps`, use `--ldaps`

```sh
./rusthound-ce -c All -i 172.16.8.3 -d inlanefreight.local -u "$user" -p "$pass" --ldaps -z
```

## Custom queries

### Direct Object Rights

This is a cypher query **used in bloodhound server**.

This query shows you what rights accounts have **directly assigned**. Very good in CTF or small AD environments.

```cypher
MATCH p=(source)-[r]->(target)
WHERE (source:Computer OR source:User)
AND type(r) <> 'MemberOf'
return p
```