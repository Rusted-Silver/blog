## `netexec`

```sh
netexec smb $target -u "$user" -p "$pass" --users
```

## `enum4linux`

```sh
enum4linux -U $target  | grep "user:" | cut -f2 -d"[" | cut -f1 -d"]"
```

## `impacket-samrdump`

```sh
impacket-samrdump $target
```

## `ldapsearch`

```sh
ldapsearch -h $target -x -b "DC=INLANEFREIGHT,DC=LOCAL" -s sub "(&(objectclass=user))"  | grep sAMAccountName: | cut -f2 -d" "
```

## `kerbrute`

Clone username list repo

```sh
git clone https://github.com/insidetrust/statistically-likely-usernames.git
```

Download `kerbrute` binary. Yes, it does not exist in kali's package manager. And sadly, at the time of writing, it hasn't been updated since 2019

```sh
wget https://github.com/ropnop/kerbrute/releases/download/v1.0.3/kerbrute_linux_amd64
mv kerbrute_linux_amd64 kerbrute
chmod +x kerbrute
```

Depends on the account naming convention, chose the correct username list file.

```sh
./kerbrute userenum -d inlanefreight.local --dc $target ./jsmith.txt
```

## `rcpclient`

null session

```sh
rpcclient -N -U '%' $target
```

Or, need valid credential

```sh
rpcclient -U "$user%$pass" $target
```

```
rpcclient $> enumdomusers
```

### Basic commands

```sh
## Domain and user unumerate
rpcclient $> srvinfo
rpcclient $> enumdomains
rpcclient $> querydominfo
rpcclient $> enumdomusers
rpcclient $> queryuser <RID>
```

### RID brute force

Brute forcing RID from 0x500 to 0x1100

```sh
for i in $(seq 500 1100);do rpcclient -N -U "" $target -c "queryuser 0x$(printf '%x\n' $i)" | grep "User Name\|user_rid\|group_rid" && echo "";done
```
