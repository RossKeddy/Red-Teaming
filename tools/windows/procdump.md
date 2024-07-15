https://learn.microsoft.com/en-us/sysinternals/downloads/procdump

The procdump utility can be used to dump process memory.

```powershell
get-process -name firefox

# -ma flag to dump the entire memory of the process.
.\procdump.exe -ma ID firefox.dmp
```

