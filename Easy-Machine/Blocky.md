# HackTheBox Blocky Easy Linux Machine Write Up

<div align="center">
  <img src=(https://github.com/user-attachments/assets/fc5e44d6-c3c4-4a7a-943d-d825e83f7e6f>
</div>
<br/><br/>

## Description
Blocky is an old very easy and straight forward linux machine released on 21 Jul 2017. Blocky is a machine that used same password to different services, and easy privilege escalation because the user is allowed to do all sudo. The main challenge for me is finding something useful in the directories.
Let's get started!

## Enumeration and Reconnaissance
While pentesting, it is a must to do enumeration. In this machine, i did port enumaration and directory enumaration.

### Port Scanning with Nmap
I used `nmap` to scan all open ports in the network.

Payload:

    nmap <target-ip> -p- -sV -sC -T5 -O -oN nmap1.txt

<div align="center">
  <img src=https://github.com/user-attachments/assets/ef87bb57-13ca-440f-83d0-42a6f17993bb>
</div>
<br/><br/>
<div align="center">nmap output</div
<br/><br/>

From the nmap output, I got 4 open ports, ftp on port 21, ssh on port 22, http on port 80, and minecraft on port 25565.

### Directory Enumeration
Because I knew that this network has http or web opened, I used `dirsearch` to do directory enumeration because its convenient. 

Payload:

    dirsearch -u <target-url> -o dirsearch1.txt

<div align="center">
  <img src=https://github.com/user-attachments/assets/d7f422fd-99cd-4637-8c5a-129acb7cec1d
</div>
<br/><br/>
<div align="center">dirsearch output</div>
<br/><br/>

From the output, I got many directories. I checked it one by one looking for something interesting. When I look into `/plugins/` directory, it has two .jar files that seems intersting. So, I downloaded both of them and check it. I check the **Blockycore.jar** first and found an sqluser and sqlpassword.

<div align="center">
  <img src=https://github.com/user-attachments/assets/34476ef0-4fe3-4fe0-9d25-344d95cca4ea>
</div>
<br/><br/>
<div align="center">Blockycore.jar</div>
<br/><br/>

## Exploitation
So, I got the sqluser `root` and sqlpassword `8YsqfCTnvxAUeduzjNSXe22` then tried it to login to `/phpmyadmin/` that I found earlier in directory enumeration. Using that credentials, I got in.
Then, I went looking around to find something. I found user `notch` in `wp_user`. 

<div align="center">
  <img src=https://github.com/user-attachments/assets/0acfde3b-afe6-43cf-acfa-166513087d8b>
</div>
<br/><br/>

Then I was thinking, maybe I can login as notch in **ssh** and use the password. But, I failed. I tried using the phpmyadmin password, and turns out, the password that notch's used was the sqlpass that I found earlier and I used to logged in to phpmyadmin. 

<div align="center">
  <img src=https://github.com/user-attachments/assets/67ac4bef-f8d9-4649-9c8c-445c1b84babd>
</div>
<br/><br/>

After finally logged in to ssh as notch, I read the user.txt

### Privilege Escelation
I tried using `sudo -l` command to see what is allowed to sudo as notch. Turns out, all of it. You can sudo all command as notch. Because of that, it's easy, I just need to `sudo su` then I got root.

![image](https://github.com/user-attachments/assets/72f2673b-736e-45a7-a1da-238f46bfc131)
