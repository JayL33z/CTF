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

exploit/windows/http/rejectto_hfs_exec is observed to be a potential exploit for CVE-2014-6287.

### 2.2. exploiting vulnerability
Hence use this exploit and then set the options accordingly to the target and attacker machine

<img width="1364" height="701" alt="image" src="https://github.com/user-attachments/assets/c78504f6-4d88-496e-a2e4-0f517e4bc82d" />

Then run the exploit to get a shell under the context of the username STEELMOUNTAIN\bill

<img width="912" height="335" alt="image" src="https://github.com/user-attachments/assets/5d143abd-e714-4e98-abb8-64d59cdf6b87" />


## 3. Post-Exploitation
### 3.1. post-exploitation enumeration

Enumerate the system using ```systeminfo | findstr /B /C:"OS Name"  /C:"OS Version" /C:"System Type```
<img width="1233" height="100" alt="image" src="https://github.com/user-attachments/assets/4e580995-d4af-4066-8853-682cfffa1e64" />
Based on the above screenshot, it is a x64-based architecture.

Amongst other areas of enumeration, one potential vector for privilege escalation is to exploit unquoted service path (Reference: https://medium.com/@SumitVerma101/windows-privilege-escalation-part-1-unquoted-service-path-c7a011a8d8ae)
This can be enumerated by the command ``` wmic service get name,pathname | findstr /i /v "C:\Windows\\" | findstr /i /v """ ```
<img width="1197" height="794" alt="image" src="https://github.com/user-attachments/assets/a3edf7c6-5c06-495f-a824-5bd4c608035c" />

The service named "AdvancedSystemCareService9" is further enumerated to check if privilege escalation via unquoted service path can be done. 

Under the current username STEELMOUNTAIN\bill, there are two key requirements:
a) STEELMOUNTAIN\bill must be able to write in a subdirectory of the service image path "C:\Program Files (x86)\IOBit\Advanced SystemCare\ASCService.exe", where there is a space within that breaks the full service path.
b) STEELMOUNTAIN\bill must be able to start/stop the service "AdvancedSystemCareService9"

For (a) using icacls to check permissions from this image path in top-down manner, it is observed that the directory "C:\Program Files (x86)\IOBit\" has permissions for STEELMOUNTAIN\bill to write into it. 
<img width="972" height="532" alt="image" src="https://github.com/user-attachments/assets/bdbbdf71-2c3c-46a2-b6d5-e3c22b5fd34a" />
Therefore, due to the unquoted string and space, the filename which be executed will append .exe at the end of the path. 
In this case, it can be executed as "C:\Program Files (x86)\IOBit\Advanced.exe".

### 3.2. privilege escalation using unquoted service path exploit


