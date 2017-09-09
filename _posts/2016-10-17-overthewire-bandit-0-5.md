---
layout: post
title: "OverTheWire: Bandit 0-5"
date: 2016-10-17 12:00:00
share: true
comments: true
tags: [OverTheWire - Bandit]
---

We all have to start somewhere, and I decided to solve OverTheWire wargames, starting with the first set of challenges called [Bandit](http://overthewire.org/wargames/bandit/).

You'll have to ssh to their server for this set of challenges, this is not always the case, Natas for example web-application based. Anyway, let's start with the first challenge.

## Bandit 0

Nothing special, ssh-ing to bandit0@bandit.labs.overthewire.org and an `ls` command shows a readme file. Let's read what's inside it with cat command.

```console
bandit0@melinda:~$ ls
readme
bandit0@melinda:~$ cat readme
boJ9jbbUNNfktd78OOpsqOltutMc3MY1
```
Yep, that's our password to bandit1.

___________________________________________

## Bandit 1

Password is in file called `-`, let's try reading the content with `cat`:

```console
bandit1@melinda:~$ ls
-
bandit1@melinda:~$ cat -
s
s
^C
```

Didn't work, hmm, cat is working as an echo to STDIN. How can we properly make it read the file?  Let's try passing the file in a different way, by using its path:

```console
bandit1@melinda:~$ cat ./-
CV1DtqXWVFXTvM2F0k09SHz0YwRINYA9
```

There you go! Our pasword for Bandit 2.

___________________________________________

## Bandit 2

Another easy one, the password is in a file called `spaces in this filename`.


`cat spaces in this file name` will assume you're trying to read files called spaces`, `in`, `this `and `filename`, which is not the case. What we need is a way to escape the space character properly in Linux.

This can be done by preceding the space character by a backslash:

```console
bandit2@melinda:~$ cat spaces in this filename
cat: spaces: No such file or directory
cat: in: No such file or directory
cat: this: No such file or directory
cat: filename: No such file or directory
bandit2@melinda:~$ cat spaces\ in\ this\ filename
UmHadQclWmgdLOKQ3YNgjWxGoRMb5luK
```

___________________________________________

## Bandit 3

Password is inside a hidden file in inhere directory, let's first navigate to it then run our ls command.

```console
bandit3@melinda:~$ ls
inhere
bandit3@melinda:~$ cd inhere
bandit3@melinda:~/inhere$ ls
bandit3@melinda:~/inhere$
```

What? Is it empty? Maybe the default ls behavior doesn't show hidden files (which start with a dot in Linux btw). Let's see what ls has to offer:

```console
bandit3@melinda:~/inhere$ ls --help
Usage: ls [OPTION]... [FILE]...
...
-a, --all         do not ignore entries starting with .
...
bandit3@melinda:~/inhere$
```

Alright, let's run `ls -a`:

```console
bandit3@melinda:~/inhere$ cat .hidden
pIwrPrtPN36QITSp3EQaw936yaFoFgAB

bandit3@melinda:~/inhere$
```

___________________________________________

## Bandit 4

Password is in one of the files inside inhere directory, with humanly readable characters. Simply running cat for every file would do. Or a 1-line Bash for loop:

```console
bandit4@melinda:~/inhere$ for i in $(seq 0 9); do echo Reading file0$i;cat ./-file0$i;echo; done
Reading file00
;▒-▒(▒▒z▒▒У▒▒ޘ▒▒8鑾
Reading file01
?▒@c
O8▒L▒▒c▒Ч7▒zb~▒▒ף▒▒U▒
Reading file02
▒g▒f▒4▒6+>"▒▒B▒Vx▒▒d▒▒;de▒O
Reading file03
▒:n▒▒▒▒8S▒▒Ѕ[▒/q▒(▒▒@▒▒M▒.▒t
Reading file04
▒▒▒▒+▒▒5▒`▒¶R
▒1*6C▒u#Nr▒
Reading file05
▒▒hZ▒▒▒P▒邚▒▒▒{#▒▒TP▒▒6▒]▒▒X:
Reading file06
▒▒▒▒!▒>P▒
d{▒▒▒▒ҏH▒▒▒xX|▒
`Reading file07
koReBOKuIDDepwhWk7jZC0RTdopnAYKh `
Reading file08
▒▒M▒▒▒▒#8B0wPg▒▒▒▒C▒▒▒@▒▒FM
Reading file09
#[:*▒▒▒?▒▒j▒▒▒U▒
bandit4@melinda:~/inhere$
```

___________________________________________

## Bandit 5

File is inside one of the directories, and is of size 1033 bytes. We'll use find command, with -type f to only scan directories and -size 1033c (c is for bytes, b is for 512-blocks).

```console
bandit5@melinda:~/inhere$ find . -type f -size 1033c
./maybehere07/.file2
bandit5@melinda:~/inhere$ cat maybehere07/.file2
DXjZPULLxYr17uwoI01bNLQbtFemEgo7
```

That's all for now, will write another guide for Bandit 6-10 soon.