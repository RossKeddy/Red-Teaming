
| MIB Values             | Windows Parameters |
| ---------------------- | ------------------ |
| 1.3.6.1.2.1.25.1.6.0   | System Processes   |
| 1.3.6.1.2.1.25.4.2.1.2 | Running Programs   |
| 1.3.6.1.2.1.25.4.2.1.4 | Processes Path     |
| 1.3.6.1.2.1.25.2.3.1.4 | Storage Units      |
| 1.3.6.1.2.1.25.6.3.1.2 | Software Name      |
| 1.3.6.1.4.1.77.1.2.25  | User Accounts      |
| 1.3.6.1.2.1.6.13.1.3   | TCP Local Ports    |
```
# Brute force community strings
# echo public > community
# echo private >> community
# echo manager >> community
# for ip in $(seq 1 254);do echo 10.11.1.$ip;done > snmp-ips
onesixtyone -c community -i snmp-ips

# Enumerate entire MIB tree
snmpwalk -c public -v1 $ip
# Enumerate specific MIB Value
snmpwalk -c public -v1 $ip $MIB_Value

snmp-check $ip
```