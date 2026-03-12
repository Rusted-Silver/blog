>Can skip this part if you know for sure there is no account lockout policy

# Linux Attacker

## `netexec`

If null session is available, use this

```sh
netexec smb -u '' -p '' $target --pass-pol
```

If not, then need valid credential

```sh
netexec smb $target -u "$user" -p "$pass" --pass-pol
```

## `rpcclient`

If null session is available, use this

```sh
rpcclient -U "" -N $target
rpcclient $> getdompwinfo
```

If not, then need valid credential

```sh
rpcclient -U "$user%$pass" $target
rpcclient $> getdompwinfo
```

## `ldapsearch`

```sh
ldapsearch -h $target -x -b "DC=INLANEFREIGHT,DC=LOCAL" -s sub "*" | grep -m 1 -B 10 pwdHistoryLength
```

# Domain joined Windows

## `net.exe`

```cmd
net accounts
```

## `Powerview`

```powershell
import-module .\PowerView.ps1
```

```powershell
Get-DomainPolicy
```