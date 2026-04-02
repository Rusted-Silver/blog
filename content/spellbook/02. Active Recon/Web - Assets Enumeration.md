# Crawling
## Katana
> `ProjectDiscovery` tool. See [ProjectDiscovery Tools]({{< relref "01. Passive Recon/Subdomain Enumeration#projectdiscovery-tools" >}})

Crawl and parse `js` files for more endpoints

```sh
katana -u domain.name -jsl | tee -a assets.txt
cat domains.txt | katana -jsl -kf all -td -d 10 | tee -a assets.txt
```
## ReconSpider
> See [Reconspider]({{< relref "03. Web Pentesting/Scripts#reconspider" >}})
> This tool crawl and parse html comments, links, emails, etc

```sh
python3 ./reconspider.py http://domain.name
```
# Vhost enumeration (No DNS)

**Brute force** subdomains, no DNS. For CTF, not real bug bounty

```sh
ffuf -w /usr/share/wordlists/SecLists/Discovery/DNS/bitquark-subdomains-top100000.txt:FUZZ -u http://$target/ -H "Host: FUZZ.$target_domain"
```

```sh
gobuster vhost -k -t 12 --append-domain -u http://inlanefreight.htb -w /usr/share/seclists/Discovery/DNS/subdomains-top1million-110000.txt
```
# Subpath (directory/file) enumeration

```sh
gobuster dir -u http://$target/ -w /usr/share/seclists/Discovery/Web-Content/common.txt
```

```sh
ffuf -w /usr/share/seclists/Discovery/Web-Content/common.txt:FUZZ -u http://$target/FUZZ
```