---
description: https://attack.mitre.org/techniques/T1558/
---

# Steal or Forge Kerberos Tickets

**ATT\&CK ID:** [T1558](https://attack.mitre.org/techniques/T1558/)

**Description**

Adversaries may attempt to subvert Kerberos authentication by stealing or forging Kerberos tickets to enable [Pass the Ticket](https://attack.mitre.org/techniques/T1550/003). Kerberos is an authentication protocol widely used in modern Windows domain environments. In Kerberos environments, referred to as "realms", there are three basic participants: client, service, and Key Distribution Center (KDC).[\[1\]](https://adsecurity.org/?p=227)

Clients request access to a service and through the exchange of Kerberos tickets, originating from KDC, they are granted access after having successfully authenticated. The KDC is responsible for both authentication and ticket granting. Adversaries may attempt to abuse Kerberos by stealing tickets or forging tickets to enable unauthorized access.The table below shows only results that are pertinent to Windows.

## Sub Techniques

### T1558.001: Golden Ticket

{% content-ref url="golden-ticket.md" %}
[golden-ticket.md](golden-ticket.md)
{% endcontent-ref %}

### T1558.002: Silver Ticket

{% content-ref url="silver-ticket.md" %}
[silver-ticket.md](silver-ticket.md)
{% endcontent-ref %}

### T1558.003: Kerberoasting

{% content-ref url="kerberoasting.md" %}
[kerberoasting.md](kerberoasting.md)
{% endcontent-ref %}

### T1558.004: AS-REP Roasting

{% content-ref url="as-rep-roasting.md" %}
[as-rep-roasting.md](as-rep-roasting.md)
{% endcontent-ref %}

## S4U2Self

{% content-ref url="s4u2self.md" %}
[s4u2self.md](s4u2self.md)
{% endcontent-ref %}

### Ticket Acquisition

{% content-ref url="ticket-aquisition.md" %}
[ticket-aquisition.md](ticket-aquisition.md)
{% endcontent-ref %}

### Constrained Delegation

{% content-ref url="constrained-delegation.md" %}
[constrained-delegation.md](constrained-delegation.md)
{% endcontent-ref %}

### Unconstrained Delegation

{% content-ref url="unconstrained-delegation.md" %}
[unconstrained-delegation.md](unconstrained-delegation.md)
{% endcontent-ref %}
