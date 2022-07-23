---
description: https://tryhackme.com/room/cyberheroes
---

# CyberHeroes

## Nmap

```
nmap 10.10.150.136 -p- -sS -sV

PORT   STATE SERVICE VERSION
22/tcp open  ssh     OpenSSH 8.2p1 Ubuntu 4ubuntu0.4 (Ubuntu Linux; protocol 2.0)
80/tcp open  http    Apache httpd 2.4.48 ((Ubuntu))
Service Info: OS: Linux; CPE: cpe:/o:linux:linux_kernel
```

With only port 80 open we browse to the root page for CyberHeros.

![](<../../../.gitbook/assets/image (175).png>)

Running the web site through ZAP proxy with attack mode enabled reveals several pages. Viewing the response results for /login.html reveals a potential user name and password. We see the password is assigned the value [#undefined](cyberheroes.md#undefined "mention")[#undefined](cyberheroes.md#undefined "mention")RevereString".

![](<../../../.gitbook/assets/image (1879).png>)

Using the command line we are able to reverse the string.

```bash
echo "<Password>" | rev
```

![](<../../../.gitbook/assets/image (544).png>)

To reveal the correct password for the user h3ck3rBoi where, we can then grab the room flag.

![](<../../../.gitbook/assets/image (339).png>)
