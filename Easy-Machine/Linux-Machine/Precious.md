![Screenshot 2024-12-13 145928](https://github.com/user-attachments/assets/1891bb42-7943-4bb6-a6f6-e4bc40f892b2)# Precious HackThebox Easy Linux Machine Write up

![Screenshot 2024-12-13 161321](https://github.com/user-attachments/assets/bc3311ec-84c3-49b9-9fbe-8f50ae32984f)

## Vulnerability Found
1. Using PDFkit version with known vulnerability (CMD Injection).
2. YAML Deserialization for Root Privilege

## Step to Re-produce

### Enumeration

### Network Enumeration

First thing I did was network enumeration to find open services in the network. I used `nmap` for this.

Command:

    nmap TARGET-IP -sV -sC -T5 -O -oN outputfile.txt

<div align="center">
  <img src="https://github.com/user-attachments/assets/7fbe4c98-63f4-41a2-a82d-2fcabdc0b3da">
</div>
<div>
  Nmap output
</div>
</br>

From the nmap output, It shows that this network has two open services, SSH on port 22 and HTTP on port 80. Next, I search the web page on firefox.

### Web Enumeration

<div align="center">
  <img src="https://github.com/user-attachments/assets/e83c614d-0250-4ec4-bec2-0616155126eb">
</div>
<div>
  Web page
</div>
</br>

Turns out this web can generate a web page from a given URL and turn it out into a PDF file. At first, I tried to input URL like https://www.google.com, but it gave me a `Cannot load a remote URL!`. So, I tried to make my own web page with a simple html file. I only create a `<h1></h1>` header and save it on my local machine. Next, I ran a python http server and input my own http server into precious.htb and it gave me the PDF file.

Step: 
1. Create HTML file with a simple html code.
2. Run a python http server on your terminal
3. Input your http server URL  to precious.htb (it should be like this: **http://your-ip:your-port/your-file-directory**) 
4. Download the PDF.

<div align="center">
  <img src="https://github.com/user-attachments/assets/7a802d20-8950-4f88-ba04-478f64500092">
</div>
<div>
  PDF File
</div>
</br>

It should be like the picture above. Other than testing how the web works in making a PDF file, I found another things. I used `Burpsuite` to see how the response after I input a URL. 

<div align="center">
  <img src="https://github.com/user-attachments/assets/05501f26-6236-4d29-889d-c28fad9f5f00">
</div>
<div>
  Web Reponse Header
</div>
</br>

From the response header, I found out that this used `Phusion Passenger` addition for the server. On the `X-Runtime` it shows that this web built using `Ruby` language.

### PDF File Enumeration

After downloading the PDF file, I started to enumerate the metadata from this file to see if there any new information that could help me to find entry point to exploit. To enumerate this, I used `exiftool`.

Command:

    exiftool [FILENAME]

<div align="center">
  <img src="https://github.com/user-attachments/assets/3e105075-1c22-4382-afaf-afa06eb0ac4a">
</div>
<div>
  PDFkit Version
</div>
</br>

From the output of exiftool, I found out that this web generated the PDF using **pdfkit**. I searched up to see if there is any exploit for this pdfkit version and found this [CVE-2022-25765](https://github.com/nikn0laty/PDFkit-CMD-Injection-CVE-2022-25765).

### Exploitation

I copied the python exploit and made a python file on my terminal and followed the **README.md** from the PoC. Other than running the script, I ran `netcat` listener too based on the port that I used on the exploit payload.

<div align="center">
  <img src="https://github.com/user-attachments/assets/6864f31f-0722-45f6-aa74-49a23b4ba082">
</div>
<div>
  Running the script
</div>
</br>



<div align="center">
  <img src="https://github.com/user-attachments/assets/55039515-9275-4aa8-b389-f29b91e70810">
</div>
<div>
  Shell Acquired
</div>
</br>

### User.txt
After acquiring the shell, I moved into `/home/henry` directory and found out that I was denied to see the user.txt because I was still user `ruby` not `henry`. In the root directory from `ruby` user, I used `ls -la` to see hidden directory and found `.bundle` directory. From there I found config file and found the password for user `henry`.

<div align="center">
  <img src="https://github.com/user-attachments/assets/48162629-5d96-4aad-9e53-f351aa16709d">
</div>
<div>
  henry's password
</div>
</br>

Next, I ran `su henry` to change super user as henry using that password and moved into henry directory to find the user.txt.

<div align="center">
  <img src="https://github.com/user-attachments/assets/905777be-8643-44b9-a307-64b5745315b2">
</div>
<div>
  user.txt
</div>
</br>

### Privilege Escelation

To find the root.txt, we need to do a privilege escelation. After acquiring the user.txt, I tried to login into SSH with user `henry` and his password and it worked. I used `sudo -l` to see if henry is allowed to run sudo. 

<div align="center">
  <img src="https://github.com/user-attachments/assets/62d7a037-6654-46a1-b479-9d5eab7e423a">
</div>
<div>
  henry permission to run sudo
</div>
</br>

From the picture above, it appeared that henry is allowed to run sudo for `usr/bin/ruby /opt/update_dependencies.rb`. Next, I checked that ruby file and found this.

<div align="center">
  <img src="https://github.com/user-attachments/assets/8f91381c-b598-4fe0-b3c9-e5d144008603">
</div>
<div>
  update_dependencies.rb file
</div>
</br>

From reviewing the code, I knew that this file read a **.yml** file. But there is a catch. The **.yml** file (in this case the name is **dependencies.yml**) the location is not specified. The `update_dependencies.rb` file read from the current directory where user is executing the file (So, if you run the **update_dependencies.rb** file in **/home/henry** directory, it will search the **dependencies.yml** file in **/home/henry** directory). But, I still didn't know how to exploit this. So I looked up the ruby version that was used in this. I ran `ruby -v` and found the version.

<div align="center">
  <img src="https://github.com/user-attachments/assets/d35a6d6b-80f7-4d45-9086-fc631462c597">
</div>
<div>
  Ruby version
</div>
</br>

I found this exploit for the ruby version. It was YAML deserialization. Exploit: [YAML Deserialization](https://github.com/swisskyrepo/PayloadsAllTheThings/blob/master/Insecure%20Deserialization/Ruby.md). From what I understand with my newbie brain, if I used the YAML deserialization one, on the `git_set`, the ruby will run that command. So, in the case from the PoC, it will run `id` command and return the id of the user. To, prove this hypothesis, I tried it. I made a new `.yml` file with the file name same as what was in the `update_dependencies.rb` code, but the content is the exploit (You can copy paste the YML deserialization code from the github and paste it into your newly made .yml file).

<div align="center">
  <img src="https://github.com/user-attachments/assets/89634aaf-a429-442b-99b9-da6ceec18c17">
</div>
<div>
  exploit output
</div>
</br>

After running the `update_dependencies.rb` file, it returned a id for root which proved my hypothesis. So, next I used a reverse shell payload from [revshell](https://www.revshells.com/) and used the ruby #1. After creating the payload, I paste it into the `git_set` of the `.yml` file that I created earlier. The next step was the same as before, running sudo. But, this time I ran up a `netcat` listener to capture the reverse shell.

### Root.txt

<div align="center">
  <img src="https://github.com/user-attachments/assets/262869db-37a9-46e9-9859-af2011259bcc">
</div>
<div>
  Root shell acquired and Root.txt
</div>
</br>

After acquiring the root shell, next looking the root.txt in `/root` directory.

