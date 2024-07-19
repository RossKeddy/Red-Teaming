https://cheatsheetseries.owasp.org/index.html
## GoBuster
https://github.com/OJ/gobuster

Directories
```
gobuster dir -u http://permx.htb/ -w ~/wordlists/wordlists/SecLists/Discovery/Web-Content/directory-list-2.3-big.txt -b "302,404" -t 50
```

Subdomains
```
gobuster dns -d runner.htb -w /usr/share/wordlists/SecLists/Discovery/DNS/subdomains-top1million-5000.txt -t 50
```

vHosts
```
gobuster vhost -u runner.htb -w /usr/share/wordlists/SecLists/Discovery/DNS/subdomains-top1million-5000.txt -t 50
```

## FeroxBuster
```
feroxbuster -u http://www.securewebinc.jet/ -w /usr/share/seclists/Discovery/Web-Content/raft-medium-files-lowercase.txt -k -B -x "txt,html,php,zip,rar,tar.gz" -v -e -o ./ferox.txt
```
## FFUF
https://github.com/ffuf/ffuf

DNS
```
ffuf -w ~/wordlists/wordlists/SecLists/Discovery/DNS/subdomains-top1million-110000.txt:FUZZ -u http://FUZZ.board.htb/
```

vHost
```
ffuf -w ~/wordlists/wordlists/SecLists/Discovery/DNS/subdomains-top1million-110000.txt:FUZZ -u http://board.htb/ -H 'Host: FUZZ.board.htb' -fs 900
```