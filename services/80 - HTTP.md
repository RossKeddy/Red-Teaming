# 80/443 - HTTP/S
https://cheatsheetseries.owasp.org/index.html

> Use [Wappalyzer](https://www.wappalyzer.com/download) to identify technologies, web server, OS, database server deployed
> 
> `View-Source` of pages to find interesting comments, directories, technologies, web application being used, etc.
> 
> Finding hidden content by scanning each sub-domain and interesting directories is a good idea

## Common Directories
```
/robots.txt
/sitemap.xml
/.htaccess
config.php / config.ini
wp-config.php
.env
web.config

# Make it throw an error
/doesnotexist
```
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
Directories
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