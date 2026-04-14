---
weight: 2
---
# Powershell

Sometimes, you need to enumerate AD on a AD joined windows computer, or you want to "living off the land". 

You need `ActiveDirectory` powershell module installed on the machine. Typically, this is already installed for you on most AD joined machine
## Query objects

The `ActiveDirectory` powershell module gives you some cmdlet to enumerate AD. Typically, those are what you want to use:
- `Get-ADUser`
- `Get-ADGroup`
- `Get-ADComputer`
- `Get-ADObject`

With `Get-ADObject`, you might need to use `-LDAPFilter` to filter specifically what object you want, but you can just use `Get-ADUser`, `Get-ADGroup`, and `Get-ADComputer` to save yourself having to remember the ldap query.

For example, this is how you query all properties of all users:

```powershell
# Get-ADObject
Get-ADObject -LDAPFilter '(&(objectCategory=person)(objectClass=user))' -Properties *
# Get-ADUser
Get-ADUser -Filter * -Properties  *
```

Groups:

```powershell
# Get-ADObject
Get-ADObject -LDAPFilter '(objectClass=group)' -Properties *
# Get-ADGroup
Get-ADGroup -Identity "Administrators" -Properties *
```

And computers

```powershell
# Get-ADObject
Get-ADObject -LDAPFilter '(objectCategory=computer)'
# Get-ADComputer
Get-ADComputer -Filter * -Properties *
```

### Select specific properties

You can pipe the received objects into `Select` to **show** only specific properties:

```powershell
# Get-ADObject
Get-ADObject -LDAPFilter '(&(objectCategory=person)(objectClass=user))' -Properties * | select logonHours
# Get-ADUser
Get-ADUser -Filter * -Properties * | select logonHours
```

### Count

You can also count the number of objects received

```powershell
# Get-ADObject
(Get-ADObject -LDAPFilter '(&(objectCategory=person)(objectClass=user))').Count
# Get-ADUser
(Get-ADUser -Filter *).Count
```

### Filter

You can utilize the `-Filter` flag to filter further things from the objects received

For example, this is how you can filter in only objects that has `Name` property `equal` to  `'sally jones'`:

```powershell
Get-ADUser -Filter "Name -eq 'sally jones'"
```

You can also pipe it to `Where` for the same purpose:

```powershell
Get-ADUser | Where Name -eq 'sally jones'
```

Here's a list of filter operators:

|**Filter**|**Meaning**|
|---|---|
|-eq|Equal to|
|-le|Less than or equal to|
|-ge|Greater than or equal to|
|-ne|Not equal to|
|-lt|Less than|
|-gt|Greater than|
|-approx|Approximately equal to|
|-bor|Bitwise OR|
|-band|Bitwise AND|
|-recursivematch|Recursive match|
|-like|Like|
|-notlike|Not like|
|-and|Boolean AND|
|-or|Boolean OR|
|-not|Boolean NOT|

## SearchBase and SearchScope

`-SearchBase` flag needs OU and DN like `OU=Employees,DC=INLANEFREIGHT,DC=LOCAL`

`-SearchScope` allows us to define how deep into the OU hierarchy we would like to search:
- `Base` - Return only the OU itself defined in `-SearchBase`
- `OneLevel` - Return objects inside the OU defined in `-SearchBase`
- `SubTree` - Return **recursively** all objects inside defined OU and nested objects

As an example, if we search for member count in an OU, with `-SearchBase` and no `-SearchScope` we got 970 users

```powershell
(Get-ADUser -SearchBase "OU=Employees,DC=INLANEFREIGHT,DC=LOCAL" -Filter *).count
970
```

When we set `-SearchScope` to `Base`, we got nothing, since it returns only the OU object itself, and no user object

```powershell
(Get-ADUser -SearchBase "OU=Employees,DC=INLANEFREIGHT,DC=LOCAL" -SearchScope Base -Filter *).Count
0
```

When we set `-SearchScope` to `OneLevel`, we got 1 user, since this is the only user **directly** inside the OU

```powershell
(Get-ADUser -SearchBase "OU=Employees,DC=INLANEFREIGHT,DC=LOCAL" -SearchScope OneLevel -Filter *).Count
1
```

Finally, when we set `-SearchScope` to `SubTree`, we got all 970 users, including nested users

```powershell
(Get-ADUser -SearchBase "OU=Employees,DC=INLANEFREIGHT,DC=LOCAL" -SearchScope Subtree -Filter *).count
970
```

## User

### Kerberoastable users

```powershell
Get-ADUser -Properties * -filter ** | where servicePrincipalName -ne $null | Select SamAccountName,MemberOf,ServicePrincipalName | fl
```

### ASREP-Roastable users

You can search for the `DoesNotRequirePreAuth` property, which mean that the user is vulnerable to ASREP-Roast attack

