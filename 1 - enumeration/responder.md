# LLMNR/NTB-NS Poisoning
https://github.com/SpiderLabs/Responder

> [!THEORY]
[Link-Local Multicast Name Resolution](https://datatracker.ietf.org/doc/html/rfc4795) (LLMNR) and [NetBIOS Name Service](https://docs.microsoft.com/en-us/previous-versions/windows/it-pro/windows-2000-server/cc940063(v=technet.10)?redirectedfrom=MSDN) (NBT-NS) are Microsoft Windows components that serve as alternate methods of host identification that can be used when DNS fails. If a machine attempts to resolve a host but DNS resolution fails, typically, the machine will try to ask all other machines on the local network for the correct host address via LLMNR. LLMNR is based upon the Domain Name System (DNS) format and allows hosts on the same local link to perform name resolution for other hosts. It uses port `5355` over UDP natively. If LLMNR fails, the NBT-NS will be used. NBT-NS identifies systems on a local network by their NetBIOS name. NBT-NS utilizes port `137` over UDP.
>
The kicker here is that when LLMNR/NBT-NS are used for name resolution, ANY host on the network can reply. This is where we come in with `Responder` to poison these requests. With network access, we can spoof an authoritative name resolution source ( in this case, a host that's supposed to belong in the network segment ) in the broadcast domain by responding to LLMNR and NBT-NS traffic as if they have an answer for the requesting host. This poisoning effort is done to get the victims to communicate with our system by pretending that our rogue system knows the location of the requested host. If the requested host requires name resolution or authentication actions, we can capture the NetNTLM hash and subject it to an offline brute force attack in an attempt to retrieve the cleartext password. The captured authentication request can also be relayed to access another host or used against a different protocol (such as LDAP) on the same host. LLMNR/NBNS spoofing combined with a lack of SMB signing can often lead to administrative access on hosts within a domain. SMB Relay attacks will be covered in a later module about Lateral Movement.
>
Let's walk through a quick example of the attack flow at a very high level:
>
>1. A host attempts to connect to the print server at \\print01.inlanefreight.local, but accidentally types in \\printer01.inlanefreight.local.
>2. The DNS server responds, stating that this host is unknown.
>3. The host then broadcasts out to the entire local network asking if anyone knows the location of \\printer01.inlanefreight.local.
>4. The attacker (us with `Responder` running) responds to the host stating that it is the \\printer01.inlanefreight.local that the host is looking for.
>5. The host believes this reply and sends an authentication request to the attacker with a username and NTLMv2 password hash.
>6. This hash can then be cracked offline or used in an SMB Relay attack if the right conditions exist.

## Responder
> [!NOTE]
From UNIX-like systems, [Responder](https://github.com/lgandx/Responder) is a tool built to listen, analyze, and poison NTLM authentications over `LLMNR`, `NBT-NS`, `MDNS` ( and many more -> `SMB, HTTP, LDAP, FTP, POP3, IMAP, SMTP`). Depending on the authenticating principal's configuration, the NTLM authentication can sometimes be downgraded with `--lm` and `--disable-ess` in order to obtain NTLMv1 responses. Analyze mode will passively listen to the network and not send any poisoned packets.
>
You should try to force a LM hashing downgrade with Responder. LM and NTLMv1 responses (a.k.a. LM/NTLMv1 hashes) from Responder can easily be cracked with [crack.sh](https://crack.sh/netntlm/). The [ntlmv1-multi](https://github.com/evilmog/ntlmv1-multi) tool (Python) can be used to convert captured responses to crackable formats by hashcat, [crack.sh](https://crack.sh/netntlm/) and so on.
>
Machine account NT hashes can be used with the silver ticket or S4U2self abuse techniques to gain admin access to it.

```bash
# -A is analyze mode
sudo responder -I tun0 -A 


sudo responder -I tun0 -wrf
```