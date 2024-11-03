# Write Up for "Cap" Easy Linux Machine From HackTheBox
![image](https://github.com/user-attachments/assets/bf290da1-bc26-4496-9621-a73f853fdffb)

## Description
Cap is an old easy Linux machine from HackTheBox. Vulnerabilities that can be found in this machine are IDOR and Linux capabilities to gain root access.

## Enumeration and Reconnaissance

### Port Scanning with Nmap
I used `nmap` to find the open ports with `-p- -T5 -sC -O -oN` tags.

<div align="center">
  <img src=https://github.com/user-attachments/assets/d6292694-6cbd-488a-8ed1-31d37bf67a52>
</div>
<br/><br/>

From the nmap result, I found out that this IP has 3 open ports, ftp on port 21, ssh on port 22, and http on port 80. I opened the webpage and found something interesting. I've logged in as user **nathan**. 

<div align="center">
  <img src=https://github.com/user-attachments/assets/9bf9e52c-199a-482a-a42f-ef849b0c1ce7>
</div>
<br/><br/>

I clicked on the security snapshot and notice on the url, it says `/data/1` that could possibly leads to IDOR that I can I exploit. Other than scanning open ports, I did directory enumration but found nothing intersting.

## Exploitation

### Exploiting IDOR
Since i know that this web potentially vulnerable to IDOR, I tried to changed the url from `/data/1` which is us to `/data/2` and see whether I can see another user's packet capture data.

<div align="center">
  <img src=https://github.com/user-attachments/assets/67495835-4002-4282-825e-4c2896201135>
</div>
<br/><br/>

It did! so I tried to look for another user's data. I tried to check `data/0`, download the PCAP file then open it with `wireshark`. From opening the file, I found the password for user **nathan** for **ftp server**

<div align="center">
  <img src=https://github.com/user-attachments/assets/534f5e2f-0b80-43bd-b492-c2f900567598>
</div>
<br/><br/>

Next, I used that user and the password to login to ftp server. After logged in, I found the user.txt and download it to my local computer. 

<div align="center">
  <img src=https://github.com/user-attachments/assets/cab1d478-3038-476a-bf3c-a7697c68d960>
</div>
<br/><br/>
After downloaded the user.txt, I read it and submit the user.txt.

![Screenshot 2024-11-03 185754](https://github.com/user-attachments/assets/76312147-2fbf-49ea-a86b-789d135f79d0)

### Privilege Escelation
After submitting the user.txt, I wondered if the password can be use too for the ssh. So, I tried it and it did used the same password. Because I already got the user.txt now I tried to use `linpeas` to find some vulnerablity.
First, I download the linpeas from my local computer to the ssh by starting http server on my local and downloaded it on ssh. After that, I ran it.

<div align="center">
  <img src=https://github.com/user-attachments/assets/3229fa5d-1547-471b-ae80-d1d41fb283f7>
</div>
<br/><br/>

Linpeas output shows that the python can run a SETUID which is can change our UID. So, I tried change my UID to 0, which is root and run a bash. First, I ran the python with `python3` command. Then, it will asked me to type some input, this is where I enter the payload below.

Payload:

    import os
    os.setuid(0)
    os.system("/bin/bash")

<div align="center">
  <img src=https://github.com/user-attachments/assets/4a80b803-cdae-4363-8fbd-84589723adb7>
</div>

Then I read the root.txt. Thank you for reading!

