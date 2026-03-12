# Ovpn

```bash
sudo openvpn --config ~/lab_Trungle256.ovpn > /dev/null 2>&1 &
# Detach ovpn process out of current terminal
disown
```

# Virt-manager setup

```bash
# sudo systemctl enable --now libvirtd
# sudo systemctl enable --now virtlogd.socket
sudo virsh net-autostart default
sudo virsh net-start default
```

# Sleep off (Xorg only)

Works for Xorg desktop managers, like XFCE. Gnome or KDE need to switch to Xorg

```sh
xset s off
```

# Kali genmon IP indicator

```sh
sh -c 'ip -o -4 addr show "$([ -d /sys/class/net/tun0 ]&&echo tun0||echo eth0)"|awk "{print \$2\": \" \$4}"|cut -d/ -f1'
```

# Docker setup

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

# Terminal logging

 Terminal logging, `script`. with `-fqa` flag, it basically allows you to log multiple terminal in one file, at the same time.

```sh
script -fqa
```

Furthermore, In case you used `clear`, and is afraid that your whole terminal log is lost. Fear not, here is how you "restore" it

```sh
sed -E 's/\x1b\[H|\x1b\[2J|\x1b\[3J//g' typescript > clean.txt
```

> Basically, when you do `clear`, it just prints these special characters as terminators that clear your screen. So when you log your terminal, those special characters is in the log file, and when you `cat` it out, the same special characters will again, clear your screen

# Quick docker

```sh
sudo apt update && sudo apt install docker.io containerd -y
sudo usermod -aG docker $USER
newgrp docker
```

Test. No need privilege since we are in docker group. After this, should log out and log in again or just... restart

```
docker run hello-world
```

# Tmux

`[CTRL + B]` as command prefix

`[CTRL + D]` or `exit` to exit. The first one put `EOF` into the terminal. Second one is a command.

`[CTRL + B] + [` To "scroll"

## "Windows"

`[CTRL + B] C` create another "window"

`[CTRL + B] 0` switch to "window" number 0

## Split

`[CTRL + B] %` Horizontal split

`[CTRL + B] "` Vertical split

`[CTRL + B]` + directional key up down right left to switch

## Rename

`[CTRL + B] ,` To rename current "window"

## Session

`[CTRL + B] D` Detach the current `tmux` session

`tmux ls` list all background sessions

`tmux rename-session -t <session number> <name>` Name session

`tmux attach -t <session number or name>` Reattach backgrounded session

`tmux kill-session -t <session number or name>` Kill session

# CVE search

[CVEdetails](https://www.cvedetails.com/)

[Exploit DB](https://www.exploit-db.com)

[Vulners](https://vulners.com)

[Packet Storm Security](https://packetstormsecurity.com)

[NIST](https://nvd.nist.gov/vuln/search?execution=e2s1)

# Quick shell

https://www.revshells.com/

# Quick hash crack

https://crackstation.net/

# Packages

```sh
# https://github.com/projectdiscovery/pdtm
sudo apt update && sudo apt install golang-go -y
go install -v github.com/projectdiscovery/pdtm/cmd/pdtm@latest

# https://vscodium.com/
wget -qO - https://gitlab.com/paulcarroty/vscodium-deb-rpm-repo/raw/master/pub.gpg \
    | gpg --dearmor \
    | sudo dd of=/usr/share/keyrings/vscodium-archive-keyring.gpg
echo 'deb [arch=amd64,arm64 signed-by=/usr/share/keyrings/vscodium-archive-keyring.gpg] https://download.vscodium.com/debs vscodium main' \
    | sudo tee /etc/apt/sources.list.d/vscodium.list

sudo sh -c 'apt update && apt install -y massdns libcap-dev codium seclists jq python3-scrapy keepassxc passwordsafe feroxbuster rlwrap obsidian flameshot faketime ntpsec-ntpdate hurl && apt -y full-upgrade'
pdtm -ia
pipx install git+https://github.com/Esonhugh/Gopherus3.git
pipx install flask-unsign
pipx install git-dumper
```

# Chatgpt report

```
Hi, can you please help me write a report for a penetration test session? I will describe the finding, and you will help me write each section of the finding.

Here's the finding:
I found a SSRF vulnerability. This let me do a port scan on the system's localhost. After doing the portscan, I found a prototype application opens only on localhost. This application is vulnerable to command injection. The final result is RCE

Here's all the sections that I need you to write:
- A proper title. Professional, short. I don't mind if the title miss some details, as long as the core impact and vulnerability is there. The missing details can be filled in on other sections.
- CWE: Only 1 CWE allowed. If it is a chained exploitation, choose CWE of the first vulnerability.
- CVSS 4.0 score, based on what you think. Must include the vectors
- Description & Cause: Very generic, not like telling a story, but more like a extremely generic overview of the vulnerability itself or the exploitation chain.
- Impact: Expand based on what i described above. However, again, generic is the key. It should be boring, not like telling a heroic story
- Reference: links
```