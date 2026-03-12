# Real environment

Use [interact.sh](https://app.interactsh.com). This is the cheapest (free) one for POC. Otherwise, setup your own exfiltration server on cloud, or on prem if you're nuts

# Lab, CTF

Create a certificate. Outside of lab environment, better set this up with `Nginx` or `Apache2` and request cert from letsencrypt

```sh
openssl req -new -x509 -keyout server.pem -out server.pem -days 365 -nodes
```

Open a server

```python
from http import server
import ssl

class CustomRequestHandler(server.SimpleHTTPRequestHandler):
    def do_OPTIONS(self):
        self.send_response(200)
        self.send_header("Access-Control-Allow-Origin", "*")
        self.send_header("Access-Control-Allow-Methods", "GET, POST, OPTIONS")
        self.send_header("Access-Control-Allow-Headers", "Content-Type")
        self.end_headers()

    def do_GET(self):
        super().do_GET()

    def do_POST(self):
        length = int(self.headers.get('Content-Length', 0))
        body = self.rfile.read(length)

        if body:
            self.log_message("[i] POST body: %s", body.decode("utf-8", errors="replace"))

        self.send_response(200)
        self.end_headers()

print("Serving HTTPS on 0.0.0.0 port 4443 (https://0.0.0.0:4443/) ...")
httpd = server.HTTPServer(('0.0.0.0', 4443), CustomRequestHandler)
context = ssl.SSLContext(ssl.PROTOCOL_TLS_SERVER)
context.load_cert_chain(certfile='./server.pem')
httpd.socket = context.wrap_socket(httpd.socket, server_side=True)
httpd.serve_forever()
```

For payloads in [XSS]({{< relref "03. Enumeration/80,443 TCP HTTP/Common Vulnerability/XSS" >}}) and [CSRF]({{< relref "03. Enumeration/80,443 TCP HTTP/Common Vulnerability/CSRF" >}})

```python
from http import server
import ssl
import json
import base64

class CustomRequestHandler(server.SimpleHTTPRequestHandler):
    def do_OPTIONS(self):
        self.send_response(200)
        self.send_header("Access-Control-Allow-Origin", "*")
        self.send_header("Access-Control-Allow-Methods", "GET, POST, OPTIONS")
        self.send_header("Access-Control-Allow-Headers", "Content-Type")
        self.end_headers()

    def do_GET(self):
        super().do_GET()

    def do_POST(self):
        length = int(self.headers.get('Content-Length', 0))
        body = self.rfile.read(length)

        if body:
            self.log_message("[i] POST body: %s", base64.b64decode(json.loads(body.decode("utf-8", errors="replace"))["data"]))

        self.send_response(200)
        self.end_headers()

print("Serving HTTPS on 0.0.0.0 port 4443 (https://0.0.0.0:4443/) ...")
httpd = server.HTTPServer(('0.0.0.0', 4443), CustomRequestHandler)
context = ssl.SSLContext(ssl.PROTOCOL_TLS_SERVER)
context.load_cert_chain(certfile='./server.pem')
httpd.socket = context.wrap_socket(httpd.socket, server_side=True)
httpd.serve_forever()
```