# HackTheBox Easy Linux Machine "Lame" Write Up

## Description
Lame is an easy Linux machine from HackTheBox.

## Enumeration

### Port Scanning with Nmap
I used `nmap` as port scanning tool to help me identify open ports in the target IP address. Nmap payload:

    'nmap <IP-TARGET> -p- -sC -T5 -O -oN <output-file>

<div align="center">
  <img src=https://github.com/user-attachments/assets/dba6d923-333e-4d23-8770-acc6a4116977>
</div>
<br/>

<div align="center">nmap output</div>
<br/>


## Exploitation

### Finding FTP exploit and Try It
From the nmap result, I got 5 ports open, FTP on port 21, SSH on port 22, SMB on port 139 and 445, and disstcd on 3632. I found out that anonymous login in ftp is allowed, but it didn't give me anything. So, since I knew the FTP's version, I looked for some possible exploit in the internet.
When I search for it, I found the exploit on Github: [vsFTPd_2.3.4_exploit](https://github.com/Hellsender01/vsftpd_2.3.4_Exploit). You can find it too on the exploitdb and metasploit but I used the one on that link because its straight forward and tells me how to use it. But, turns out the exploit didn't work. 

### Enumerating SMB service and Find the Exploit
So, I tried to find another way in. I knew that this machine has another services like SMB. I tried to enumerate it with `enum4linux` tool.

enum4linux payload:

    enum4linux -a <target-ip> // I used -a to scan all that can be scanned.

From the enum4linux scan, I found the Samba's version, which is **3.0.20**. Next, I looked for the exploit in `Metasploit` and I found it.

<div align="center">
<img src=https://github.com/user-attachments/assets/10ba9a7a-3361-49a1-be2d-69089dc3e23c>
</div>
<br/>

I chose that module and use it to exploit the SMB. Don't forget to set the LHOST, LPORT, RHOSTS, and RPORT. You can see it by typing `options` after using the module. To set it, you just type `set <what you wanna set>`.

Example:

    set LHOST <Your-VPN-IP>

After you set the configuration, you just type run and enter.

<div align="center">
  <img src=https://github.com/user-attachments/assets/203ac9b3-9c88-464c-bf95-d0b1a7f8ee19>
</div>
<br/>

I got the shell but its not a good shell so I ran a python script.

Python script:

    python -c 'import pty; pty.spawn("/bin/bash")'

<div align="center">
  <img src=https://github.com/user-attachments/assets/13bff275-31f3-4c33-815d-beda983e51c0>
</div>
<br/>

Just like you can see in the result, we are already root.

<div align="center">
  <img src=https://github.com/user-attachments/assets/a2a247d2-a71f-4c93-9d7b-bff2f6a9f068>
</div>
<br/>

### User.txt

I just needed to change directory to `/home` and found the user named **makis** and changed to that directory and read the user.txt.

<div align="center">
<img src=https://github.com/user-attachments/assets/f06136a1-2009-4385-873f-f2d784bf6a60>
  </div>

### Root.txt

Well, since I am a root, I just needed to change to `/root` directory and read the root.txt

<div align="center">
<img src=https://github.com/user-attachments/assets/8602c604-4285-4bb9-b89b-56d776cc722d>
  </div>


Thank you for reading!
