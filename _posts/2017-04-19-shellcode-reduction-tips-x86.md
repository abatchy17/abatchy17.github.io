---
layout: post
title: " Shellcode reduction tips (x86) "
date: 2017-04-19 12:00:00
share: true
comments: true
tags: [Exploit Development, Shellcoding, SLAE, OSCE Prep]
---

**Public reminder to add some tricks sometime** 
  

### 1\. XOR Instruction

Setting a register to null is usually a problem when writing shellcode due to couple of reasons:  
  
* It contains null characters, will fuck up your string.  
* Size limitations.  
  
Example:  

```nasm
nasm > mov al, 0  
00000000  B000          ; 2 bytes  
nasm > mov ax, 0  
00000000  66B80000      ; 4 bytes  
nasm > mov eax, 0  
00000000  B800000000    ; 5 bytes
```
  
Instead, XORing a register with itself 1) clears it out as well, 2) is null-free and 3) generates less shellcode.  
  
```nasm
nasm > xor al,al  
00000000  30C0          ; 2 bytes 
nasm > xor ax,ax  
00000000  6631C0        ; 4 bytes 
nasm > xor eax,eax  
00000000  31C0          ; 2 bytes
```
  
Although `xor ax,ax` didn't generate less shellcode, it doesn't contain a null character.  
  

### 2\. CBW, CWD, CDQ Instructions

`CBW`, `CWD` and `CDQ` extends the sign bit of `al`/`ax`/`eax` to `ah`/`dx`/`edx`. This is particularly useful due to a couple of reasons:  
  
* `EAX` is essential in system calls.  
* Many times you'll need a register zeroed out. As most system calls won't "set" the flag bit in `EAX`, calling C{B/W/D}Q is an easy way to use `EDX` when NULL is needed.  
  
Example:

```nasm
mov al, 5       ; al's sign bit is 0 (positive)
cbw             ; ah is now 0x00
mov ax, -5
cbd             ; dx is now 0xffff
mov eax, 1337
cdq             ; eax is now 0x00000000
```
  
A common way to set a register to zero is by XORing itself (tip 1). This is 2-4 bytes long, while using any of the previous commands knowing/controlling the value of the corresponding register, we can reduce it to a single byte, cutting down the size to 50-75%.  

```nasm
nasm > xor eax, eax  
00000000  31C0  
nasm > xor edx, edx  
00000000  31D2  
nasm > xor ah,ah  
00000000  30E4  
nasm > cbw  
00000000  6698  
nasm > cwd  
00000000  6699  
nasm > cdq  
00000000  99  
```

### 3\. Push, POP Reg32

XORing (Tip 1) is great to zero out a register, but what if we want to initialize it with a certain value?

```nasm
nasm > xor eax, eax  
00000000  31C0  
nasm > mov al, 0x1  
00000000  B001
```
  
How about we utilize the stack? Pushing 0x1 for example will be automatically aligned as a DWORD.  

```nasm
nasm > push byte 0x1  
00000000  6A01              push byte +0x1  
nasm > pop eax  
00000000  58                pop eax
```


\- Abatchy

