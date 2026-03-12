> The general direction is, mount the virtual drive on attack host, then extract hash from `<Mount_Point>/Windows/System32/config/` using `impacket-secretsdump`

# Mount VMDK on Linux

```sh
guestmount -a SQL01-disk1.vmdk -i --ro /mnt/vmdk
```

# Mount VHD/VHDX on Linux

```sh
guestmount --add WEBSRV10.vhdx  --ro /mnt/vhdx/ -m /dev/sda1
```

# Mount Bitlocker VHD

# Crack hash

## Rip hash

```sh
bitlocker2john -i Backup.vhd > backup.hashes
```

> `bitlocker2john` creates 4 hashes. The first two correspond to the BitLocker password, while the latter two is the recovery key hash. Recovery key is string with length of 48, so recommended not cracking that. The hash starts with `$bitlocker$0$` is the password hash

```sh
grep "bitlocker\$0" backup.hashes > backup.hash
```

## Crack

```sh
john --wordlist=/usr/share/wordlists/rockyou.txt ./backup.hash
```

# Mount

## Install tool

```sh
sudo apt-get install dislocker
```

## Mount

Create 2 mount dir

```sh
sudo mkdir -p /media/bitlocker
sudo mkdir -p /media/bitlockermount
```

Use `losetup` to configure the VHD as [loop device](https://en.wikipedia.org/wiki/Loop_device)

```sh
sudo losetup -f -P Backup.vhd
```

See what name is the loop device, in this case, `loop0p1`

```sh
lsblk              
NAME      MAJ:MIN RM  SIZE RO TYPE MOUNTPOINTS
loop0       7:0    0   64M  0 loop 
└─loop0p1 259:0    0   61M  0 part
```

Decrypt the drive using `dislocker`. `-u` to specify password. `--` to mark end of program's options

```sh
sudo dislocker /dev/loop0p1 -u1234qwer -- /media/bitlocker
```

Mount the decrypted volume

```sh
sudo mount -o loop /media/bitlocker/dislocker-file /media/bitlockermount
```
