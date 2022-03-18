# 1099 | Java RMI

## Enumeration

Identified as 'GNU Classpath grmiregistry on Linux based systems.

```bash
nmap -sT -p 1099 -sV <IP>
nmap -sV --script "rmi-dumpregistry or rmi-vuln-classloader" -p <PORT> <IP>

# Metasploit
use auxiliary/scanner/misc/java_rmi_server
use auxiliary/gather/java_rmi_registry
```

## Exploitation

```bash
exploit/multi/misc/java_rmi_server
```
