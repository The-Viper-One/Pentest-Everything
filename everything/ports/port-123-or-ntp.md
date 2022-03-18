# Port 123 | NTP

### **Clock Skew errors**

**Update system time**

```bash
sudo ntpdate <DC-IP>
sudo ntpdate <DC FQDN>
```

**Get remote system time through Nmap and manually update local system time**

```bash
sudo nmap -sU -p 123 --script ntp-info <IP>

PORT    STATE SERVICE
123/udp open  ntp
| ntp-info: 
|_  receive time stamp: 2022-03-18T02:29:45

# Then copy paste time into date command
sudo date -s "2022-03-18T02:29:45"
```

****

Disable time sync in Virtualbox if clock skew errors are persistent

```
VBoxManage.exe setextradata "Kali" "VBoxInternal/Devices/VMMDev/0/Config/GetHostTimeDisabled" 1
```

