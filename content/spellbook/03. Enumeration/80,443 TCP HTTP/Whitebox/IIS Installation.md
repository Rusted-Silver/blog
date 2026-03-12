# Installation
Go to windows features, tick all these boxes
![](/images/42b8202c39ef13719ac074c31e68d678.png)
# Configuration
To add a site, go to ISS manager -> Content View -> Sites -> Add Website
![](/images/d5356748b64a69f18a085c34d384ffaf.png)
Then enter some info, run either localhost or private IP, or VPN IP
![](/images/e98152be6fbb26361e31aa27e5436f14.png)
If later on, you want to edit IP address, port, go to IIS manager -> Content View -> Sites -> right click on \<Your_site\> -> Edit Bindings
![](/images/577e9cd77fc67684223c9bc900ff84b6.png)
# Debugging
## Disable JIT optimization
By default, IIS have JIT optimization that makes debugging complicated. We can download [this script](https://gist.github.com/richardszalay/59664cd302e66511618f51eaaa77db26) to disable that. First, open powershell as admin
```powershell
wget https://gist.github.com/richardszalay/59664cd302e66511618f51eaaa77db26/raw/e6f28fd32b693f9f98c538c63880c3eb50e317f4/IISAssemblyDebugging.psm1 -o IISAssemblyDebugging.psm1
powershell -ep bypass
Import-Module .\IISAssemblyDebugging.psm1
Enable-IISAssemblyDebugging C:\inetpub\wwwroot\TeeTrove.Publish\
```
## Install dnSpy
[This repository](https://github.com/dnSpy/dnSpy/releases) is archived, but whatever. Download the zip, run the thing
Then File -> Open or just `CTRL + O`
And open every executable in the website directory
![](/images/2592026651885b3538a17cded67df7e4.png)
Next, Debug -> Attach to Process or `CTRL + ALT + P`
![](/images/9ef1b73b8b2b9ab10ecbfddd0501fe9a.png)
Find the `w3wp.exe` process. If not appear, run dnSPY as admin. If not appear again, try send a request and refresh
![](/images/cebfe3dcaed87f8c098eb61ab5ba414d.png)
