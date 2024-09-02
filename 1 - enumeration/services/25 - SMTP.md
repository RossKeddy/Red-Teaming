https://github.com/jetmore/swaks
---

```bash
#Version Detection
nc -nv <IP> 25 

# -M means mode, it can be RCPT, VRFY, EXPN
smtp-user-enum -M VRFY -U username.txt -t <IP> 

#Sending emain with valid credentials, the below is an example for Phishing mail attack
sudo swaks -t daniela@beyond.com -t marcus@beyond.com --from john@beyond.com --attach @config.Library-ms --server 192.168.50.242 --body @body.txt --header "Subject: Staging Script" --suppress-data -ap
```