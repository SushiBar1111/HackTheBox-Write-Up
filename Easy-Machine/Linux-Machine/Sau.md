![Screenshot 2024-11-20 003625](https://github.com/user-attachments/assets/1c17e219-f68f-4f49-a4b5-22a0cf45fe83)# Sau, Easy Linux Machine from HackTheBox Write up

![image](https://github.com/user-attachments/assets/67356553-be73-4a83-a3c2-f14520741217)

## Machine Description 

`Sau` is an Easy Difficulty Linux machine that features a `Request Baskets` instance that is vulnerable to Server-Side Request Forgery (SSRF) via `[CVE-2023-27163](https://nvd.nist.gov/vuln/detail/CVE-2023-27163)`. Leveraging the vulnerability we are to gain access to a `Maltrail` instance that is vulnerable to Unauthenticated OS Command Injection, which allows us to gain a reverse shell on the machine as `puma`. A `sudo` misconfiguration is then exploited to gain a `root` shell. 

## Information Gathering

### Network Enumeration

For network enumeration, I used `nmap` as the tool. 

<div align="center">
  <img src=https://github.com/user-attachments/assets/917778e1-5607-4c69-88e2-56eb96284599>
</div>

<div align="center">
  Nmap output
</div>
</br>

From the nmap output, I found 4 services. Two of four (port 80 and 8338) is filtered, likely by a firewall. While port 22 and port 55555 is open. 

### Web Information Gathering

Next, I visited the url with port 55555. It returned with a web page. Here, is the page:

<div align="center">
  <img src="https://github.com/user-attachments/assets/ec119c61-cfa2-4c8d-8057-e0a0ab78783f">
</div>

<div align="center">
  Port 55555 Page
</div>
</br>

At the bottom of the page, I found that this service is powered by **request-baskets** with version **1.2.1**. Next, I searched for known exploit and found this:
(https://github.com/entr0pie/CVE-2023-27163)[CVE-2023-27163]

From what I found, it seems that this service is vulnerable to SSRF. When I tried to use that exploit, it didn't work. I don't know whether i did something wrong or in this machine we can't use that PoC.

This service allows us to create a request basket where we can find the request that are send by or to the basket. After I created the basket, I noticed I can add a forwarded URL.

<div align="center">
  <img src="https://github.com/user-attachments/assets/0d72cfbd-15f7-4eb8-bab6-48a05414b337">
</div>

<div align="center">
  Basket setting for Forwaded URL
</div>
</br>

## Exploitation

### Escalating as User
So, after I found that I can add a URL, I tried to add my local IP and port. I added `http://my-local-IP:my-open-port` to the forwarded URL. Next, I set up `netcat` to listen to any request sent to the basket and use `curl` to send a request to the basket (You can do it by searching the basket's url in the web).

curl request: 

    curl -X GET http://your-machine-IP:machineport/your-basket-name -H "Content-Type: application/json"

<div align="center">
  <img src="https://github.com/user-attachments/assets/5d8f5531-6dbd-4ea9-90a0-a1cfc9c1013f">
</div>

<div align="center">
  Successful request received by added forwarded URL
</div>
</br>

I knew that I got the request after adding my URL as the forwarded URL and used netcat. So, I tried to change the forwarded URL to the web's local server, which is `127.0.0.1` (Don't forget to add the `http://`).

<div align="center">
  <img src="https://github.com/user-attachments/assets/e2b0366d-f10b-419e-b31f-ef8df2f1ff31">
</div>

<div align="center">
  Changing the forwarded URL
</div>
</br>

Next, I tried to search the basket's url in web browser and it redirected me to this:

<div align="center">
  <img src="https://github.com/user-attachments/assets/26337fa4-8b23-44b3-b5bd-513ce7ec3767">
</div>

<div align="center">
  Maltrail page.
</div>
</br>

I found the used Maltrail's version

<div align="center">
  <img src="https://github.com/user-attachments/assets/6108cd98-6185-4b8d-9fd2-2933fe344fbc">
</div>

<div align="center">
  Maltrail version
</div>
</br>

Next, I tried to search if there are any exploit for this version. I stumbled upon this: (https://github.com/spookier/Maltrail-v0.53-Exploit)[Maltrail-v0.53].

I ran a `netcat` listener and ran the exploit following the readme.

<div align="center">
  <img src="![image](https://github.com/user-attachments/assets/e43eb3d2-e8e7-4013-90c8-648a212082b8">
</div>

<div align="center">
  Connect to the Maltrail server
</div>
</br>

<div align="center">
  <img src="![image](https://github.com/user-attachments/assets/5789883c-2a0f-4689-82a7-8294b6e2447c">
</div>

<div align="center">
  user.txt
</div>
</br>

### Privilege Escelation

Next, to do a privilege escelation, I ran a `sudo -l` to see what user `puma` can run with sudo.

<div align="center">
  <img src="![image](https://github.com/user-attachments/assets/8ddc9d0e-5ef3-446d-97fb-d7f3fc7480b2">
</div>

Turns out, user `puma` may run 

    /usr/bin/systemctl status trail.service

I looked up to find what version this systemctl run on with this command `systemctl --version`

<div align="center">
  <img src="![image](https://github.com/user-attachments/assets/4f5a5de6-59de-4689-869d-441d1108ae68">
</div>
</br>

<div align="center">
  systemctl version
</div>

After finding the version, I searched for the exploit and found this (https://packetstormsecurity.com/files/174130/systemd-246-Local-Root-Privilege-Escalation.html)[systemctl privilege escelation]

Following the PoC that provided in that link, I successfully escalate my privilege as root. Next, I found the root.txt on the root directory

<div align="center">
  <img src="https://github.com/user-attachments/assets/35816c1d-1d4c-4977-8829-937985332d01">
</div>

<div align="center">
  root.txt
</div>
