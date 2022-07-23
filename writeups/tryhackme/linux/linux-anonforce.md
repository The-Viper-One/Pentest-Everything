---
description: https://tryhackme.com/room/bsidesgtanonforce
---

# Anonforce

## Nmap

```
nmap 10.10.13.132 -A -T4 -p- 

PORT   STATE SERVICE VERSION
21/tcp open  ftp     vsftpd 3.0.3
| ftp-anon: Anonymous FTP login allowed (FTP code 230)
| drwxr-xr-x    2 0        0            4096 Aug 11  2019 bin
| drwxr-xr-x    3 0        0            4096 Aug 11  2019 boot
| drwxr-xr-x   17 0        0            3700 Sep 05 00:59 dev
| drwxr-xr-x   85 0        0            4096 Aug 13  2019 etc
| drwxr-xr-x    3 0        0            4096 Aug 11  2019 home
| lrwxrwxrwx    1 0        0              33 Aug 11  2019 initrd.img -> boot/initrd.img-4.4.0-157-generic
| lrwxrwxrwx    1 0        0              33 Aug 11  2019 initrd.img.old -> boot/initrd.img-4.4.0-142-generic
| drwxr-xr-x   19 0        0            4096 Aug 11  2019 lib
| drwxr-xr-x    2 0        0            4096 Aug 11  2019 lib64
| drwx------    2 0        0           16384 Aug 11  2019 lost+found
| drwxr-xr-x    4 0        0            4096 Aug 11  2019 media
| drwxr-xr-x    2 0        0            4096 Feb 26  2019 mnt
| drwxrwxrwx    2 1000     1000         4096 Aug 11  2019 notread [NSE: writeable]
| drwxr-xr-x    2 0        0            4096 Aug 11  2019 opt
| dr-xr-xr-x  102 0        0               0 Sep 05 00:59 proc
| drwx------    3 0        0            4096 Aug 11  2019 root
| drwxr-xr-x   18 0        0             540 Sep 05 00:59 run
| drwxr-xr-x    2 0        0           12288 Aug 11  2019 sbin
| drwxr-xr-x    3 0        0            4096 Aug 11  2019 srv
| dr-xr-xr-x   13 0        0               0 Sep 05 00:59 sys
|_Only 20 shown. Use --script-args ftp-anon.maxlist=-1 to see all.
| ftp-syst: 
|   STAT: 
| FTP server status:
|      Connected to ::ffff:10.9.29.24
|      Logged in as ftp
|      TYPE: ASCII
|      No session bandwidth limit
|      Session timeout in seconds is 300
|      Control connection is plain text
|      Data connections will be plain text
|      At session startup, client count was 2
|      vsFTPd 3.0.3 - secure, fast, stable                                                                                                                                                                                                 
|_End of status                                                                                                                                                                                                                            
22/tcp open  ssh     OpenSSH 7.2p2 Ubuntu 4ubuntu2.8 (Ubuntu Linux; protocol 2.0)                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                               
Service Info: OSs: Unix, Linux; CPE: cpe:/o:linux:linux_kernel 
```

Looks like we have port 21 and port 22 open on the server. port 21 looks to be misconfigured lets have a closer look.

We can log in to FTP with the anonymous user name and specify anything for the password.

Straight away we can cd into home and identify a possible user for SSH and grab the user.txt flag.

![Grabbing user flag from melodias](<../../../.gitbook/assets/image (76) (1).png>)

Looking in the directorys we can see a directory called "notread" which list the follow contents:

![notread directory](<../../../.gitbook/assets/image (77).png>)

We can download both of these files with the `mget *` command.

After downloading the files we need to decrypt the files so we should first start by using the `gpg` command to try and import the privatekey file private.asc. Unfortunately we need a passphrase to complete this process.

![Attempting to import the private key](<../../../.gitbook/assets/image (78) (1).png>)

We can attempt to hash the file and run it through John The Ripper. We need the module gpg2john which usually comes preinstalled with John The Ripper. As you can see below we locate the module and then execute it and define the private key file and a locate to output the hash:

![Hashing the private key](<../../../.gitbook/assets/image (79) (1).png>)

After hashing we can run John. I have removed the password from the following image as per guidelines from THM.

![Cracking the hash](<../../../.gitbook/assets/image (80).png>)

We can now try importing the private.asc key again and when prompted for a password I entered the one cracked by John.

![Importing the key](<../../../.gitbook/assets/image (81) (1).png>)

We have now imported the key. Next we need to see if this key will decrypt the backup.pgp file we downloaded earlier.

After run the command below and entering the password we retrieved earlier we can see backup.pgp appears to be a backup of /etc/shadow.

![hashes have been removed from this image](<../../../.gitbook/assets/image (82) (1).png>)

We can run this file against John to see if we can crack the account hashes.

![Cracking the account hashes](<../../../.gitbook/assets/image (83) (1).png>)

In this instance I was only able to find a hash for the root account. I tried a few more passwords and was unable to hit the user account. Lets see if we can SSH into root seeing as port 22 is open.

![Logging in at root on port 22](<../../../.gitbook/assets/image (84) (1).png>)

As you can see above I was able to login with the password cracked from John and was able to read the root flag.
