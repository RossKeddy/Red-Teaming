```
# login if you have relevant creds or based on nmpa scan find out whether this has # anonymous login or not, then loginwith Anonymous:password

ftp <IP>
ftp> USER anonymous
ftp> PASS anonymous

# Uploading file
put <file> 

# Downloading file
get <file> 
wget -m --no-passive ftp://anonymous:anonymous@<target>

# NSE
locate .nse | grep ftp
nmap -p21 --script=<name> <IP>

# bruteforce
hydra -L users.txt -P passwords.txt <IP> ftp #'-L' for usernames list, '-l' for username and viceversa
```