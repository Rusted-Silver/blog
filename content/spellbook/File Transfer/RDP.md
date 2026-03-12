Mount a drive from linux machine when connect to rdp. Connect with `\\tsclient\`

```sh
xfreerdp3 /u:"$user" /p:"$pass" /v:$target /drive:linux,/home/kali/payloads
```

```sh
rdesktop $target -d HTB -u administrator -p 'Password0@' -r disk:linux='/home/kali/payloads'
```