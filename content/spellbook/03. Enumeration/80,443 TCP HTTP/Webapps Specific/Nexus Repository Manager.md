# RCE

First, somehow login as `admin`

![](/images/d2c6b2d73b548d28ed18ad7debd91524.png)

Then navigate to admin settings (the cog icon on top) -> `Tasks` -> `Create Task`

![](/images/062668b813cbe031a5d2f5b3ab3e0a42.png)

Create a `Admin - Execute script` task

![](/images/1219816e11214dbd98018cd8bf0af728.png)

Fill out the task. Use a `groovy` reverse shell script from https://www.revshells.com

![](/images/f155ebe5d7e0a84364a041343c8ce76a.png)

After creating the task, it will redirect us back to this `Tasks` page. Click on the task we just created

![](/images/8c2ccdde8a08be308f884a60a8da8309.png)

Run a netcat listener on your side first

```sh
rlwrap nc -nvlp 9001
```

Finally, run the task, and you should have the shell on your netcat listener

![](/images/c9d4084ab4847bfafc3f013d62d5f9ad.png)