# HackerFest2019

## Nmap

```
sudo nmap   192.168.152.32 -p- -sS -sV

PORT      STATE SERVICE  VERSION
21/tcp    open  ftp      vsftpd 3.0.3
22/tcp    open  ssh      OpenSSH 7.4p1 Debian 10+deb9u7 (protocol 2.0)
80/tcp    open  http     Apache httpd 2.4.25 ((Debian))
10000/tcp open  ssl/http MiniServ 1.890 (Webmin httpd)
Service Info: OSs: Unix, Linux; CPE: cpe:/o:linux:linux_kernel
```

First up checking FTP we have anonymous access. In what appears to be a Wordpress directory. First we can grab the wp-config.php as this will likely contain credentials we can use.

![](<../../../.gitbook/assets/image (1215) (1).png>)

Reading the contents of wp-config.php shows some credentials we can use later. The credentials are: `wordpress:nvwtlRqkD0E1jBXu`

![](<../../../.gitbook/assets/image (1216).png>)

Running `dirsearch.py` against port 80 reveals the directory /phpmyadmin

```
python3 dirsearch.py -u http://192.168.152.32  -w /usr/share/seclists/Discovery/Web-Content/big.txt -t 60 --full-url
```

![](<../../../.gitbook/assets/image (1217).png>)

I then tried to login with simple credentials such as `root:root` and was informed by the web server we cannot use root as a login.

![](<../../../.gitbook/assets/image (1218) (1).png>)

I tried the database credentials from earlier and was permitted access: `wordpress:nvwtlRqkD0E1jBXu`

![](<../../../.gitbook/assets/image (1219).png>)

Opening up the Wordpress database we find a password hash for the user webmaster.

![](<../../../.gitbook/assets/image (1220) (1).png>)

This hash was cracked with `hashcat` on Windows.

![](<../../../.gitbook/assets/image (1221).png>)

We have the credentials: `webmaster:kittykat1` We can then browse to [http://192.168.152.32/wp-admin/](http://192.168.152.32/wp-admin/) and login with the credentials above.

Once logged in we notice we are working in a language other than English. Follow the image below to change this back to English if required.

![](<../../../.gitbook/assets/image (1222).png>)

After doing so we can head over to Appearance > Theme Editor and replace the contents of index.php with a [PHP Reverse shell](https://github.com/pentestmonkey/php-reverse-shell).

![](<../../../.gitbook/assets/image (1223).png>)

Once completed start a `netcat` listener then browse to the main index.php page to execute the shell.

![](<../../../.gitbook/assets/image (1224).png>)

From here the path to root is super simple. As the user webmaster exists on this machine we can simply `su` into the user with the credentials we obtained earlier. Check `sudo -l` and then run /bin/bash using sudo.

```
su webmaster
sudo -l
sudo /bin/bash
```

![](<../../../.gitbook/assets/image (1225) (1).png>)
