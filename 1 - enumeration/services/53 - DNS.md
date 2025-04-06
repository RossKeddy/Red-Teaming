* https://book.hacktricks.xyz/network-services-pentesting/pentesting-dns
* https://securitytrails.com/blog/most-popular-types-dns-attacks
---
## Finding Domain Controllers

AD-DS (Active Directory Domain Services) rely on DNS SRV RR (service location resource records). Those records can be queried to find the location of some servers: the global catalog, LDAP servers, the Kerberos KDC and so on.

The [nmap](https://nmap.org/) tool can be used with its [dns-srv-enum.nse](https://nmap.org/nsedoc/scripts/dns-srv-enum.html) script to operate those queries.
```bash
nmap --script dns-srv-enum --script-args dns-srv-enum.domain=$FQDN_DOMAIN
```

> [!TIP]
In order to function properly, the tools need to know the domain name and which nameservers to query. That information is usually [sent through DHCP offers](https://www.thehacker.recipes/ad/recon/dhcp) and stored in the `/etc/resolv.conf` or `/run/systemd/resolve/resolv.conf` file in UNIX-like systems.

If needed, the nameservers may be found with a port scan on the network by looking for DNS ports `53/TCP` and `53/UDP`.
```bash
nmap -v -sV -p 53 $SUBNET/$MASK
nmap -v -sV -sU -p 53 $SUBNET/$MASK
```

> The DNS service is usually offered by all domain controllers

## DIG
### Queries
```bash
# SOA
dig soa www.inlanefreight.com

# Reverse lookup
dig -x 10.13.37.10 @10.13.37.10

# Reverse SOA to find hidden domains
nslookup               
> server 10.13.37.10
Default server: 10.13.37.10
Address: 10.13.37.10#53
> set type=SOA
> 10.13.37.10
Server:         10.13.37.10
Address:        10.13.37.10#53

10.37.13.10.in-addr.arpa        name = www.securewebinc.jet.

# TXT Query
dig TXT ctf.games

# MX Query
dig MX ctf.games

# NS Query
dig ns domain.com @10.129.14.128

# Version Query
dig CH TXT version.bind 10.129.120.85

# Any Query
dig any inlanefreight.htb @10.129.240.190

# AXFR Zone Transfer
dig axfr domain.com @10.129.14.128

# Subdomain Bruteforce
for sub in $(cat /usr/share/wordlists/SecLists/Discovery/DNS/subdomains-top1million-110000.txt);do dig $sub.inlanefreight.htb @10.129.240.190 | grep -v ';\|SOA' | sed -r '/^\s*$/d' | grep $sub | tee -a subdomains.txt;done
```

## DNS Recon
```bash
dnsrecon -r 127.0.0.0/24 -n 192.168.213.129 -d blackpearl.tcm
```

## DNSenum
```
dnsenum --dnsserver 10.129.240.190 --enum -p 0 -s 0 -o subdomains.txt -f /opt/useful/SecLists/Discovery/DNS/subdomains-top1million-110000.txt inlanefreight.htb
```
## Bind9 DNS
https://www.cvedetails.com/product/144/ISC-Bind.html?vendor_id=64
### Zone Files
```bash
cat /etc/bind/db.domain.com
```
### Reverse Name Resolution Zone Files
```bash
cat /etc/bind/db.10.129.14
```

### Dangerous Settings

| **Option**        | **Description**                                                                |
| ----------------- | ------------------------------------------------------------------------------ |
| `allow-query`     | Defines which hosts are allowed to send requests to the DNS server.            |
| `allow-recursion` | Defines which hosts are allowed to send recursive requests to the DNS server.  |
| `allow-transfer`  | Defines which hosts are allowed to receive zone transfers from the DNS server. |
| `zone-statistics` | Collects statistical data of zones.                                            |
|                   |                                                                                |


For subdomain hunting visit [[80 - HTTP]]