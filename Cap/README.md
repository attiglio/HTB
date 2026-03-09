
<img width="507" height="286" alt="banner" src="https://github.com/user-attachments/assets/79800b1f-9bc4-449e-a2e1-b11900879d12" />

# Easy Hackthebox challange machine.

To solve this **machine**, we begin by enumerating open ports – finding ports *21, 22, and 80* open. 
On the webserver, we are able to exploit an *Insecure Direct Object Reference (IDOR)* vulnerability to obtain system credentials as the nathan user. 
Using the credentials, we are able to SSH into the machine, and read user.txt. 
Through automated, local enumeration, we learn a system package is incorrectly configured to allow setuid. Using this knowledge, we are able to exploit the package to get a shell as root – gaining access to root.txt.

Nmap scan result :
```
nmap/nmap -A -T4 -sS -sCV -p- --open -oN nmap/all_tcp.md 10.129.3.102
Nmap scan report for 10.129.3.102
Host is up (0.018s latency).
Not shown: 65532 closed tcp ports (reset)
PORT   STATE SERVICE VERSION
21/tcp open  ftp     vsftpd 3.0.3
22/tcp open  ssh     OpenSSH 8.2p1 Ubuntu 4ubuntu0.2 (Ubuntu Linux; protocol 2.0)
80/tcp open  http    Gunicorn
|_http-server-header: gunicorn
|_http-title: Security Dashboard
Device type: general purpose|router
```
<img width="808" height="471" alt="nmapscan" src="https://github.com/user-attachments/assets/fcd4038f-744e-427c-a1ba-344cf0c24edc" />


Since FTP is running, we attempt to log in anonymously, however, this is disabled. 
Next, we enumerate port 80 using tools like **gobuster and ffuf**. Unfortunately, these tools do not yield any useful results.

FFUF directory scan result using raft-medium-directories list :

```

ffuf -u http://10.129.3.102/FUZZ -w /opt/seclists/Discovery/Web-Content/raft-medium-directories.txt -c -v -o gobuster/ffufdir.md
 :: Method           : GET
 :: URL              : http://10.129.3.102/FUZZ
 :: Wordlist         : FUZZ: /opt/seclists/Discovery/Web-Content/raft-medium-directories.txt
 :: Output file      : gobuster/ffufdir.md
________________________________________________
[Status: 302, Size: 208, Words: 21, Lines: 4, Duration: 28ms]
| URL | http://10.129.3.102/data
| URL | http://10.129.3.102/ip
| URL | http://10.129.3.102/capture
| --> | http://10.129.3.102/data/3
    * FUZZ: capture
```
<img width="1102" height="641" alt="fuffdirscan" src="https://github.com/user-attachments/assets/84e6d7f8-d510-47db-a30a-9348e4ab91b8" />

Gobuster directory scan result using raft-medium-directories list :

```
gobuster dir -u http://10.129.3.102/ -w /opt/seclists/Discovery/Web-Content/raft-medium-directories.txt -t 20 -o gobuster/gobusterdir.md
===============================================================
Gobuster v3.8.2
by OJ Reeves (@TheColonial) & Christian Mehlmauer (@firefart)
===============================================================
[+] Url:                     http://10.129.3.102/
[+] Wordlist:                /opt/seclists/Discovery/Web-Content/raft-medium-directories.txt
[+] User Agent:              gobuster/3.8.2
[+] Timeout:                 10s
===============================================================
Starting gobuster in directory enumeration mode
===============================================================
data                 (Status: 302) [Size: 208] [--> http://10.129.3.102/]
ip                   (Status: 200) [Size: 17446]
capture              (Status: 302) [Size: 220] [--> http://10.129.3.102/data/1]
Progress: 29999 / 29999 (100.00%)
===============================================================
```
<img width="1159" height="402" alt="gobuster" src="https://github.com/user-attachments/assets/14ed6605-b226-4c20-a167-ea550a32905c" />

 Since we did not find any further results from our automated tools, we decide to dig deeper into the web pages. 
Reflecting back on the “Security Snapshot” page, we generated a pcap, the URL path was /data/1. 
 This suggests a possible **Insecure Direct Object Reference (IDOR)** vulnerability may exist, if there is no authorization checks around the request. 
To test this, we can change this number, and monitor the results. In the case the object is invalid, we are redirected to the dashboard, 
however, when we set the number to “0”, we are given a valid capture file that we are able to download. 

