# Amnesiac

### **Description**

This module automatically starts Amnesiac C2 in a seperate process on the attacking system. PsMapExec will then execute the appropriate payload on specified remote systems in order to establish a persistent connection back to the Amnesiac console window.

Once a session has been established on the required remote systems, it is highly recommended to consult the Amnesiac documentation to aid in post-exploitation.

Github: [https://github.com/Leo4j/Amnesiac](https://github.com/Leo4j/Amnesiac)

Documentation: [https://leo4j.gitbook.io/amnesiac/get-started/quick-start](https://leo4j.gitbook.io/amnesiac/get-started/quick-start)

### **Supported Methods**

* MSSQL  (Requires Fix)
* SMB&#x20;
* SessionHunter (WMI)
* WMI&#x20;
* WinRM

### Optional Parameters

<table><thead><tr><th width="180">Parameter</th><th width="127.33333333333331">Value</th><th>Description</th></tr></thead><tbody><tr><td>-Scramble</td><td>N/A</td><td>Scrambles the pipe name to a alternate value</td></tr><tr><td>-SuccessOnly</td><td>N/A</td><td>Display only successful results</td></tr></tbody></table>

### Usage

{% code overflow="wrap" %}
```powershell
# Standard execution
PsMapExec -targets [All] -Method [Method] -Module Amnesiac
```
{% endcode %}

<figure><img src="../../.gitbook/assets/image (2114).png" alt=""><figcaption></figcaption></figure>
