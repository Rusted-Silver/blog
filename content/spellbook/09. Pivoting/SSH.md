# Port forwards
## Dynamic port forward

>Use when you want to connect from attacker host to internal host, via a pivot box (maybe an edge server with ssh)
>Basically, you can think the ssh pivot box as our socks 4 proxy server.

Set ssh dynamic port forwarding

```sh
ssh -D 9050 "$user@$target"
```

Set up `proxychains`

```sh
$ tail -2 /etc/proxychains4.conf
socks4  127.0.0.1 9050
```

Run some random commands

```sh
sudo proxychains nmap -sT -F --max-retries 0 $target
```

>Yes, `-sT`. You cannot use a half `SYN` handshake here. You have to establish a full TCP handshake.
>Also, yes, `sudo`. For some reason, `nmap` with normal user privilege doesn't work so well with `proxychains`
## Local port forward

>Use when want to connect to port(s) on internal host

Establish local port forward

```sh
ssh -L 8080:172.0.12.13:80 -L 4433:172.0.12.13:443 "$user@$target"
```

interact with internal host

```sh
curl localhost:8080
curl localhost:4433
```

## Remote port forward

>Use when we want reverse shell. Or when you want to expose attacker host's ports to that internal host

Establish remote port forward

```sh
ssh -R 0.0.0.0:8080:0.0.0.0:4444 "$user@$target"
```

> `0.0.0.0:8080` means we want pivot box to listen on port 8080 on every interface.
> `0.0.0.0:4444` means we want the traffics to forward to our attacker host's port `4444`

Set up listener

```sh
nc -nvlp 4444
```