## Self-signed Certificate generation

```sh
openssl req -x509 -newkey rsa:4096 -keyout key.pem -out selfsigned.pem -sha256 -days 365
```

## Certificate Conversion

Download certificate of any service, like `https`

```sh
openssl s_client -connect example.com:443 | openssl x509 > example.pem
```

This stores the certificate in the `PEM` format. We can convert from `PEM` to other formats such as `DER` and `PKCS#7`

```sh
## PEM to DER
Ag2S@htb[/htb]$ openssl x509 -outform der -in hackthebox.pem -out hackthebox.der

## PEM to PKCS#7
Ag2S@htb[/htb]$ openssl crl2pkcs7 -nocrl -certfile hackthebox.pem -out hackthebox.p7
```

## Key Generation

Generate a 2048 bit `RSA` private key

```sh
openssl genrsa -out key.pem 2048
```

Extract the public key from the private key generated

```sh
openssl rsa -in key.pem -pubout > key_pub.pem
```

## Encryption and Decryption

Encryption using the public key key we generated in [#Key Generation]({{< relref "#key-generation" >}})

```sh
openssl pkeyutl -encrypt -inkey key_pub.pem -pubin -in msg.txt -out msg.enc
```

Decryption using private key

```sh
openssl pkeyutl -decrypt -inkey key.pem -in msg.enc > decrypted.txt
```

## Brute force decrypt `.gz`

This is an example. `.gz` files does not have a native way to encrypt, but people usually use `openssl` to encrypt them. This is how to decrypt, brute-force-ly

```sh
for i in $(cat rockyou.txt);do openssl enc -aes-256-cbc -d -in GZIP.gzip -k $i 2>/dev/null| tar xz;done
```