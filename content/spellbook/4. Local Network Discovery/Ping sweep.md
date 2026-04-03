## Calculate subnet

I would never do this manually. Only CCNA exam make you do this manually, which is a nightmare

```sh
ipcalc 172.16.5.129/23
```

## Windows

### CMD

```cmd
for /L %i in (1 1 254) do ping 172.16.5.%i -n 1 -w 100 | find "Reply"
```

### Powershell

```powershell
1..254 | % {"172.16.5.$($_): $(Test-Connection -count 1 -comp 172.16.5.$($_) -quiet)"}
```

## Linux

```sh
for i in {1..254} ;do (ping -c 1 172.16.5.$i | grep "bytes from" &) ;done
```

```sh
fping -asgq 172.16.4.0/23
```