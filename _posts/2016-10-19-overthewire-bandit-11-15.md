---
layout: post
title: "OverTheWire: Bandit 11-15"
date: 2016-10-19 12:00:00
share: true
comments: true
tags: [OverTheWire - Bandit]
---

## Bandit 11

Letters are rotated by 13 positions, which means _abc..xyz_ becomes _opq...lmn. _A useful tool to use would be [`tr`](http://www.tutorialspoint.com/unix_commands/tr.htm). 

```console
bandit11@melinda:~$ tr --help
`Usage: tr [OPTION]... SET1 [SET2]`
Translate, squeeze, and/or delete characters from standard input,
writing to standard output.
...
`  [:alpha:]    all letters`
...
```

We know from previous passwords that the password contains upper and lowercase characters, so we can use `[:alpha:]` as our first set. Uppercase letters come first. The set we want to translate it from is `O..No..n`.

```console
bandit11@melinda:~$ cat data.txt | tr [:alpha:] [OPQRSTUVWXYZABCDEFGHIJKLMNopqrstuvwxyzabcdefghijklmn]
The password is 5Te8Y4drgCRfCx8ugdwuEX8KFC6k2EUu
```

Can we do it in a nicer way? Yep:

```console
bandit11@melinda:~$ cat data.txt | tr [:alpha:] '[O-ZA-No-za-n]'
The password is 5Te8Y4drgCRfCx8ugdwuEX8KFC6k2EUu
```

___________________________________________

## Bandit 12

Very interesting challenge, actually one of my favourites in the bandit series. First we need to make a temporary directory as suggested to unzip the files. Then copy `data.txt` and convert the ASCII text to binary (basically undumping the content), for that we'll be using [`xxd`](http://linuxcommand.org/man_pages/xxd1.html).

```console
bandit12@melinda:~$ mkdir /tmp/abatchy0
bandit12@melinda:~$ cp data.txt /tmp/abatchy0
bandit12@melinda:~$ cd /tmp/abatchy0
bandit12@melinda:/tmp/abatchy0$ ls
data.txt
bandit12@melinda:/tmp/abatchy0$ xxd -r data.txt > data.bin
```

Next we want to figure out how this file is compressed, for that we'll use `file`.

```console
bandit12@melinda:/tmp/abatchy0$ file data.bin
data.bin: gzip compressed data, was "data2.bin", from Unix, last modified: Fri Nov 14 10:32:20 2014, max compression
```

Awesome! Let's decompress it.

```console
bandit12@melinda:/tmp/abatchy0$ gzip -d data.bin
gzip: data.bin: unknown suffix -- ignored
```

Damn, gzip expects a proper extension, let's rename it.

```console
bandit12@melinda:/tmp/abatchy0$ mv data.bin data.gz
bandit12@melinda:/tmp/abatchy0$ gzip -d data.gz
bandit12@melinda:/tmp/abatchy0$ ls
data data.txt
bandit12@melinda:/tmp/abatchy0$ file data
data: bzip2 compressed data, block size = 900k
bandit12@melinda:/tmp/abatchy0$ mv data data.bz2
bandit12@melinda:/tmp/abatchy0$ bzip2 -d data.bz2
bandit12@melinda:/tmp/abatchy0$ ls
data data.txt
bandit12@melinda:/tmp/abatchy0$ file data
data: gzip compressed data, was "data4.bin", from Unix, last modified: Fri Nov 14 10:32:20 2014, max compression
```

Only a few more times...

```console
bandit12@melinda:/tmp/abatchy0$ mv data data.gz
bandit12@melinda:/tmp/abatchy0$ gzip -d data.gz
bandit12@melinda:/tmp/abatchy0$ ls
data data.txt
bandit12@melinda:/tmp/abatchy0$ file data
data: POSIX tar archive (GNU)
```

_Crickets chirp..._

```console
bandit12@melinda:/tmp/abatchy0$ mv data data.gz
bandit12@melinda:/tmp/abatchy0$ gzip -d data.gz
bandit12@melinda:/tmp/abatchy0$ ls
data data.txt
bandit12@melinda:/tmp/abatchy0$ file data
data: POSIX tar archive (GNU)
bandit12@melinda:/tmp/abatchy0$ mv data data.tar
bandit12@melinda:/tmp/abatchy0$ tar -xvf data.tar
data5.bin
bandit12@melinda:/tmp/abatchy0$ file data5.bin
data5.bin: POSIX tar archive (GNU)
bandit12@melinda:/tmp/abatchy0$ mv data5.bin data5.tar
bandit12@melinda:/tmp/abatchy0$ tar -xf data5.tar
bandit12@melinda:/tmp/abatchy0$ tar -xf data5.tar
bandit12@melinda:/tmp/abatchy0$ mv data5.bin data5.tarl^C
bandit12@melinda:/tmp/abatchy0$ ls
data.txt data5.tar data6.bin
bandit12@melinda:/tmp/abatchy0$ rm data5.tar
bandit12@melinda:/tmp/abatchy0$ file data6.bin
data6.bin: bzip2 compressed data, block size = 900k
bandit12@melinda:/tmp/abatchy0$ mv data6.bin data6.bz2
bandit12@melinda:/tmp/abatchy0$ bzip2 -d data6.bz2
bandit12@melinda:/tmp/abatchy0$ ls
data.txt data6
bandit12@melinda:/tmp/abatchy0$ file data6
data6: POSIX tar archive (GNU)
bandit12@melinda:/tmp/abatchy0$ mv data6 data6.tar
bandit12@melinda:/tmp/abatchy0$ tar -xf data6.tar
bandit12@melinda:/tmp/abatchy0$ ls
data.txt data6.tar data8.bin
bandit12@melinda:/tmp/abatchy0$ rm data6.tar
```

Is this ever going to end?

```console
bandit12@melinda:/tmp/abatchy0$ file data8.bin
data8.bin: gzip compressed data, was "data9.bin", from Unix, last modified: Fri Nov 14 10:32:20 2014, max compression
bandit12@melinda:/tmp/abatchy0$ mv data8.bin data8.gz
bandit12@melinda:/tmp/abatchy0$ gzip -d data8.gz
bandit12@melinda:/tmp/abatchy0$ ls
data.txt data8
bandit12@melinda:/tmp/abatchy0$ file data8
data8: ASCII text
bandit12@melinda:/tmp/abatchy0$ cat data8
The password is 8ZjyCRiBWFYkneahHwxCv3wb2a1ORpYL
```

Got it!

___________________________________________

## Bandit 13

I'm currently using cygwin, but following any tutorial for adding ssh keys on linux will do.

___________________________________________

## Bandit 14

You'll need to play around with NC if you don't know what it's for. Great tool for any network engineer/system administrator/professional boxer.

```console
bandit14@melinda:~$ nc -nv 127.0.0.1 30000 < /etc/bandit_pass/bandit14
Connection to 127.0.0.1 30000 port [tcp/*] succeeded!
Correct!
BfMYroe26WYalil77FoDi9qh59eK5xNr
```

___________________________________________

## Bandit 15

NC is outdated and doesn't support SSL, we'll be used the modernized version
of it called ncat which is part of the nmap project.

```console
bandit15@melinda:~$ ncat --ssl -nv 127.0.0.1 30001
Ncat: Version 6.40 ( http://nmap.org/ncat )
Ncat: SSL connection to 127.0.0.1:30001.
Ncat: SHA-1 fingerprint: 6872 7805 D7EC 03BA 51E2 B301 2651 8989 0556 7D66
BfMYroe26WYalil77FoDi9qh59eK5xNr
Correct!
cluFn7wTiGryunymYOu4RcffSxQluehd
^C
```

15 levels done, 12 to go!