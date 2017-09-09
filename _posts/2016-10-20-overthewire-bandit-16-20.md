---
layout: post
title: "OverTheWire: Bandit 16-20"
date: 2016-10-20 12:00:00
share: true
comments: true
tags: [OverTheWire - Bandit]
---

## Bandit 16

As indicated, we need to submit the current password to one of the ports between 31000 and 32000. For that we'll be using one of the greatest port scanning tools out there, [`nmap`](https://nmap.org/).

First, let's check which ports are open and which aren't. For a full list, run `nmap -h` or just `nmap` on your terminal. We'll also be using the `-sV` option to detect which service is listening on the open ports.

```console
bandit16@melinda:~$ nmap -p 31000-32000 127.0.0.1
Starting Nmap 6.40 ( http://nmap.org ) at 2016-10-19 21:32 UTC
Nmap scan report for localhost (127.0.0.1)
Host is up (0.00033s latency).
PORT      STATE SERVICE VERSION
31046/tcp open  echo
31518/tcp open  msdtc   Microsoft Distributed Transaction Coordinator (error)
31691/tcp open  echo
31790/tcp open  msdtc   Microsoft Distributed Transaction Coordinator (error)
31960/tcp open  echo
Service Info: OS: Windows; CPE: cpe:/o:microsoft:windows

Service detection performed. Please report any incorrect results at http://nmap.org/submit/ .
Nmap done: 1 IP address (1 host up) scanned in 41.18 seconds
```

Awesome, 3 ports don't interest us since they'll just echo your input. This leaves us port 31518 and 31790. We'll use `ncat` again to establish a secure connection.

```console
bandit16@melinda:~$ `ncat --ssl -nv 127.0.0.1 31518`
Ncat: Version 6.40 ( http://nmap.org/ncat )
Ncat: SSL connection to 127.0.0.1:31518.
Ncat: SHA-1 fingerprint: 6872 7805 D7EC 03BA 51E2 B301 2651 8989 0556 7D66
cluFn7wTiGryunymYOu4RcffSxQluehd
cluFn7wTiGryunymYOu4RcffSxQluehd
^C
bandit16@melinda:~$ `ncat --ssl -nv 127.0.0.1 31790`
Ncat: Version 6.40 ( http://nmap.org/ncat )
Ncat: SSL connection to 127.0.0.1:31790.
Ncat: SHA-1 fingerprint: 6872 7805 D7EC 03BA 51E2 B301 2651 8989 0556 7D66
cluFn7wTiGryunymYOu4RcffSxQluehd
Correct!
-----BEGIN RSA PRIVATE KEY-----
MIIEogIBAAKCAQEAvmOkuifmMg6HL2YPIOjon6iWfbp7c3jx34YkYWqUH57SUdyJ
imZzeyGC0gtZPGujUSxiJSWI/oTqexh+cAMTSMlOJf7+BrJObArnxd9Y7YT2bRPQ
Ja6Lzb558YW3FZl87ORiO+rW4LCDCNd2lUvLE/GL2GWyuKN0K5iCd5TbtJzEkQTu
DSt2mcNn4rhAL+JFr56o4T6z8WWAW18BR6yGrMq7Q/kALHYW3OekePQAzL0VUYbW
JGTi65CxbCnzc/w4+mqQyvmzpWtMAzJTzAzQxNbkR2MBGySxDLrjg0LWN6sK7wNX
x0YVztz/zbIkPjfkU1jHS+9EbVNj+D1XFOJuaQIDAQABAoIBABagpxpM1aoLWfvD
KHcj10nqcoBc4oE11aFYQwik7xfW+24pRNuDE6SFthOar69jp5RlLwD1NhPx3iBl
J9nOM8OJ0VToum43UOS8YxF8WwhXriYGnc1sskbwpXOUDc9uX4+UESzH22P29ovd
d8WErY0gPxun8pbJLmxkAtWNhpMvfe0050vk9TL5wqbu9AlbssgTcCXkMQnPw9nC
YNN6DDP2lbcBrvgT9YCNL6C+ZKufD52yOQ9qOkwFTEQpjtF4uNtJom+asvlpmS8A
vLY9r60wYSvmZhNqBUrj7lyCtXMIu1kkd4w7F77k+DjHoAXyxcUp1DGL51sOmama
+TOWWgECgYEA8JtPxP0GRJ+IQkX262jM3dEIkza8ky5moIwUqYdsx0NxHgRRhORT
8c8hAuRBb2G82so8vUHk/fur85OEfc9TncnCY2crpoqsghifKLxrLgtT+qDpfZnx
SatLdt8GfQ85yA7hnWWJ2MxF3NaeSDm75Lsm+tBbAiyc9P2jGRNtMSkCgYEAypHd
HCctNi/FwjulhttFx/rHYKhLidZDFYeiE/v45bN4yFm8x7R/b0iE7KaszX+Exdvt
SghaTdcG0Knyw1bpJVyusavPzpaJMjdJ6tcFhVAbAjm7enCIvGCSx+X3l5SiWg0A
R57hJglezIiVjv3aGwHwvlZvtszK6zV6oXFAu0ECgYAbjo46T4hyP5tJi93V5HDi
Ttiek7xRVxUl+iU7rWkGAXFpMLFteQEsRr7PJ/lemmEY5eTDAFMLy9FL2m9oQWCg
R8VdwSk8r9FGLS+9aKcV5PI/WEKlwgXinB3OhYimtiG2Cg5JCqIZFHxD6MjEGOiu
L8ktHMPvodBwNsSBULpG0QKBgBAplTfC1HOnWiMGOU3KPwYWt0O6CdTkmJOmL8Ni
blh9elyZ9FsGxsgtRBXRsqXuz7wtsQAgLHxbdLq/ZJQ7YfzOKU4ZxEnabvXnvWkU
YOdjHdSOoKvDQNWu6ucyLRAWFuISeXw9a/9p7ftpxm0TSgyvmfLF2MIAEwyzRqaM
77pBAoGAMmjmIJdjp+Ez8duyn3ieo36yrttF5NSsJLAbxFpdlc1gvtGCWW+9Cq0b
dxviW8+TFVEBl1O4f7HVm6EpTscdDxU+bCXWkfjuRb7Dy9GOtt9JPsX8MBTakzh3
vBgsyi/sN3RqRBcGU40fOoZyfAMT8s1m/uYv52O6IgeuZ/ujbjY=
-----END RSA PRIVATE KEY-----
Thanks!
^C
```


