---
description: https://attack.mitre.org/techniques/T1556/
---

# 🔨 Modify Authentication Process

**ATT\&CK ID:** [T1556](https://attack.mitre.org/techniques/T1556/)

**Description**

Adversaries may modify authentication mechanisms and processes to access user credentials or enable otherwise unwarranted access to accounts. The authentication process is handled by mechanisms, such as the Local Security Authentication Server (LSASS) process and the Security Accounts Manager (SAM) on Windows

By modifying an authentication process, an adversary may be able to authenticate to a service or system without using Valid Accounts.

Adversaries may maliciously modify a part of this process to either reveal credentials or bypass authentication mechanisms. Compromised credentials or access may be used to bypass access controls placed on various resources on systems within the network and may even be used for persistent access to remote systems and externally available services, such as VPNs, Outlook Web Access and remote desktop.

\[[Source](https://attack.mitre.org/techniques/T1556/)]

## Sub Techniques

### T1556.001: Domain Controller Authentication

{% content-ref url="domain-controller-authentication-skeleton-key.md" %}
[domain-controller-authentication-skeleton-key.md](domain-controller-authentication-skeleton-key.md)
{% endcontent-ref %}

### T1556.002:

\<WIP>

### T1556.005: Reversible Encryption

{% content-ref url="reversible-encryption.md" %}
[reversible-encryption.md](reversible-encryption.md)
{% endcontent-ref %}
