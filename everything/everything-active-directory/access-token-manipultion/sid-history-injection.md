---
description: https://attack.mitre.org/techniques/T1134/005/
---

# 🔨 SID-History Injection

**ATT\&CK ID:** [T1134.005](https://attack.mitre.org/techniques/T1134/005/)

**Permissions Required:** <mark style="color:red;">**Administrator**</mark> | <mark style="color:red;">**SYSTEM**</mark>

**Description**

Adversaries may use SID-History Injection to escalate privileges and bypass access controls. The Windows security identifier (SID) is a unique value that identifies a user or group account. SIDs are used by Windows security in both security descriptors and access tokens. An account can hold additional SIDs in the SID-History Active Directory attribute, allowing inter-operable account migration between domains (e.g., all values in SID-History are included in access tokens).

With Domain Administrator (or equivalent) rights, harvested or well-known SID values may be inserted into SID-History to enable impersonation of arbitrary users/groups such as Enterprise Administrators. This manipulation may result in elevated access to local resources and/or access to otherwise inaccessible domains via lateral movement techniques such as Remote Services, SMB/Windows Admin Shares, or Windows Remote Management.

[\[Source\]](https://attack.mitre.org/techniques/T1134/005/)

## Techniques

### Empire

```
powershell/persistence/misc/add_sid_history
```

### Mimikatz

```powershell
Invoke-Mimikatz -Command '"sid::patch" "sid::add /sid: /sam:"'
```

## Mitigation

Clean up SID-History attributes after legitimate account migration is complete.

Consider applying SID Filtering to interforest trusts, such as forest trusts and external trusts, to exclude SID-History from requests to access domain resources. SID Filtering ensures that any authentication requests over a trust only contain SIDs of security principals from the trusted domain (i.e preventing the trusted domain from claiming a user has membership in groups outside of the domain).

SID Filtering of forest trusts is enabled by default, but may have been disabled in some cases to allow a child domain to transitively access forest trusts. SID Filtering of external trusts is automatically enabled on all created external trusts using Server 2003 or later domain controllers. However note that SID Filtering is not automatically applied to legacy trusts or may have been deliberately disabled to allow inter-domain access to resources.

SID Filtering can be applied by:

* Disabling SIDHistory on forest trusts using the netdom tool (`netdom trust /domain: /EnableSIDHistory:no` on the domain controller)
* Applying SID Filter Quarantining to external trusts using the netdom tool (`netdom trust /domain: /quarantine:yes` on the domain controller)
* Applying SID Filtering to domain trusts within a single forest is not recommended as it is an unsupported configuration and can cause breaking changes. If a domain within a forest is untrustworthy then it should not be a member of the forest. In this situation it is necessary to first split the trusted and untrusted domains into separate forests where SID Filtering can be applied to an interforest trust

## Further Reading

**Windows Security Identifier (SID) History Injection Exposure:** [https://www.attivonetworks.com/blogs/windows-sid-history-injection-exposure-blog/](https://www.attivonetworks.com/blogs/windows-sid-history-injection-exposure-blog/)
