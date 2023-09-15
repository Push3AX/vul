# Information

**Affected product:** Vitogate 300

**Vendor of the product:** Viessmann

**Product page:** https://connectivity.viessmann.com/gb/mp-fp/vitogate/vitogate-300-bn-mb.html

**Affected system version: **2.1.3.0 and lower

**Firmware download address:** https://connectivity.viessmann.com/gb/mp-fp/vitogate/vitogate-300-bn-mb/release-historie.html

**Reported by:**  Wentao Yang (pushe4x@gmail.com)



# Description

Vitogate 300 is a gateway for connecting the Viessmann LON to the BACnet or Modbus. Two hardcoded administrative accounts that hardcoded in /cgi-bin/vitogate.cgi allows full access to the web management interface. One of them is a hidden account, absent from the backend account management and immune to removal or password modification.

# Vulnerability details

The isValidUser() function in the `/ugw/httpd/html/cgi-bin/vitogate.cgi` binary is utilised to authenticate credentials during user sign-in.

![image-20230916033726118](https://raw.githubusercontent.com/Push3AX/vul/main/pic/image-20230916033726118.png)

The function contains two hardcoded administrative accounts: "vitomaster" and "vitogate", with passwords "viessmann1917" and "viessmann". 

Notably, "vitomaster" is a hidden account, absent from the backend account management and immune to removal or password modification.

![image-20230916034449441](https://raw.githubusercontent.com/Push3AX/vul/main/pic/image-20230916034449441.png)

# POC:

Vulnerable Vitogate 300 device incorporates the following hardcoded administrative accounts:

1. Username: vitomaster, Password: viessmann1917
2. Username: vitogate, Password: viessmann
