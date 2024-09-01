https://book.hacktricks.xyz/network-services-pentesting/pentesting-dns
https://securitytrails.com/blog/most-popular-types-dns-attacks
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