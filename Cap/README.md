
<img width="507" height="286" alt="banner" src="https://github.com/user-attachments/assets/79800b1f-9bc4-449e-a2e1-b11900879d12" />

# Easy Hackthebox challange machine.

To solve this **machine**, we begin by enumerating open ports – finding ports *21, 22, and 80* open. 
On the webserver, we are able to exploit an *Insecure Direct Object Reference (IDOR)* vulnerability to obtain system credentials as the nathan user. 
Using the credentials, we are able to SSH into the machine, and read user.txt. 
Through automated, local enumeration, we learn a system package is incorrectly configured to allow setuid. Using this knowledge, we are able to exploit the package to get a shell as root – gaining access to root.txt.

**Let's jump in and start to hack !:).

# Enumeration

We are using network map scanner with aggressive mode recon for service and version number.

Nmap scan printscreen result :

<img width="808" height="471" alt="nmapscan" src="https://github.com/user-attachments/assets/fcd4038f-744e-427c-a1ba-344cf0c24edc" />


Next, we enumerate (fuzzing) the port 80 using tools like **gobuster and ffuf** for directory listing. 

We are using both fuzzing scanner with same fuzzing list to compare the result.
First we are using FFUF scanner for directory fuzzing with raft-medium-directories list :

Fuzzing with FFuf printscreen result :

<img width="1102" height="641" alt="fuffdirscan" src="https://github.com/user-attachments/assets/84e6d7f8-d510-47db-a30a-9348e4ab91b8" />

In second stage we are using Gobuster scanner for directory fuzzing with raft-medium-directories list :

Fuzzing gobuster print screen result :

<img width="1159" height="402" alt="gobuster" src="https://github.com/user-attachments/assets/14ed6605-b226-4c20-a167-ea550a32905c" />

 Both fuzzing scanner give a same result .Since we did not find any further results from our automated tools.We decide to dig deeper into the web pages. 
Reflecting back on the “Security Snapshot” page, we generated a pcap, the URL path was /data/1. 
 This suggests a possible **Insecure Direct Object Reference (IDOR)** vulnerability may exist, if there is no authorization checks around the request. 
To test this, we can change this number, and monitor the results. In the case the object is invalid, we are redirected to the dashboard, 
 however, when we set the number to “0”, we are given a valid capture file that we are able to download. 

INSTERT IMAGE

 After we download the capture file, we open it in a packet analysis tool, such as **Wireshark**. To help hunt for interesting information, we open the “Statistics” menu, and launch the Protocol Hierarchy window. In it, we see there are FTP packets that were captured. 
  Since FTP is a clear-text protocol, we know there may be credentials captured that we can get. To check this, we set the display filter to “ftp”. Once we set the filter, within the first few packets, we see credentials for the **nathan** user.

<img width="1464" height="848" alt="wireshark" src="https://github.com/user-attachments/assets/352ca591-b42c-421d-b23a-fd6227f064d6" />

# Gaining acces
We have credentials for user **nathan** 
```
Credentials: Nathan : Buck3tH4TF0RM3!
```

This Pcap file has username and password for FTP service. Let’s try using these credentials to login via SSH and read the first flag.

<img width="667" height="399" alt="ftpdirlogin" src="https://github.com/user-attachments/assets/a276a7b8-4fa9-4148-9f30-4ed3af616539" />

<img width="605" height="205" alt="sshlogindir" src="https://github.com/user-attachments/assets/bcda57a1-d086-4ae8-b982-013bf74428a6" />

```
nathan@cap:~$ ls -la
total 28
drwxr-xr-x 3 nathan nathan 4096 May 27  2021 .
drwxr-xr-x 3 root   root   4096 May 23  2021 ..
lrwxrwxrwx 1 root   root      9 May 15  2021 .bash_history -> /dev/null
-rw-r--r-- 1 nathan nathan  220 Feb 25  2020 .bash_logout
-rw-r--r-- 1 nathan nathan 3771 Feb 25  2020 .bashrc
drwx------ 2 nathan nathan 4096 May 23  2021 .cache
-rw-r--r-- 1 nathan nathan  807 Feb 25  2020 .profile
lrwxrwxrwx 1 root   root      9 May 27  2021 .viminfo -> /dev/null
-r-------- 1 nathan nathan   33 Mar  8 23:49 user.txt
nathan@cap:~$ cat user.txt
***ce3f90fe646c14a033f2cff997f3b5c4***

```
# Privilege Escalation

Let's find linpeas some any escalation paths.

<img width="917" height="155" alt="suidlinpeas" src="https://github.com/user-attachments/assets/6e2dc037-3842-4525-9f42-870fed7f9033" />

We trying and testing manual !

<img width="882" height="143" alt="capsuid" src="https://github.com/user-attachments/assets/bb3e80e0-b63f-4d90-aa47-47fa378fd060" />

 LinPeas result reveals that ‘cap_setuid’ capability is enabled on python3.8 binary. This simply means, the user has privilege to run this program as root.

https://gtfobins.github.io/gtfobins/python/#capabilities

<img width="758" height="365" alt="flag2" src="https://github.com/user-attachments/assets/bdac6f58-f0f2-41a0-9071-1819d9869a8f" />


We got access to root shell and read the root flag.