31518 was just another echo service, submitting our password via port 31790 reveals another SSH private key. After logging as Bandit17, the password for it is `xLYVMN9WE5zQ5vHacb0sZEVqbrp7nBTn`.

___________________________________________

## Bandit 17

Quite straight forward, the `diff` command seems to be the sensible option. Let's check how to use it.

```console
bandit17@melinda:~$ diff --help
Usage: diff [OPTION]... FILES
Compare FILES line by line.
Mandatory arguments to long options are mandatory for short options too.
--normal         output a normal diff (the default)
...
```

`--normal` is used by default, so `diff password.old password.new` should do it.

Done! New password is the one on the right `kfBf3eYk5BPBRzwjqutbbfE887SVc5Yd`.

___________________________________________

## Bandit 18

Hmm, once you ssh into the server you just log out. We need to do 2 things:

* Figure out why we log out once connected.
* Find a way to stay connected.

Let's start by our first objective. You might've noticed that some hidden files exist in the home directory, like `.bashrc` or `.bash_profile`. We need to find a way to read their content. SSH is the only command we used, so let's check the help pages, maybe there's a way to run a command upon executing?

`ssh --help` didn't reveal much info, let's try the man pages. -t looks promising.

```console
abatchy@abatchy ~
$ `ssh bandit18@bandit.labs.overthewire.org -t "ls -al" `
This is the OverTheWire game server. More information on http://www.overthewire.org/wargames
Please note that wargame usernames are no longer level<X>, but wargamename<X>
e.g. vortex4, semtex2, ...
Note: at this moment, blacksun is not available.
bandit18@bandit.labs.overthewire.org's password:
total 24
drwxr-xr-x  2 root   root   4096 Nov 14 2014 .
drwxr-xr-x 172 root   root   4096 Jul 10 14:12 ..
-rw-r--r--  1 root   root   220 Apr 9 2014 .bash_logout
-rw-r-----  1 bandit19 bandit18 3660 Nov 14 2014 .bashrc
-rw-r--r--  1 root   root   675 Apr 9 2014 .profile
-rw-r-----  1 bandit19 bandit18  33 Nov 14 2014 readme
Connection to bandit.labs.overthewire.org closed.
abatchy@abatchy ~
$ `ssh bandit18@bandit.labs.overthewire.org -t "cat readme"`
This is the OverTheWire game server. More information on http://www.overthewire.org/wargames
Please note that wargame usernames are no longer level<X>, but wargamename<X>
e.g. vortex4, semtex2, ...
Note: at this moment, blacksun is not available.
bandit18@bandit.labs.overthewire.org's password:
` IueksS7Ubh8G3DCwVzrTd8rAVOwq3M5x  `
Connection to bandit.labs.overthewire.org closed.
abatchy@abatchy ~
$ ssh bandit18@bandit.labs.overthewire.org -t "sh"
This is the OverTheWire game server. More information on http://www.overthewire.org/wargames
Please note that wargame usernames are no longer level<X>, but wargamename<X>
e.g. vortex4, semtex2, ...
Note: at this moment, blacksun is not available.
bandit18@bandit.labs.overthewire.org's password:
$ ls
readme
$
```

