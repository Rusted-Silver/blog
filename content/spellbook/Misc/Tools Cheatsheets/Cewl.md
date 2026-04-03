## `cewl`

`cewl` crawls the website and returns words. Could be useful for creating wordlist

- `-m` minimum word length
- `-d` crawl depth
- `-w` file out

```sh
cewl https://www.inlanefreight.com -d 4 -m 6 --lowercase -w inlane.wordlist
```