## Password file search

From HTB

```sh
for l in $(echo ".conf .config .cnf");do echo -e "\nFile extension: " $l; find / -name *$l 2>/dev/null | grep -v "lib\|fonts\|share\|core" ;done
```
```sh
for i in $(find / -name *.cnf 2>/dev/null | grep -v "doc\|lib");do echo -e "\nFile: " $i; grep "user\|password\|pass" $i 2>/dev/null | grep -v "\#";done
```
```sh
for l in $(echo ".sql .db .*db .db*");do echo -e "\nDB File extension: " $l; find / -name *$l 2>/dev/null | grep -v "doc\|lib\|headers\|share\|man";done
```
```sh
find /home/* -type f -name "*.txt" -o ! -name "*.*"
```
```sh
for l in $(echo ".py .pyc .pl .go .jar .c .sh");do echo -e "\nFile extension: " $l; find / -name *$l 2>/dev/null | grep -v "doc\|lib\|headers\|share";done
```

[MiniPenguin](https://github.com/huntergregal/mimipenguin)(Need privilege)

[Firefox Decrypt](https://github.com/unode/firefox_decrypt)

Homemade

```sh
for e in $(echo ".conf .config .json .yaml .yml .toml");do find / -name "*$e" 2>/dev/null -exec grep -HiE "passw|pwd" {} \;;done
```