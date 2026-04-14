# Deleted objects

## From Windows

Query everything of deleted object

```sh
Get-ADObject -Filter 'IsDeleted -eq $true' -IncludeDeletedObjects -Properties *
```

Query specific things like Description,ObjectSid,ObjectGUID,LastKnownParent

```sh
Get-ADObject -Filter 'IsDeleted -eq $true' -IncludeDeletedObjects -Properties Description,ObjectSid,ObjectGUID,LastKnownParent
```

### Restore object

Restore the object, using `ObjectGUID`

```sh
Restore-ADObject -Identity aaaaaaaa-aaaa-aaaa-aaaa-aaaaaaaaaaaa
```

## From linux

```sh
# Query everything
bloodyAD --host dc.domain.name -d domain.name get search -c '1.2.840.113556.1.4.2064' -c '1.2.840.113556.1.4.2065' --filter '(isDeleted=TRUE)'
# Query specific properties
bloodyAD --host dc.domain.name -d domain.name get search -c '1.2.840.113556.1.4.2064' -c '1.2.840.113556.1.4.2065' --attr 'Description,ObjectSid,ObjectGUID,LastKnownParent' --filter '(isDeleted=TRUE)'
```

> If you want Kerberos authentication with `ldapsearch`, set `KRB5CCNAME=user.ccache` variable add `-Tx -Y GSSAPI` flag

```sh
# Query everything
ldapsearch -H ldap://dc.domain.name \
  -D "username@domain.name" -w 'password' \
  -b "CN=Deleted Objects,DC=domain,DC=name" \
  -E "1.2.840.113556.1.4.417" \
  "(isDeleted=TRUE)"
# Query specific properties
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
