# 🔨 RDP MiTM

## Description

RDP MiTM attacks are possible through the usage of a tool called Seth.&#x20;

Seth is a tool written in Python and Bash to MitM RDP connections by attempting to downgrade the connection in order to extract clear text credentials. It was developed to raise awareness and educate about the importance of properly configured RDP connections in the context of pentests, workshops or talks. The author is Adrian Vollmer (SySS GmbH).

## Scenario Lab

* Victim - Windows 10 21H1 - 10.10.10.X
* Attacking-PC - Kali Linux - 10.10.10.6
* DC01 - Windows Server 2019 - 10.10.10.10
* Gateway - 10.10.10.1

All Windows systems are fully updated and running latest versions of Windows Defender with default settings.

To start Seth is initiated with the following syntax and command.

```
sudo ./seth.sh <interface> <Attacker-PC> <Victim-PC> <Server>
sudo ./seth.sh eth1 10.10.10.6 10.10.10.7 10.10.10.10 
```

![](<../../../.gitbook/assets/image (1950).png>)

Then on the Victim-PC the victim initiates an RDP session as part of a routine task to the Domain Controller on 10.10.10.10.

On connection request we see the following.

![MiTM intercepted connection](<../../../.gitbook/assets/image (1948).png>)

Where normally an RDP connection will produce results similar to that of below. Taking note of the extra warnings provided above as opposed to the legitimate request below.

![Legitimate Connection](<../../../.gitbook/assets/image (1949).png>)

Assuming the victim accepts the malicious certificate request the Attacking-PC will receive a Network NTLM hash for the user and the users password in cleartext.

![](<../../../.gitbook/assets/image (1951).png>)

The victim user should continue to connect to the destination server and be none the wiser to the attack.

## Mitigation

### Group Policy: Require user authentication for remote connections by using Network Level Authentication

If you want to restrict who can access your PC, choose to allow access only with Network Level Authentication (NLA). When you enable this option, users have to authenticate themselves to the network before they can connect to your PC

Domain joined computers by default have this option enabled. It would however, be more appropriate to ensure this is enforced through Group Policy.

Computer Configuration > Policies > Administrative Templates > Windows Components > Remote Desktop Services > Remote Desktop Session Host > Security > Require user authentication for remote connections by using Network Level Authentication > Enabled.

Whilst the attack can be still be performed to the same effect where a NTLM hash is captured and a password is presented in cleartext this policy will stop the user from finalizing the connection to the intended RDP destination.

![](<../../../.gitbook/assets/image (1966).png>)

This has the possibility of alerting the users IT teams to the anomaly which could lead to detection of the adversary on the network.

An additional as a result of a failure to connect is that an adversary cannot use Seth to inject key presses into the RDP session.

### Group Policy: Disallow connections if the certificate cannot be validated

Computer configuration > Policies > Administrative Templates > Windows Components >Remote Desktop Services (or Terminal Services) > Remote Desktop Connection Client > Configure server authentication for client.

This policy setting pushed out to endpoints will partially stop the attack. When performing the attack steps again we are instead presented with the following warning after attempting to send credentials for authentication.

![](<../../../.gitbook/assets/image (1953).png>)

As far as the Attacker-PC goes we are unable to view clear text credentials. We do still have a Network NTLM hash which can potentially be cracked or relayed elsewhere to bridge further attack vectors.

![](<../../../.gitbook/assets/image (1954).png>)

## References:

{% embed url="https://github.com/SySS-Research/Seth/blob/master/doc/paper/Attacking_RDP-Paper.pdf" %}

## Tools used

### Seth: [https://github.com/SySS-Research/Seth](https://github.com/SySS-Research/Seth)

```
sudo git clone https://github.com/SySS-Research/Seth.git
```

### dSniff: [https://www.monkey.org/\~dugsong/dsniff/](https://www.monkey.org/\~dugsong/dsniff/)

```
sudo apt install dsniff -y
```
