# Spray

The spray module provides different password spraying techniques. PsMapExec takes into account the default domain policy's account lockout threshold to prevent user account lockouts. However, it does not consider fine-grained password policies. It's advisable to assess whether such policies are in place within the environment to avoid potentially locking out a significant number of user accounts.

<figure><img src="../../.gitbook/assets/image (2118).png" alt=""><figcaption></figcaption></figure>

## Usage

### Targets

When using the Spray method `-Targets` parameter can be provided. Specifying "all" we spray all enabled user accounts in the domain. Otherwise, any other value will be treated as a group name. When `-Targets` is omitted, PsMapExec will spray all enabled active directory accounts.

```powershell
PsMapExec -Method Spray                           # Sprays all
PsMapExec -Method Spray -Targets all              # Sprays all
PsMapExec -Method Spray -Targets "C:\Users.txt"   # Sprays users from list (SamAccountNames)
PsMapExec -Method Spray -Targets "AdminCount=1"   # Sprays targets with AdminCount=1
PsMapExec -Method Spray -Targets "Group Name"     # Sprays members of group
```



### Hash

Hash authentication supports RC4/NT, NTLM and AES256 hashes

```powershell
PsMapExec -Method Spray -SprayHash [RC4]
PsMapExec -Method Spray -SprayHash [AES256]
PsMapExec -Method Spray -SprayHash [NTLM]
```

### Password

```powershell
PsMapExec -Method Spray -SprayPassword [Password]
```

### AccountAsPasswords

Sets the password to the username value. This switch will also attempt to authenticate as computer accounts to identify any that might be current or legacy Pre-Windows 2000 Compatible Computers.

```powershell
PsMapExec -Method Spray -AccountAsPassword
```

### EmptyPassword

Authentication attempts are performed with empty password values.

```powershell
PsMapExec -Method Spray -EmptyPassword
```

### SuccessOnly

Displays only successful authentication attempts.

```
PsMapExec -Method Spray -SprayPassword Password123! -SuccessOnly
```

##
