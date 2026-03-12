# `hashcat`

- `-a` specifies attack mode
- `-m` specifies hash type

## Dictionary

```sh
hashcat -a 0 -m 0 ./md5hash /usr/share/wordlists/rockyou.txt
```

## Rules

Try password from wordlist with some modification, like `password123` to `password123!@#`

```sh
hashcat -a 0 -m 0 ./md5hash /usr/share/wordlists/rockyou.txt -r /usr/share/hashcat/rules/best64.rule
```

## Mask

- `?u` is uppercase letter
- `?l` is lowercase letter
- `?d` is digit
- `?s` is symbol

Aside from that, we can specify `-1`, `-2`, `-3`, `-4` to "combine" charsets

Example, `Password123` would be `?u?l?l?l?l?l?l?l?d?d?d`. If you know exactly the format of the password, which, in most case, we don't.

In that case, we can "guess". If we know exactly password length is 3 and only have uppercase and lowercase letters, digits and symbol:

```sh
hashcat -a 3 -m 0 -1 '?u?l?d?s' ./md5hash '?1?1?1'
```

Or we know some starting characters but not password length or charset

```sh
hashcat -a 3 -m 0 --increment ./md5hash 'Pass?a?a?a?a?a?a?a?a?a'
```

The reason why we specify so many `?a` is because we have `--increment` flag. It will try adding one by one guessed character to the password.

## Custom rule

| **Function** | **Description**                                  |
| ------------ | ------------------------------------------------ |
| `:`          | Do nothing                                       |
| `l`          | Lowercase all letters                            |
| `u`          | Uppercase all letters                            |
| `c`          | Capitalize the first letter and lowercase others |
| `sXY`        | Replace all instances of X with Y                |
| `$!`         | Add the exclamation character at the end         |

See [more](https://hashcat.net/wiki/doku.php?id=rule_based_attack)

Example rules

```
:
c
so0
sa@
si1
si!
ss$
$0$8
$0$5
$1$9$9$8
$!
```

Quick script to combine rules (from the example above)

```python
import itertools

# Read rules from file, stripping whitespace
with open('custom.rules', 'r') as f:
    rules = [line.strip() for line in f if line.strip()]

# Generate all non-empty permutations
for i in range(1, len(rules)+1):
    for combo in itertools.combinations(rules, i):
        print(''.join(combo))
```
