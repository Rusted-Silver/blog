## Change current shell to bash

Before we even get a shell, first switch to `bash` shell on our attacker machine first, since `zsh` is not going to work correctly

```sh
kali$> bash
```

## Spawn Interactive shell

Then, after establishing a shell with target, on target machine, we spawn a interactive shell

```sh
python3 -c 'import pty; pty.spawn("/bin/bash")'
```

## Pause current listener

We need to pause our current shell listener (maybe `netcat`) to connect our pty to the reverse shell

```sh
target$> ^Z
```

## Connect our PTY to the reverse shell

```sh
kali$> stty raw -echo
## note down these 2 output
kali$> echo $TERM
kali$> stty size
## bring up netcat listener again
kali$> fg

target$> export TERM=xterm-256color
## Use the values of `stty size` we noted down
target$> stty rows 61 columns 161
```