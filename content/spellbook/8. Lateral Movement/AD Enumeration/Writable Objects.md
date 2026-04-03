# Writable Objects

Sometimes, we have permission to write specific things on another object, and bloodhound doesn't show that.

```sh
bloodyAD --host "dc01.domain.name" -d "domain.name" -u 'user' -p 'pass' get writable --detail
```