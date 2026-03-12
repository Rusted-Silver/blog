# Connect
`openssl`

```sh
openssl s_client -connect $target:imaps
```

`curl`

```sh
curl -k "imaps://$target" --user user:p4ssw0rd -v
```

# Login, read, send, everything

Just... use a GUI app. Thunderbird especially sucks at trusting self signed certs, hence `evolution`

```sh
sudo apt install evolution
```
