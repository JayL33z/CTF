# HackTheBox: Bastard
https://www.hackthebox.com/machines/bastard

## 1. Enumeration
### 1.1. port scanning
Scan the ports of the target machine
```sudo nmap -T 4 -p- -A X.X.X.X ```

<img width="1909" height="651" alt="image" src="https://github.com/user-attachments/assets/8885c3de-0d87-4e7d-beac-67668e85dac7" />

As observed in above screenshot, open ports discovered are 80, 135 and 49154.

### 1.2. web subdirectory enumeration on Port 80 (HTTP)
Checking out the HTTP service on port 80, the landing page is a login panel as below.
<img width="1593" height="641" alt="image" src="https://github.com/user-attachments/assets/b3f45e91-454b-4051-9272-4f8753c927c9" />


From the earlier nmap scan, noticed a few subdirectories had been found such as CHANGELOG.txt.

<img width="957" height="862" alt="image" src="https://github.com/user-attachments/assets/8563b079-4925-4d94-b04c-eefeac2c2fc3" />

At *http://X.X.X.X:80/CHANGELOG.txt*, the file here revealed that the web service could be running on Drupal 7.54 which is open-source content management system (CMS).

## 2. Exploitation

### 2.1. researching vulnerability
Online researching *CMS Made Simple version 2.2.8* :
- Found possible vulnerability CVE-2018-7600 (https://nvd.nist.gov/vuln/detail/cve-2018-7600) that allows remote attackers to execute arbitrary code.
- Found a publicly available exploit (https://github.com/pimps/CVE-2018-7600)

### 2.2. exploiting vulnerability
Hence we downloaded the exploit (python script) and execute it against the target server where remote commands can be executed with the parameter for ```-c``` switch.
For example ```python3 drupa7-CVE-2018-7600.py http://X.X.X.X:80 -c "systeminfo"``` to run ```systeminfo``` command

<img width="761" height="834" alt="image" src="https://github.com/user-attachments/assets/bc4f6011-b821-4b52-b205-a3bd900a66bc" />


