---
description: https://owasp.org/www-community/vulnerabilities/Unrestricted_File_Upload
---

# File Upload

### Description <a href="#description" id="description"></a>

Uploaded files represent a significant risk to applications. The first step in many attacks is to get some code to the system to be attacked. Then the attack only needs to find a way to get the code executed. Using a file upload helps the attacker accomplish the first step.

The consequences of unrestricted file upload can vary, including complete system takeover, an overloaded file system or database, forwarding attacks to back-end systems, client-side attacks, or simple defacement. It depends on what the application does with the uploaded file and especially where it is stored.

{% hint style="info" %}
The following uses the DVWA as a base for showcasing the file upload techniques. In these examples an image file is expected to be uploaded by the web application.
{% endhint %}

## Simple file upload

In rare instances a web application may allow an upload of a malicious file which can be used for code execution on the target web server. In the example below the web server is running PHP and allows file upload without any security restrictions.

Below the file 'webshell.php' was uploaded without any restrictions to the target web server.

![](<../../.gitbook/assets/image (1882) (1).png>)

However, turning the security level up and attempting this again produces some type of filter stopping us from upload a file that is not JPEG or PNG format.

![](<../../.gitbook/assets/image (1883).png>)

As such the following techniques will demonstrate how to bypass this.

## Null Byte

A null byte or `%00` calls for early termination of a string. Where a file such as `shell.php%00.png`will execute as `shell.php` but will be read as a `.png` file since this is technically what the file name ends in.

Below, the webshell.php file was renamed to webshell.php%00.png and as shown was successfully uploaded to the target web server.

![](<../../.gitbook/assets/image (1884).png>)

## MIME Type

The MIME type of a file can also be adjusted in an attempt to bypass file upload restrictions. Shown below is a request for a normal PHP file upload to the target web server.

As shown on line 22 the application type is set to 'application/x-php'. When attempting to upload webshell.php again we are restricted from uploading.

![](<../../.gitbook/assets/image (1885).png>)

Resending the request for upload webshell.php again however, this time the MIME type on line 22 is changed to 'Content-type: image/jpeg'. We see the request is successful.

![](<../../.gitbook/assets/image (1886).png>)

![](<../../.gitbook/assets/image (1887).png>)
