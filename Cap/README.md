Let's have a look at the website first. We can run different tools, we can see ipconfig, netstat and we can capture pcap files for five seconds and download them afterwards. A lot to look at but there wasn't any command injection or anything similar possible.

However we noticed, that after we captured our pcap file we get redirected to /data/1 to download it. So what's /data/0 ?

Nmap scan result :

<img width="808" height="471" alt="nmapscan" src="https://github.com/user-attachments/assets/fcd4038f-744e-427c-a1ba-344cf0c24edc" />

OS: Linux

Web-Technology:

IP:10.10.10.10

USERS: root, nathan

CREDENTIALS (ANY): Buck3tH4TF0RM3! ,
Flas:
```
1. ce3f90fe646c14a033f2cff997f3b5c4
2. 1d96b9552d8caefcd6e857075c0cc138
```
Ports (To-Try List): 
80 -port try ==> gobuster, ffuf no enumerat files and dirs
21 -port ftp try to login nathan with Buck3tH4TF0RM3!
22 -port SSH try same cred to login

nmap result here:
root@kali:~/Labs/HTB/Easy-HTB/HTB-CAP]
nmap -A -T4 -sS -sCV -p- --open -oN nmap/all_tcp.md $ip
Starting Nmap 7.98 ( https://nmap.org ) at 2026-03-08 19:52 -0400
Nmap scan report for 10.129.3.102
Host is up (0.018s latency).
Not shown: 65532 closed tcp ports (reset)
PORT   STATE SERVICE VERSION
21/tcp open  ftp     vsftpd 3.0.3
22/tcp open  ssh     OpenSSH 8.2p1 Ubuntu 4ubuntu0.2 (Ubuntu Linux; protocol 2.0)
| ssh-hostkey:
|   3072 fa:80:a9:b2:ca:3b:88:69:a4:28:9e:39:0d:27:d5:75 (RSA)
|   256 96:d8:f8:e3:e8:f7:71:36:c5:49:d5:9d:b6:a4:c9:0c (ECDSA)
|_  256 3f:d0:ff:91:eb:3b:f6:e1:9f:2e:8d:de:b3:de:b2:18 (ED25519)
80/tcp open  http    Gunicorn
|_http-server-header: gunicorn
|_http-title: Security Dashboard
Device type: general purpose|router
Running: Linux 5.X, MikroTik RouterOS 7.X
OS CPE: cpe:/o:linux:linux_kernel:5 cpe:/o:mikrotik:routeros:7 cpe:/o:linux:linux_kernel:5.6.3
OS details: Linux 5.0 - 5.14, MikroTik RouterOS 7.2 - 7.5 (Linux 5.6.3)
Network Distance: 2 hops
Service Info: OSs: Unix, Linux; CPE: cpe:/o:linux:linux_kernel

TRACEROUTE (using port 22/tcp)
HOP RTT      ADDRESS
1   20.95 ms 10.10.14.1
2   21.05 ms 10.129.3.102


==========================================================================
Web Services Enumeration:
- NIKTO :

- ffuf and gobuster here :

gobuster dir -u http://10.129.3.102/ -w /opt/seclists/Discovery/Web-Content/raft-medium-directories.txt -t 20 -o gobuster/gobusterdir.md
===============================================================
Gobuster v3.8.2
by OJ Reeves (@TheColonial) & Christian Mehlmauer (@firefart)
===============================================================
[+] Url:                     http://10.129.3.102/
[+] Method:                  GET
[+] Threads:                 20
[+] Wordlist:                /opt/seclists/Discovery/Web-Content/raft-medium-directories.txt
[+] Negative Status codes:   404
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
Finished
===============================================================
FFUF directory scan
# ffuf -u http://10.129.3.102/FUZZ -w /opt/seclists/Discovery/Web-Content/raft-medium-directories.txt -c -v -o gobuster/ffufdir.md

        /'___\  /'___\           /'___\
       /\ \__/ /\ \__/  __  __  /\ \__/
       \ \ ,__\\ \ ,__\/\ \/\ \ \ \ ,__\
        \ \ \_/ \ \ \_/\ \ \_\ \ \ \ \_/
         \ \_\   \ \_\  \ \____/  \ \_\
          \/_/    \/_/   \/___/    \/_/

       v2.1.0-dev
________________________________________________

 :: Method           : GET
 :: URL              : http://10.129.3.102/FUZZ
 :: Wordlist         : FUZZ: /opt/seclists/Discovery/Web-Content/raft-medium-directories.txt
 :: Output file      : gobuster/ffufdir.md
 :: File format      : json
 :: Follow redirects : false
 :: Calibration      : false
 :: Timeout          : 10
 :: Threads          : 40
 :: Matcher          : Response status: 200-299,301,302,307,401,403,405,500
________________________________________________

[Status: 302, Size: 208, Words: 21, Lines: 4, Duration: 28ms]
| URL | http://10.129.3.102/data
| --> | http://10.129.3.102/
    * FUZZ: data

[Status: 200, Size: 17454, Words: 7275, Lines: 355, Duration: 54ms]
| URL | http://10.129.3.102/ip
    * FUZZ: ip

[Status: 302, Size: 220, Words: 21, Lines: 4, Duration: 5178ms]
| URL | http://10.129.3.102/capture
| --> | http://10.129.3.102/data/3
    * FUZZ: capture
===========================================================================
Files / (Web Root)

ftp login dir -a
