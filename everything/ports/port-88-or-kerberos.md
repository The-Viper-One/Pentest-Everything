# Port 88 | Kerberos

### User Enumeration

#### Nmap

```bash
nmap -p 88 --script=krb5-enum-users --script-args krb5-enum-users.realm=<Domain>,userdb=<Wordlist> <IP>
```

#### Metasploit

```
use auxiliary/gather/kerberos_enumusers
```

#### Kerbrute

**Download:** [https://github.com/ropnop/kerbrute](https://github.com/ropnop/kerbrute)

```python
./kerbrute userenum <UserList> --dc <IP> --domain <Domain>
```

#### Rubeus

```python
# with a list of users
.\Rubeus.exe brute /users:<UserList> /passwords:<Wordlist> /domain:<Domain>

# Check all domain users again password list
.\Rubeus.exe brute /passwords:<Wordlist>
```
