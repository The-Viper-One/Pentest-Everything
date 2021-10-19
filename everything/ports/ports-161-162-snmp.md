# Ports 161 | 162 | SNMP



```bash
#OneSixtyOne
onesixtyone -c <wordlist> -w 5 <ip>

# Nmap
sudo nmap -Pn -sU -p 161 --script=snmp-brute <IP>
sudo nmap -Pn -sU -p 161 --script=snmp-interfaces <IP>
sudo nmap -Pn -sU -p 161 --script=snmp-netstat <IP>
sudo nmap -Pn -sU -p 161 --script=snmp-processes <IP>
sudo nmap -Pn -sU -p 161 --script=snmp-win32-services <IP>
sudo nmap -Pn -sU -p 161 --script=snmp-win32-shares <IP>
sudo nmap -Pn -sU -p 161 --script=snmp-win32-software <IP>
sudo nmap -Pn -sU -p 161 --script=snmp-win32-users <IP>
sudo nmap -Pn -sU -p 161 --script=snmp-brute,snmp-hh3c-logins,snmp-info,snmp-interfaces,snmp-ios-config,snmp-netstat,snmp-processes,snmp-sysdescr,snmp-win32-services,snmp-win32-shares,snmp-win32-software,snmp-win32-users  <IP> 

# SNMP Walk
snmpwalk -c <CommunityString> -v1 <IP> 1

# Metasploit
use auxiliary/scanner/snmp/snmp_enum

# Impacket
python2 samdump.py SNMP <IP>
```
