_This will be continuously improved as I learn more about shellcoding._  
  

### 1\. XOR Instruction

Setting a register to null is usually a problem when writing shellcode due to
couple of reasons:  
  
1\. It contains null characters, will fuck up your string.  
2\. Size limitations.  
  
Example:  
  

nasm &gt; mov al, 0  
00000000  B000                  ; **2 bytes**  
nasm &gt; mov ax, 0  
00000000  66B80000          ; **4 bytes**  
nasm &gt; mov eax, 0  
00000000  B800000000      ; **5 bytes**

  
Instead, XORing a register with itself 1) clears it out as well, 2) is null-
free and 3) generates less shellcode.  
  

nasm &gt; xor al,al  
00000000  30C0                 ; **2 bytes**  
nasm &gt; xor ax,ax  
00000000  6631C0             ; **4 bytes**  
nasm &gt; xor eax,eax  
00000000  31C0                 ; **2 bytes**

  
Although **xor ax,ax** didn't generate less shellcode, it doesn't contain a
null character.  
  

### 2\. CBW, CWD, CDQ Instructions

CBW, CWD and CDQ extends the sign bit of al/ax/eax to ah/dx/edx. This is
particularly useful due to a couple of reasons:  
  
1\. EAX is essential in system calls.  
2\. Many times you'll need a register zeroed out. As most system calls won't
"set" the flag bit in EAX, calling C{B/W/D}Q is an easy way to use EDX when
NULL is needed.  
  
Example:  
  

    
    
    mov al, 5       ; al's sign bit is 0 (positive)
    
    
    cbw             ; ah is now 0x00
    
    
     
    
    
    mov ax, -5
    
    
    cbd             ; dx is now 0xffff
    
    
     
    
    
    mov eax, 1337
    
    
    cdq             ; eax is now 0x00000000

  
A common way to set a register to zero is by XORing itself (tip 1). This is
**2-4 bytes long**, while using any of the previous commands
knowing/controlling the value of the corresponding register, w**e can reduce
it to one byte**, cutting down the size to 50-75%.  
  

nasm &gt; xor eax, eax  
00000000  31C0  
nasm &gt; xor edx, edx  
00000000  31D2  
nasm &gt; xor ah,ah  
00000000  30E4  
nasm &gt; cbw  
00000000  6698  
nasm &gt; cwd  
00000000  6699  
nasm &gt; cdq  
00000000  99  
nasm &gt;

  

### 3\. Push 0x1, POP EAX

XORing (Tip 1) is great to zero out a register, but what if we want to
initialize it with a certain value?  
  

nasm &gt; xor eax, eax  
00000000  31C0  
nasm &gt; mov al, 0x1  
00000000  B001

  
How about we utilize the stack? Pushing 0x1 for example will be automatically
aligned as a DWORD.  
  

nasm &gt; push byte 0x1  
00000000  6A01              push byte +0x1  
nasm &gt; pop eax  
00000000  58                   pop eax

  
\- Abatchy

