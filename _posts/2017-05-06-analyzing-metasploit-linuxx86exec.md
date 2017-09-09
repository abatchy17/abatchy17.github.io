---
layout: post
title: "Analyzing Metasploit linux/x86/exec module using Ndisasm"
date: 2017-05-06 12:00:00
share: true
comments: true
tags: [Exploit Development, Shellcoding, SLAE, OSCE Prep]
---

_This blog post has been created for completing the requirements of the [SecurityTube Linux Assembly Expert certification](http://securitytube-training.com/online-courses/securitytube-linux-assembly-expert/)._
_**Student ID:** SLAE-885_
_**Assignment number:** 5.3_
_**Github repo:** <https://github.com/abatchy17/SLAE>_  

---  
  
In the third and final part of assignment 5 in SLAE, I'll be analyzing the `linux/x86/exec` module using Ndisasm.  
  
Ndisasm comes with Nasm assembler, unlike GDB and Libemu it doesn't execute any code, just converts shellcode to the corresponding instructions, so we'll be analyzing the instructions by hand.  
  

## 1\. Setting payload parameters

```console
root@kali:~# msfvenom -p linux/x86/exec --payload-options  
Options for payload/linux/x86/exec:  
  
  
       Name: Linux Execute Command  
     Module: payload/linux/x86/exec  
   Platform: Linux  
       Arch: x86  
Needs Admin: No  
 Total size: 36  
       Rank: Normal  
  
Provided by:  
    vlad902 <vlad902@gmail.com>  
  
Basic options:  
Name  Current Setting  Required  Description  
----  ---------------  --------  -----------  
CMD                    yes       The command string to execute  
  
Description:  
  Execute an arbitrary command  
```    

  
Payload requires a single command, so let's use something simple like `/bin/date`.  

```console
root@kali:~# msfvenom -p linux/x86/exec CMD=/bin/date -f c  
No platform was selected, choosing Msf::Module::Platform::Linux from the payload  
No Arch selected, selecting Arch: x86 from the payload  
No encoder or badchars specified, outputting raw payload  
Payload size: 45 bytes  
Final size of c file: 213 bytes  
unsigned char buf[] =   
"\x6a\x0b\x58\x99\x52\x66\x68\x2d\x63\x89\xe7\x68\x2f\x73\x68"  
"\x00\x68\x2f\x62\x69\x6e\x89\xe3\x52\xe8\x0a\x00\x00\x00\x2f"  
"\x62\x69\x6e\x2f\x64\x61\x74\x65\x00\x57\x53\x89\xe1\xcd\x80";  
```

---


## 2\. Analyzing payload with Ndisasm
```console
root@kali:~# echo -ne "\x6a\x0b\x58\x99\x52\x66\x68\x2d\x63\x89\xe7\x68\x2f\x73\x68\x00\x68\x2f\x62\x69\x6e\x89\xe3\x52\xe8\x0a\x00\x00\x00\x2f\x62\x69\x6e\x2f\x64\x61\x74\x65\x00\x57\x53\x89\xe1\xcd\x80" | ndisasm -u -  
```

```nasm 
00000000  6A0B              push byte +0xb  
00000002  58                pop eax  
00000003  99                cdq  
00000004  52                push edx  
00000005  66682D63          push word 0x632d  
00000009  89E7              mov edi,esp  
0000000B  682F736800        push dword 0x68732f  
00000010  682F62696E        push dword 0x6e69622f  
00000015  89E3              mov ebx,esp  
00000017  52                push edx  
00000018  E80A000000        call dword 0x27  
0000001D  2F                das  
0000001E  62696E            bound ebp,[ecx+0x6e]  
00000021  2F                das  
00000022  6461              fs popad  
00000024  7465              jz 0x8b  
00000026  005753            add [edi+0x53],dl  
00000029  89E1              mov ecx,esp  
0000002B  CD80              int 0x80  
```

Okay let's go through every few commands and get an idea what's going on.  

```nasm
push byte +0xb  
pop eax                 ; EAX = 0xb  
cdq                     ; Extend EAX -> EDX = 0x0  
push edx                ; Push zero byte  
```

  
Nothing special here so far.  
  
```nasm
push word 0x632d        ; Translates to "-c"  
mov edi,esp             ; Make EDI point to top of stack, which contains "-c" terminated with a null character  
```  

This part is constructing the command to be executed, `-c` will execute the command.  

```nasm
push dword 0x68732f     ; PUSH "hs/"  
push dword 0x6e69622f   ; PUSH "nib/"  
mov ebx,esp             ; Make EBX point to top of stack "/bin/sh -c"  
push edx                ; Push another NULL byte  
```    

  
Next part is interesting, code tells us to jump to instruction 27, but there isn't one! Reason is that there is a string starting at 1D till 27.  
  
```console
root@kali:~# echo -ne "\x2f\x62\x69\x6e\x2f\x64\x61\x74\x65\x00"  
/bin/date  
```

Remaining instructions (after 27) are interpreted incorrectly, so let's pipe them again to ndisasm and come up with the correct instructions.

```
root@kali:~# echo -ne "\x57\x53\x89\xe1\xcd\x80" | ndisasm -u -  
00000000  57                push edi  
00000001  53                push ebx  
00000002  89E1              mov ecx,esp  
00000004  CD80              int 0x80  
```    
  
Awesome, this makes more sense now. Let's rewrite the assembly code in a more understandable structure.  

```nasm   
00000000    push byte +0xb  
00000002    pop eax                 ; EAX = 0xb  
00000003    cdq                     ; Extend EAX -> EDX = 0x0  
00000004    push edx                ; Push zero byte  
00000005    push word 0x632d        ; Translates to "-c"  
00000009    mov edi,esp             ; Make EDI point to top of stack, which contains "-c" terminated with a null character  
0000000B    push dword 0x68732f     ; PUSH "hs/"  
00000010    push dword 0x6e69622f   ; PUSH "nib/"  
00000015    mov ebx,esp             ; Make EBX point to top of stack "/bin/sh"  
00000017    push edx                ; Push another NULL byte  
00000018    call dword 0x27         ; Address of "/bin/date" is pushed onto stack through "call"  
0000001D    "/bin/date"             ; Not executed  
00000027    push edi                ; Push pointer to "-c"  
00000027    push ebx                ; Push pointer to "/bin/sh"  
00000029    mov ecx,esp             ; ECX point to top of stack  
                                    ; This is the "meat" of the shellcode, ECX points to [addr1][addr2][addr3][null]  
                                    ; addr1 points to /bin/sh (0B, 10 and 15)  
                                    ; addr2 points to -c (05 and 09)  
                                    ; addr3 points to "/bin/date" (18 and 1D)  
  
                                    ; NULL indicates end of argv[]  
0000002B    int 0x80                ; EAX = 0xb, which is syscall for execve()  
                                    ; EBX points to argv[0]  
                                    ; ECX points to &argv[0]  
```

### TL;DR

  
1\. `EAX` is set to 0xb, which is the syscall number for `execve()`  
2\. `EBX` points to `/bin/sh` 
3\. `ECX` points to `argv[]` which is `{ "/bin/sh", "-c", "/bin/date" }`
  
That's it!  
  
\- Abatchy

