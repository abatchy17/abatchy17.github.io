---
layout: post
title: "OverTheWire: Bandit 6-10"
date: 2016-10-18 12:00:00
share: true
comments: true
tags: [OverTheWire - Bandit]
---

## Bandit 6

We'll be using `find` command again. File has the following properties:
* Owned by user bandit7
* Owned by group bandit6
* Size is 33 bytes

```console
bandit6@melinda:~$ find / -user bandit7 -group bandit6 -size 33c 2>/dev/null
/var/lib/dpkg/info/bandit7.password
bandit6@melinda:~$ cat /var/lib/dpkg/info/bandit7.password
HKBPTKQnIay4Fw76bEy8PVxKEDQRKTzs
```

Notice that we redirected stderr to avoid seeing the permission-denied messages.

___________________________________________

## Bandit 7

The password  is in data.txt next to the word millionth, of course we can just use cat and waste our time searching for the string millionth. Instead let's use [`grep`](https://www.gnu.org/software/grep/manual/grep.html) and pipe the output of cat to it.
 
To get an idea of how the file looks we can also use `head` or `tail`.

```console
bandit7@melinda:~$ cat data.txt | grep millionth
millionth    cvX2JJa4CFALtqS87jk27qwqGhBM9plV
```

___________________________________________

## Bandit 8

More piping! You'll need to play more with other famous unix commands like sort and uniq. First let's sort the strings, get the count of how many times they appear in `data.txt` and sort them again, so the string with appearance of 1 is on top. (piping it one more time to `grep` with `-v` 10 will only reveal the string we want, but you already figured out how many times other strings appear by that point).

Also important to note is if you don't sort them first, `uniq` uses a greedy algorithm and doesn't care if the string will show again later. 

```console
bandit8@melinda:~$ cat data.txt | sort | uniq -c | sort
1 UsvVyFSfZZWbi6wgC7dAFyFuR6jQQUhR
10 0dJUVh7xSLq6OkSLaxUydzRBVVJlc78x
10 1JF4GVFmFLq7XT2mYPpCzEl2aT33zxfh
10 1i6J1JQ6VDg2GYSqsgiwS1R6roZyHcm3
...
bandit8@melinda:~$ cat data.txt | sort | uniq -c | sort | grep -v 10
1 UsvVyFSfZZWbi6wgC7dAFyFuR6jQQUhR
```

___________________________________________

## Bandit 9

Human readable strings? Let's use strings this time, then pipe the output to `grep`.

```console
bandit9@melinda:~$ strings data.txt | grep ==
========== the6
========== password
========== ism
========== truKLdjsbJ5g7yyJ2X2R0o3a5HQJFuLk
```

___________________________________________

## Bandit 10

`data.txt` is encoded in [base64](https://en.wikipedia.org/wiki/Base64) format, let's decode it.

```console
bandit10@melinda:~$ cat data.txt | base64 -d
The password is IFukwKGsFW8MOq3IRFqrxE1hxTNEbUPR
```

That wasn't so hard, was it?