So basically, we have two solutions:

* `ssh bandit18@bandit.labs.overthewire.org -t "ls -al" `


* `ssh bandit18@bandit.labs.overthewire.org -t "cat readme" `

Or directly calling `sh` and have the regular interactive shell.

___________________________________________

## Bandit 19

In this challenge, we'll get introduced to a new concept, which is [`setuid`](https://en.wikipedia.org/wiki/Setuid). Any file/directory in Unix has read, write, execute (if not directory) permissions for owner, group or other. You can read more about permissions [here](https://www.tutorialspoint.com/unix/unix-file-permission.htm).

A misconfigured setuid program could be disastrous as it could give a limited account root access. [Root-me.org](https://www.root-me.org/en/Challenges/App-Script) has a few great examples on how to exploit it. Another interesting read is [nmap's interactive mode](https://gist.github.com/dergachev/7916152).

Alright, let's see what's the setuid program does.

```console
bandit19@melinda:~$ `ls -al`
total 28
drwxr-xr-x  2 root   root   4096 Nov 14 2014 .
drwxr-xr-x 172 root   root   4096 Jul 10 14:12 ..
-rw-r--r--  1 root   root   220 Apr 9 2014 .bash_logout
-rw-r--r--  1 root   root   3637 Apr 9 2014 .bashrc
-rw-r--r--  1 root   root   675 Apr 9 2014 .profile
-rw`s`r-x---  1 bandit20 bandit19 7370 Nov 14 2014 bandit20-do
bandit19@melinda:~$ ./bandit20-do
Run a command as another user.
Example: ./bandit20-do id


bandit19@melinda:~$ `./bandit20-do "whoami"`
bandit20


bandit19@melinda:~$ `./bandit20-do cat /etc/bandit_pass/bandit20`
GbKksEFF4yrVs6il55v6gwY5aVje5f0j
```

___________________________________________

## Bandit 20

Hmm, so the setuid will be acting as the service listening to the port we specify, we'll fire up another terminal to make it act as the host. What are we waiting for?

#### Terminal 1 (host):

```console
bandit20@melinda:~$ nc -nvlp 44444
Listening on [0.0.0.0] (family 0, port 44444)
```


#### Terminal 2 (client):

```console
bandit20@melinda:~$ ls
suconnect
bandit20@melinda:~$ ./suconnect
Usage: ./suconnect <portnumber>
This program will connect to the given port on localhost using TCP. If it receives the correct password from the other side, the next password is transmitted back.
bandit20@melinda:~$ ./suconnect 4444
ERROR: Can't connect
bandit20@melinda:~$ ./suconnect 44444
ERROR: Can't connect
bandit20@melinda:~$ ./suconnect 44444
Read: GbKksEFF4yrVs6il55v6gwY5aVje5f0j
Password matches, sending next password
```

#### Terminal 1 (server):

```console
bandit20@melinda:~$ nc -nvlp 44444
Listening on [0.0.0.0] (family 0, port 44444)
Connection from [127.0.0.1] port 44444 [tcp/*] accepted (family 2, sport 33272)
GbKksEFF4yrVs6il55v6gwY5aVje5f0j
gE269g2h3mw3pwgrj0Ha9Uoqen1c9DGr
```

Done. To sum it up:

1. On terminal 1 (host), start netcat as server: `nc -nvlp 44444`.
2. On terminal 2 (client/setuid), run suconnect on same port: `./suconnect 44444`.
3. Back to terminal 1, send the password of current level.
4. Suconnect will show you the password.

