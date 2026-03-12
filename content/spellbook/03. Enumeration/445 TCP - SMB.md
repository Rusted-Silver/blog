# Recon

```sh
nmap --script smb-os-discovery.nse -p445 $target
```

# No credential (Null session)

>If guest account is enable, you can set `user='guest'` and no `pass`.
## List available shares

```sh
smbclient -N -L //$target/
```

```sh
smbmap -H $target
```

## Access share

```sh
smbclient //$target/<dir> -U "%"
```

## `enum4linux-ng`

Utilizes `nmblookup`, `net`, `rpcclient`, and `smbclient` to automate some common enumeration from SMB targets such as:

- Workgroup/Domain name
- Users information
- Operating system information
- Groups information
- Shares Folders
- Password policy information

```sh
sudo apt install enum4linux-ng
```

```sh
enum4linux-ng $target -A -C -oA adenum
```

## Spider for interesting files

After running this, it will output to a file, which will be shown in the command output. Read that json file for a list of files in the network share

```sh
nxc smb $target -u '' -p '' -M spider_plus
```

### Spider and search

Search the the word 'passw' in every file

```sh
netexec smb $target -u "" -p "" --spider IT --content --pattern "passw"
```

### Download all files from smb

```sh
netexec smb $target -u "" -p """ -M spider_plus -o DOWNLOAD_FLAG=True --smb-timeout 60 -t 15
```

## Mount smb

### Linux

## Without credential

```sh
sudo mount -t cifs -o domain=. //192.168.220.129/Finance /mnt/Finance
```

## With credential

```sh
sudo mount -t cifs -o username=plaintext,password=Password123,domain=. //192.168.220.129/Finance /mnt/Finance
```

Or, with credential file

```sh
mount -t cifs //192.168.220.129/Finance /mnt/Finance -o credentials=/path/credentialfile
```

credentialfile

```
username=user
password=pass
domain=domain.local
```

### Windows - CMD

## Without credential

```cmd
net use n: \\192.168.220.129\Finance
```

## With credential

```cmd
net use n: \\192.168.220.129\Finance /user:plaintext Password123
```

### Windows - Powershell

## Without credential

```powershell
New-PSDrive -Name "N" -Root "\\192.168.220.129\Finance" -PSProvider "FileSystem"
```

## With credential

```powershell
$username = 'plaintext'
$password = 'Password123'
$secpassword = ConvertTo-SecureString $password -AsPlainText -Force
$cred = New-Object System.Management.Automation.PSCredential $username, $secpassword
New-PSDrive -Name "N" -Root "\\192.168.220.129\Finance" -PSProvider "FileSystem" -Credential $cred
```
