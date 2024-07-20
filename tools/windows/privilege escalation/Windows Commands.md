### Who Am I?
```powershell
whoami
echo %username%

# Check Groups
whoami /groups

# Check Privileges
whoami /priv

whoami /all
```

### Where Am I?
```powershell
hostname
echo %hostname%
```
### Anyone home?
Local users
```powershell
net users
```

Domain users
```powershell
net users /domain
```
### What am I part of?
Local groups
```powershell
net groups
```

Domain groups
```powershell
net groups /domain
```
### What is this place?
```text
systeminfo
```
### Is it fancy?
Both should be the same for ease of exploitation, if either is 32-bit then try to gain a 64-bit shell.  
Use PowerShell
```powershell
[environment]::Is64BitOperatingSystem
[environment]::Is64BitProcess
```
### Am I tied up?
Check LanguageMode. FullLanguage is nicer to have.  
Use PowerShell
```powershell
$ExecutionContext.SessionState.LanguageMode
```
### Anything reachable?
Use PowerShell
```powershell
Get-AppLockerPolicy -Effective
Get-AppLockerPolicy -Effective | select -ExpandedProperty RuleCollections
```
### What does the inside look like?
Look for interesting services
```powershell
netstat -aon
```
### Leave me alone
Do you have admin privs?  

Disable Windows Defender real time monitoring
```powershell
Set-MpPreference -DisableRealTimeMonitoring $true	
```

Disable Windows Defender scanning for all files downloaded
```powershell
Set-MpPreference -DisableIOAVProtection $true	
```