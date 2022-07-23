---
description: PG Practice Algernon writeup
---

# Algernon

## Nmap

```
sudo nmap 192.168.147.65 -p- -sV -sS    

PORT      STATE SERVICE       VERSION
21/tcp    open  ftp           Microsoft ftpd
80/tcp    open  http          Microsoft IIS httpd 10.0
135/tcp   open  msrpc         Microsoft Windows RPC
139/tcp   open  netbios-ssn   Microsoft Windows netbios-ssn
445/tcp   open  microsoft-ds?
9998/tcp  open  http          Microsoft IIS httpd 10.0
17001/tcp open  remoting      MS .NET Remoting services
Service Info: OS: Windows; CPE: cpe:/o:microsoft:windows
```

Port 80 redirects to the default IIS page.

![](<../../../.gitbook/assets/image (807).png>)

Port 9998 directs us to the following login page for SmarterMail.

![](<../../../.gitbook/assets/image (806) (1).png>)

Researching exploits for SmarterMail on Google we come across an interesting exploit:

{% embed url="https://www.exploit-db.com/exploits/49216" %}

Looking at the description for this is exploit we have the following:

![](<../../../.gitbook/assets/image (808) (1).png>)

Looking at the `nmap` results from earlier we do have .NET remoting running on port 17001. As such this exploit should be applicable to the target machine.

I downloaded the exploit and edited the following portion to match my IP and to use the local port of 21.

![](<../../../.gitbook/assets/image (809).png>)

I then set up a `netcat` listener for port 21.

```
sudo nc -lvp 21
```

Then executed the exploit and received a reverse shell as SYSTEM.

![](<../../../.gitbook/assets/image (810).png>)
