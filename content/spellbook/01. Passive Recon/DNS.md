# Dig

NS query for more name server for enumerate. `$target` is dns server's ip address

```sh
dig ns domain.name @10.10.11.78
```

Find IP of a domain, less info but more effective for scripts

```sh
dig +short domain.name @10.10.11.78
```

Dig dns server's version, if there is a record in the dns server

```sh
dig CH TXT version.bind @10.10.11.78
```

Dig all

```sh
dig any inlanefreight.htb @10.10.11.78
```

Zone transfer

```sh
dig axfr inlanefreight.htb @10.10.11.78
```
# NSLookup

Basic syntax, more info than `dig +short`

```sh
nslookup
> server 10.10.11.78
> domain.name
```
# NSUpdate

Basic syntax, use to update dns records if the dns server has no authentication

```sh
nsupdate
> server 10.10.11.78
> update add sub.domain.name 3600 A IP
> send
```
## Basic DNS records

- `A` records: IP that the domain points to.
- `MX` records: Mail server record. Who or "what" is responsible for managing the emails for the company.
- `NS` records: Points to a name server which will resolve their domain name into IP.
- `TXT` records: Contains verification keys, such as [SPF](https://datatracker.ietf.org/doc/html/rfc7208), [DMARC](https://datatracker.ietf.org/doc/html/rfc7489), and [DKIM](https://datatracker.ietf.org/doc/html/rfc6376), which are responsible for verifying and confirming the origin of the emails sent.

## Tips
A DNS zone usually has a `SOA` record. For example, if you 

```sh
dig soa dev.domain.name
```

and it returns a valid `SOA` record, it usually means there are further subdomain of that `dev` subdomain, like `mail.dev.doamin.name`.
