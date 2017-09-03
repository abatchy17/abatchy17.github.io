_**Note:** This post summarizes the techniques discussed in Corelan
[post](https://www.corelan.be/index.php/2009/07/23/writing-buffer-overflow-
exploits-a-quick-and-basic-tutorial-part-2/) on jumping to shellcode. Read it
for better understanding._  
  
So you managed to control EIP and you want to jump to your shellcode. Check
the condition for each technique and see if you can make it work to reach your
shellcode.  
  

### 1\. JMP/CALL register

_**Conditions:**_  
    ○ A register points to shellcode.  
_**Example: **_  
    ○ [Register] → [Shellcode]  
    ○ EIP → JMP/CALL [register]  
    ○ EIP now points to [register] where shellcode.  
_**Pros:**_  
    ○ Easy to apply.  
    ○ Common instruction.  
_**Cons:**_  
    ○ None.  
  

### 2\. POP RET / POP POP RET / POP POP POP RET

_**Condition: **_  
    ○ Address at [ESP+4], [ESP+8], [ESP+12] (and so on) points to address to shellcode **OR **directly to shellcode.  
_**Example 1: **_  
    ○ [ESP] → [4 bytes][Address to shellcode].  
    ○ EIP → POP [register] followed by RET.  
        § POP [register]: ESP now points to old_ESP + 4  
        § RET: EIP now contains address to shellcode.  
    ○ EIP now points to shellcode.  
_**Example 2: **_  
    ○ [ESP] → [8 bytes][Address to JMP ESP][shellcode].  
    ○ EIP → POP [register] followed by POP [register] followed by RET:  
        § POP [register], POP [register] will get rid of 8 bytes, new ESP → old ESP + 8 (Address to JMP ESP)  
        § RET places address to JMP ESP in EIP and now ESP now points to shellcode.  
        § JMP ESP: EIP now contains ESP  
    ○ EIP points to current ESP value which points to start of shellcode.  
_**Pros:**_  
    ○ Straightforward (slightly harder than method 1).  
    ○ Lots of possible combinations to find instruction.  
Cons:  
    ○ Less frequent pattern.  
  

### 3\. PUSH  RET

_**Condition: **_  
    ○ A register points to shellcode and can't/don't want to use method 1.  
_**Example: **_  
    ○ [Register]→ [Shellcode]  
    ○ EIP → PUSH [register], followed by RET  
        § Stack will first push register, then pop it to EIP.   
    ○ EIP now points to shellcode.  
_**Pros:**_  
    ○ Easy to apply.  
    ○ Fallback if method 1 can't be done.  
_**Cons:**_  
    ○ None.  
  

### 4\. JMP [register + offset]

_**Condition: **_  
    ○ A register + offset points to shellcode.  
_**Example: **_  
    ○ [Register] → [Shellcode]  
    ○ EIP → JMP [register + offset]  
    ○ EIP will point to [register + offset] where the shellcode starts.  
_**Pros:**_  
    ○ Easy to apply.  
_**Cons:**_  
    ○ Doesn't work if [register + offset] points to address to shellcode as ESP doesn't change and a RET won't work.  
      

### 5\. Blind return

_**Condition: **_  
    ○ Shellcode is always loaded to the same address.  
    ○ Address doesn't contain a null byte.  
    ○ You control at least the first 4 bytes at [ESP]  
_**Example: **_  
    ○ Shellcode is always at 0xdeadbeef  
    ○ Since you control the first 4 bytes at ESP, put 0xdeadbeef at ESP.  
    ○ By pointing EIP to a RET, address at ESP will be popped to EIP.  
    ○ EIP now points to address 0xdeadbeef where the shellcode starts.  
_**Pros:**_  
    ○ Easiest method, only need a RET.  
    ○ Fixed address.  
_**Cons:**_  
    ○ Heavy dependency on hardcoded address.  
    ○ Address can't contain null byte (good luck with stack at low address).  
    ○ Assumes no ASLR.  
  

### 6\. POPAD

_**Condition: **_  
    ○ Shellcode is located at [ESP + 32x + offset]  
    ○ Enough controllable space to execute POPAD y times then JMP ESP.  
_**Example: **_  
    ○ [ESP + 240] → [Shellcode]  
    ○ [ESP + 32 * 7] → [NOP sled]  
    ○ [ESP] → POPAD 7 times followed by JMP ESP  
    ○ EIP → [JMP ESP]  
        § EIP will execute POPAD 7 times, ESP = old_ESP + 224  
        § EIP goes over NOP sled  
    ○ EIP after executing NOP sled, it will point to [ESP + 240] where the shellcode starts.  
_**Pros:**_  
    ○ POPAD is a single byte.  
    ○ ESP gets incremented with 32 every time POPAD is executed.  
_**Cons:**_  
    ○ Requires NOP sled.  
    ○ Less reliable than Method 1/2/3/4  
  

### 7\. Short jumps (backwards, forwards, conditional)

_**Condition:**_  
    ○ Shellcode is located at [ESP + offset] where -128 &lt; offset &lt; 127.  
_**Example:**_  
    ○ [ESP + 30] → [Shellcode]  
    ○ [ESP] → JMP 30  
    ○ EIP → [JMP ESP]  
        § EIP will execute a short JMP  
    ○ EIP will point to [ESP + 30] where the shellcode starts.  
_**Pros:**_  
    ○ Simple instruction.  
    ○ Reliable, no NOP sled is needed.  
_**Cons:**_  
    ○ Restricted by being a short JMP.  
      

### 8\. Hardcoded address

_**Condition: **_  
    ○ Shellcode is always located at specific address.  
_**Example: **_  
    ○ 0xdeadbeef → [shellcode]  
    ○ EIP → JMP ESP  
    ○ ESP → JMP 0xdeadbeef  
_**Pros:**_  
    ○ Simple instruction.  
_**Cons:**_  
    ○ Unreliable.  
    ○ Address can't contain null bytes.  
  
Following repo contains some snippets on various payload generation for the
techniques below:
<https://github.com/abatchy17/ExploitDevSnippets/tree/master/Corelan>

