## Full TCP scan
### RustScan
I usually use rustscan for pure port scan since it will be faster and not much syntax. I saved it in my `PATH` to call it easier, but you do you

[Github Repo](https://github.com/bee-san/RustScan)

Download and extract binary

```sh
wget https://github.com/bee-san/RustScan/releases/download/2.4.1/x86_64-linux-rustscan.tar.gz.zip
unzip x86_64-linux-rustscan.tar.gz.zip
tar -xzvf x86_64-linux-rustscan.tar.gz
```

Quick all ports tcp scan.

```sh
./rustscan -a 10.10.10.10,10.10.10.20 -g
```
### Nmap

```sh
nmap -sS -p- --min-rate=1000 -T4 $target --max-retries 0 | grep '^[0-9]' | cut -d '/' -f 1 | tr '\n' ',' | sed s/,$//
```
## Script scan

I do this as my "default" scan

```sh
nmap -sVC -T4 -vv --reason -Pn -oA nmap/script -p $ports 10.10.10.10
```

Then do this to have an easy document to look at. I prefer this than the `nmap` output text

```sh
xsltproc nmap/script.xml -o nmap/script.html
open nmap/script.html
```
## Host discovery

```sh
nmap 10.129.2.0/24 -sn -oA nmap-host-discovery | grep for | cut -d" " -f5
```
## Scan delayed response time services

```sh
nmap --min-rtt-timeout 15 $target
```
## Interesting flags

| Flag                 | What do?                                                   |
| -------------------- | ---------------------------------------------------------- |
| `-Pn`                | No ICMP ping                                               |
| `--reason`           | Displays the reason for specific result                    |
| `--packet-trace`     | Shows all packets sent and received                        |
| `-iL`                | Scan IPs in a list file                                    |
| `--disable-arp-ping` | Self explainatory, use in case you only want ICMP ping     |
| `--top-ports=%`      | Scan `%` number of most scanned ports on `nmap` database   |
| `-F`                 | Scan top 100 ports                                         |
| `-n`                 | Disable DNS resolving                                      |
| `--stats-every=5s`   | Show status of the scan every 5s                           |
| `--max-retries`      | Self explainatory                                          |
| `-g`                 | Specify source port to scan. Usually `-g 53` to bypass IDS |
| `--script-trace`     | Traces commands sent to service, response                  |
## TTL to OS

Yes, you can find out from [here](https://subinsb.com/default-device-ttl-values/) and [here](https://ostechnix.com/identify-operating-system-ttl-ping/) the host's OS by time to live (`TTL`) parameter in the response packet. This includes UDP, TCP, ICMP packets