https://github.com/PowerShellMafia/PowerSploit/blob/dev/Recon/PowerView.ps1

PowerView is a PowerShell tool to gain network situational awareness on Windows domains. It contains a set of pure-PowerShell replacements for various windows "net *" commands, which utilize PowerShell AD hooks and underlying Win32 API functions to perform useful Windows domain functionality.

It also implements various useful metafunctions, including some custom-written user-hunting functions which will identify where on the network specific users are logged into. It can also check which machines on the domain the current user has local administrator access on. Several functions for the enumeration and abuse of domain trusts also exist. See function descriptions for appropriate usage and available options. For detailed output of underlying functionality, pass the -Verbose or -Debug flags.

For functions that enumerate multiple machines, pass the -Verbose flag to get a progress status as each host is enumerated. Most of the "meta" functions accept an array of hosts from the pipeline.

```powershell
. ./PowerView.ps1
Import-Module .\PowerView.ps1

//  Create Credential Object
$passwd = ConvertTo-SecureString "password" -AsPlainText -Force
$creds = New-Object System.Management.Automation.PSCredential ("fakedomain\user", $passwd)

// Gather List of all workstations on domain and store in variable
$comps = Get-NetComputer -Domain msp.local -Credential $creds

// Attempt to issue a command on each machine - any results indicate local admin on that machine
Invoke-Command -ScriptBlock{hostname} -Computer ($comps.dnshostName) -Credential $creds -ErrorAction SilentlyContinue



```