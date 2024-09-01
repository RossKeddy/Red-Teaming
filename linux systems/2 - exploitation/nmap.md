
nmap scan
- -A - OS detection
- -sC - Default Scripts
- -sV - Probe ports for info
- -T5 - Use 5 threads
- -Pn - Assume hosts are online
- -p- - Scan all ports

```bash
nmap -A -sC -sV -T5 -Pn -p- 10.129.142.35
```

nmap udp scan
```bash
sudo nmap -p1-500 -sU -T5- 10.129.196.125
```

nmap Stealth Scan
```bash
# Stealthy
nmap -sS 10.11.1.X
```
