>The pivot box will listen on 1 port and forward all traffics received on that port to the target host:port
# Socat

Use on linux pivot box. If they have it.

```sh
socat TCP4-LISTEN:8080,fork TCP4:10.10.14.18:80
```

# Netsh

Use on windows host

```cmd
netsh.exe interface portproxy add v4tov4 listenport=8080 listenaddress=10.129.235.175 connectport=445 connectaddress=172.16.5.5
```

verify

```cmd
netsh.exe interface portproxy show v4tov4
```
