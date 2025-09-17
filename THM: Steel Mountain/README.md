# TryHackMe: Steel Mountain
https://tryhackme.com/room/steelmountain

## 1. Enumeration

### 1.1. port scanning
Scan the ports of the target machine
```sudo nmap -T 4 -p- -A X.X.X.X ```
<img width="723" height="795" alt="image" src="https://github.com/user-attachments/assets/b96f777b-a4b5-420b-ade9-fd6834f4ffe1" />
<img width="921" height="381" alt="image" src="https://github.com/user-attachments/assets/adb8ee0c-39ee-4856-8be0-3b484344b6d0" />

As observed in above screenshot, open ports discovered are 80, 135, 139, 445, 3389, 5985, 8080, 47001 and 49XXX.
The service HttpFileServer httpd 2.3. stands out in particular. Hence, it is worth to probe further.

### 1.2. web enumeration on Port 8080 (HTTP)

<img width="933" height="695" alt="image" src="https://github.com/user-attachments/assets/a25c9470-bea4-41aa-a1ed-e91febc9395c" />

The server information shows "HttpFileServer 2.3". A vulnerability exploit can be searched on this software version. 


## 2. Exploitation

### 2.1. researching vulnerability

Online researching *HttpFileServer 2.3* : 
- Found possible vulnerability CVE-2014-6287 (https://nvd.nist.gov/vuln/detail/CVE-2014-6287)

After launching Metasploit (msfconsole), searching *httpfileserver* :
<img width="1110" height="765" alt="image" src="https://github.com/user-attachments/assets/99997455-8128-42e3-9de4-ef2934a3a239" />

It is observed to be a potential exploit for CVE-2014-6287.


