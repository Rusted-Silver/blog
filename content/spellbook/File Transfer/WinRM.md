This is for pentest on windows machine

Create session. Only when you have right on the target machine (Remote Management Users group)

```powershell
$Session = New-PSSession -ComputerName DATABASE01
```

```powershell
Copy-Item -ToSession $Session -Path .\samplefile.txt -Destination C:\Users\User\Desktop\
```

```powershell
Copy-Item -FromSession $Session -Path "C:\Users\Administrator\Desktop\DATABASE.txt" -Destination C:\
```