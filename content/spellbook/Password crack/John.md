# `john`

## Rip hash

Rip hash from encrypted files, for example:  .zip`, `.pdf
`
There are tools usually ends with `2john` that can do this for us

```sh
locate *2john*
```

### `.zip` file

```sh
zip2john ./sensitive.zip > ./sensitive.zip.hash
```

## Hash format

Specify which hash format `john` should use to crack. eg `raw-sha1`

```sh
john --format=raw-sha1 ./hash
```

## Single

Useful for Linux credentials. Password generated based on the victim's username, home directory name, and [GECOS](https://en.wikipedia.org/wiki/Gecos_field) values (full name, room number, phone number, etc.).

```sh
echo 'r0lf:$6$ues25dIanlctrWxg$nZHVz2z4kCy1760Ee28M1xtHdGoy0C2cYzZ8l2sVa1kIa8K9gAcdBP.GI6ng/qA4oaMrgElZ1Cb9OeXO4Fvy3/:0:0:Rolf Sebastian:/home/r0lf:/bin/bash' > passwd
john --single ./passwd
```

## Dictionary

```sh
john --wordlist=/usr/share/wordlists/rockyou.txt ./hash
```

## Incremental

Bruteforce all possible password

```sh
john --incremental ./hash
```

