# Write Up for Linux Machine DEVVORTEX From HackTheBox!

![image](https://github.com/user-attachments/assets/4dd5af75-f9a8-499e-a1e8-31b9dc296b8e)

## Machine Description
Devvortex is an easy difficulty Linux machine from HackTheBox. The end goals of this machine is for us to get user privilege and read the user.txt and do privilege escelation and read the root.txt.
Devvortex is a retired machine, which means you can follow the guided mode to solve this machine. But, I challenged my self to try solve this without guided mode. First you need to start the machine. Let's go!

## Enumeration & Reconnaissance 
Enumeration is the first thing that we need to do while doing penetration testing, in this case, we want to infiltrate to devvortex. So, we need to know about this machine and looking for entry point.

### Port scanning with Nmap
The goal to do port scanning is to find which ports are open in the target IP address/network. I used `nmap` to do port scanning because i'm familiar with it.

![image](https://github.com/user-attachments/assets/6f6f6bb7-1c21-4f07-9397-d97ac44a8778)

                                    (nmap)

The result from the nmap, tells me that there are 2 open ports, **port 22 for ssh**, and **port 80 for http**, and one port which is filtered. So, from this i knew that devvortex has a web page and browse that on my browser.

### Subdomain Enmuration 
Other than finding open ports, sometime we need to find another sub-domain that a web has. I used `ffuf` to try finding the subdomain of devvortex.htb. Here is my payload: `ffuf -c -w /usr/share/wordlists/amass/subdomains-top1mil-5000.txt -u http://FUZZ.devvortex.htb -o subdomain1.txt -p 2`

![image](https://github.com/user-attachments/assets/c3b492fc-f091-4b71-a594-540f36f20248)

                                  (subdomain)

From the output, i found that devvortex has another subdomain named **dev**. Maybe this is where the entry point for us.

### Directory Enumeration
Because i've found the subdomain of devvortex, i tried to do another enumeration, which is directory enumeration, to find maybe there are some critical directory that are allowed for normal user. I used `dirsearch` to help me find the directories.
Payload: > dirsearch -u http://dev.devvortex.htb -o dirsearch1.txt

**Dirsearch output**

![image](https://github.com/user-attachments/assets/9855a7a3-439c-40ac-9ff4-2a3ba50d68ff)

I found interesting directory that normal user are allowed to enter. When i enter the url, turns out that was a page to login to Joomla, a CMS, that are used by devvortex.

Here is the page view:

![image](https://github.com/user-attachments/assets/6573cd78-0ef0-43a4-bf83-a22131733c16)

                                  (Joomla login page)

Now, i wanted to find the Joomla version, but it was not there in source page. I look for ways to view Joomla version in Google and i found it. Turns out you can view the Joomla version if you know the directory.

Viewing Joomla version: > http://www.[thejoomlawebsite].com/administrator/manifests/files/joomla.xml.

Now change the the url to domain to dev.devvortex.htb. Here is the version of devvortex's Joomla:

![image](https://github.com/user-attachments/assets/67f3a279-c622-41f8-a89f-f69c4999bf42)

                                (Joomla version)

Now, I searched for exploit for Joomla with that version and found it here: [https://vulncheck.com/blog/joomla-for-rce]. The exploit is we can curl specific directory from the webpage and make it public so it leaks to us (something like that, sorry if i wrong).

## Exploitation

### Getting User Privilege
Now, after i've found some data like from doing enumeration and found out that devvortex use Joomla as its CMS and the version is vulnerable and i found the login page for administrator, next thing is try to exploit that vulnerable.

From the web link above, there are a 2 ways to exploit the vulnerability. I used the first one, when i tried the second one, it only gave me the user, user type, email, username not with the password.

Payload: > curl -v http://dev.devvortex.htb/api/index.php/v1/config/application?public=true

Result:

![Screenshot 2024-11-02 152825](https://github.com/user-attachments/assets/fe3f4e00-ae69-44d7-afd4-9165d38c76d4)

                            (user lewis and password)

I found a username and the password and tried it to login to administrator page. After successfully logged in, i was kinda clueless what to do next. So, i read again the website that explain about the vulnerability that i mention before (because i didn't read it fully, my bad). I found that we can modify a template to execute arbitrary code and php reverse shell.


![image](https://github.com/user-attachments/assets/065a13e7-c327-4626-8b55-d56ade97a148)

Next, i go to settings and choose one of the templates. I chose Site Template -> cassiopeia -> error.php file. I change the content of it to php pentestmonkey that i got from [revshells.com] and save the file. Next, i used netcat to listen and curl to sending a request to error.php so that the web will run it.

Curl payload: > curl -k http://dev.devvortex.htb/templates/cassiopeia/error.php

![image](https://github.com/user-attachments/assets/450dce9b-59fa-488c-9f5d-06c811fee2c5)

After successful connect to the web server, I change directory to /home and found that logan is the only user.

![image](https://github.com/user-attachments/assets/3175333a-3872-491f-bb8c-b74bea535c86)

![image](https://github.com/user-attachments/assets/c714cc63-61ac-425a-8254-dd4615d95723)

logan has the user.txt but i cannot read it because i am not logan, i am still www-data. I read from the explanation website again and see that Joomla used mysql. So, i tried to connect to mysql with user lewis and his password and look if i could find logan's password in the database.

mysql command: > mysql -u lewis -p 

![image](https://github.com/user-attachments/assets/9d1cea69-2a2d-4a4f-9778-ad8fff0f125b)

                                  (mysql databases)

So the database that interesting is joomla of course. Then, i used it and list all the tables.

![image](https://github.com/user-attachments/assets/3fdca95f-49ab-4e95-92a5-2a9024150969)

                                  (sd4fg_users)

sdf4g_users table is one of the interesting one so i select that table and view all the value. I found logan's encrypted password.

![Screenshot 2024-11-02 163748](https://github.com/user-attachments/assets/ec17184f-c6cd-48d0-9033-a13c905cdddd)

                                  (logan's encrypted password)

Next, i used online cipher identifier, [https://www.dcode.fr/cipher-identifier], and found out it was blowfish. The only way to crack this is by brute-force it.
I used `JohnTheRipper` to crack it with rockyou.txt wordlist. 

![image](https://github.com/user-attachments/assets/84c332d9-ff98-43db-a8bc-8608d690d617)

                                  (logan's password decrypted)

Next, su as logan in the shell earlier and read the user.txt


![image](https://github.com/user-attachments/assets/61fa3bf7-7f28-4999-8503-cd65123d3e7f)


### Privilege Escelation

After finding the user.txt, i changed to connect to ssh using logan's credentials. I ran `sudo -l` to find if logan is allowed to run something. Turns out, he did.

![image](https://github.com/user-attachments/assets/f964a301-298d-484b-bea0-0c72cba8cdb9)

                                (logan's permission to run)

Searching exploit for `/usr/bin/apport-cli`, I found the PoC from github, [https://github.com/diego-tella/CVE-2023-1326-PoC]. If i am not wrong understanding that, basically we can run `sudo /usr/bin/apport-cli` to run some .crash file and get root privilege from that. So, I tried it. But, i didn't follow the PoC because i didn't have the crash file and when i make a new one i don't know it wont work.
So, I read another write up, [https://medium.com/@aniketdas07770/hackthebox-devvortex-writeup-b6fa1f007dff]. From that write up, he/she used --file-bug so i followed it and gain root access.

![Screenshot 2024-11-02 165839](https://github.com/user-attachments/assets/ab2863f2-e3cb-4154-96e4-9242fb908e78)

                                          (root.txt)

Thank you for **Aniket Das** for the write up you help me did the privilege escelation when i couldn't figure out why i can't do the exploit following the PoC.

Okay, that's all. Thank you for reading!.











