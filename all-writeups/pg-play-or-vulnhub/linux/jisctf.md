# JISCTF

## Nmap

```
sudo nmap 192.168.152.25 -p- -sS -sV

PORT   STATE SERVICE VERSION
22/tcp open  ssh     OpenSSH 7.2p2 Ubuntu 4ubuntu2.1 (Ubuntu Linux; protocol 2.0)
80/tcp open  http    Apache httpd 2.4.18 ((Ubuntu))
Service Info: OS: Linux; CPE: cpe:/o:linux:linux_kernel
```

Checking out port 80 directs us to a login page on /login.php.

![](<../../../.gitbook/assets/image (1204).png>)

Running `dirsearch.py` against the web server reveals robots.txt

```
python3 dirsearch.py -u http://192.168.152.25  -w /usr/share/seclists/Discovery/Web-Content/big.txt -t 60 --full-url  
```

![](<../../../.gitbook/assets/image (1205).png>)

The contents of robots.txt is shown below:

```
User-agent: *
Disallow: /
Disallow: /backup
Disallow: /admin
Disallow: /admin_area
Disallow: /r00t
Disallow: /uploads
Disallow: /uploaded_files
Disallow: /flag
```

Browsing to /admin\_area shows the page below.

![](<../../../.gitbook/assets/image (1206).png>)

Viewing the source reveals sensitive information:

![](<../../../.gitbook/assets/image (1207).png>)

We can then login to /login.php with the credentials shown above. The following page reveals a web page for uploading files.

![](<../../../.gitbook/assets/image (1208).png>)

I then uploaded a [PHP reverse shell](https://github.com/pentestmonkey/php-reverse-shell) which after upload showed a 'success' status message. Knowing the directory /uploaded\_files/ exists we can then browse to this followed by the uploaded files name: [http://192.168.152.25/uploaded\_files/phpshell.php](http://192.168.152.25/uploaded\_files/phpshell.php).

The page should hang and we will receive a shell on our `netcat` listener.

![](<../../../.gitbook/assets/image (1209).png>)

I could not see that Python was installed on this machine so I instead used the following command to upgrade the shell:

```
/usr/bin/script -qc /bin/bash /dev/null
```

![](<../../../.gitbook/assets/image (1210).png>)

From here I transferred over `linpeas` from my attacking machine and let it run. The script picks up the username 'technawi' which is an alternative user on the box.

![](<../../../.gitbook/assets/image (1211).png>)

Running `cat` on the credentials.txt reveals login information. We can then use `su` to switch to the technawi user.

![](<../../../.gitbook/assets/image (1212).png>)

Checking `sudo -l` against the user reveals we can any command as any user on this machine.

![](<../../../.gitbook/assets/image (1213).png>)

We can then run the command below to spawn a root shell.

```
sudo /bin/bash
```

![](<../../../.gitbook/assets/image (1214).png>)
