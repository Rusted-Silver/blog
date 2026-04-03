Install client

```sh
go install github.com/nats-io/natscli/nats@latest
```

Create a context, this helps you not having to type out password every time
```sh
nats context add somename --user 'user' --password 'password' --server nts://10.10.11.78:4222
```

See list of commands

```sh
nats -h
```

List streams

```sh
nats --context somename stream ls
```
