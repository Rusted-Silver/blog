
> Window's built in group
> Able to read event log

# `Wevutil.exe`

Since windows commands usually have an option to pass credentials into the command line, we can try to search for them in event log

```cmd
wevtutil qe Security /rd:true /f:text | Select-String "/user"
```

If we have a reverse shell or situations where above command doesn't work, we can pass credential into `wevutil`

```
wevtutil qe Security /rd:true /f:text /r:share01 /u:julie.clay /p:Welcome1 | findstr "/user"
```

# `Get-WinEvent`

```powershell
Get-WinEvent -LogName security | where { $_.ID -eq 4688 -and $_.Properties[8].Value -like '*/user*'} | Select-Object @{name='CommandLine';expression={ $_.Properties[8].Value }}
```
