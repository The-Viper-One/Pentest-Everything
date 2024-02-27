# Inject

A simple method, Inject is used to inject a kerberos ticket in memory. There are two primary reasons for performing this method\


* You do not have any kerberos tickets already in memory, for example when working from a non-domain joined system
* You wish to revert to different "credentials" after performing impersonation in PsMapExec

### Ticket

A base64 encoded Kerberos ticket can be supplied to the `-Ticket` parameter either directly into the console or can be loaded from file.

the username switch does not need to be provided as a valid kerberos ticket will contain the intended username for the ticket anyway.

```powershell
PsMapExec -Method Inject -Ticket "doIhsj..."
PsMapExec -Method Inject -Ticket "C:\ticket.txt"
```

### Username and Hash

A username and hash combination can also be provided for authentication. The following hashes are currently accepted:

* RC4 / NT
* NTLM
* AES256 HMAC&#x20;

{% code overflow="wrap" %}
```powershell
PsMapExec -Method Inject -Username [User] -Hash [Hash]
```
{% endcode %}

### Username and Password

Of course a traditional username and password combination is also supported. If the provided password contains a "$" ensure the password is wrapped in single quotes.

{% code overflow="wrap" %}
```powershell
PsMapExec Method Inject -Username [User] -Password [Password]
```
{% endcode %}

<figure><img src="../../.gitbook/assets/image (2122).png" alt=""><figcaption></figcaption></figure>
