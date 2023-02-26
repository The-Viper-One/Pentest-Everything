---
description: https://attack.mitre.org/techniques/T1070/006/
---

# Timestomp

**ATT\&CK ID:** [T1070.007](https://attack.mitre.org/techniques/T1070/006/)

**Description**

Adversaries may modify file time attributes to hide new or changes to existing files. Timestomping is a technique that modifies the timestamps of a file (the modify, access, create, and change times), often to mimic files that are in the same folder. This is done, for example, on files that have been modified or created by the adversary so that they do not appear conspicuous to forensic investigators or file analysis tools.

Timestomping may be used along with file name Masquerading to hide malware and tools.

\[[Source](https://attack.mitre.org/techniques/T1070/006/)]

## Techniques

### Empire

```
usemodule powershell/management/timestomp

> set FilePath "C:\Secrets\SecretNotes.txt"
> set Modified 10/10/2020 12:00pm
> set Accessed 10/10/2020 13:00pm
> execute 
```

![](<../../../../.gitbook/assets/image (73) (2).png>)

### Metasploit

```
# Meterpreter shell

Usage: timestomp <file(s)> OPTIONS

OPTIONS:

    -a   Set the "last accessed" time of the file
    -b   Set the MACE timestamps so that EnCase shows blanks
    -c   Set the "creation" time of the file
    -e   Set the "mft entry modified" time of the file
    -f   Set the MACE of attributes equal to the supplied file
    -h   Help banner
    -m   Set the "last written" time of the file
    -r   Set the MACE timestamps recursively on a directory
    -v   Display the UTC MACE values of the file
    -z   Set all four attributes (MACE) of the file
```

View current MACE information

```
timestomp "C:\Secrets\SecretNote.txt" -v
```

![](<../../../../.gitbook/assets/image (1088) (2) (1) (1).png>)

Set all MACE attributes

```
timestomp "C:\Secrets\SecretNote.txt" -z "02/02/2020 23:13:23"
```

![](<../../../../.gitbook/assets/image (78).png>)

Recursively blank all MACE Attributes

```
timestomp C:\\Secrets\\ -r
```

![](<../../../../.gitbook/assets/image (81).png>)

## Mitigation

This type of attack technique cannot be easily mitigated with preventive controls since it is based on the abuse of system features.

## Further Reading

**Timestomp:** [https://www.offensive-security.com/metasploit-unleashed/timestomp/](https://www.offensive-security.com/metasploit-unleashed/timestomp/)
