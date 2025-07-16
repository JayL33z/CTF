# HackTheBox: Legacy
https://www.hackthebox.com/machines/legacy

## 1. Enumeration

### 1.1. port scanning
Scan the ports of the target machine
```sudo nmap -T 4 -p- -A X.X.X.X ```
<img width="861" height="829" alt="image" src="https://github.com/user-attachments/assets/11eda684-e209-436d-ae32-18498e0bd8a1" />


As observed in above screenshot, open ports discovered are 135, 139 and 445.

### 1.2. further smb enumeration
Run additional nmap scripts to find smb vulnerabilities
```sudo nmap --script smb-vuln* -p 135,139,455 X.X.X.X ```
<img width="859" height="717" alt="image" src="https://github.com/user-attachments/assets/247b4820-67bc-445c-8565-76b6bf858496" />

Possible vulnerabilities found:
- CVE-2008-4250 (https://nvd.nist.gov/vuln/detail/cve-2008-4250)
- CVE-2017-0143 (https://nvd.nist.gov/vuln/detail/cve-2017-0143)

## 2. Exploitation

### 2.1. researching vulnerability CVE-2008-4250 on Metasploit
Using Metasploit console (msfconsole) search, modules targeting CVE-2008-4250 is found.
<img width="1374" height="699" alt="image" src="https://github.com/user-attachments/assets/4cbceb40-f83c-4cb8-92b2-f8f07a9189de" />

As per above screenshot, use the one that does "Automatic Target"

### 2.2. exploiting vulnerability and gaining foothold via Metasploit

And as per below screenshot, setting the relevant options for the module and launching it, the exploitation is successful.
With a reverse shell as the payload, attackers gaining a foothold into the system.
<img width="861" height="337" alt="image" src="https://github.com/user-attachments/assets/29e6466f-081d-4bf1-92d1-69266078acdb" />
