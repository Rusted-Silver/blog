> Head to https://chaos.projectdiscovery.io/
# DNS

```sh
dnsenum --dnsserver 8.8.8.8 --enum -p 0 -s 0 --threads 12 -f /usr/share/seclists/Discovery/DNS/subdomains-top1million-20000.txt domain.name
```

```sh
gobuster dns -d domain.name -w /usr/share/SecLists/Discovery/DNS/namelist.txt
```

---
# ProjectDiscovery Tools

Install the tool manager

```sh
sudo apt update && sudo apt install golang-go -y
go install -v github.com/projectdiscovery/pdtm/cmd/pdtm@latest
```

Export go tools in `PATH`. Append this at the end of `~/.bashrc` or `~/.zshrc`

```sh
export GOPATH="$HOME/go"
export PATH="$PATH:$GOPATH/bin"
```

Install all tools. Documentation is [here](https://docs.projectdiscovery.io/opensource)

```sh
sudo apt update && sudo apt install massdns libcap-dev -y
pdtm -ia
```

## Subfinder
> OSINT. It finds subdomains from multiple sources

Should config subfinder a little more. Add some provider API keys

```sh
vim ~/.config/subfinder/config.yaml
vim ~/.config/subfinder/provider-config.yaml
```

```sh
subfinder -d domain.name -all | tee -a domains.txt
```
## Assetfinder
> Not a `ProjectDiscovery` tool.
> Find subdomains and other related domains through OSINT

Install

```sh
go get -u github.com/tomnomnom/assetfinder
```

```sh
assetfinder domain.name -subs-only | tee -a domains.txt
```
## ShuffleDNS
> This tool try to brute force subdomains by resolving them using multiple resolvers. If the domain resolves, it means that it exists.
> You can think of this tool's functionality like gobuster dns

Download resolvers ip list. Not necessarily needed, you can just use `8.8.8.8` but just in case, use other resolvers too for better result

```sh
wget https://github.com/trickest/resolvers/raw/refs/heads/main/resolvers.txt
```

Bruteforce subdomain of a domain name. Need a subdomain wordlist, resolver list

```sh
shuffledns -d domain.com -w /usr/share/wordlists/seclists/Discovery/DNS/shubs-subdomains.txt -r resolvers.txt -mode bruteforce | tee -a ./domains.txt
```

Validate if domains exists by simply resolving the whole domain in a list `domains.txt`

```sh
shuffledns -list ./domains.txt -r resolvers.txt -mode resolve | tee -a domains.txt
```

## Alterx
> This tool take a list of domain names and create some permutations like from `api.domain.name` to `dev-api.domain.name`
> Does not resolve. Should pipe this into `dnsx` to confirm

Should config this a little more

```sh
vim ~/.config/alterx/config.yaml
vim ~/.config/alterx/permutation*.yaml
```

```sh
cat domains.txt | alterx | tee -a altered_DN.txt
```
## DnsX
> This tool try to resolve the domain name. If the domain resolves, it means that it exists.

```sh
cat altered_DN.txt | dnsx | tee -a all_domains.txt
```
## Subzy
> Not a `ProjectDiscovery` tool. This tool find subdomains that we can takeover

```sh
subzy run --targets domains.txt
```
# Httpx
> This tool do a simple `GET` to the target domain(s) and return a status code like `200`, `302`, `404`, etc
> It can also spit more info like title, content length, etc.
> We can find which domains are live with status code `200`. Sometimes, a `404` doesn't mean that it's not live, do directory brute force.

```sh
cat all_domains.txt | httpx -sc > status.txt
cat status.txt | grep '200'
```

```sh
cat all_domains.txt | httpx -sc -cl -location > httpx.txt
```
## Eyewitness
> Not a `ProjectDiscovery` tool
> This tool do a `GET` on the website, take a screenshot, then spits out a report
> Helps us find interesting targets to begin hacking

```sh
eyewitness -f interesting_domains.txt -d eyewitness
```

---
# Shodan
Get list of IP from domain name, then search on shodan

```sh
for i in $(cat subdomainlist);do host $i | grep "has address" | grep inlanefreight.com | cut -d" " -f4 >> ip-addresses.txt;done

for i in $(cat ip-addresses.txt);do shodan host $i;done
```
# Registered SSL Certs

```sh
curl -s 'https://crt.sh/?q=domain.name&output=json' | jq . > domains.json
```

Select and sort any certs that have the word `dev` in `name_value` field

```sh
curl -s 'https://crt.sh/?q=domain.name&output=json' | jq -r '.[]
 | select(.name_value | contains("dev")) | .name_value' | sort -u
```