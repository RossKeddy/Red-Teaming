# Ascension
![[writeups/hacksmarter/ascension (easy)/screenshots/logo.webp]]
## Scenario

This is the Capstone Challenge for Ryan's [Hacking Linux course on Simply Cyber Academy](https://academy.simplycyber.io/l/pdp/linux-hacking). As a result, this lab isn't strictly focused on realism, but rather teaching proper enumeration, lateral movement, and privilege escalation on a Linux machine.

There are 6 flags on the machine (you can see the location of each by clicking the 'hint' button to make it less of a rabbit chase). There are also multiple ways to solve the machine... so if you solve it in one way, you can go back and see if you can find the 2nd way.

Happy hacking!
## Enumeration
Starting with a standard nmap scan of `10.1.137.114`, the attacker looks for points of entry.
```bash
PORT      STATE SERVICE  VERSION
21/tcp    open  ftp      vsftpd 3.0.5
22/tcp    open  ssh      OpenSSH 9.6p1 Ubuntu 3ubuntu13.14 (Ubuntu Linux; protocol 2.0)
80/tcp    open  http     Apache httpd 2.4.58 ((Ubuntu))
111/tcp   open  rpcbind  2-4 (RPC #100000)
2049/tcp  open  nfs_acl  3 (RPC #100227)
35843/tcp open  mountd   1-3 (RPC #100005)
36015/tcp open  status   1 (RPC #100024)
38857/tcp open  nlockmgr 1-4 (RPC #100021)
42197/tcp open  mountd   1-3 (RPC #100005)
53591/tcp open  mountd   1-3 (RPC #100005)
```

Starting from top to bottom, the attacker tries logging in with the default `ftp:ftp` credentials and successfully compromised the FTP service. Inside was an interesting file labeled `pwlist.txt`
![[writeups/hacksmarter/ascension (easy)/screenshots/ftp.png]]

The list is fairly generic, and all of these options will be available with `rockyou.txt`.
![[writeups/hacksmarter/ascension (easy)/screenshots/passwords.png]]

The attacker will pivot to NFS shares. Checking the mounts reveals one is available.
![[writeups/hacksmarter/ascension (easy)/screenshots/user1nfs.png]]

The attacker will mount this locally for enumeration.
![[writeups/hacksmarter/ascension (easy)/screenshots/user1nfsmounting.png]]

Inside the share is a key pair. 
![[writeups/hacksmarter/ascension (easy)/screenshots/nfscredfiles.png]]
## Privilege Escalation
The private key requires a passwords, therefor the attacker will utilize [ssh2john](https://github.com/openwall/john/blob/bleeding-jumbo/run/ssh2john.py) to attempt to crack it and obtain access.
![[writeups/hacksmarter/ascension (easy)/screenshots/ssh2john.png]]

Utilizing [john](https://www.openwall.com/john/) on the converted hash reveals the keypair password is `sammie1`
![[writeups/hacksmarter/ascension (easy)/screenshots/cracker.png]]

Armed with the password the attacker has compromised the user1 account and retrieves the flag.
![[writeups/hacksmarter/ascension (easy)/screenshots/user1flag.png]]

### Credentialed Enumeration
Checking `/etc/passwd` reveals some interesting users.
![[writeups/hacksmarter/ascension (easy)/screenshots/passwd.png]]
 
 With the list of users obtained, the attacker will password spray the `ftpuser` using [hydra](https://github.com/vanhauser-thc/thc-hydra) with the passwords obtained earlier. This yields a valid login to the ftp server but there's nothing of interest there. ![[writeups/hacksmarter/ascension (easy)/screenshots/ftpuser.png]]

The attacker will try a standard login to that user which was successful and obtains the `ftpuser` flag.
![[writeups/hacksmarter/ascension (easy)/screenshots/ftpuserflag.png]]

The attacker identifies some database credentials that will be utilized later
![[writeups/hacksmarter/ascension (easy)/screenshots/dbcreds.png]]

[linpeas](https://github.com/peass-ng/PEASS-ng/tree/master/linPEAS) revealed nothing of interest. [pspy64](https://github.com/DominicBreuker/pspy) revealed the following script running as `UID=1002`:
![[writeups/hacksmarter/ascension (easy)/screenshots/pspy64.png]]

## Lateral Movement
However no files exist in `/tmp`, the attacker will create `backup.sh` and populate it with a reverse shell. This file is an attempt to gain shell as user2 (UID 1002).
![[writeups/hacksmarter/ascension (easy)/screenshots/tmpdir.png]]

![[writeups/hacksmarter/ascension (easy)/screenshots/filereplacement.png]]

Starting a listener on [penelope](https://github.com/brightio/penelope) and waiting eventually yields a shell. This is successful because the script is running on a timer and the attacker was able to write to `/tmp/` and force it to run anything the attacker wants.
![[writeups/hacksmarter/ascension (easy)/screenshots/user2shell.png]]

Now as user2 the second flag is obtained.
![[writeups/hacksmarter/ascension (easy)/screenshots/user2flag.png]]

## Lateral Movement
Now the attacker will utilize the DB credentials from earlier. Issuing a `ss -tupln` reveals SQL is running on the standard port, therefor they will try and connect. Displaying the tables yields flags and users. The attacker will select all from those tables and utilize them for further lateral movement.
![[writeups/hacksmarter/ascension (easy)/screenshots/sqlpwned.png]]

With those credentials found, the attacker will try and login as that user.
![[writeups/hacksmarter/ascension (easy)/screenshots/user3pwned.png]]

## Privilege Escalation
Interestingly in the user3 home directory, there is a python3 binary. This binary does not have SUID set.
![[writeups/hacksmarter/ascension (easy)/screenshots/python3getcap.png]]

However, this binary has the Linux CAP_SETUID capability set, so it can be used as a backdoor to maintain privileged access by manipulating its own process UID.
![[writeups/hacksmarter/ascension (easy)/screenshots/getcap.png]]

With that discovery, a simple binary privilege escalation exploit will allow the attacker to become the root user and obtain the sixth and final flag.
![[writeups/hacksmarter/ascension (easy)/screenshots/privesc.png]]