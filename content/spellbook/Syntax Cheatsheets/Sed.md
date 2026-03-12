Dear god, regular expression

The first `/` is search, the second `/` is command. in this case, we don't have command, so it replace. **(misinfo alert)**

```sh
sed '/search/replaced/g' file
```

`-i` edit the file directly without printing to `stdout` first. I prefer avoid using this, better debug

```sh
sed -i '/search/replaced/g' file
```

`a` command insert our text after printing the `search` string. Meaning it will search our `search` text, then print our `inserted` text the line below

```sh
sed '/search/a inserted' file
```

`i` command insert our text before printing the `search` string

```sh
sed '/search/i inserted' file
```

`r` insert content of another file after printing the `search` string

```sh
sed '/search/r insert.txt' file
```

`e` insert a command output before printing the `search` string

```sh
sed '/search/e cat insert.txt' file
```