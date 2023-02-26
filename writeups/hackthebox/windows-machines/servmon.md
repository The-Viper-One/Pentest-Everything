---
description: https://www.hackthebox.eu/home/machines/profile/240
---

# Servmon

![](<../../../.gitbook/assets/image (424).png>)

## Nmap

```
sudo nmap 10.10.10.184 -p- -T4

PORT      STATE SERVICE
21/tcp    open  ftp
22/tcp    open  ssh
80/tcp    open  http
135/tcp   open  msrpc
139/tcp   open  netbios-ssn
445/tcp   open  microsoft-ds
5040/tcp  open  unknown
5666/tcp  open  nrpe
6063/tcp  open  x11
6699/tcp  open  napster
7680/tcp  open  pando-pub
8443/tcp  open  https-alt
49664/tcp open  unknown
49665/tcp open  unknown
49666/tcp open  unknown
49667/tcp open  unknown
49668/tcp open  unknown
49669/tcp open  unknown
49670/tcp open  unknown
```

## SMB

A quick check on SMB with `smbmap` using null authentication and using enum4linux with the `-a` switch reveals no access.

![](<../../../.gitbook/assets/image (425).png>)

## Rpcclient

A quick check with null credentials with `rpcclient` displays "NT\_STATUS\_ACCESS\_DENIED."

```
rpcclient -U "" 10.10.10.184
```

![](<../../../.gitbook/assets/image (426).png>)

## FTP

As port 21 is open for FTP we can check with the `nmap` ftp-anon script for anonymous login.

```
nmap 10.10.10.184 -p 21 --script=ftp-anon
```

![](<../../../.gitbook/assets/image (427).png>)

`nmap` has confirmed we can login to the FTP with anonymous credentials. We can then login to FTP with the following command specifying the user as 'anonymous' and using a blank password.

```
ftp 10.10.10.184
```

Once logged in we find a 'Users' directory and under the folders Nathan and Nadine we find some text files we are able to download using the get command.

![](<../../../.gitbook/assets/image (428) (1).png>)

I was unable to find any other files or folders on the FTP server. We can now take a look at the documents we have retrieved so far.

![](<../../../.gitbook/assets/image (429).png>)

Looking at both of these documents we have the following information:\\

* A file named 'Passwords.txt' exists on Nathan's Desktop.
* Public access is enabled to the NVMS
* Secret files are located somewhere and have yet to been uploaded to SharePoint.

## HTTP

On port 80 the root page redirects to [http://10.10.10.184/Pages/login.htm](http://10.10.10.184/Pages/login.htm) I ran `dirb` and `nikto` against this and was unable to find anything of interest. After a quick Google search for NVMS-1000 exploits we can see `metasploit` has a module which can be used by us.

{% embed url="https://www.rapid7.com/db/modules/auxiliary/scanner/http/tvt_nvms_traversal/" %}

Load up `metasploit` and search for the module. Once selected set the RHOSTS value and set the FILEPATH value.

Knowing that Nathan has a file called 'Passwords.txt' on his Desktop we can attempt to read this. I set the file path to the following `'/users/nathan/Desktop/Passwords.txt'` We can then run the exploit.

![](<../../../.gitbook/assets/image (431).png>)

We have managed to read the file and obtain some passwords.

At this point we have two confirmed usernames and a small selection of passwords. We can run the credentials against a service and see what we can get.

We can run the credentials against the `metasploit` module `auxiliary/scanner/smb/smb_login` and we get a successful attempt.

![](<../../../.gitbook/assets/image (432).png>)

nadine:L1k3B1gBut7s@W0rk

## User Shell

With these credentials I tried logging into SMB and was allowed access but was unable to access any interesting shares. RPC was allowed as a login with `Rpcclient` however, very limited access gave no information.

### SSH

I was able to log into SSH on the server with the given credentials.

```
ssh nadine@10.10.10.184
```

![](<../../../.gitbook/assets/image (433).png>)

We now grab the user.txt flag.

![](<../../../.gitbook/assets/image (434).png>)

## Privilege Escalation

After searching through the machine manually I could not find much in terms of interesting information until I took a look at the 'Program Files' directory where we can see a non default installation of 'NSClient++'

Searching for exploits related to this on Google we come to a privilege escalation exploit that includes detailed instructions on how to perform the exploit.

{% embed url="https://www.exploit-db.com/exploits/46802" %}

Following from the instructions on the exploit page firstly, we can run the following command to get the Administrator web credentials.

```
nscp web -- password --display
```

![](<../../../.gitbook/assets/image (435).png>)

Password:`ew2x6SsGTxjRwXOT`

If we recall back to our `nmap` results from earlier we have a web server running on port 8443 in which the root page redirects us to the following:

![](<../../../.gitbook/assets/image (436) (1).png>)

We get the error "403 Your not allowed" When attempting to login with the Administrator credentials. If we look at the `nsclient.ini` file again we can see that on logins from the localhost address are allowed.

![](<../../../.gitbook/assets/image (437).png>)

We can get around this by running the command listed below from the terminal on the attacking machine.

```
ssh -L 8443:127.0.0.1:8443 nadine@10.10.10.184
```

Once completed we can now access the same page again over the localhost address 127.0.0.1.

![](<../../../.gitbook/assets/image (438).png>)

Now the GUI here is not very nice to use. For me this was unstable and difficult to work with. When researching exploits for NSClient earlier I did come across a python script that allowed RCE providing we have administrator credentials.

{% embed url="https://www.exploit-db.com/exploits/48360" %}

I downloaded the script and tested a command for account creation to confirm if working.

![](<../../../.gitbook/assets/image (439) (1).png>)

I then checked from the users perspective on the victim machine to see if the account was created.

![](<../../../.gitbook/assets/image (440) (1).png>)

We have confirmed command execution. From here I added our user Nadine into the local administrators group. Logged out of SSH and back in for the group changes to take place on her account.

![](<../../../.gitbook/assets/image (441).png>)

![](<../../../.gitbook/assets/image (442).png>)

We are now part of the 'Administrators' group. From here I attempted to read the root flag on the Administrator desktop.

![](<../../../.gitbook/assets/image (443).png>)

No Access.. We can try to login with a Psexec session using one of the Impacket's script. Hopefully this will spawn us in as 'NT Authority\System'

```
sudo python psexec.py servmon.htb.local/nadine:L1k3B1gBut7s@W0rk@10.10.10.184
```

We now have access as 'System'.

![](<../../../.gitbook/assets/image (445).png>)
