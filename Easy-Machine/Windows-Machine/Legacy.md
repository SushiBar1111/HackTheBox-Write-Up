# Legacy HackTheBox Easy Windows Machine Write up

![image](https://github.com/user-attachments/assets/012c83f6-ab6a-44f1-910a-686396802dd6)

## Vulnerability Found
1. Using Windows version for SMB server with known vulnerability


## Step to Re-produce

### Enumeration

### Network Enumeration

I used `nmap` to do a network enumeration.

Payload:

    nmap TARGET-IP -sV -sC -T5 -oN outputfile.txt
    
<div align="center">
  <img src="https://github.com/user-attachments/assets/66bbb2db-86b3-43cc-ac60-790e6722dc35">
</div>
<div align="center">
  Nmap output
</div>
</br>

From nmap output, I found that this network has a SMB server. Next, I ran another nmap scan with a script to find what vulnerability that this SMB could have.

Payload:

    nmap TARGET-IP --script=smb-vuln* -T5 -oN outputfile.txt
    
<div align="center">
  <img src="https://github.com/user-attachments/assets/b1d7f4ad-170a-4161-b531-1243028ea4d9">
</div>
<div align="center">
  Exploit found for the SMB server
</div>
</br>

Next, I searched the exploit in the `metasploit` and found it.

<div align="center">
  <img src="https://github.com/user-attachments/assets/a2a06a8c-5a38-40f9-922f-a0a135d38501">
</div>
<div align="center">
  Exploit in metasploit
</div>
</br>

### Exploitation

For exploitation it was very straight forward. I set up the LHOST and LPORT to my IP and PORT and RHOSTS and RPORT to target IP and 445. Next, I ran it and got the shell on the SMB server.

<div align="center">
  <img src="https://github.com/user-attachments/assets/4d9b58f8-70fa-475e-b7f7-f93b5b046585">
</div>
<div align="center">
  Shell on SMB server
</div>
</br>

### User.txt
To find the `user.txt`, I moved into `C:\Documents and Settings\john\Desktop` directory.

<div align="center">
  <img src="https://github.com/user-attachments/assets/ab233f1b-5a6c-419c-a4c4-2c36fa9d91c8">
</div>
<div align="center">
  user.txt
</div>
</br>

### root.txt
To find the `root.txt`, I moved into `C:\Documents and Settings\Administrator\Desktop` directory.

<div align="center">
  <img src="https://github.com/user-attachments/assets/45f79cb3-2b59-4a95-b29d-b31cd5bf194e">
</div>
<div align="center">
  root.txt
</div>
</br>
