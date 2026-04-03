## Linux

- List all network interfaces

```sh
ip link
ip a
```

- Default gateway? Next hop?

```sh
route
```

- ARP cache?

```sh
arp -a
```

- DNS server?

```sh
cat /etc/resolv.conf
```

- Non-standard domain names?

```sh
cat /etc/hosts
```

## Windows

- All network interfaces

```cmd
ipconfig /all
```

List all IP in arp table

```cmd
arp -a
```

List routing table

```cmd
route print
```
