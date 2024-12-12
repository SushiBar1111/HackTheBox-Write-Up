# Blue HackTheBox Easy Windows Machine Write Up

<div align="center">
  <img src="https://github.com/user-attachments/assets/f731495c-247f-4738-a221-221195040940">
</div>
</br>

## Vulnerability Found
1. Using Windows OS version with known vulnerability.


## Step to Re-produce

### Enumeration

### Network Enumeration

I used `nmap` to do a network enumeration and found several open TCP services.

Payload:

    nmap TARGET-IP -sV -sC -T5 -O -oN outputfile.txt

<div align="center">
  <img src="https://github.com/user-attachments/assets/b93178dd-c3b5-464a-8d53-fbba80f7ff46">
</div>
</br>
<div align="center">
  Nmap Output
</div>
</br>

From the output, I found that this network has open port on 139, 445 and 135 while the other are different port but same service as on port 135. On port 139 and 445 it was an SMB service. From the nmap script scan, I found out about the Windows version that were used, user's username, and another data.


### SMB Enumeration

Using `enum4linux`, I found out that I can connect to SMB using '' as username and '' as password.

<div align="center">
  <img src="https://github.com/user-attachments/assets/db6b91e0-383e-424a-b753-72963e242010">
</div>
</br>
<div align="center">
  enum4linux output
</div>
</br>

Next, I tried to look into `metasploit` to find any exploit for this SMB server. With a luck guess, I search **blue** in metasploit, command: `search  blue`, and found an exploit.

<div align="center">
  <img src="https://github.com/user-attachments/assets/98d32876-2a44-4a54-ad9c-3a9918fe76de">
</div>
</br>
<div align="center">
  SMB Exploit
</div>
</br>

### Exploitation

After finding out about the exploit, I set the payload that were required to run the exploit and got the shell on the SMB server.

<div align="center">
  <img src="https://github.com/user-attachments/assets/61339044-132e-4452-919b-7ea82e9c6622">
</div>
</br>
<div align="center">
  Shell on SMB server
</div>
</br>

To find `user.txt`, I moved into `C:\Users\haris\Desktop` directory.

<div align="center">
  <img src="https://github.com/user-attachments/assets/0dc35c12-8d60-4d56-a63e-0b1ceb789eb4">
</div>
</br>
<div align="center">
  user.txt
</div>
</br>

and to find `root.txt`, I moved into `C:\Users\Administrator\Desktop` directory.

<div align="center">
  <img src="https://github.com/user-attachments/assets/91615257-2294-4130-bf92-33a0002fe843">
</div>
</br>
<div align="center">
 root.txt
</div>
</br>
