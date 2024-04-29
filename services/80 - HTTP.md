
Directories
```
gobuster dir -u http://runner.htb/ -w /usr/share/wordlists/ -b "302,404" -t 50
```

Subdomains
```
gobuster dns -d runner.htb -w /usr/share/wordlists/SecLists/Discovery/DNS/subdomains-top1million-5000.txt -t 50
```

vHosts
```
gobuster vhost -u runner.htb -w /usr/share/wordlists/SecLists/Discovery/DNS/subdomains-top1million-5000.txt -t 50
```