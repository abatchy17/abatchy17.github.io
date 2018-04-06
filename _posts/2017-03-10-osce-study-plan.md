---
layout: post
title: "OSCE Study Plan"
date: 2017-03-10 12:00:00
share: true
comments: true
tags: [OSCE Prep]
---

[TL;DR](http://www.wikihow.com/Avoid-Becoming-a-Script-Kiddie)  
    

## Table of Content

1. [OSCE OSCE Course Outline](#osce-course-outline)
2. [Online Study Resources](#online-study-resources)
3. [Offline Study Resources](#offline-study-resources)
4. [Practice](#practice)

---

## OSCE Course Outline

  

#### 1\. Advanced Web Attacks

* HTML Injection and XSS  
* Bypassing CSRF protection  
* LFI to RCE  

#### 2\. Backdooring PE

#### 3\. Bypassing AV

#### 4\. Exploit development

* Automated fuzzing (Spike)  
* Assembly and Shellcode basics  
* Stack overflow  
* SEH  
* Egghunting  
* Bypassing ASLR  

#### 5\. Advanced Network Attacks

* Using Scapy  
* Bypassing ACL  
* Exploiting SNMP  
* MiTM attacks  

#### 6\. Study cases

* MS07-017  
* Open TFTP 1.4 (CVE-2008-1611)  
* HP OpenView NNM  
* Bypassing Cisco ACL using Spoofed SNMP Requests  

---

## Online Study Resources

  

#### 1\. Advanced Web Attacks

* [Advanced XSS Knowledge](https://www.exploit-db.com/papers/13646/)  
* [LFI to RCE Exploit with Perl Script](https://www.exploit-db.com/papers/12992/)   
* [Bypass CSRF Protection via XSS](http://blog.safebuff.com/2016/05/26/Bypass-CSRF-Protection-via-XSS/)   

#### 2\. Backdooring PE

* Backdooring PE Files - [Part 1](http://sector876.blogspot.com/2013/03/backdooring-pe-files-part-1.html) [Part 2  ](http://sector876.blogspot.ca/2013/03/backdooring-pe-files-part-2.html)_(Nice intro on basic PE backdooring)_  
* [Manually Adding Shellcode to Windows Executables](https://v00d00sec.com/2015/09/14/manually-backdooring-windows-executables/) _(Short and to the point)_    
* [Introduction to Manual Backdooring](http://www.abatchy.com/2017/05/introduction-to-manual-backdooring_24.html) by your favourite llama  
* [The Beginners Guide to Codecaves](https://www.codeproject.com/Articles/20240/The-Beginners-Guide-to-Codecaves) (Good read but don't spend too much time on it)

#### 3\. Bypassing AV

* [Bypassing Anti-Virus Scanners](https://dl.packetstormsecurity.net/papers/bypass/bypassing-av.pdf)  
* [Introduction to AV &amp; Detection Techniques](https://pentest.blog/art-of-anti-detection-1-introduction-to-av-detection-techniques/)    

#### 4\. Exploit development

1. Fuzzing  
* [The Very Unofficial Dummies Guide To Scapy Assembly](https://theitgeekchronicles.files.wordpress.com/2012/05/scapyguide1.pdf)  
* [An Introduction to SPIKE, the Fuzzer Creation Kit - Black Hat ](https://www.blackhat.com/presentations/bh-usa-02/bh-us-02-aitel-spike.ppt)  
* [An Introduction to Fuzzing](http://resources.infosecinstitute.com/intro-to-fuzzing/)  

2. Assembly and Shellcode basics  
* [SLAE ](http://www.securitytube-training.com/online-courses/securitytube-linux-assembly-expert/index.html) _(Great course for assembly fresh-up and shellcoding basics)_  
* SLAE alternatives for ASM [1](http://cs.lmu.edu/~ray/notes/nasmtutorial/) [2](http://www.cs.virginia.edu/~evans/cs216/guides/x86.html)  
* [Understanding Windows Shellcode](http://www.hick.org/code/skape/papers/win32-shellcode.pdf) by Skape  
* [Corelan: Introduction to Win32 shellcoding](https://www.corelan.be/index.php/2010/02/25/exploit-writing-tutorial-part-9-introduction-to-win32-shellcoding/)  
* [FuzzySecurity:  Writing W32 shellcode](http://fuzzysecurity.com/tutorials/expDev/6.html)   

3. Stack Based Overflow
* Corelan [1 ](https://www.corelan.be/index.php/2009/07/19/exploit-writing-tutorial-part-1-stack-based-overflows/)and [2](https://www.corelan.be/index.php/2009/07/23/writing-buffer-overflow-exploits-a-quick-and-basic-tutorial-part-2/)  
* FuzzySecurity's Exploit Development [1 ](http://fuzzysecurity.com/tutorials/expDev/1.html) and [2](http://fuzzysecurity.com/tutorials/expDev/2.html)   
* Securitysift's Windows Exploit Development [1](http://www.securitysift.com/windows-exploit-development-part-1-basics/), [2](http://www.securitysift.com/windows-exploit-development-part-2-intro-stack-overflow/), [3](http://www.securitysift.com/windows-exploit-development-part-3-changing-offsets-and-rebased-modules/) and [4](http://www.securitysift.com/windows-exploit-development-part-4-locating-shellcode-jumps/)  
4. SEH  
* Corelan [3a](https://www.corelan.be/index.php/2009/07/25/writing-buffer-overflow-exploits-a-quick-and-basic-tutorial-part-3-seh/) and [3b](https://www.corelan.be/index.php/2009/07/28/seh-based-exploit-writing-tutorial-continued-just-another-example-part-3b/)   
* FuzzySecurity's Exploit Development [3](http://fuzzysecurity.com/tutorials/expDev/3.html)  
* Securitysift's Windows Exploit Development [6](http://www.securitysift.com/windows-exploit-development-part-6-seh-exploits/)  
* [The need for a POP POP RET instruction sequence ](https://dkalemis.wordpress.com/2010/10/27/the-need-for-a-pop-pop-ret-instruction-sequence/)  
5. Egghunting  
* [Skape's Whitepaper on egg-hunting](http://web.archive.org/web/20061010194043/http://www.hick.org/code/skape/papers/egghunt-shellcode.pdf)  
* Corelan [8](https://www.corelan.be/index.php/2010/01/09/exploit-writing-tutorial-part-8-win32-egg-hunting/)  
* FuzzySecurity's Exploit Development [4](http://fuzzysecurity.com/tutorials/expDev/4.html)  
* Securitysift's Windows Exploit Development [5](http://www.securitysift.com/windows-exploit-development-part-5-locating-shellcode-egghunting/)  
* [egg hunter - Exploit-DB](https://www.exploit-db.com/docs/18482.pdf)   
6. Bypassing ASLR
* Corelan Series [6](https://www.corelan.be/index.php/2009/09/21/exploit-writing-tutorial-part-6-bypassing-stack-cookies-safeseh-hw-dep-and-aslr/)  
* [Bypassing ASLR ](https://www.exploit-db.com/docs/18744.pdf)  

#### 5\. Advanced Network Attacks

* [Bypassing Router’s Access Control List](https://securityshards.wordpress.com/2016/02/05/bypassing-routers-access-control-list-acl/)   
* [Firewall ACL Bypass](http://www.ipv6.cisco.com/c/en/us/td/docs/ios/sec_data_plane/configuration/guide/12_4/sec_data_plane_12_4_book/sec_fwall_acl_bypass.pdf)   
* [Hacking networks with SNMP](https://0x41.no/hacking-networks-with-snmp/)   
* [TCP Session Hijacking](https://www.exploit-db.com/papers/13587/)   
* [Cisco SNMP configuration attack with a GRE tunnel](https://www.symantec.com/connect/articles/cisco-snmp-configuration-attack-gre-tunnel)  
* [Exploiting Cisco Routers](https://www.symantec.com/connect/articles/cisco-snmp-configuration-attack-gre-tunnel)   

#### 6\. Study cases:

* [NNM Exploit](https://www.youtube.com/watch?v=axTthxE-z6A)  
* [Bypassing Cisco Access Lists using Spoofed SNMP Requests](http://web.archive.org/web/20051024151559/http://new.remote-exploit.org/index.php/SNMP_Spoof)   
  
---

## Offline Study Resources

  
1. Hacking: The Art of Exploitation: Chapter 1,2,3 and 5 are relevant to OSCE.  
2. Assembly Language Step-by-Step: Programming with Linux  
3. The Shellcoder's Handbook: Discovering and Exploiting Security Holes  

---

## Practice

1. <http://overthewire.org/wargames/narnia/> 
2. <http://www.thegreycorner.com/2010/12/introducing-vulnserver.html>
3. <http://canyouhack.us>
4. <https://holidayhackchallenge.com/2016/  >
5. <https://exploit-exercises.com/protostar/> 
6. <https://exploit-exercises.com/fusion/>
7. <http://io.netgarage.org:84/> (Thanks WhizzMan!)

---

_Note: I'm no longer seeking OSCE, but this post has proven to be useful to many. If you think a link should (not) be here, please let me know in the comments._  


\- Abatchy
