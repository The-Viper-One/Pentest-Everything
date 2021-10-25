# Port 53 | DNS

```python
nslookup -query=mx '<Domain>' -server '<DNS-IP>'                                                                                                                                                         
nslookup -query=ns '<Domain>'  -server '<DNS-IP>'
nslookup -query=any '<Domain>' -server '<DNS-IP>'
dig '<Domain>'
dig '<Domain>' A
dig '<Domain>' AAAA
dig '<Domain>' PTR
dig '<Domain>' NS
dig '<Domain>' MX

nmap --script dns-brute --script-args dns-brute.threads=12 '<Domain>'

fierce -dns '<Domain>'
fierce -dns '<Domain>' -dnsserver '<DNS>'

dnsenum --dnsserver '<IP>' --enum '<Domain>'
```

Resolve DNS IP to Domain name.

```python
dig '@172.16.5.10' -x '172.16.5.10' +nocookie
```

Brute force

```python
fierce --domain '<Domain>' --range <Range> --dns-servers '<IP>' --subdomain-file '<wordlist>'
```

Brute force with Bash

```python
for name in $(cat /usr/share/seclists/Discovery/DNS/subdomains-top1million-20000.txt); do host $name.sportsfoo.com '172.16.5.10' -W 2; done | grep 'has address'
```

Zone Transfer

```python
dig '@<IP>' -t AXFR '<Domain>' +nocookie
```
