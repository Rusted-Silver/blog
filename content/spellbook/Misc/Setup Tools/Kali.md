## Packages

```sh
## https://github.com/projectdiscovery/pdtm
sudo apt update && sudo apt install golang-go -y
go install -v github.com/projectdiscovery/pdtm/cmd/pdtm@latest

## https://vscodium.com/
wget -qO - https://gitlab.com/paulcarroty/vscodium-deb-rpm-repo/raw/master/pub.gpg \
    | gpg --dearmor \
    | sudo dd of=/usr/share/keyrings/vscodium-archive-keyring.gpg
echo 'deb [arch=amd64,arm64 signed-by=/usr/share/keyrings/vscodium-archive-keyring.gpg] https://download.vscodium.com/debs vscodium main' \
    | sudo tee /etc/apt/sources.list.d/vscodium.list

## https://cloudsmith.io/~kong/repos/insomnia/setup/#formats-deb
## Since they don't have package for kali, we spoof to ubuntu focal, works
## Related issue: https://github.com/Kong/insomnia/issues/6990
curl -1sLf \
  'https://packages.konghq.com/public/insomnia/setup.deb.sh' \
  | sudo -E distro=ubuntu codename=focal arch=amd64 component=main bash

## Install useful packages
sudo apt update
sudo apt install -y seclists jq python3-scrapy feroxbuster rlwrap faketime ntpsec-ntpdate hurl
pipx install git+https://github.com/Esonhugh/Gopherus3.git
pipx install flask-unsign
pipx install git-dumper
## Password vault apps, if found password file 
sudo apt install -y keepassxc passwordsafe
## Good for reporting
sudo apt install -y obsidian flameshot

## Install codium (vscode but open source)
sudo apt install -y codium

## Install insomnia (API testing client)
sudo apt install -y insomnia

## Install all ProjectDiscovery tools
sudo apt install -y massdns libpcap-dev
pdtm -ia

## Full kali upgrade
sudo apt -y full-upgrade
```

## Kali genmon IP indicator

```sh
sh -c 'ip -o -4 addr show "$([ -d /sys/class/net/tun0 ]&&echo tun0||echo eth0)"|awk "{print \$2\": \" \$4}"|cut -d/ -f1'
```

## Setup `flameshot`

Go to `Settings Manager`
![](/images/0087cfed585a0ba7af6ea10d8d602aac7135e11adcaa4baf6bb40e0236ff658e.jpeg)

![](/images/4425a49ac00a9798ffa6cd8e95e27dcbcccdc7b8fa7fc481a3bd4ba06dfcbfa7.jpeg)

![](/images/92f7c2e5776219aa76a8f3a91cb7c00fbb3deb2b10d98c49dd5c0761e36fb3b8.jpeg)

![](/images/3db640718674728e2ac0ffb3b7d15ae536c135f0bdf1ac508cb3d2820d57be01.jpeg)

`/usr/bin/flameshot gui`
![](/images/8423e73a6d97b1049b6b864c453de39bd3b607ce98fb255af065cb1ffacd1105.jpeg)

## Sleep off, lockscreen off

Go to `Settings Manager`

![](/images/0087cfed585a0ba7af6ea10d8d602aac7135e11adcaa4baf6bb40e0236ff658e.jpeg)

![](/images/5c090d71cb7e48dc92c0a1439b2a1f787c57abb51799b8154445630c49324435.jpeg)

![](/images/ee6afb9f549669209ea30954cd666b7b786c13c7bc0b656379a9627f62469faf.jpeg)

![](/images/3e447d8852ffb6e6990117f3fdb7dbb984186defd943a1369090077805a5b9a8.jpeg)

![](/images/8ef7d4757b4a1459a3fc34ee13bdb8662f518a2c3fd0a7f8844eee5c9b04592d.jpeg)

![](/images/a1c871b324bc8aadb9ae68d25bf44895efbebdbe237e753226bd301cd34c9621.jpeg)

![](/images/a79d44253ceb75535732446228e601870d804187f2a2f98689d737ffc58d4e62.jpeg)
