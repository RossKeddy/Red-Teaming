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

Assuming that latest PowerView script (master and dev are the same) has been loaded in memory.

### Domain Enumeration
Get basic information of the domain
```powershell
Get-Domain
```

Get domain SID
```powershell
Get-DomainSID
```

Get domain policies
```powershell
Get-DomainPolicy [-Domain <target>]
```

Get domain Kerberos policy
```powershell
(Get-DomainPolicy).KerberosPolicy
```

Get list of DCs
```powershell
Get-DomainController [-Domain <target>]
```

Get DC IP
```powershell
nslookup <target_dc>
```

### Forest Enumeration
Get current forest
```powershell
Get-Forest
```

Get a list of domains
```powershell
Get-ForestDomain [-Forest <target>]
```

#### User Enumeration
Get a list of users
```powershell
Get-NetUser [-Domain <target>] [user_name]
```

```powershell
net user /domain
```

Get a count of users
```powershell
(Get-NetUser).count
```

Get a list of users with some specific properties
```powershell
Get-NetUser [-Properties <>] 
```

Get a list of users with their logon counts, bad password attempts where attempts are greater than 0
```powershell
Get-NetUser | select cn, logoncounts, badpwdcount | ? {$_.badpwdcount -gt 0}
```

Finding users with SPN
```powershell
Get-NetUser -SPN
```

Finding users who are AllowedToDelegateTo
```powershell
Get-NetUser -TrustedToAuth
```

Finding users who can be delegated
```powershell
Get-NetUser -AllowDelegation
```
### Computer Enumeration
Get a list of computers
```powershell
Get-NetComputer [-Domain <target>] [-OperatingSystem "*2016*"] [-Properties <>]
```

Get a list of computers with Unconstrained delegation
```powershell
Get-NetComputer -Unconstrained
```

Finding users who are AllowedToDelegateTo
```powershell
Get-NetComputer -TrustedToAuth
```
### Group Enumeration
Get a list of groups in a domain
```powershell
net group /domain
```

Get a list of groups in a domain
```powershell
Get-NetGroup [-Domain <target>] [-FullData] [-GroupName "*admin*"] [-Username 'user_name']
```

Get group membership
```powershell
Get-NetGroupMember [-GroupName 'group_name'] [-Recurse]
```
### Share Enumeration
List shares user have access to
```powershell
Invoke-ShareFinder -CheckShareAccess -ErrorAction SilentlyContinue [-ComputerDomain <target_domain>]
```
### ACL Enumeration
Get resolved ACEs, optionally for a specific user/group and domain
```powershell
Get-ObjectAcl [-Identity <user_name>] [-Domain <target_domain>] -ResolveGUIDs
```

Get interesting resolved ACLs
```powershell
Invoke-ACLScanner [-Domain <target_domain>] -ResolveGUIDS
```

Get interesting resolved ACLs owned by specific object (ex. noobsec)
```powershell
Invoke-ACLScanner -ResolveGUIDS \| ?{$_.IdentityReference -match 'noobsec'}
```
### Session Enumeration
Finding sessions on a computer
```powershell
Get-NetSession [-Computer <comp_name>]
```

Get who is logged on locally where
```powershell
Get-LoggedOnLocal [-ComputerName <comp_name>]
```
### User Hunting
Get list of machines where current user has local admin access
```powershell
Find-LocalAdminAccess [-Domain <target_domain>]
```

Find machines where members of specific groups have sessions. Default: Domain Admins
```powershell
Invoke-UserHunter [-GroupName <group_name>]
```

Find machines where current user has local admin access AND specific group sessions are present
```powershell
Invoke-UserHunter -CheckAccess
```
