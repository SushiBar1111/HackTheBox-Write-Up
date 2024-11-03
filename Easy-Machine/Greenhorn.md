# Write up for HackTheBox Greenhorn machine!
<div align="center">
  <img src=https://github.com/user-attachments/assets/39a34900-15d2-43bd-aa5a-da1dcf0be8e5>
</div>
Greenhorn is a Linux machine from hackthebox that we can try to pwn to sharpen our penetration testing skill. Like the other machine, our objectives are looking for user.txt and root.txt. You can start the machine and get the IP.
Don't forget to add the machine IP and domain to your `/etc/hosts/`. Let's start!

## **Reconnaissance**
First thing to do in penetration testing is doing reconnaissance or recon, so we can understand what services running in the network. To do this, i used `nmap`, a recon tool that used for mapping a network and finding what service that running on a network.
I used a few tags that provided by nmap. Here is the full command `nmap <machine-ip> -sC -p- -T5 -O -oN nmap1.txt`. -sC to run a script, -p- means search in all ports, -T5 is the speed, -O find the OS version, and -oN to create an output file.

After running nmap, i found 3 services that are running in 10.10.11.25 (machine ip).

<div align="center">
  <img src=https://github.com/user-attachments/assets/d28e5112-10b5-4792-af3a-35ab81df6812> 
</div>
<br/>
<div align="center">
(nmap output)
</div>
<br/>

From the nmap output, i knew that ssh and http are running. But, the intersting one is the service that running in port 3000, but let me take a look of the http service.
<br/>
<div align="center">
  <img src=https://github.com/user-attachments/assets/fe97cbcf-acbf-4198-b8d0-746de60b0794>
</div>
<br/>
<div align="center">
(HTTP page)
</div>
<br/>

Okay, that is how the page looks like. From this, i know that this website use Pluck CMS. I tried to view the page source and found the pluck's version, which is **4.7.18**.

<div align="center">
  <img src=https://github.com/user-attachments/assets/3c02a83a-de07-4703-8aa5-c0b554d09832> 
</div>
<br/>

<div align="center">
(Pluck CMS Version)
</div>
<br/>

Next, when i saw the url, i thought maybe i can get a file from the server by using the url `http://greenhorn.htb/?file=<file-name>`. I tried to change the file name to /etc/passwd/ to see if this web is vulnerable to this attack. This is the result that i got:
<br/>
<div align="center">
  <img src=https://github.com/user-attachments/assets/0fb4288e-bf66-415e-8b36-dc408fd4387f>
</div>
<br/>

<div align="center">
(failed attempt LFI)
</div>
<br/>

So the website using some kind of security measure that prevent me to do that attack. Okay, i need to find another vulnerability.

I used `dirsearch`, a recon tool to find any directory that the web's have.

<div align="center">
  <img src=https://github.com/user-attachments/assets/39b28a01-d973-4c59-a996-1ec50b4c2737>
</div>
<br/>

<div align="center">
<img src=https://github.com/user-attachments/assets/be253cea-9251-484a-9584-4c6615b00be4> 
</div>
<br/>
<div align="center">
(dirsearch output)
</div>
<br/>

I found a few directories, but i only take two that i think i can find a vulnerability, login.php and admin.php. When i tried to access admin.php, it needs me to logged in first (of course).

Because i knew this pluck's version, i searched the exploit for this. I found from the CVE database, few github repository, but what i used is this one: [CVE-2023-50564_Pluck-v4.7.18_PoC](https://github.com/Rai2en/CVE-2023-50564_Pluck-v4.7.18_PoC). It has the poc.py and shell.rar. This is a RCE vulnerability where we can upload a file containing php reverse shell and then we run that shell and we can get the shell of the website and run some command there.

You can read the README.md to understand how to use this exploit, i will not explain it here because the README is easy to understand.

Next, we look for the contents of poc.py. It needs us to change the web login url, web admin to install a module url, password, username, and where the RCE will be located after you upload them. Well, its very straight forward for me because the user who created this exploit already tell us where to change (in the `<url>` place).
But the problem for now is, i didn't know the password yet. This when i remembered something. Other than ssh and http, there is other service that running in port 3000. I tried to access it by changing the port from 80 to 3000, and it direct me to this.