```powershell
Get-ADUser -Filter {DoesNotRequirePreAuth -eq 'True'}
```

### Trusted for delegation

A user account can be set to be trusted for delegation.

```powershell
Get-ADUser -Properties * -LDAPFilter '(userAccountControl:1.2.840.113556.1.4.803:=524288)' | Select Name,memberof,ServicePrincipalName,TrustedForDelegation | fl
```

### Description

Sometimes, description of a user account might have password

```powershell
Get-ADUser -Properties * -LDAPFilter '(description=*)' | Select SAMAccountname,Description
```

### Blank password

Find users with `useraccountcontrol` attribute set with the flag `PASSWD_NOTREQD` meaning the account **can** have a blank password set

```powershell
Get-AdUser -LDAPFilter '(userAccountControl:1.2.840.113556.1.4.803:=32)' -Properties * | Select Name,Memberof | fl
```

### Groups of a user

Find all group **recursively** that `Harry Jones` is a part of

```powershell
Get-ADGroup -Filter 'member -RecursiveMatch "CN=Harry Jones,OU=Network Ops,OU=IT,OU=Employees,DC=INLANEFREIGHT,DC=LOCAL"' | Select Name
# With ldap OID filter
Get-ADGroup -LDAPFilter '(member:1.2.840.113556.1.4.1941:=CN=Harry Jones,OU=Network Ops,OU=IT,OU=Employees,DC=INLANEFREIGHT,DC=LOCAL)' | Select Name
```

### UAC attributes

UAC attributes are not [UAC]({{< relref "6. Privilege Escalation/Windows/UAC bypass" >}}) windows feature.

You can enumerate user accounts and their UAC values like this. UAC value of 512 can be ignored:

```powershell
Get-ADUser -Filter {UserAccountControl -ne 512} -Properties UserAccountControl | select Name,UserAccountControl
```

To resolve what the UAC value means, you can paste the number into [this web tool](https://towerofpower256.github.io/web-useraccountcontrol-tool/)

Or you can refer to this table:

| Value                          | Name     |
| ------------------------------ | -------- |
| PASSWD_CANT_CHANGE             | 64       |
| ENCRYPTED_TEXT__PWD_ALLOWED    | 128      |
| TEMP_DUPLICATE_ACCOUNT         | 256      |
| NORMAL_ACCOUNT                 | 512      |
| INTERDOMAIN_TRUST_ACCOUNT      | 2048     |
| WORKSTATION_TRUST_ACCOUNT      | 4096     |
| SERVER_TRUST_ACCOUNT           | 8192     |
| DONT_EXPIRE_PASSWORD           | 65536    |
| MNS_LOGON_ACCOUNT              | 131072   |
| SMARTCARD_REQUIRED             | 262144   |
| TRUSTED_FOR_DELEGATION         | 524288   |
| NOT_DELEGATED                  | 1048576  |
| USE_DES_KEY_ONLY               | 2097152  |
| DONT_REQ_PREAUTH               | 4194304  |
| PASSWORD_EXPIRED               | 8388608  |
| TRUSTED_TO_AUTH_FOR_DELEGATION | 16777216 |
| PARTIAL_SECRETS_ACCOUNT        | 67108864 |

An easier way is to use `powerview`, which will resolve uac values for you

```powershell
Get-DomainUser * | Select SAMAccountname,UserAccountControl
```

### Disabled user

```powershell
Get-ADUser -LDAPFilter '(userAccountControl:1.2.840.113556.1.4.803:=2)' | Select Name
```

Sometimes, a user can also be disabled by setting `logonHours` to `000000000000000000000`. It was 21 bytes of 0, no need to count

We can query such users like this:

```powershell
Get-ADUser -LDAPFilter '(logonHours=*)' -Properties * | Where {(logonHours -join '') -match '^0+$'}
```

## Groups

### Find admin groups

```powershell
Get-ADGroup -Filter "adminCount -eq 1" | Select Name
```

### Members of group

Enumerate members **directly** in `Administrators` group

```powershell
(Get-ADGroup -identity "Administrators" -Properties *).Members
Get-ADGroup -identity "Administrators" -Properties * | Select Members | fl
```

Enumerate members indirectly (nested) and directly in `Administrator` group

```powershell
Get-ADGroupMember -Identity "Administrator"
```

## Computers

### Domain Controllers

```powershell
Get-ADComputer -LDAPFilter '(userAccountControl:1.2.840.113556.1.4.803:=8192)'
```

### Trusted for delegation

```powershell
Get-ADComputer -Properties * -LDAPFilter '(userAccountControl:1.2.840.113556.1.4.803:=524288)' | Select DistinguishedName,SServicePrincipalName,TrustedForDelegation | fl
```