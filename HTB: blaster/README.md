# TryHackMe: blaster
https://tryhackme.com:8443/room/blaster


## 1. Enumeration

### 1.1. port scanning
Scan the ports of the target machine
```sudo nmap -T 4 -p- -A X.X.X.X ```
<img width="827" height="730" alt="image" src="https://github.com/user-attachments/assets/d292d495-f370-4638-b381-8053778bb688" />

As observed in above screenshot, open ports discovered are 80 (HTTP) and 3389 (RDP).
It is discovered that port 80 is a Microsoft IIS web server.

### 1.2. web subdirectory enumeration on Port 80 (HTTP)
Running ```dirbuster&```, the below fields are configured with a directory list:
<img width="763" height="537" alt="image" src="https://github.com/user-attachments/assets/43669004-33c1-4fbb-b305-c8fb2b1843f8" />

The subdirectory /retro is found.
<img width="761" height="536" alt="image" src="https://github.com/user-attachments/assets/a52e8565-a512-47ac-aafc-8001b971fa06" />

<img width="1254" height="787" alt="image" src="https://github.com/user-attachments/assets/7ecd9042-bcdf-4283-ab96-3dffe5a7024a" />

Manually browsing the website, a possible username "wade" could have leaked their password "parzival".
<img width="1249" height="786" alt="image" src="https://github.com/user-attachments/assets/fd26e8e9-6376-4533-8412-84299d68b5fb" />

This credentials can be tested on the RDP service.

## 2. Gaining initial foothold through user credentials

Using ```rdesktop -u wade X.X.X.X``` and input the password "parzival", RDP can be accessed

<img width="1626" height="525" alt="image" src="https://github.com/user-attachments/assets/f1539776-e338-42d4-97d0-7f6b56e81173" />


## 3.  researching vulnerability to escalate privileges 
Privilege escalation using CVE-2019-1388

From the user's desktop, a file named "hhupd.exe" is found.

Online researching "hhupd.exe" for indicators:
- Found possible vulnerability CVE-2019-1388 relating to hhupd.exe
    <img width="1097" height="694" alt="image" src="https://github.com/user-attachments/assets/cc960ced-b0d9-4a59-9d2e-02686f9618c2" />
- Found a exploitation method: https://sotharo-meas.medium.com/cve-2019-1388-windows-privilege-escalation-through-uac-22693fa23f5f

## 4.  Privilege escalation using CVE-2019-1388

Following through the exploitation method:

1. Right click "hhupd.exe" file and "Run as Administrator". 
<img width="476" height="472" alt="image" src="https://github.com/user-attachments/assets/a4bb7cd1-4b7b-4467-a81e-3c91d6dd21fa" />

2. Click "Show more Details"
<img width="455" height="485" alt="image" src="https://github.com/user-attachments/assets/4e98fabb-612d-4ff9-bf5c-17d719865759" />

3. Click "Show information about the publisher’s certificate".
<img width="456" height="523" alt="image" src="https://github.com/user-attachments/assets/5aa8fac3-2737-4490-97e0-de7e6861c0df" />

4. Click "VeriSign Commercial Software Publishers CA". Then "Ok" and "No" to exit out. This essentially opens Internet Explorer under the context of the user NT Authority\System
<img width="403" height="508" alt="image" src="https://github.com/user-attachments/assets/78fd17b9-197a-432e-96a4-e9bb1c91aeb5" />

5. Save the webpage: File -> Save as... to see the error “Location is not available.”
<img width="1846" height="462" alt="image" src="https://github.com/user-attachments/assets/daa07714-57cb-48fc-9609-843d12d20512" />

6. Click "OK" to close the error
<img width="1506" height="624" alt="image" src="https://github.com/user-attachments/assets/79097493-8ef4-4b7f-9e39-494a3ad41799" />

7. Under "File name" field, search "C:\Windows\System32\*.*"
<img width="614" height="435" alt="image" src="https://github.com/user-attachments/assets/abc528e6-1f25-47b0-8b96-5f5b6a1c6282" />

8. Then open the "cmd" application
<img width="611" height="439" alt="image" src="https://github.com/user-attachments/assets/e64157f8-facb-4502-ade3-911819e35319" />

9. A command-line interface under NT Authority\System is launched.
<img width="740" height="475" alt="image" src="https://github.com/user-attachments/assets/3e6fcc04-dada-4d03-b49d-a97019394c93" />


