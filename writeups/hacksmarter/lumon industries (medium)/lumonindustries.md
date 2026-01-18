# Lumon Industries
![](./screenshots/logo.png)

## Objective / Scope

Lumon Industries will soon be integrating a high-value employee into the organization. In accordance with internal security protocols, a comprehensive penetration test and internal access verification must be conducted prior to full onboarding.

For the purposes of this evaluation, you will be provided the assigned credentials and access permissions corresponding to the subject employee. Your objective is to assess the scope and boundaries of these permissions, ensuring compliance with all Lumon security standards and operational safeguards.
### [](https://www.hacksmarter.org/events/1e17172b-97a8-4c4b-83fa-0abf061887a2#user-content-starting-credentials)Starting Credentials

```
hellyr:H3lenaR!2025
```
## Enumeration

The attacker will start by organizing `hosts.txt`
```        
10.1.28.90
10.1.32.242
```

Then scan the hosts with nmap
```zsh
Not shown: 65512 filtered tcp ports (no-response)
PORT      STATE SERVICE       VERSION
53/tcp    open  domain        Simple DNS Plus
88/tcp    open  kerberos-sec  Microsoft Windows Kerberos (server time: 2026-01-15 21:12:04Z)
135/tcp   open  msrpc         Microsoft Windows RPC
139/tcp   open  netbios-ssn   Microsoft Windows netbios-ssn
389/tcp   open  ldap          Microsoft Windows Active Directory LDAP (Domain: lumons.hacksmarter, Site: Default-First-Site-Name)
445/tcp   open  microsoft-ds?
464/tcp   open  kpasswd5?
593/tcp   open  ncacn_http    Microsoft Windows RPC over HTTP 1.0
636/tcp   open  ssl/ldap      Microsoft Windows Active Directory LDAP (Domain: lumons.hacksmarter, Site: Default-First-Site-Name)
3268/tcp  open  ldap          Microsoft Windows Active Directory LDAP (Domain: lumons.hacksmarter, Site: Default-First-Site-Name)
3269/tcp  open  ssl/ldap      Microsoft Windows Active Directory LDAP (Domain: lumons.hacksmarter, Site: Default-First-Site-Name)
3389/tcp  open  ms-wbt-server
9389/tcp  open  mc-nmf        .NET Message Framing
49664/tcp open  msrpc         Microsoft Windows RPC
49667/tcp open  msrpc         Microsoft Windows RPC
49668/tcp open  msrpc         Microsoft Windows RPC
49669/tcp open  msrpc         Microsoft Windows RPC
49685/tcp open  msrpc         Microsoft Windows RPC
49686/tcp open  ncacn_http    Microsoft Windows RPC over HTTP 1.0
49704/tcp open  msrpc         Microsoft Windows RPC
49718/tcp open  msrpc         Microsoft Windows RPC
49758/tcp open  msrpc         Microsoft Windows RPC
63599/tcp open  msrpc         Microsoft Windows RPC

Nmap scan report for Intranet (10.1.32.242)
Host is up (0.027s latency).
Not shown: 65526 filtered tcp ports (no-response)
PORT      STATE SERVICE       VERSION
80/tcp    open  http          Microsoft IIS httpd 10.0
135/tcp   open  msrpc         Microsoft Windows RPC
139/tcp   open  netbios-ssn   Microsoft Windows netbios-ssn
443/tcp   open  ssl/https?
445/tcp   open  microsoft-ds?
3389/tcp  open  ms-wbt-server
5985/tcp  open  http          Microsoft HTTPAPI httpd 2.0 (SSDP/UPnP)
49669/tcp open  msrpc         Microsoft Windows RPC
49671/tcp open  msrpc         Microsoft Windows RPC
```

The attacker can confirm the credentials are valid.
```
➜ nxc smb hosts.txt -u 'hellyr' -p 'H3lenaR!2025'          
SMB         10.1.28.90      445    DC01             [*] Windows 11 / Server 2025 Build 26100 x64 (name:DC01) (domain:lumons.hacksmarter) (signing:True) (SMBv1:False) 
SMB         10.1.28.90      445    DC01             [+] lumons.hacksmarter\hellyr:H3lenaR!2025 
SMB         10.1.32.242     445    INTRANET         [*] Windows 11 / Server 2025 Build 26100 x64 (name:INTRANET) (domain:lumons.hacksmarter) (signing:False) (SMBv1:False) 
SMB         10.1.32.242     445    INTRANET         [+] lumons.hacksmarter\hellyr:H3lenaR!2025 
Running nxc against 2 targets ━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━ 100% 0:00:00
```

