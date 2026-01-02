# Building Magic
![[writeups/hacksmarter/buildingmagic (easy)/screenshots/buildingmagic-logo.png]]
## Scope and Objective

**Objective**: As a penetration tester on the Hack Smarter Red Team, your objective is to achieve a full compromise of the Active Directory environment.

**Initial Access**: A prior enumeration phase has yielded a leaked database containing user credentials (usernames and hashed passwords). This information will serve as your starting point for gaining initial access to the network.

**Execution**: Your task is to leverage the compromised credentials to escalate privileges, move laterally through the Active Directory, and ultimately achieve a complete compromise of the domain.

***Note to user**: *To access the target machine, you must add the following entries to your /etc/hosts file:

    buildingmagic.local
    dc01.buildingmagic.local

**Leaked Database File**:
```
id	username	full_name	role		password
1	r.widdleton	Ron Widdleton	Intern Builder	c4a21c4d438819d73d24851e7966229c
2	n.bottomsworth	Neville Bottomsworth Plannner	61ee643c5043eadbcdc6c9d1e3ebd298
3	l.layman	Luna Layman	Planner		8960516f904051176cc5ef67869de88f
4	c.smith		Chen Smith	Builder		bbd151e24516a48790b2cd5845e7f148
5	d.thomas	Dean Thomas	Builder		4d14ff3e264f6a9891aa6cea1cfa17cb
6	s.winnigan	Samuel Winnigan	HR Manager	078576a0569f4e0b758aedf650cb6d9a
7	p.jackson	Parvati Jackson	Shift Lead	eada74b2fa7f5e142ac412d767831b54
8	b.builder	Bob Builder	Electrician	dd4137bab3b52b55f99f18b7cd595448
9	t.ren		Theodore Ren	Safety Officer	bfaf794a81438488e57ee3954c27cd75
10	e.macmillan	Ernest Macmillan Surveyor	47d23284395f618bea1959e710bc68ef
```

## Preparation
Since a list of users and hashes is provided, the attacker will create separate lists for each.

users.txt
```
r.widdleton
n.bottomsworth
l.layman
c.smith
d.thomas
s.winnigan
p.jackson
b.builder
t.ren
e.macmillan
```

hashes.txt
```
c4a21c4d438819d73d24851e7966229c
61ee643c5043eadbcdc6c9d1e3ebd298
8960516f904051176cc5ef67869de88f
bbd151e24516a48790b2cd5845e7f148
4d14ff3e264f6a9891aa6cea1cfa17cb
078576a0569f4e0b758aedf650cb6d9a
eada74b2fa7f5e142ac412d767831b54
dd4137bab3b52b55f99f18b7cd595448
bfaf794a81438488e57ee3954c27cd75
47d23284395f618bea1959e710bc68ef
```

Crackstation reveals two of the md5 hashes are easily cracked.
![[writeups/hacksmarter/buildingmagic (easy)/screenshots/crackstation.png]]

