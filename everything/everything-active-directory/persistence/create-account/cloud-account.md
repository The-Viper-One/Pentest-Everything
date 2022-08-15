---
description: https://attack.mitre.org/techniques/T1136/003/
---

# Cloud Account

**ATT\&CK ID:** [T1136.003](https://attack.mitre.org/techniques/T1136/002/)

**Permissions Required:** <mark style="color:red;">**Global Administrator**</mark> | <mark style="color:red;">**User Administrator**</mark>

**Description**

Adversaries may create a cloud account to maintain access to victim systems. With a sufficient level of access, such accounts may be used to establish secondary credentialed access that does not require persistent remote access tools to be deployed on the system.

Adversaries may create accounts that only have access to specific cloud services, which can reduce the chance of detection.

## Techniques

### Azure PowerShell

```powershell
# Create Azure user
$PasswordProfile = New-Object -TypeName Microsoft.Open.AzureAD.Model.PasswordProfile
$PasswordProfile.Password = "<Password>"

New-AzureADUser `
    -DisplayName "New User" ` 
    -PasswordProfile $PasswordProfile `
    -UserPrincipalName "NewUser@contoso.com" `
    -AccountEnabled $true `
    -MailNickName "Newuser"

# Add user to Azure Group
Add-AzureADGroupMember -ObjectId "<ObjectID" -RefObjectId "<RefObject>"
```

## Mitigation

* Use multi-factor authentication for user and privileged accounts.
* Configure access controls and firewalls to limit access to critical systems and domain controllers. Most cloud environments support separate virtual private cloud (VPC) instances that enable further segmentation of cloud systems.

## Further Reading

**Azure PowerShell:** [https://docs.microsoft.com/en-us/powershell/module/azuread/new-azureaduser?view=azureadps-2.0](https://docs.microsoft.com/en-us/powershell/module/azuread/new-azureaduser?view=azureadps-2.0)