<div align="center">
  <img src=https://github.com/user-attachments/assets/3e04feb0-4a67-4311-9bac-20cd93285ecf>
</div>
<br/>

<div align="center">
(Greenhorn:3000)
</div>
<br/>

It looks like greenhorn is some kind of repository like github. I explored that web and find the user.

<div align="center">
  <img src=https://github.com/user-attachments/assets/6520dd32-449a-4167-a7f4-7bad820b3a0c>
</div>
<br/>
<div align="center">
(greenhorn:3000 user)
</div>
<br/>

I clicked the user and find out the data in the repository

<div align="center">
<img src=https://github.com/user-attachments/assets/a5cb55db-16aa-4f4b-ae36-11286d71ec5a>
</div>
<br/>

Next, i tried to find the password. You can explore it yourself, but i found the password in `/data/settings/pass.php`. 

<div align="center">
  <img src=https://github.com/user-attachments/assets/255750f3-c62b-4416-a184-71fef96e8b21>
</div>
<br/>

Next, copy paste this password to [dcode](https://www.dcode.fr/cipher-identifier) to identify what encryption is this, and it said SHA-512. Use decoder online and then i found the password, which is **iloveyou1**. Okay recon is done!.

## **Exploit**

To run this exploit, you need to extract the **shell.rar** file that provided in the CVE's github (but it's not neccessary, you can create your own reverse shell file). I change the IP and port to mine. Next, save it and zip it using `zip <file-name> <fileoutput-name>`.
Next, i start netcat to listen into port that i used in the shell.php before. The command are `nc -lvnp <your-port>`. After starting the netcat listener, i run the poc.py. It will ask you the path of the zip file you created before.

<div align="center">
  <img src=https://github.com/user-attachments/assets/f521aeb1-1a21-4113-854a-109d48688cdb>
</div>
<br/>

<div align="center">
(running exploit).
</div>
<br/>

While i ran the exploit, my netcat successfully connected and i got the shell. I went straight to `/home` directory because that is usually where user.txt located. I knew from this too that the user is junior 

<div align="center">
<img src=https://github.com/user-attachments/assets/57a7b7ef-d3bd-42c9-975f-df7d88b2f5db>
</div>
<br/>

<div align="center">
(home directory)
</div>
<br/>

Next, i changed to junior using `su junior` with the password that i've obtained before and read the user.txt.

<div align="center">
<img src=https://github.com/user-attachments/assets/a42586e1-1c82-407b-bcc8-6c3c25c84a25>
</div>
<br/>


## **Privileged Escelation**

To read the root.txt, i need to perform a privileged escelation. su root is denied, so i tried to find another way. What intersting was, in junior directory, there is another file. I went straight to download it to my computer and access it.

<div align="center">
  <img src=https://github.com/user-attachments/assets/4c74bbf5-6a42-489f-a667-a85abee7fe58>
</div>
<br/>
<div align="center">
(openvas.pdf)
</div>
<br/>

Turns out, it was some note that contain a root password but it was pixelated. To be honest, i was stuck here for like 30 minutes and then i gave up and search for another write up, i found the write up from Iam Gh0st in medium (thank you for Iam Gh0st for the write up you helped me. You can check it if you want with this link: [medium](https://medium.com/@iam.gh0s700/hackthebox-greenhorn-95245df0a39d).

It turns out, i need to use a tool to un-pixelated the image using Depix. Here is the tool's repository: [Depix](https://github.com/spipm/Depix). 

I seperated the image from the pdf using online tool and run the Depix tool following the guide. 
Eventually, i got the password:

<div align="center">
  <img src=https://github.com/user-attachments/assets/a4a67b8e-441d-43b0-beda-dd4fab36f975>
</div>
<br/>

<div align="center">
(root password)
</div>
<br/>

Next, i ran `su root` command and enter the password and got the root privileged. Next, read the root.txt.

<div align="center">
  <img src=https://github.com/user-attachments/assets/670f7a4e-8a8b-4e5b-8cd4-6d04c8e98174>
</div>


It's done. Thank you for reading my write up!








