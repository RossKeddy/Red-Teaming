```bash
# Better version than snmpwalk as it displays more user friendly
snmpcheck -t <IP> -c public 

# Displays entire MIB tree, MIB Means Management Information Base
snmpwalk -c public -v1 -t 10 <IP> 
snmpwalk -c public -v1 <IP> 1.3.6.1.4.1.77.1.2.25 #Windows User enumeration
snmpwalk -c public -v1 <IP> 1.3.6.1.2.1.25.4.2.1.2 #Windows Processes enumeration
snmpwalk -c public -v1 <IP> 1.3.6.1.2.1.25.6.3.1.2 #Installed software enumeraion
snmpwalk -c public -v1 <IP> 1.3.6.1.2.1.6.13.1.3 #Opened TCP Ports

#Windows MIB values
1.3.6.1.2.1.25.1.6.0 - System Processes
1.3.6.1.2.1.25.4.2.1.2 - Running Programs
1.3.6.1.2.1.25.4.2.1.4 - Processes Path
1.3.6.1.2.1.25.2.3.1.4 - Storage Units
1.3.6.1.2.1.25.6.3.1.2 - Software Name
1.3.6.1.4.1.77.1.2.25 - User Accounts
1.3.6.1.2.1.6.13.1.3 - TCP Local Ports
```