<img width="253" height="19" alt="image" src="https://github.com/user-attachments/assets/32e0be08-14e3-4fe8-bf7d-c682a330139b" /># TryHackMe: UltraTecch
https://tryhackme.com/Room/ultratech1

## 1. Enumeration

### 1.1. port scanning
Scan the ports of the target machine
```sudo nmap -T 4 -p- -A X.X.X.X ``` <br>

<img width="1909" height="546" alt="image" src="https://github.com/user-attachments/assets/9230c1f0-25da-4ef3-bceb-cd62c42b052a" />

As observed in above screenshot, open ports discovered are 21 (FTP), 22 (SSH), 8081 (HTTP) and 31331 (HTTP).
8081 and 31331 are observed to be web servers, in which 8081 is to Express with Node.js.


Below shows the pages at 8081 which is an API. <br>
<img width="617" height="278" alt="image" src="https://github.com/user-attachments/assets/2902923d-68a4-4277-878b-f63b67dfe726" />
<br>
Below shows the pages at 31331 which is a web portal. <br> 
<img width="1258" height="699" alt="image" src="https://github.com/user-attachments/assets/d9705eb4-34e5-4f59-8501-89c115088a3f" />


### 1.2. subdirectory enumeration on Port 8081 and 31331 (HTTP)

Running the command: ```gobuster dir -u http://X.X.X.X:8081 -w /usr/share/wordlists/dirb/common.txt -t 5 -k```
<img width="811" height="370" alt="image" src="https://github.com/user-attachments/assets/24ac6bb8-c902-41af-9f69-d6f09b0aef9a" />
There are likely to be two API calls; auth and ping.

Running the command: ```gobuster dir -u http://X.X.X.X:31331 -w /usr/share/wordlists/dirb/common.txt -t 5 -k```
<img width="813" height="502" alt="image" src="https://github.com/user-attachments/assets/db8dabbb-f75e-46da-83ec-b03144ed1f87" />
robots.txt can be checked if there is any further information there.

## 2. Gaining initial foothold through user credentials

Navigating to *http://X.X.X.X:31331/robots.txt<br>*
<img width="916" height="208" alt="image" src="https://github.com/user-attachments/assets/c8c4926a-f02b-4c2c-a61c-3bf1d5b57094" />

From above screenshot, it is discovered that there is a sitemap at /utech_sitemap.txt. <br>

Then navigating to http://X.X.X.X:31331/utech_sitemap.txt: <br>
<img width="973" height="223" alt="image" src="https://github.com/user-attachments/assets/5fcbdc08-5342-4dbd-812b-3dd353de9ff9" />

From above screenshot, it is discovered that there is a subdirectory at /partners.html which was previously not known, hence it can be probed further. <br>
Then navigating to http://X.X.X.X:31331/partners.html: <br>
<img width="1734" height="868" alt="image" src="https://github.com/user-attachments/assets/5a877576-01d9-4774-a570-105e3251465c" />

From above screenshot, it appears that  http://X.X.X.X:31331/partners.html is a login page. <br>
NOTE: An attacker may also probe the login page for web vulnerabilites. <br>

In this case, the attacker view the page source to find vulnerabilities. <br>
<img width="1337" height="696" alt="image" src="https://github.com/user-attachments/assets/4a5ba1bd-19d8-44ee-9bb2-b6c116729033" /><br>

From above screenshot, it is observed that the login page uses embbeded Javascript code. It calls *js/api.js* which can be probed further as earlier we discovered it has an API server running.
At this stage, one can deduce that the web application running on port 31331 calls the API server running on port 8081 to perform certain functions.

<img width="927" height="640" alt="image" src="https://github.com/user-attachments/assets/c94e7744-7807-4917-8d5d-fef9578fae65" /> <br>
From the above screenshot, it is observed that the API call for the command *ping* is hardcoded into the Javascript code.

