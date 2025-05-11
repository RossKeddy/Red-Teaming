# Services
In an Active Directory domain, domain controllers can be easily spotted depending on what services they host. Each service is usually accessible specific TCP and/or UDP port(s) making the DCs stand out in the network. Here is a list of ports to look for when hunting for domain controllers.

- `53/TCP` and `53/UDP` for DNS
- `88/TCP` for Kerberos authentication
- `135/TCP` and `135/UDP` MS-RPC epmapper (EndPoint Mapper)
- `137/TCP` and `137/UDP` for NBT-NS
- `138/UDP` for NetBIOS datagram service
- `139/TCP` for NetBIOS session service
- `389/TCP` for LDAP
- `636/TCP` for LDAPS (LDAP over TLS/SSL)
- `445/TCP` and `445/UDP` for SMB
- `464/TCP` and `445/UDP` for Kerberos password change
- `3268/TCP` for LDAP Global Catalog
- `3269/TCP` for LDAP Global Catalog over TLS/SSL

The [nmap](https://nmap.org/) utility can be used to scan for open ports in an IP range.
- -A - OS detection
- -sC - Default Scripts
- -sV - Probe ports for info
- -T5 - Use 5 threads (4 threads are better in real world exercises)
- -Pn - Assume hosts are online
- -p- - Scan all ports

```bash
nmap -A -sC -sV -T5 -Pn -p- 10.129.142.35

# Fast Scanning
nmap -T5 -p- $ip
# Define ports found 
nmap -T5 -A $ip

```

nmap udp scan
```bash
sudo nmap -p1-500 -sU -T4- 10.129.196.125
```

nmap Stealth Scan
```bash
# Stealthy
nmap -sS 10.11.1.X
```