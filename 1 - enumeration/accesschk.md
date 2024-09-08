https://learn.microsoft.com/en-gb/sysinternals/downloads/accesschk
### Checking Access using Accesschk.exe
Below should give you an idea of some of the useful flags
```powershell
.\accesschk.exe /accepteula
# -c : Name a windows service, or use * for all
# -d : Only process directories
# -k : Name a registry key e.g., hklm/software
# -q : Omit banner
# -s : Recurse
# -u : Suppress errors
# -v : Verbose
# -w : Show objects with write access
```

Checking service permissions

> ALWAYS RUN THE FOLLOWING TO CHECK IF YOU’VE PERMISSIONS TO START AND STOP THE SERVICE
```powershell
.\accesschk.exe /accepteula -ucqv <user> <svc_name>
```

Get all writable services as per groups
```powershell
.\accesschk.exe /accepteual -uwcqv Users *
.\accesschk.exe /accepteula -uwcqv "Authenticated Users" *
```

Check unquoted service paths by testing if directories are writable
```powershell
.\accesschk.exe /accepteula -uwdv "C:\Program Files"
```

Check user permissions on an executable
```powershell
.\accesschk.exe /accepteula -uqv "C:\Program Files\abcd\file.exe"
```

Find all weak permissions in folders
```powershell
.\accesschk.exe /accepteula -uwdqs Users c:\
.\accesschk.exe /accepteula -uwdqs "Authenticated Users" c:\
```

Files
```powershell
.\accesschk.exe /accepteula -uwqs Users c:\*.*
.\accesschk.exe /accepteula -uwqs "Authenticated Users" c:\*.*
```

Weak registry permissions
```powershell
.\accesschk.exe /accepteula -uvwqk HKLM\System\CurrentControlSet\Services\svc_name
```