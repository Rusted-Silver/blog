## Ovpn

```bash
sudo openvpn --config ~/lab_Trungle256.ovpn > /dev/null 2>&1 &
## Detach ovpn process out of current terminal
disown
```

## Interactive shell

Before we even get a shell, first switch to `bash` shell on our attacker machine first, since `zsh` is not going to work correctly

```sh
kali$> bash
```

Then, after establishing a revshell with target, **on the reverse shell**, we spawn a interactive shell

```sh
target$> python3 -c 'import pty; pty.spawn("/bin/bash")'
```

We need to pause our current shell listener (maybe `netcat`) to connect our pty to the reverse shell

```sh
target$> ^Z
```

Then, we connect our PTY to the reverse shell

```sh
kali$> stty raw -echo
# note down these 2 output
kali$> echo $TERM
kali$> stty size
# bring up netcat listener again
kali$> fg

target$> export TERM=xterm-256color
# Use the values of `stty size` we noted down
target$> stty rows 61 columns 161
```

## Terminal logging

 Terminal logging, `script`. with `-fqa` flag, it basically allows you to log multiple terminal in one file, at the same time.

```sh
script -fqa
```

Furthermore, In case you used `clear`, and is afraid that your whole terminal log is lost. Fear not, here is how you "restore" it

```sh
sed -E 's/\x1b\[H|\x1b\[2J|\x1b\[3J//g' typescript > clean.txt
```

> Basically, when you do `clear`, it just prints these special characters as terminators that clear your screen. So when you log your terminal, those special characters is in the log file, and when you `cat` it out, the same special characters will again, clear your screen

## CVE search

- [CVEdetails](https://www.cvedetails.com/)
- [Exploit DB](https://www.exploit-db.com)
- [Vulners](https://vulners.com)
- [Packet Storm Security](https://packetstormsecurity.com)
- [NIST](https://nvd.nist.gov/vuln/search?execution=e2s1)

## Quick Online Tools

- [Revshell generator](https://www.revshells.com/)
- [CrackStation (rainbow table)](https://crackstation.net/)

## AI report prompt

```
Hi, can you please help me write a report for a penetration test session? I will describe the finding, and you will help me write each section of the finding.

Here's the finding:
I found a SSRF vulnerability. This let me do a port scan on the system's localhost. After doing the portscan, I found a prototype application opens only on localhost. This application is vulnerable to command injection. The final result is RCE

Here's all the sections that I need you to write:
- A proper title. Professional, short. I don't mind if the title miss some details, as long as the core impact and vulnerability is there. The missing details can be filled in on other sections.
- CWE: Only 1 CWE allowed. If it is a chained exploitation, choose CWE of the first vulnerability.
- CVSS 4.0 score, based on what you think. Must include the vectors
- Description & Cause: Very generic, not like telling a story, but more like a extremely generic overview of the vulnerability itself or the exploitation chain.
- Impact: Expand based on what i described above. However, again, generic is the key. It should be boring, not like telling a heroic story
- Reference: links
```