# RCE
## CVE-2018-9276

Version: below 18.2.39. From like 2018, Authenticated command injection

Log in -> setup -> Account Settings -> Notifications

![](/images/c2fe21cb715fc2c028b5b78a3af2028b.png)

Add new notification

![](/images/da2d2e6a8c3c5c747d0bef525c8198b6.png)

Scroll down to Execute program, then do the command injection thing.

![](/images/67e810d1414b594f664b6dd9d8008125.png)

Then send a test notification, which will send us our revshell

![](/images/f0b61fb21ac8d30887686380be7f66b6.png)