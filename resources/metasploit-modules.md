# Metasploit Modules

## Active Directory Modules

```
use post/windows/gather/enum_ad_to_wordlist
use post/windows/gather/enum_ad_bitlocker
use post/windows/gather/enum_ad_computers
use post/windows/gather/enum_ad_groups
use post/windows/gather/enum_ad_managedby_groups
use post/windows/gather/enum_ad_service_principal_names 
use post/windows/gather/enum_ad_user_comments 
use post/windows/gather/credentials/enum_laps
```

## Multiplatform

### Post

```
use post/multi/gather/env
use post/multi/gather/ping_sweep
use post/windows/gather/hashdump
```

#### Privilege Escalation

```
use post/multi/recon/local_exploit_suggester
```

## Windows Modules

```
use post/windows/capture/lockout_keylogger
use post/windows/gather/bitlocker_fvek
use post/windows/gather/cachedump
use post/windows/gather/credentials/credential_collector
use post/windows/gather/credentials/outlook
use post/windows/gather/credentials/enum_cred_store
use post/windows/gather/credentials/rdc_manager_creds
use post/windows/gather/credentials/skype
use post/windows/gather/credentials/sso
use post/windows/gather/credentials/windows_autologin
use post/windows/gather/enum_services
use post/windows/gather/enum_unattend
use post/windows/gather/tcpnetstat
use post/windows/gather/lsa_secrets
use post/windows/gather/netlm_downgrade
use post/windows/gather/phish_windows_credentials
use post/windows/manage/wdigest_caching
```

### 3rd Party Applications

```
use post/windows/gather/enum_putty_saved_sessions
use post/windows/gather/credentials/teamviewer_passwords
use post/multi/gather/filezilla_client_cred
use post/windows/gather/credentials/filezilla_server
use post/windows/gather/credentials/steam
use post/windows/gather/credentials/vnc
use post/windows/gather/credentials/winscp
```

### Information Gathering

```
use post/windows/gather/arp_scanner
use post/windows/gather/bitcoin_jacker
use post/windows/gather/checkvm
use post/windows/gather/dnscache_dump
use post/windows/gather/enum_applications
use post/windows/gather/enum_av_excluded
use post/windows/gather/enum_hostfile
use post/windows/gather/enum_logged_on_users
use post/windows/gather/enum_ms_product_keys
use post/windows/gather/enum_patches
use post/windows/gather/enum_services
use post/windows/gather/enum_shares
use post/windows/gather/enum_termserv
use post/windows/gather/enum_trusted_locations
use post/windows/gather/outlook
use post/windows/gather/usb_history
use post/windows/recon/computer_browser_discovery
use post/windows/wlan/wlan_profile
```

### Enumeration Modules

```
use auxiliary/scanner/portscan/tcp
use auxiliary/gather/dns_enum
use auxiliary/server/ftp
use auxiliary/server/socks4
```

### Privilege Escalation

#### Recon

```
use post/multi/recon/local_exploit_suggester
```

```
use post/windows/escalate/droplnk
use post/windows/escalate/getsystem
use post/windows/escalate/golden_ticket
use post/windows/escalate/screen_unlock
use post/windows/escalate/unmarshal_cmd_exec
use exploit/windows/local/trusted_service_path
```

### Spy modules

```
use post/windows/capture/keylog_recorder
use post/windows/gather/screen_spy
use post/windows/manage/webcam
```

### Forensic Modules

```
use post/windows/gather/dumplinks
use post/windows/gather/enum_muicache
use post/windows/gather/file_from_raw_ntfs
use post/windows/gather/forensics/enum_drives
use post/windows/gather/forensics/imager
use post/windows/gather/forensics/recovery_files
```

### Generic

```
use post/windows/manage/killav
use post/windows/manage/download_exec
use post/windows/manage/enable_rdp
use post/windows/manage/exec_powershell
use post/windows/manage/inject_host
use post/windows/manage/migrate
use post/windows/manage/reflective_dll_inject
use post/windows/manage/rollback_defender_signatures
use post/windows/manage/vss_mount
```

## Browser Modules

### Browser modules (Firefox)

```
use post/firefox/gather/cookies
use post/firefox/gather/history
use post/firefox/gather/passwords
use post/firefox/manage/webcam_chat
use post/multi/gather/firefox_creds
```

### Browser modules (Chrome)

```
use post/multi/gather/chrome_cookies
use post/windows/gather/enum_chrome
```

### Browser modules (Internet Explorer)

```
use post/windows/gather/enum_ie
use post/windows/manage/ie_proxypac
```

### Browser modules (Multiple)

```
use post/windows/gather/forensics/browser_history
```

## Packet Capture

### Sniffer

```bash
use sniffer
sniffer_interfaces
sniffer_start <ID>
sniffer_dump <ID> /tmp/sniff.pcap
sniffer_stop <ID>
sniffer_release <ID>
```
