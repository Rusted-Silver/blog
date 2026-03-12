# AD common sense

- Computer accounts usually has a complex random long password set and rotate automatically. Cracking them is impossible.
- Computer accounts **cannot** do cross-forest kerberoast. However, they can do kerberoast in the same domain
- Computer accounts in AD change their passwords automatically in 30 days
- Computer accounts that is member of `PRE-WINDOWS 2000 COMPATIBLE ACCESS` have their password set to the same as username.
	- However, being a member of a group that is a member of `PRE-WINDOWS 2000 COMPATIBLE ACCESS` **DOES NOT** count. it isn't inheritable
- If you authenticate via kerberos, and change something in AD, but you don't the privilege you should have had, you should request a ticket again. This is because the old ticket don't update your privilege
## Covert attacks common sense

- Always request kerberos ticket with aes-256 key. RC4(NTLM) encryption is not the default windows behavior. `impacket-getTgT` default always use Rc4, which is bad
- Never dump a `.kirbi` file. Only `mimikatz` and some other **pentest/POC** tools does this. NO SINGLE app on windows create a `.kerbi` file. Always dump base64 on terminal
- Use `Rubeus`. `mimikatz` is a POC tool. `Rubeus` is a tool designed by an operator with the mindset of a red team operator.

# Deleted objects

## From linux

### BloodyAD method

Query everything of deleted object

```sh
bloodyAD --host dc.domain.name -d domain.name get search -c '1.2.840.113556.1.4.2064' -c '1.2.840.113556.1.4.2065' --filter '(isDeleted=TRUE)'
```

Query specific things like Description,ObjectSid,ObjectGUID,LastKnownParent

```sh
bloodyAD --host dc.domain.name -d domain.name get search -c '1.2.840.113556.1.4.2064' -c '1.2.840.113556.1.4.2065' --attr 'Description,ObjectSid,ObjectGUID,LastKnownParent' --filter '(isDeleted=TRUE)'
```

### Ldapsearch method

> For Kerberos authentication, set `KRB5CCNAME=user.ccache` variable add `-Tx -Y GSSAPI` flag

Query everything of deleted object

```sh
ldapsearch -H ldap://dc.domain.name \
  -D "username@domain.name" -w 'password' \
  -b "CN=Deleted Objects,DC=domain,DC=name" \
  -E "1.2.840.113556.1.4.417" \
  "(isDeleted=TRUE)"
```

Query specific things like Description,ObjectSid,ObjectGUID,LastKnownParent

```sh
ldapsearch -H ldap://dc.domain.name \
  -D "username@domain.name" -w 'password' \
  -b "CN=Deleted Objects,DC=domain,DC=name" \
  -E "1.2.840.113556.1.4.417" \
  "(isDeleted=TRUE)" \
  Description objectSid ObjectGUID LastKnownParent
```

### Restore object

Restore the object. Should use SID for a surefire

```sh
bloodyAD --host dc01.domain.name -d domain.name -u user -p 'password' set restore deleted_user
```

## From `Powershell`

Query everything of deleted object

```sh
Get-ADObject -Filter 'IsDeleted -eq $true' -IncludeDeletedObjects -Properties *
```

Query specific things like Description,ObjectSid,ObjectGUID,LastKnownParent

```sh
Get-ADObject -Filter 'IsDeleted -eq $true' -IncludeDeletedObjects -Properties Description,ObjectSid,ObjectGUID,LastKnownParent
```

Restore the object, using ObjectGUID

```sh
Restore-ADObject -Identity aaaaaaaa-aaaa-aaaa-aaaa-aaaaaaaaaaaa
```

# Writable Objects

Sometimes, we have permission to write specific things on another object, and bloodhound doesn't show that.

```sh
bloodyAD --host "dc01.domain.name" -d "domain.name" -u 'user' -p 'pass' get writable --detail
```

# Get Object

See all attributes of an object

```sh
bloodyAD --host "dc01.domain.name" -d "domain.name" -u 'user' -p 'pass' get object 'victim' --resolve-sd
```

Or, get a specific attribute

```sh
bloodyAD --host "dc01.domain.name" -d "domain.name" -u 'user' -p 'pass' get object 'victim' --attr logonHours 
```

# Cypher queries

These is a cypher query **used in bloodhound**. See how to set up bloodhound at [10. Lateral Movement/Bloodhound#Server]({{< relref "10. Lateral Movement/Bloodhound#server" >}})
## All DACL of User or Computer Accounts

This query will show every computer or user accounts' DACL. Very good in CTF or small AD environments.

```cypher
MATCH p=(source)-[r]->(target)
WHERE (source:Computer OR source:User)
AND type(r) <> 'MemberOf'
return p
```