---
description: https://app.hackthebox.com/machines/Help
---

# Help

## Nmap

```
sudo nmap 10.10.10.121 -p- -sS -sV   

PORT     STATE SERVICE VERSION
22/tcp   open  ssh     OpenSSH 7.2p2 Ubuntu 4ubuntu2.6 (Ubuntu Linux; protocol 2.0)
80/tcp   open  http    Apache httpd 2.4.18
3000/tcp open  http    Node.js Express framework
Service Info: Host: 127.0.1.1; OS: Linux; CPE: cpe:/o:linux:linux_kernel
```

**Note:** Add `10.10.10.121 help.htb` to `/etc/hosts`.

### Web Server

Viewing the web server running on port 80 we are greeted with the Apache2 default page.

<figure><img src="../../../.gitbook/assets/image (5) (1) (2) (1).png" alt=""><figcaption></figcaption></figure>

### Directory Brute Force

With nothing of interest left in the page source we change over to directory brute forcing with `feroxbuster`.

```
feroxbuster -u http://help.htb -w /usr/share/seclists/Discovery/Web-Content/directory-list-2.3-medium.txt
```

<figure><img src="../../../.gitbook/assets/image (68).png" alt=""><figcaption></figcaption></figure>

### HelpDeskZ

`feroxbuster` discovers the /support/ directory. Navigating to the directory we are directed to  a login page for "HelpDeskZ".

<figure><img src="../../../.gitbook/assets/image (8) (1) (3).png" alt=""><figcaption></figcaption></figure>

A quick search with `searchsploit` shows that HelpDeskZ may be vulnerable to arbitrary file upload attack.

<figure><img src="../../../.gitbook/assets/image (14) (4) (1).png" alt=""><figcaption></figcaption></figure>

Further enumeration with `feroxbuster` on the /support/ directory picks up `/readme.html`.

<figure><img src="../../../.gitbook/assets/image (4) (2) (1).png" alt=""><figcaption></figcaption></figure>

Viewing this we see the version of `HelpDeskZ` running is v1.0.2 which should be vulnerable to the arbitrary file upload.

<figure><img src="../../../.gitbook/assets/image (6) (1) (1) (1).png" alt=""><figcaption></figcaption></figure>

The basis of this exploit can be found here: [https://www.exploit-db.com/exploits/40300](https://www.exploit-db.com/exploits/40300). I was not able to get the included exploit to work however, used an alternative script linked further below to complete the exploit.

### File Upload

Firstly, navigate to the "Submit a Ticket" page. Fill in information as required and attach a PHP reverse shell.

<figure><img src="../../../.gitbook/assets/image (73).png" alt=""><figcaption></figcaption></figure>

### Reverse Shell

On upload the web page gives a "disallowed file" type error when uploaded PHP. This error can be disregarded. I then used the script linked below to complete the exploit and receive a shell on my `netcat` listener.

**Exploit:** [https://cxsecurity.com/issue/WLB-2017080112](https://cxsecurity.com/issue/WLB-2017080112)

<figure><img src="../../../.gitbook/assets/image (69).png" alt=""><figcaption></figcaption></figure>

### User Flag

After connecting the reverse shell we can navigate to /home/help to grab the `user.txt` flag

<figure><img src="../../../.gitbook/assets/image (45).png" alt=""><figcaption></figcaption></figure>

### Privilege Escalation

After grabbing the flag we upload a copy of `linpeas.sh` and let the script run. After a short while we see the binary `s-mail-privep` has the SUID bit set.

<figure><img src="../../../.gitbook/assets/image (9) (1) (1).png" alt=""><figcaption></figcaption></figure>

Looking for way to exploit the binary I came across the following bash exploit script in order to escalate privileges.

**ExploitDB:** [https://www.exploit-db.com/exploits/47172](https://www.exploit-db.com/exploits/47172)

### Root Shell

After uploading the script I ran it a few times before it worked correctly and gave a **root** shell.

<figure><img src="../../../.gitbook/assets/image (71).png" alt=""><figcaption></figcaption></figure>
