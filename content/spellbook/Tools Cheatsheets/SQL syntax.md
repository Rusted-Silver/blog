# MSSQL

List all DB

```sql
SELECT name FROM master.dbo.sysdatabases;
```

List table in `somedb` DB

```sql
SELECT TABLE_NAME FROM somedb.INFORMATION_SCHEMA.TABLES WHERE TABLE_TYPE = 'BASE TABLE';
```

List columns in `users` table

```sql
SELECT COLUMN_NAME FROM INFORMATION_SCHEMA.COLUMNS WHERE TABLE_NAME = 'users';
```

Select first one in ascending order of username

```sql
SELECT TOP 1 username FROM users ORDER BY username ASC;
```

Select username of users whose first name starts with `S`

```sql
SELECT username FROM users WHERE firstName LIKE 'S%';
```

Select username of users whose email longer than 20 characters

```sql
SELECT username FROM users WHERE LEN(email) > 20;
```

Join 2 tables and select all post contents and author's username

```sql
SELECT TOP 1 username,content FROM users JOIN posts ON posts.authorId = users.id;
```
