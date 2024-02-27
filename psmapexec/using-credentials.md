# Using Credentials

Most methods within PsMapExec supports using various credential material.&#x20;

### CurrentUser

This switch allows for the current user context to be used across authentication. Its not really required as PsMapExec should determine if this should be in use anyway.

```powershell
PsMapExec -Targets All -Method WMI -CurrentUser -Command ps
```

### Ticket

A base64 encoded Kerberos ticket can be supplied to the `-Ticket` parameter either directly into the console or can be loaded from file.

the username switch does not need to be provided as a valid kerberos ticket will contain the intended username for the ticket anyway.

```powershell
PsMapExec -Targets All -Method WMI -Ticket "doIhsj..."
PsMapExec -Targets All -Method WinRM -Ticket "C:\ticket.txt"
```

### Username and Hash

A username and hash combination can also be provided for authentication. The following hashes are currently accepted:

* RC4 / NT
* NTLM
* AES256 HMAC&#x20;

{% code overflow="wrap" %}
```powershell
PsMapExec -Targets All -Method WMI  -Command "net user" -Username [User] -Hash [Hash]
```
{% endcode %}

### Username and Password

Of course a traditional username and password combination is also supported. If the provided password contains a "$" ensure the password is wrapped in single quotes.

{% code overflow="wrap" %}
```powershell
PsMapExec -Targets All -Method WinRM -Username [User] -Password [Password]
```
{% endcode %}

### Local Authentication

Currently, the switch `-LocalAuth` is only supported across the following methods:

* WMI
* RDP
* MSSQL (MSSQL SQL Server authentication)

Local authentication currently only supports a username and password combination.

{% code overflow="wrap" %}
```powershell
# WMI
PsMapExec -Targets All -Method WMI -LocalAuth -Username [User] -Password [Password]

# MSSQL
PsMapExec -Targets All -Method MSSQL -LocalAuth -Username sa -Password sa

# RDP
PsMapExec -Targets All -Method RDP -LocalAuth -Username [User] -Password [Password]
```
{% endcode %}
