
nmap scan
- -A - OS detection
- -sC - Default Scripts
- -sV - Probe ports for info
- -T5 - Use 5 threads
- -Pn - Assume hosts are online
- -p- - Scan all ports

```
nmap -A -sC -sV -T5 -p- 10.129.142.35
```


nmap udp scan
```
sudo nmap -p1-500 -sU 10.129.142.35
```
