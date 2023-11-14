---
description: https://attack.mitre.org/techniques/T1003/006/
---

# DCSync

**ATT\&CK ID:** [T1003.006](https://attack.mitre.org/techniques/T1003/006/)

**Permissions Required:** <mark style="color:red;">**Administrator**</mark> | <mark style="color:orange;">**DCSync rights**</mark>

**Description**

Adversaries may attempt to access credentials and other sensitive information by abusing a Windows Domain Controller's application programming interface (API) to simulate the replication process from a remote domain controller using a technique called DCSync.

A DCSync attack is where an adversary impersonates a Domain Controller (DC) and requests replication changes from a specific DC. The DC in turn returns replication data to the adversary, which includes account hashes.

By default the following groups have permissions to perform this action:

* Administrators
* Domain Admins
* Enterprise Admins
* Domain Controllers

However, in an incorrectly configured environment it may be possible to hunt down users who have the required individual permissions without being in any of the aforementioned groups. These individual permissions are:

* Replicating Directory Changes
* Replicating Directory Changes All
* Replicating Directory Changes In Filtered Set

## Techniques

### Secretsdump.py

Impacket's `secretsdumpy.py` can be used to dump all domain hashes, providing the hash or password is known for an account with permission to perform replication.

```bash
sudo python2 secretsdump.py <Domain>/<User>:<Password>@<IP>
sudo python2 secretsdump.py security/Moe:'Password123!'@10.10.10.10
```

![](<../../../../../.gitbook/assets/image (1972).png>)

###



### Further Reading

{% content-ref url="dcsync-attack.md" %}
[dcsync-attack.md](dcsync-attack.md)
{% endcontent-ref %}
