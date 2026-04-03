## Recon

```sh
nmap -sV -p 873 $target
```

## Probing share dirs

[More](https://book.hacktricks.xyz/network-services-pentesting/873-pentesting-rsync)

```sh
nc -nv $target 873
```

```sh
rsync -av --list-only rsync://$target/<dir>
```

## Retrieve shared files

```sh
rsync -av rsync://$target/<dir>
```

[More](https://phoenixnap.com/kb/how-to-rsync-over-ssh)

```sh
rsync -av rsync://$target/<dir> -e 'ssh -p22'
```
