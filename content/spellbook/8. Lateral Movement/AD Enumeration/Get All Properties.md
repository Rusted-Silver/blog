# Get All Properties

See all attributes of an object

```sh
bloodyAD --host "dc01.domain.name" -d "domain.name" -u 'user' -p 'pass' get object 'victim' --resolve-sd
```

Or, get a specific attribute

```sh
bloodyAD --host "dc01.domain.name" -d "domain.name" -u 'user' -p 'pass' get object 'victim' --attr logonHours 
```
