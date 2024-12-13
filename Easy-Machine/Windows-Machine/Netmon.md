# Netmon HackTheBox Easy Windows Machine Write up

![image](https://github.com/user-attachments/assets/162d74bc-2036-4119-95e7-3aab39f0a8c1)

## Vulnerability Found
1. FTP anonymous login allowed
2. Using PRTG Version with known vulnerability

## Step to Re-produce

### Enumeration

### Network Enumeration

For network enumeration, I used `nmap` to found the running services in the network.

<div align="center">
  <img src="https://github.com/user-attachments/assets/f2e47518-2b38-414d-8a25-fc54df232ef2">
</div>
<div align="center">
  Nmap output
</div>
</br>

From the nmap output, I found out that the FTP server is allowing us to login as anonymous, which is a big security issue.

<div align="center">
  <img src="https://github.com/user-attachments/assets/2fa2cfc5-76f4-41b9-b7fc-7fbc676de9f4">
</div>
<div align="center">
  FTP anonymous login allowed
</div>
</br>

Other than anonymous login in FTP, I found out that this network serve HTTP service for PRTG Network Monitor (like the machine name netmon).

<div align="center">
  <img src="https://github.com/user-attachments/assets/3cfda3ab-16af-45ab-bfc5-9fe31c3c232f">
</div>
<div align="center">
  PRTG Network Monitor 
</div>
</br>

### Exploitation

After finding out about FTP allowing anonymous login, I tried that and got connected into the FTP server.

<div align="center">
  <img src="https://github.com/user-attachments/assets/71190d36-620b-463b-9b02-e867d1fe8869">
</div>
<div align="center">
  FTP Login Success
</div>
</br>

### user.txt

To find `user.txt`, it was very simple. I moved into `C:\Users\Public\` directory and found it there.

<div align="center">
  <img src="https://github.com/user-attachments/assets/c96d046d-41db-4ea7-bf2a-00b3fc074fae">
</div>
<div align="center">
  user.txt
</div>
</br>

I downloaded the file using `get` command and opened it in my local. Next, looking for root.txt. Firstly, I tried to moved into `C:\Users\Administrator\` directory but it was denied. So, I looked for another way to privilege escelation

### Privilege Escelation

After exploring the files in the FTP server, I found a configuration file for the PRTG Network Monitor. It was located in `C:\Users\All Users\Paessler\PRTG Network Monitor\`. To find the `All Users` directory, you need to use `ls -al` command to list all hidden directory. The configuration file that I've located was `PRTG Configuration.old.bak`. Next, I downloaded it to my local machine using `get`.

<div align="center">
  <img src="https://github.com/user-attachments/assets/fb85bcdd-9545-45e0-9e06-d3cc9c2dd717">
</div>
<div align="center">
  PRTG configuration file
</div>
</br>

After downloaded the file, I ran `grep` command

Command:

    `grep -C 10 password 'PRTG Configuration.old.bak` | less

I found the username and password.

<div align="center">
  <img src="https://github.com/user-attachments/assets/e64add8e-7763-4444-a56e-15441f403112">
</div>
<div align="center">
  username and password
</div>
</br>

Next, I tried to use it to log in into the PRTG web in port 80 but it was incorrect. With a lucky guess, I was thinking maybe it could use a pattern on the `2018`. So, I changed it into `2019` and it worked.

<div align="center">
  <img src="https://github.com/user-attachments/assets/10568f71-6d79-44de-9ff7-b6d057eb15f5">
</div>
<div align="center">
  PRTG Version
</div>
</br>

After logged in into the web, I found the version of the PRTG. Next, I looked on internet to find the exploit and I found this [PRTG Exploit](https://github.com/A1vinSmith/CVE-2018-9276). But, I didn't use it. I found it on metasploit too so I used from the metasploit one.

<div align="center">
  <img src="https://github.com/user-attachments/assets/84866ab7-f87e-4f20-97f5-d7339e96236d">
</div>
<div align="center">
  metasploit 
</div>
</br>

I set up the LHOST, LPORT, RHOSTS, ADMIN_PASSWORD, ADMIN_USERNAME and run it and got the shell.

<div align="center">
  <img src="https://github.com/user-attachments/assets/4fd78bcb-3eeb-4ca4-81e5-59d50fa0e0ca">
</div>
<div align="center">
  shell acquired 
</div>
</br>

Next, I moved into `C:\Users\Administrator\Desktop` directory to find the root.txt

<div align="center">
  <img src="https://github.com/user-attachments/assets/3cc23ee5-9017-439e-a0dc-22b74fe2ad08">
</div>
<div align="center">
  root.txt 
</div>
</br>


