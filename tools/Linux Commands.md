


```bash
# Find useable Linux files
find / -perm -u=s -type f 2>/dev/null

# Disk Usage
sudo du -hsx ./* | sort -rh | head -n 40
```