# Talisman
![](./screenshots/logo.png)

# Scenario
### [](https://www.hacksmarter.org/courses/5e5b9833-e6be-4fa0-aa4d-efd3086a612c#user-content-objective-and-scope)Objective and Scope

You have been assigned a penetration test on a critical Linux server in the client's environment. The scope is strictly limited to a **single Linux server environment** designated as the target. The primary objective is to gain **root-level access** to this system to demonstrate maximum impact and the full extent of the security compromise to the client.

A set of leaked credentials, recently recovered from a third-party data breach, have been provided. While the specific service or application these credentials belong to is unknown, they serve as the initial vector for establishing a foothold.

#### [](https://www.hacksmarter.org/courses/5e5b9833-e6be-4fa0-aa4d-efd3086a612c#user-content-leaked-credentials)Leaked Credentials
```
jane / Greattalisman1! 
```
## Enumeration
```
➜ nmap -sV -T4 -p- 10.1.88.76
Starting Nmap 7.98 ( https://nmap.org ) at 2026-01-10 08:23 -0500

PORT     STATE SERVICE VERSION
22/tcp   open  ssh     OpenSSH 8.0 (protocol 2.0)
8978/tcp open  unknown
1 service unrecognized despite returning data. If you know the service/version, please submit the following fingerprint at https://nmap.org/cgi-bin/submit.cgi?new-service :
SF-Port8978-TCP:V=7.98%I=7%D=1/10%Time=6962568F%P=x86_64-pc-linux-gnu%r(Ge
SF:tRequest,18A8,"HTTP/1\.1\x20200\x20OK\r\nDate:\x20Sat,\x2010\x20Jan\x20
SF:2026\x2013:39:41\x20GMT\r\nCache-Control:\x20no-cache,\x20no-store,\x20
SF:must-revalidate\r\nContent-Type:\x20text/html\r\nExpires:\x200\r\nConte
SF:nt-Length:\x206145\r\n\r\n<!doctype\x20html>\n<html\x20lang=\"en\"\x20d
SF:ir=\"ltr\"\x20data-version=\"25\.2\.0\.202509010904\">\n\x20\x20<head>\
SF:n\x20\x20\x20\x20<meta\x20charset=\"UTF-8\"\x20/>\n<meta\x20name=\"view
SF:port\"\x20content=\"width=device-width,\x20initial-scale=1,\x20shrink-t
SF:o-fit=no\"\x20/>\n<meta\x20content=\"text/html;\x20charset=utf-8\"\x20/
SF:>\n<link\x20rel=\"manifest\"\x20href=\"/manifest\.webmanifest\"\x20/>\n
SF:\n<link\x20rel=\"alternate\x20icon\"\x20type=\"image/png\"\x20href=\"/f
SF:avicon\.png\"\x20sizes=\"16x16\"\x20/>\n<link\x20rel=\"icon\"\x20type=\
SF:"image/png\"\x20sizes=\"32x32\"\x20href=\"/favicon-32x32\.png\"\x20/>\n
SF:<link\x20rel=\"icon\"\x20type=\"image/svg\+xml\"\x20href=\"/favicon\.sv
SF:g\"\x20/>\n<link\x20rel=\"apple-touch-icon\"\x20sizes=\"180x180\"\x20hr
SF:ef=\"/apple-touch-icon\.png\"\x20/>\n<link\x20rel=\"prefetch\"\x20href=
SF:\"/icons/info_icon\.svg\"\x20/>\n<link\x20rel=\"prefetch\"\x20href=\"/i
SF:cons/info_icon_sm\.svg\"\x20/>\n<link\x20rel=\"prefetch\"\x20href=\"")%
SF:r(HTTPOptions,66,"HTTP/1\.1\x20200\x20OK\r\nDate:\x20Sat,\x2010\x20Jan\
SF:x202026\x2013:39:42\x20GMT\r\nAllow:\x20GET,\x20HEAD,\x20OPTIONS\r\nCon
SF:tent-Length:\x200\r\n\r\n")%r(RTSPRequest,220,"HTTP/1\.1\x20505\x20HTTP
SF:\x20Version\x20Not\x20Supported\r\nDate:\x20Sat,\x2010\x20Jan\x202026\x
SF:2013:39:42\x20GMT\r\nCache-Control:\x20must-revalidate,no-cache,no-stor
SF:e\r\nContent-Type:\x20text/html;charset=iso-8859-1\r\nContent-Length:\x
SF:20349\r\n\r\n<html>\n<head>\n<meta\x20http-equiv=\"Content-Type\"\x20co
SF:ntent=\"text/html;charset=ISO-8859-1\"/>\n<title>Error\x20505\x20Unknow
SF:n\x20Version</title>\n</head>\n<body>\n<h2>HTTP\x20ERROR\x20505\x20Unkn
SF:own\x20Version</h2>\n<table>\n<tr><th>URI:</th><td>/badMessage</td></tr
SF:>\n<tr><th>STATUS:</th><td>505</td></tr>\n<tr><th>MESSAGE:</th><td>Unkn
SF:own\x20Version</td></tr>\n</table>\n\n</body>\n</html>\n");
```

The service on 8978 is a web service called [CloudBeaver](https://github.com/dbeaver/cloudbeaver) and the credentials are valid to access the portal. It also appears to be containerized.

![Beaver Cloud](./screenshots/sql.png)

Immediately the attacker can identify this is an Oracle database.

The attacker will enumerate tables and views.
![Selecting all views and tables](./screenshots/sql-2.png)

From that the attacker can further enumerate users.
![Selecting all users](sql-3.png)

The attacker will enumerate their current privileges.
![](./screenshots/sql-4.png)
![](./screenshots/sql-4-1.png)

`INHERIT PRIVILEGES`
Allows the current user to inherit the definer rights of another schema.

`READ / WRITE / EXECUTE` on `EXT_DATA`
This privilege is interesting because it may allow for data exfiltration/file overwrite/or staging for some kind of RCE.

Checking the object type shows that it's a directory which is the most dangerous type. This means it's mapped to the OS.
![](./screenshots/sql-5.png)

The attacker can check where the folder is mounted. In this case it's `/opt/data`
![[sql-6.png]]

With the `CREATE ANY DIRECTORY` and `DROP ANY DIRECTORY` privileges, the user can create a directory object pointing to any OS path Oracle can access.