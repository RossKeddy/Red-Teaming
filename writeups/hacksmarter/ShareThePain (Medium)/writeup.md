# ShareThePain
![[writeups/hacksmarter/sharethepain (medium)/screenshots/logo.jpg]]
## Scope and Objective

**Objective:** You're a **penetration tester** on the **Hack Smarter Red Team**. Your mission is to infiltrate and seize control of the client's entire Active Directory environment. This isn't just a test; it's a full-scale assault to expose and exploit every vulnerability.

**Initial Access:** For this engagement, you've been granted **direct network access** to the client's network. The door is open, but you're starting with **zero credentials**. From here, every move counts.

**Execution:** Your objective is simple but demanding: **enumerate, exploit, and own.** Your ultimate goal is not just to get in, but to achieve a **full compromise**, elevating your privileges until you hold the keys to the entire domain.

## Enumeration
Firstly, the attacker scans `10.1.46.94` with [nmap](https://nmap.org/) to find services that may be vulnerable to attack.
```bash
PORT      STATE SERVICE       VERSION
53/tcp    open  domain        Simple DNS Plus
88/tcp    open  kerberos-sec  Microsoft Windows Kerberos (server time: 2025-12-30 14:11:06Z)
135/tcp   open  msrpc         Microsoft Windows RPC
139/tcp   open  netbios-ssn   Microsoft Windows netbios-ssn
389/tcp   open  ldap          Microsoft Windows Active Directory LDAP (Domain: hack.smarter0., Site: Default-First-Site-Name)
445/tcp   open  microsoft-ds?
464/tcp   open  kpasswd5?
593/tcp   open  ncacn_http    Microsoft Windows RPC over HTTP 1.0
636/tcp   open  tcpwrapped
3268/tcp  open  ldap          Microsoft Windows Active Directory LDAP (Domain: hack.smarter0., Site: Default-First-Site-Name)
3269/tcp  open  tcpwrapped
3389/tcp  open  ms-wbt-server Microsoft Terminal Services
5985/tcp  open  http          Microsoft HTTPAPI httpd 2.0 (SSDP/UPnP)
```

Next, the attacker checks if anonymous authentication is enabled on the domain.
![[writeups/hacksmarter/sharethepain (medium)/screenshots/anonauth.png]]

With anonymous authentication enabled, the attacker enumerates the available domain shares. This reveals that `\Share` has anonymous READ/WRITE permissions allowed.
![[writeups/hacksmarter/sharethepain (medium)/screenshots/shares.png]]

## Exploitation
Identifying this becomes a classic attack using [Greenwolf's ntlm_theft](https://github.com/Greenwolf/ntlm_theft) toolkit. First, the attacker will start [Responder](https://github.com/SpiderLabs/Responder).
![[writeups/hacksmarter/sharethepain (medium)/screenshots/responder.png]]

Then, utilizing the aforementioned tool, generate some phishing payloads.
![[writeups/hacksmarter/sharethepain (medium)/screenshots/ntlmtheft.png]]

The attacker will connect to the share and upload one of the files there.
![[writeups/hacksmarter/sharethepain (medium)/screenshots/urgentlnk.png]]

Immediately the attacker sees `HACK\bob.ross` get poisoned with [Responder](https://github.com/SpiderLabs/Responder). NTLMv2 is difficult to crack but an attempt will be made to crack their password.
![[writeups/hacksmarter/sharethepain (medium)/screenshots/bobrosshash.png]]

Fortunately, this user was utilizing a weak password, allowing it to be cracked in just a few seconds.
 Revealing the `bob.ross` password to be `137Password123!@#`
 ![[writeups/hacksmarter/sharethepain (medium)/screenshots/hashcat1.png]]
![[writeups/hacksmarter/sharethepain (medium)/screenshots/hashcat2.png]]

The attacker confirms that this user has been pwned with [NetExec](https://www.netexec.wiki/).
![[writeups/hacksmarter/sharethepain (medium)/screenshots/bobrosspwned.png]]
### Credentialed Enumeration
With a foothold on the domain, the attacker is going to analyze [bloodhound](https://bloodhound.specterops.io/get-started/introduction) attack paths for further lateral movement or privilege escalation.
![[writeups/hacksmarter/sharethepain (medium)/screenshots/bloodhound.png]]

`bob.ross` has significant permissions over `alice.wonderland`.
![[writeups/hacksmarter/sharethepain (medium)/screenshots/bobownsalice.png]]
## Exploitation
Utilizing the above permissions and [BloodyAD](https://github.com/CravateRouge/bloodyAD), the attacker will simply change the `alice.wonderland` user's password.
![[writeups/hacksmarter/sharethepain (medium)/screenshots/alicepasschange.png]]

The attacker confirms Alice has been pwned. Referencing the object in [Bloodhound](https://bloodhound.specterops.io/get-started/introduction) reveals `alice.wonderland` has the `Remote Management Users` group assigned.
![[writeups/hacksmarter/sharethepain (medium)/screenshots/alicepwned.png]]

`alice.wonderland` allows the attacker to obtain the user flag.
![[writeups/hacksmarter/sharethepain (medium)/screenshots/userpwned.png]]

### Credentialed Enumeration
Manually reviewing files reveals the server has SQL files, suggesting that the service may be installed.
![[writeups/hacksmarter/sharethepain (medium)/screenshots/sql.png]]

The attacker confirms that a SQL server is running by checking the services that are listening for connections.
![[writeups/hacksmarter/sharethepain (medium)/screenshots/sqlport.png]]

## Exploitation
To attack this we'll need to utilize a tool to forward the port, in this instance the attacker will use [sliver](https://sliver.sh/). But it's also highly recommended to use [ligolo-ng](https://github.com/nicocha30/ligolo-ng). The attacker is going to generate an implant to callback on port 443. uploading it to the host, and start the listener.
![[writeups/hacksmarter/sharethepain (medium)/screenshots/sliver.png]]

![[writeups/hacksmarter/sharethepain (medium)/screenshots/uploadshell.png]]

Start the listener
![[writeups/hacksmarter/sharethepain (medium)/screenshots/sliverjobs.png]]

Run the implant
![[writeups/hacksmarter/sharethepain (medium)/screenshots/shellstart.png]]

And here the machine has successfully connected back to the attacker.
![[writeups/hacksmarter/sharethepain (medium)/screenshots/sliversession.png]]

Now the attacker is starting a SOCKS tunnel to interact with the machine.
![[writeups/hacksmarter/sharethepain (medium)/screenshots/socks5.png]]

With the SOCKS tunnel established, the attacker will utilize [mssqlpwner](https://github.com/ScorpionesLabs/MSSqlPwner) to attack the server.
![[writeups/hacksmarter/sharethepain (medium)/screenshots/mssqlpwner.png]]

Issuing the `enumerate` command displays that the attacker has access to the Administrator account.
![[writeups/hacksmarter/sharethepain (medium)/screenshots/impersonate.png]]

The attacker will set the chain to Administrator.
![[writeups/hacksmarter/sharethepain (medium)/screenshots/admin1.png]]

Utilizing the above sliver instructions, the attacker uploads another implant through [mssqlpwner](https://github.com/ScorpionesLabs/MSSqlPwner) and executes it. 
![[writeups/hacksmarter/sharethepain (medium)/screenshots/shell2.png]]

Back on the [sliver](https://sliver.sh/) server, a new session as `MSSQL$SQLEXPRESS` is obtained.
![[writeups/hacksmarter/sharethepain (medium)/screenshots/sliversessions2.png]]

## Privilege Escalation
Checking the user's privileges reveals `SeImpersonatePrivilege`, this will allow for a classic [potato](https://github.com/BeichenDream/GodPotato) privilege escalation.
![[writeups/hacksmarter/sharethepain (medium)/screenshots/sqlprivs.png]]

Within Sliver, the attacker executes the following command to utilize [GodPotato](https://github.com/BeichenDream/GodPotato) and creates another [sliver](https://sliver.sh/) session.
```bash
execute-assembly /home/rosskeddy/tools/potatos/GodPotato-NET4.exe -cmd "C:\Temp\shell3.exe"
```

The session connects back immediately and launching the interactive shell reveals `NT AUTHORITY\SYSTEM` is obtained - fully compromising the environment.
![[writeups/hacksmarter/sharethepain (medium)/screenshots/potato.png]]