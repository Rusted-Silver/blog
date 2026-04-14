## Linux setup

### Install `krb5-user`

```sh
sudo apt-get install krb5-user
```

>During the installation, you will be prompted to enter domain name, domain controller. However, if you have already installed it before, or missed the config, then:

`/etc/krb5.conf` is the location of `krb5-user` config file. Adjust the `default realm` and `kdc`.

```
[libdefaults]
        default_realm = INLANEFREIGHT.HTB

...SNIP...

[realms]
    INLANEFREIGHT.HTB = {
        kdc = dc01.inlanefreight.htb
    }

...SNIP...
```

### `/etc/hosts`

You might also need to adjust `/etc/hosts` file to add the domain

```
172.16.1.10 inlanefreight.htb   inlanefreight   dc01.inlanefreight.htb  dc01
172.16.1.5  ms01.inlanefreight.htb  ms01
```

## Create Keberos ticket with password

### Sync time with target

```sh
faketime "$(ntpdate -q $target | awk '{print $1" "$2}')" zsh
```

### Request ticket

```sh
impacket-getTGT domain.local/"$user":"$pass" -dc-ip $target
export KRB5CCNAME=ticket.ccache
```

## Convert `.ccache` to `.kirbi`

Useful for using on windows attack host.

```sh
impacket-ticketConverter ticket.ccache julio.kirbi
```
