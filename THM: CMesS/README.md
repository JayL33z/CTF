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

The user credentials for username *andre@cmess.thm* seemed to have been leaked, with the password being *"KPFTN_f2yxe%"*.
Also, based on this development log, there is mention of an *admin panel*. 
Next step is enumerate the web service to verify whether an admin panel indeed exists so that these leaked credentials can be tested against.

### 1.3. web subdirectory enumeration on Port 80 (HTTP)

Using DirBuster by running ```dirb&``` , with the below configuration:

<img width="762" height="531" alt="image" src="https://github.com/user-attachments/assets/8b0ffe11-f6d3-4be1-b66e-4fa0778a2f32" />

<img width="757" height="539" alt="image" src="https://github.com/user-attachments/assets/a77bf7c6-d001-48be-92a0-ac6304dca540" />

As per above, /admin subdirectory is found.

And navigating to http://cmess.thm/admin shows a login page.

<img width="1274" height="733" alt="image" src="https://github.com/user-attachments/assets/864afe30-e1b8-4031-a8f6-de64e6ef4034" />



### 1.4. Gaining initial foothold through user credentials

<img width="695" height="471" alt="image" src="https://github.com/user-attachments/assets/6807dbce-b933-47f2-aeed-80127de4e328" />






