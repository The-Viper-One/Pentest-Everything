# Hunit (WIP)

## Nmap

```
sudo nmap   192.168.79.125 -p- -sS -sV

Not shown: 65531 filtered ports
PORT      STATE SERVICE    VERSION
8080/tcp  open  http-proxy
12445/tcp open  unknown
18030/tcp open  http       Apache httpd 2.4.46 ((Unix))
43022/tcp open  ssh        OpenSSH 8.4 (protocol 2.0)
```

Browsing to port 8080 takes us to a web page for haikus.

![](<../../.gitbook/assets/image (1314).png>)

We can individually browse to each haiku.

![](<../../.gitbook/assets/image (1315).png>)

Checking the source page for any haiku reveals a comment refer to API.

![](<../../.gitbook/assets/image (1316).png>)

Running curl against the API reveals further information

```
curl http://192.168.79.125:8080/api/
```

![](<../../.gitbook/assets/image (1317).png>)

Runnining curl against the user API directory reveals sensitive information regarding each user.

```
curl http://192.168.79.125:8080/api/user/
```

![](<../../.gitbook/assets/image (1318).png>)

Compiling the passwords and login names of each provides us with a users and password list.

{% tabs %}
{% tab title="Users" %}
```
rjackson
dademola
jvargas
jsanchez
```
{% endtab %}
{% endtabs %}

{% tabs %}
{% tab title="Passwords" %}
```
yYJcgYqszv4aGQ
ExplainSlowQuest110
KTuGcSW6Zxwd0Q
d52cQ1BzyNQycg
OuQ96hcgiM5o9w
```
{% endtab %}
{% endtabs %}

I then tried bruteforcing this with Hydra and was unable to get a result.

![](<../../.gitbook/assets/image (1319).png>)

Inspecting our found information further we find that all the users are 'Editors' and David is a admin. The password associated with David is also greatly different from the rest. I then tried a manual login with SSH.

```
ssh -p 43022 dademola@192.168.79.125
```

Valid credentials: `dademola:ExplainSlowQuest110`

![](<../../.gitbook/assets/image (1320).png>)

Looking for other users in /home/ we see we have the Git user. Checking contents of the directory we also have a id\_rsa key.

![](<../../.gitbook/assets/image (1321).png>)
