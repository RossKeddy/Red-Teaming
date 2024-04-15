
Directories
```
gobuster dir -u http://<DOMAIN> -w wordlists/dirbuster/directory-list-1.0.txt -b "302,404"
```

vHosts
```
gobuster vhost -u <DOMAIN> -w /usr/share/wordlists/SecLists/Discovery/DNS/subdomains-top1million-5000.txt
```
