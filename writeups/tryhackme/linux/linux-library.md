---
description: https://tryhackme.com/room/bsidesgtlibrary
---

# Library

## Nmap

```
nmap 10.10.241.233 -T4 -A -p-

PORT   STATE SERVICE VERSION

22/tcp open  ssh     OpenSSH 7.2p2 Ubuntu 4ubuntu2.8 (Ubuntu Linux; protocol 2.0)

80/tcp open  http    Apache httpd 2.4.18 ((Ubuntu))
| http-robots.txt: 1 disallowed entry 
|_/
|_http-server-header: Apache/2.4.18 (Ubuntu)
|_http-title: Welcome to  Blog - Library Machine
Service Info: OS: Linux; CPE: cpe:/o:linux:linux_kernel
```

As port 80 is open lets run `nikto` and `gobuster`:

![nikto](<../../../.gitbook/assets/image (85) (1).png>)

![Gobuster](<../../../.gitbook/assets/image (87) (1).png>)

Nmap reports an entry for robots.txt and we found the following information:

```
User-agent: rockyou 
Disallow: /
```

At this point Gobuster only found a /images/ directory so before checking that out I am not sure what User-agent: rockyou implies so I will run Gobuster with the rockyou.txt wordlist. Usually this is used for passwords however, it does not hurt to run it against `gobuster`just in-case.

The /images/ directory did not have anything outstanding inside it however, we might be able to exploit a PUT request.

![/images/ directory](<../../../.gitbook/assets/image (88) (1).png>)

Before we do lets check out the root directory:

![comments on the root directory](<../../../.gitbook/assets/image (89) (1).png>)

![possible user account](<../../../.gitbook/assets/image (90) (1).png>)

We have gathered some interesting information as per the comments above and a possible user account for SSH with the user "meliodas". I am going to run `Hydra` against SSH based on the fact we have a possible user and potentially a wordlist hint in regards to the rockyou agent in /robots.txt/

I ran quick PUT request test using Burpsuite and received the follwing.

![testing a HTTP PUT request with Burp](<../../../.gitbook/assets/image (91).png>)

No luck here as PUT requests are now allowed. Lets check on Hydra.

![Hydra has been successful against SSH](<../../../.gitbook/assets/image (92).png>)

Looks like Hydra has a hit. Lets test them on SSH.

![Logging in and grabbing user.txt](<../../../.gitbook/assets/image (94) (1).png>)

We managed to log in and grab the user.txt flag.

Lets check to see if we can do anything with `sudo -l`:

![checking sudoers permissions](<../../../.gitbook/assets/image (95) (1).png>)

Looks like we can run python as root on the bak.py file in our current home directory. Lets see what the file does.

![](<../../../.gitbook/assets/image (96) (1).png>)

I run the script as root and all this did was create a zip file from the contents of the /var/www/html directory. Nothing interesting.

Seeing as we can run bak.py as root and not allowed to edit the contents of the file we can instead delete it with the `rm` command

```
rm /home/meliodas/bak.py
```

Create a new bak.py file with the touch command:

```
touch /home/meliodas/bak.py
```

Now all we need to do is put a python shell command in this file. We achieve this either through `echo` or `nano`.

```
echo 'import pty;pty.spawn("/bin/bash")' > /home/meliodas/bak.py
```

Now we can run the script with the sudo command.

![Grabbing the root.txt flag](<../../../.gitbook/assets/image (97) (1).png>)
