---
description: https://attack.mitre.org/techniques/T1543/003/
---

# Windows Service

**ATT\&CK ID:** [T1543.003](https://attack.mitre.org/techniques/T1543/003/)

**Permissions Required:** <mark style="color:red;">**Administrator**</mark> | <mark style="color:red;">**SYSTEM**</mark>

**Description**

Adversaries may create or modify Windows services to repeatedly execute malicious payloads as part of persistence. When Windows boots up, it starts programs or applications called services that perform background system functions. Windows service configuration information, including the file path to the service's executable or recovery programs/commands, is stored in the Windows Registry.

Adversaries may install a new service or modify an existing service to execute at startup in order to persist on a system. Service configurations can be set or modified using system utilities (such as sc.exe), by directly modifying the Registry, or by interacting directly with the Windows API.

Services may be created with administrator privileges but are executed under SYSTEM privileges, so an adversary may also use a service to escalate privileges. Adversaries may also directly start services through Service Execution. To make detection analysis more challenging, malicious services may also incorporate Masquerade Task or Service (ex: using a service and/or payload name related to a legitimate OS or benign software component).

[\[Source\]](https://attack.mitre.org/techniques/T1543/003/)

## Techniques

### Empire

```
// Some code
```

### Metasploit

```
// Some code
```

## Mitigation

## Further Reading
