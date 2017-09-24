---
layout: post
title: "[Pwnable.kr] Toddler's Bottle: flag"
date: 2017-09-11 12:00:00
share: true
comments: true
description: Walkthroughs for Pwnable.kr challenge (flag)
tags: [Pwnable.kr]
---

## Toddler's bottle: `flag`

Suggested reads: 
* <https://en.wikipedia.org/wiki/Executable_compression>


This is a very beginner-friendly reversing challenge, according to the description you only need the binary.

When reversing, there are a few tools that come in handy before things get too complicated:

* [`file`]() which basically tries to recognize what the "other" file passed to it is, it could be a binary, an image, hex data, ...
* [`strings`](https://linux.die.net/man/1/strings) which looks for printable strings in an executable, supports both ASCII and UNICODE. By default it only shows strings of size 4 or more.
* [`ltrace`](https://linux.die.net/man/1/ltrace) which intercepts library calls made from an executable.
* Its brother [`strace`](https://linux.die.net/man/1/strace) which intercepts syscalls/signals made from an executable

First, let's run `file` on `flag`. 

```console
abatchy@ubuntu:~/Desktop/tmp$ file flag
flag: ELF 64-bit LSB executable, x86-64, version 1 (GNU/Linux), statically linked, stripped
```

What does that mean? It's a 64-bit executable that does not need any external libraries to function properly, thus the statically linked part. Stripped means it does not contain any debug information, this is common for release binaries as they perform better and are of smaller size, but also common for malware as reversing them will be harder without symbols.

Okay, what next? Let's try running it, you need to set the executable bit first by `chmod u+x flag`.

```console
abatchy@ubuntu:~/Desktop/tmp$ ./flag
I will malloc() and strcpy the flag there. take it.
```

Awesome, we can just use `ltrace` to see what parameters get passed, right? That'd be possible if the binary wasn't statically linked (makes dynamic calls), but that's not the case...

```console
abatchy@ubuntu:~/Desktop/tmp$ ltrace ./flag
Couldn't find .dynsym or .dynstr in "/proc/42394/exe"
```

How do we go from there? Let's run `strings`.

```console
abatchy@ubuntu:~/Desktop/tmp$ strings flag
UPX!
@/x8
gX lw_
H/\_@
	Kl$
H9\$(t
[]]y
nIV,Uh
AWAVAUATS
uSL9

// redacted
```

If you ran `strings` on a binary before, you'll notice that some common strings you find don't exist at all, and there seems to be a lot of uh, garbage? Also notice "UPX!" at the top.

Still clueless? Try `strings` again with a size parameter:

```console
abatchy@ubuntu:~/Desktop/tmp$ strings -20 flag
_~SO/IEC 14652 i18n FDC
*+,-./0>3x6789:;<=>?
@ABCDEFGHIJKLMNOPQRSTUVWXYZ[\]^_
`abcdefghijklmnopqrstuvwxyz{|}~
 "9999$%&/999956799999:<DG9999HI_`
 ''''!#$`''''abcd''''efgh''''ijkl''''mnop''''qrst''''uvwx''''yz{|''''}~
Q2R''''STUV''''WXYZ''''[\]^''''_
MNONNNNPRTUNNNNVWYZNNNN[\_`NNNNabcdNNNNefhi
 rrrr!"#$rrrr%&'(rrrr)*+,rrrr-./0rrrr1234rrrr5678rrrr9;<=rrrr>@ABrrrrCDFJrrrrKLMNrrrrOPRSrrrrTUVWrrrrXYZ[rrrr\]^_rrrr`abcrrrrdefgrrrrhijkrrrrlmnorrrrpqrsrrrrtuvwrrrrxyz{rrrr|}~
 !"9999#$%&9999'()*9999+,-.9999/012999934569999789:9999;<=>9999?@AB9999CDEF9999GHIJ9999KLMN9999OPQR9999STUV9999WXYZ9999[\]^9999_`ab9999cdef9999ghij9999klmn9999opqr9999stuv9999wxyz9999{|}~9999
'12Wr%W345%Wr%67x!Wr892
b'cdr%WrefgWr%Whij%Wr%klr%WrmnoWr%Wpqr%Wr%str%WruvwWr%Wxyz%Wr%ABr%WrCDEWr%WFGH%Wr%IJr%WrKLMWr%WNOP%Wr%QRr%WrSTUWr%WVWX%Wr%YZ
 $9999(/6>9999HQXa9999eimq9999uy}
&9223372036854775807L`
PROT_EXEC|PROT_WRITE failed.
$Info: This file is packed with the UPX executable packer http://upx.sf.net $
$Id: UPX 3.08 Copyright (C) 1996-2011 the UPX Team. All Rights Reserved. $
GCC: (Ubuntu/Linaro 4.6.3-1u)#
```

Notice the few readable strings:

```$Info: This file is packed with the UPX executable packer http://upx.sf.net $```

`UPX` is a "packer" for binaries, which compresses binaries and at runtime decompresses them. Decompressing `flag` will make our lives easier so let's install it.

```console
abatchy@ubuntu:~/Desktop/tmp$ sudo apt search upx
[sudo] password for abatchy: 
Sorting... Done
Full Text Search... Done
clamav/xenial-updates,xenial-security 0.99.2+dfsg-0ubuntu0.16.04.2 amd64
  anti-virus utility for Unix - command-line interface

upx-ucl/xenial,now 3.91-1 amd64 [installed]
  efficient live-compressor for executables

abatchy@ubuntu:~/Desktop/tmp$ sudo apt install upx-ucl
```

Next we decompress the binary.

```console
abatchy@ubuntu:~/Desktop/tmp$ ls -al
total 336
drwxrwxr-x 2 abatchy abatchy   4096 Sep 14 19:53 .
drwxr-xr-x 8 abatchy abatchy   4096 Sep  2 11:49 ..
-rwxrw-r-- 1 abatchy abatchy 335288 Jul 14  2015 flag
abatchy@ubuntu:~/Desktop/tmp$ upx -d flag 
                       Ultimate Packer for eXecutables
                          Copyright (C) 1996 - 2013
UPX 3.91        Markus Oberhumer, Laszlo Molnar & John Reiser   Sep 30th 2013

        File size         Ratio      Format      Name
   --------------------   ------   -----------   -----------
    887219 <-    335288   37.79%  linux/ElfAMD   flag

Unpacked 1 file.
abatchy@ubuntu:~/Desktop/tmp$ ls -al
total 872
drwxrwxr-x 2 abatchy abatchy   4096 Sep 14 19:53 .
drwxr-xr-x 8 abatchy abatchy   4096 Sep  2 11:49 ..
-rwxrw-r-- 1 abatchy abatchy 883745 Jul 14  2015 flag
```

If you run `flags` again you'll see much more readable strings and the flag is probably one of them, but we don't know that yet ;)

Anyway, since we know it'll make some calls to `malloc` and `strcpy` we can put a breakpoint in GDB and see how it behaves.

```console
abatchy@ubuntu:~/Desktop/tmp$ gdb -q flag
Reading symbols from flag...(no debugging symbols found)...done.
gdb-peda$ disass main
Dump of assembler code for function main:
   0x0000000000401164 <+0>:	push   rbp
   0x0000000000401165 <+1>:	mov    rbp,rsp
   0x0000000000401168 <+4>:	sub    rsp,0x10
   0x000000000040116c <+8>:	mov    edi,0x496658
   0x0000000000401171 <+13>:	call   0x402080 <puts>
   0x0000000000401176 <+18>:	mov    edi,0x64
   0x000000000040117b <+23>:	call   0x4099d0 <malloc>
   0x0000000000401180 <+28>:	mov    QWORD PTR [rbp-0x8],rax
-> 0x0000000000401184 <+32>:	mov    rdx,QWORD PTR [rip+0x2c0ee5]        # 0x6c2070 <flag>
   0x000000000040118b <+39>:	mov    rax,QWORD PTR [rbp-0x8]
   0x000000000040118f <+43>:	mov    rsi,rdx
   0x0000000000401192 <+46>:	mov    rdi,rax
   0x0000000000401195 <+49>:	call   0x400320
   0x000000000040119a <+54>:	mov    eax,0x0
   0x000000000040119f <+59>:	leave  
   0x00000000004011a0 <+60>:	ret    
End of assembler dump.
```

Uhh, someone is being extra nice and added a comment with the flag address.

```console
gdb-peda$ x/s *0x6c2070
0x496628:	"flag_for_flag"
```

\- Abatchy
