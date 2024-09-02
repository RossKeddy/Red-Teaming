```bash
# HTTP - Apache2
# cp file /var/www/html/file_name
sudo service apache2 start

# HTTP - Python. Default port 8000
# python2
sudo python -m SimpleHTTPServer 80
# python3
sudo python3 -m http.server 80

# SMB
sudo impacket-smbserver <share_name> <path/to/share>

# FTP
# apt-get install python-pyftpdlib
sudo python -m pyftpdlib -p 21

# TFTP (UDP)
sudo atftpd --daemon -port 69 /path/to/serve

# Netcat
nc -nvlp <port> < file/to/send
```