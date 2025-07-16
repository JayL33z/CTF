# TryHackMe: Simple CTF
https://tryhackme.com/room/easyctf

## 1. Enumeration

### 1.1. port scanning
Scan the ports of the target machine
```sudo nmap -T 4 -p- -A X.X.X.X ```
<img width="1894" height="800" alt="image" src="https://github.com/user-attachments/assets/b055bfd1-1086-42af-8438-54e509c1e648" />
As observed in above screenshot, open ports discovered are 21 (FTP), 80 (HTTP) and 2222 (SSH).

### 1.2. web subdirectory enumeration on Port 80 (HTTP)
Using dirbuster on http://X.X.X.X:80, the subdirectory http://X.X.X.X:80/simple/index.php is discovered,
<img width="757" height="538" alt="image" src="https://github.com/user-attachments/assets/13722eed-2a1c-4c5b-a1e0-c9a04c74cb60" />

Navigating to site on web browser, we observed that ***CMS Made Simple version 2.2.8*** is used for the site.
<img width="1268" height="768" alt="image" src="https://github.com/user-attachments/assets/3ac64a86-3060-4a0f-98c1-aa37366940b1" />

## 2. Exploitation

### 2.1. researching vulnerability
Online researching *CMS Made Simple version 2.2.8* :
- Found possible vulnerability CVE-2019-9053 (https://nvd.nist.gov/vuln/detail/CVE-2019-9053) that allows to perform unauthenticated blind time-based SQL injection attack.
- Found a publicly available exploit (https://github.com/Mahamedm/CVE-2019-9053-Exploit-Python-3) that allows to extract data from the database by monitoring delays of the responses.

### 2.2. exploiting vulnerability
Hence we downloaded the exploit (python script) and execute it against the target.

<img width="599" height="109" alt="image" src="https://github.com/user-attachments/assets/75e0cb90-6fb5-4e29-b54e-15fc73f928a7" />
<img width="554" height="90" alt="image" src="https://github.com/user-attachments/assets/a7ddabd1-6718-4282-b93f-c60e0746740b" />

As per above screenshot, the exploit extracted data from the database such as:
- Salt for password
- Username named *mitch*
- Email address *admin@admin.com*
- Password Hash (MD5)

### 2.3. cracking password
With password hash (MD5) and its salt, further online research is done (https://hashcat.net/wiki/doku.php?id=example_hashes) to determine that it can be cracked via ```hashcat -m 20``` command.
<img width="667" height="523" alt="image" src="https://github.com/user-attachments/assets/bd2f5aad-6815-4f7b-b1a7-95b195913d63" />

As per above screenshot, using the list */usr/share/wordlists/rockyou.txt*, the cracked password is *secret* 


### 2.4. gaining initial foothold
With username *mitch* and password *secret*, login via SSH on port 2222 is tested using these known credentials to log in and gain a foothold on the target machine.
```ssh mitch@X.X.X.X -p 2222```
Login via SSH is successful.

## 3. Post-Exploitation
### 3.1. post-exploitation enumeration
Running ```sudo -l ``` shows commands which user *mitch* can run with root permissions
<img width="733" height="364" alt="image" src="https://github.com/user-attachments/assets/e6f8b01d-32f6-46b3-a056-bf58475e37ae" />
As per above screenshot, it is discovered that user *mitch* can run */usr/bin/vim* which is a text-editor.

### 3.2. privilege escalation using sudo permission
Online researching on exploiting sudo permissions for vim (https://gtfobins.github.io/gtfobins/vim/) found some commands which can be used to exploit vim with sudo permissions for privilege escalation.

<img width="924" height="398" alt="image" src="https://github.com/user-attachments/assets/df56945e-7c78-4fd0-aafe-c88e732ca76c" />


Using the command ``` sudo vim -c ':!/bin/sh' ```, root shell was obtained as per below screenshot

<img width="467" height="227" alt="image" src="https://github.com/user-attachments/assets/6f95e8b7-5a9c-4fdf-93bb-5d8491285ebc" />