<img width="965" height="232" alt="image" src="https://github.com/user-attachments/assets/b9ae8d7a-ef37-473a-be20-a2c1b81e58b5" /> <br>
From the above screenshot, the API call for the ping command is tested. It returns the output of the ping command which appears to be from the output of running the command on a Linux CLI.

<img width="931" height="137" alt="image" src="https://github.com/user-attachments/assets/0f723d22-e03a-4f4b-9899-93916ef347cd" /> <br>
As per above screenshot, one can use the backticks for command substitution in Linux so that output of the command inside the backticks is executed first and then used as an argument for the outer command.
In this case, the inner command *whoami* returns the *www* username. This appears to have a local file inclusion (LFI) vulnerability.

<img width="957" height="178" alt="image" src="https://github.com/user-attachments/assets/4f0d4ab8-579a-4e78-994a-aabec0164475" /> <br>
From the above screenshot, the API call inner command *ls* is crafted to list files in directory. From the returned output, there is a file named *utech.db.sqlite*.

<img width="1171" height="133" alt="image" src="https://github.com/user-attachments/assets/729f2c16-234d-4955-a88b-f5fbb9adba2c" /><br>
From the above screenshot, the contents of the *utech.db.sqlite* file is examined to expose what looked like: two usernames r00t and admin, along with two MD5-hashed strings respectively.

From the below two screenshots, the site https://crackstation.net is used to cracked both hashes, which appears to be passwords <br>
<img width="1443" height="650" alt="image" src="https://github.com/user-attachments/assets/1e473a75-0255-4a31-bc9f-3b2124d0cb97" /><br>
<img width="1323" height="650" alt="image" src="https://github.com/user-attachments/assets/a817b99c-a489-41a3-8b67-2d6adcc435db" /> <br>
Therefore, it is likely that credentials are found for two users:
- r00t:n100906
- admin:mrsheafy

Since there is a SSH service running on the target server, the credentials username r00t and password n100906 are tested.
<img width="764" height="590" alt="image" src="https://github.com/user-attachments/assets/2a738601-5c97-4ab1-a770-fcbd47456296" /> <br>
As per above screenshot, SSH login was successful for username r00t and password n100906.


## 3. Post-Exploitation
### 3.1. post-exploitation enumeration

The LinPEAS (Linux Privilege Escalation Awesome Script) from https://github.com/peass-ng/PEASS-ng/tree/master/linPEAS can be used to automate the enumeration for vectors to escalate privileges.
As per below screenshot, it can be downloded to target machine as *linpeas.sh* and set permissions to execute. <br>
<img width="681" height="218" alt="image" src="https://github.com/user-attachments/assets/9fadcccf-8a3d-4874-b634-4e7a190f3dd4" /> <br>

Then run command ```./linpeas.sh``` to run the script. <br>

<img width="796" height="430" alt="image" src="https://github.com/user-attachments/assets/8572be93-650e-44ac-bca5-28de6841852d" /><br>
From the above screenshot, a finding that stands out as a high probability attack vector is that the user r00t is part of the docker group.

<img width="912" height="378" alt="image" src="https://github.com/user-attachments/assets/34de228e-617c-4281-be55-01239d2176e8" /> <br>
From the above screenshot, it is verified that the Docker application is present on the target machine.

### 3.2. privilege escalation using Docker

From below screenshot, an online search on https://gtfobins.github.io/gtfobins/docker/ shows a probable attack method using docker.
<img width="1312" height="499" alt="image" src="https://github.com/user-attachments/assets/97452b78-8650-43fb-a194-b1789b449b4e" /> <br>
NOTE: *alphine* is not used here, hence the method needs to be modified to *bash*

<img width="673" height="115" alt="image" src="https://github.com/user-attachments/assets/e0e36636-5f47-4bd3-b4df-391fb9f43c03" /> <br>
As per the above screensht, running the command: ```docker run -v /:/mnt --rm -it bash chroot /mnt sh``` allows to get to the root shell.