The attacker utilizes hashcat to confirm. [Hashcat](https://github.com/hashcat/hashcat) only cracks one password.
![[writeups/hacksmarter/buildingmagic (easy)/screenshots/hashcat1.png]]
![[writeups/hacksmarter/buildingmagic (easy)/screenshots/hashcat2.png]]

```
c4a21c4d438819d73d24851e7966229c:r.widdleton:lilronron
bfaf794a81438488e57ee3954c27cd75:t.ren:shadowhex7
```

## Enumeration

Firstly, the attacker will check the network services on the target.
```bash
➜ nmap -sV -T4 -p- 10.1.69.13
Starting Nmap 7.95 ( https://nmap.org ) at 2025-12-14 13:35 EST
Nmap scan report for buildingmagic.local (10.1.69.13)
Host is up (0.027s latency).
Not shown: 65515 filtered tcp ports (no-response)
PORT      STATE SERVICE       VERSION
53/tcp    open  domain        Simple DNS Plus
80/tcp    open  http          Microsoft IIS httpd 10.0
88/tcp    open  kerberos-sec  Microsoft Windows Kerberos (server time: 2025-12-14 18:44:13Z)
135/tcp   open  msrpc         Microsoft Windows RPC
139/tcp   open  netbios-ssn   Microsoft Windows netbios-ssn
389/tcp   open  ldap          Microsoft Windows Active Directory LDAP (Domain: BUILDINGMAGIC.LOCAL0., Site: Default-First-Site-Name)
445/tcp   open  microsoft-ds?
464/tcp   open  kpasswd5?
636/tcp   open  tcpwrapped
3268/tcp  open  ldap          Microsoft Windows Active Directory LDAP (Domain: BUILDINGMAGIC.LOCAL0., Site: Default-First-Site-Name)
3269/tcp  open  tcpwrapped
3389/tcp  open  ms-wbt-server Microsoft Terminal Services
5985/tcp  open  http          Microsoft HTTPAPI httpd 2.0 (SSDP/UPnP)
9389/tcp  open  mc-nmf        .NET Message Framing
49664/tcp open  msrpc         Microsoft Windows RPC
49668/tcp open  msrpc         Microsoft Windows RPC
49678/tcp open  ncacn_http    Microsoft Windows RPC over HTTP 1.0
49679/tcp open  msrpc         Microsoft Windows RPC
49722/tcp open  msrpc         Microsoft Windows RPC
49841/tcp open  msrpc         Microsoft Windows RPC
Service Info: Host: DC01; OS: Windows; CPE: cpe:/o:microsoft:windows
```

While the scan is happening, the attacker checks for null authentication on the domain with [NetExec](https://www.netexec.wiki/). The output confirms null authentication is enabled on DC01.
![[writeups/hacksmarter/buildingmagic (easy)/screenshots/nullauth.png]]

The attacker confirms that `r.widdleton` has valid credentials. This user is now compromised.
![[writeups/hacksmarter/buildingmagic (easy)/screenshots/r.widdleton.png]]

Unfortunately, t.ren is not a valid logon.
![[writeups/hacksmarter/buildingmagic (easy)/screenshots/t.ren.png]]

Utilizing the valid credentials, the attacker will use [NetExec](https://www.netexec.wiki/smb-protocol/enumeration/enumerate-domain-users) to enumerate all users on the domain.
![[writeups/hacksmarter/buildingmagic (easy)/screenshots/netexec-users.png]]

With the new users, the attacker will update `users.txt` to the following:
```
Administrator
r.widdleton
n.bottomsworth
l.layman
c.smith
d.thomas
s.winnigan
p.jackson
b.builder
t.ren
e.macmillan
h.potch
r.haggard
h.grangon
a.flatch
```

With an updated list of users, the attacker will [check the domain password policy](https://www.netexec.wiki/smb-protocol/enumeration/enumerate-domain-password-policy-1) and try to spray the new users with the cracked passwords.
![[writeups/hacksmarter/buildingmagic (easy)/screenshots/pass-pol.png]]

With the password policy being incredibly lax, the attacker can spray with no issues.
![[writeups/hacksmarter/buildingmagic (easy)/screenshots/spray1.png]]

Unfortunately, no valid logons are obtained. The attacker will pivot to [BloodHound](https://github.com/SpecterOps/BloodHound) for further enumeration. Using [bloodhound-python](https://github.com/dirkjanm/BloodHound.py/tree/bloodhound-ce) to collect domain information and uploading it to [BloodHound](https://github.com/SpecterOps/BloodHound). Once uploaded the attacker adds r.widdleton to Owned.
![[writeups/hacksmarter/buildingmagic (easy)/screenshots/bloodhound collection.png]]

- Checking the `Shortest Path from Owned Objects` yields no valuable attack paths. 
- Checking ASREP Roastable users reveals nothing of value.
- Checking Kerberoastable users reveals `r.haggard` has a Service Principal Name set.
- Checking Administrators reveals `a.flatch` as a high value target

Since the attacker discovered `r.haggard` is kerberoastable. They can dump the ticket hash using [impacket](https://github.com/fortra/impacket/blob/master/examples/GetUserSPNs.py) or [NetExec](https://www.netexec.wiki/ldap-protocol/kerberoasting).
![[writeups/hacksmarter/buildingmagic (easy)/screenshots/kerberoast1.png]]

Cracking the hash was easy. The attacker marks this user as owned in [BloodHound](https://github.com/SpecterOps/BloodHound).
![[writeups/hacksmarter/buildingmagic (easy)/screenshots/kerberoastcrack.png]]
![[writeups/hacksmarter/buildingmagic (easy)/screenshots/kerberoast2.png]]

With [BloodHound](https://github.com/SpecterOps/BloodHound) the attacker can see `r.haggard` has outbound object control on h.potch.
![[writeups/hacksmarter/buildingmagic (easy)/screenshots/forcechangepass.png]]

With [BloodyAD](https://github.com/CravateRouge/bloodyAD) this is an easy account takeover. 
![[writeups/hacksmarter/buildingmagic (easy)/screenshots/bloodyad.png]]

Utilizing [NetExec](https://www.netexec.wiki/) the attacker confirms successful compromise of the account and marks the object as Owned in [BloodHound](https://github.com/SpecterOps/BloodHound).
![[writeups/hacksmarter/buildingmagic (easy)/screenshots/hpotch.png]]

With this user compromised, the attacker will use the credentials to enumerate shares. The account has READ/WRITE on `File-Share` so this will be the next target.
![[writeups/hacksmarter/buildingmagic (easy)/screenshots/shares.png]]

Greenwolf provides an excellent [tool](https://github.com/Greenwolf/ntlm_theft), ntlm_theft, which can be used in conjunction with [Responder](https://github.com/SpiderLabs/Responder) to capture NTLM authentication. An attacker can generate malicious file payloads (such as LNK or SCF files) and upload them to a network share. When a user browses or interacts with the share, the payload can trigger an outbound NTLM authentication attempt, which Responder can then capture or relay.
![[writeups/hacksmarter/buildingmagic (easy)/screenshots/ntlmtheft.png]]

![[writeups/hacksmarter/buildingmagic (easy)/screenshots/responder.png]]

![[writeups/hacksmarter/buildingmagic (easy)/screenshots/smbpoison.png]]

Once the file is put into the share responder will begin capturing hashes. NTLMv2 is typically very difficult to crack, however the attacker will attempt it regardless.
![[writeups/hacksmarter/buildingmagic (easy)/screenshots/ntlmv2.png]]

```
[SMB] NTLMv2-SSP Client   : 10.1.69.13
[SMB] NTLMv2-SSP Username : BUILDINGMAGIC\h.grangon
[SMB] NTLMv2-SSP Hash     : h.grangon::BUILDINGMAGIC:c9924cfadecf2446:DAB1E183C1688D669D0A76E231215B2A:01010000000000000019AF7D0A6DDC012F67FF94EC9649120000000002000800330046005400580001001E00570049004E002D00410050004C005700570055003100440041004C00500004003400570049004E002D00410050004C005700570055003100440041004C0050002E0033004600540058002E004C004F00430041004C000300140033004600540058002E004C004F00430041004C000500140033004600540058002E004C004F00430041004C00070008000019AF7D0A6DDC01060004000200000008003000300000000000000000000000004000005DF107079896AE9F678A67AAD0D8AFB1FCB48BCC6D27A6428B72DA8CAA52C2970A001000000000000000000000000000000000000900240063006900660073002F00310030002E003200300030002E00320033002E003100310033000000000000000000          
```

The password ended up being easily cracked and further lateral movement was possible.
![[writeups/hacksmarter/buildingmagic (easy)/screenshots/hashcatgrangon.png]]
![[writeups/hacksmarter/buildingmagic (easy)/screenshots/grangoncracked.png]]

The attacker confirms `h.grangon` is compromised and marks the object as owned in [BloodHound](https://github.com/SpecterOps/BloodHound). This user is also a member of `Remote Management Users` meaning the attacker now has direct access to the machine using evil-winrm.
![[writeups/hacksmarter/buildingmagic (easy)/screenshots/grangonowned.png]]

![[writeups/hacksmarter/buildingmagic (easy)/screenshots/grangonwinrm.png]]

## Privilege Escalation
Browsing to `h.grangon`'s desktop reveals the user flag. The next step is to enumerate what `h.grangon` has or can do, with the end goal of obtaining Domain Admin privileges. Checking this user's privileges reveals two overpermissioned grants called [SeMachineAccountPrivilege](https://www.semperis.com/blog/mitigating-active-directory-domain-service-privilege-escalation-security-flaws/) & [SeBackupPrivilege](https://juggernaut-sec.com/sebackupprivilege/). For Privilege Escalation the attacker will use [SeBackupPrivilege](https://juggernaut-sec.com/sebackupprivilege/).
![[writeups/hacksmarter/buildingmagic (easy)/screenshots/grangonprivs.png]]

The attacker will save the [SAM & SYSTEM hives](https://www.thehacker.recipes/ad/movement/credentials/dumping/sam-and-lsa-secrets) and download those using [evil-winrm](https://github.com/Hackplayers/evil-winrm).
```powershell
reg save HKLM\sam.hive "C:\Users\h.grangon\sam.hive"
reg save HKLM\system.hive "C:\Users\h.grangon\system.hive"
```

![[writeups/hacksmarter/buildingmagic (easy)/screenshots/regdownload.png]]

With both hives the attacker can simply use [secretsdump.py](https://github.com/fortra/impacket/blob/master/examples/secretsdump.py) to obtain the NT Hashes for accounts.
![[writeups/hacksmarter/buildingmagic (easy)/screenshots/secretsdump.png]]

Spraying the Administrator hash reveals `a.flatch` (The user identified from [BloodHound](https://github.com/SpecterOps/BloodHound) earlier) is Administrator and utilizing this credential allows for full compromise of the Active Directory domain.
![[writeups/hacksmarter/buildingmagic (easy)/screenshots/adminpwned.png]]

![[writeups/hacksmarter/buildingmagic (easy)/screenshots/flatch.png]]