[TL;DR](http://www.wikihow.com/Avoid-Becoming-a-Script-Kiddie)  
  
_Note: I'm currently studying through this myself, so links will continuously
be updated. If you think a link should (not) be here, please let me know in
the comments._  
  

### [1\. OSCE topics and study
cases](https://www.blogger.com/blogger.g?blogID=8077124728429326883#tag1)

### [2\. Online Study
Resources](https://www.blogger.com/blogger.g?blogID=8077124728429326883#tag2)

### [3\. Offline Study
Resources](https://www.blogger.com/blogger.g?blogID=8077124728429326883#tag3)

### [4\.
Practice](https://www.blogger.com/blogger.g?blogID=8077124728429326883#tag4)

  

## OSCE topics and study cases

  

#### 1\. Advanced Web Attacks

    1.1 HTML Injection and XSS  
    1.2 Bypassing CSRF protection  
    1.3 LFI to RCE  

#### 2\. Backdooring PE

#### 3\. Bypassing AV

#### 4\. Exploit development

    4.1 Automated fuzzing (Spike)  
    4.2 Assembly and Shellcode basics  
    4.3 Stack overflow  
    4.4 SEH  
    4.5 Egghunting  
    4.6 Bypassing ASLR  

#### 5\. Advanced Network Attacks

    5.1 Using Scapy  
    5.2 Bypassing ACL  
    5.3 Exploiting SNMP  
    5.4 MiTM attacks  

#### 6\. Study cases

    1\. MS07-017  
    2\. Open TFTP 1.4 (CVE-2008-1611)  
    3\. HP OpenView NNM  
    4\. Bypassing Cisco ACL using Spoofed SNMP Requests  
  

## Online Study Resources

  

#### 1\. Advanced Web Attacks

    1.1 [Advanced XSS Knowledge](https://www.exploit-db.com/papers/13646/)  
    1.2 [LFI to RCE Exploit with Perl Script](https://www.exploit-db.com/papers/12992/)   
    1.3 [Bypass CSRF Protection via XSS](http://blog.safebuff.com/2016/05/26/Bypass-CSRF-Protection-via-XSS/)   

#### 2\. Backdooring PE

    2.1 Backdooring PE Files - [Part 1](http://sector876.blogspot.com/2013/03/backdooring-pe-files-part-1.html) [Part 2  ](http://sector876.blogspot.ca/2013/03/backdooring-pe-files-part-2.html)_(Nice intro on basic PE backdooring)_  
    2.2 [Manually Adding Shellcode to Windows Executables](https://v00d00sec.com/2015/09/14/manually-backdooring-windows-executables/) _(Short and to the point)_    
    2.3 [Introduction to Manual Backdooring](http://www.abatchy.com/2017/05/introduction-to-manual-backdooring_24.html) by your favourite llama  
    2.4 [The Beginners Guide to Codecaves](https://www.codeproject.com/Articles/20240/The-Beginners-Guide-to-Codecaves) _(Good read but don't spend too much time on it) _   

#### 3\. Bypassing AV

    3.1 [Bypassing Anti-Virus Scanners](https://dl.packetstormsecurity.net/papers/bypass/bypassing-av.pdf)  
    3.2 [Introduction to AV &amp; Detection Techniques](https://pentest.blog/art-of-anti-detection-1-introduction-to-av-detection-techniques/)    

#### 4\. Exploit development

**    4.1 Fuzzing**  
        4.1.1 [The Very Unofficial Dummies Guide To Scapy Assembly](https://theitgeekchronicles.files.wordpress.com/2012/05/scapyguide1.pdf)  
        4.1.2 [An Introduction to SPIKE, the Fuzzer Creation Kit - Black Hat ](https://www.blackhat.com/presentations/bh-usa-02/bh-us-02-aitel-spike.ppt)  
        4.1.3 [An Introduction to Fuzzing](http://resources.infosecinstitute.com/intro-to-fuzzing/)  
**    4.2 Assembly and Shellcode basics**  
        4.2.1 [SLAE ](http://www.securitytube-training.com/online-courses/securitytube-linux-assembly-expert/index.html) _(Great course for assembly fresh-up and shellcoding basics)_  
        4.2.2 SLAE alternatives for ASM [1](http://cs.lmu.edu/~ray/notes/nasmtutorial/) [2](http://www.cs.virginia.edu/~evans/cs216/guides/x86.html)  
        4.2.3 [Understanding Windows Shellcode](http://www.hick.org/code/skape/papers/win32-shellcode.pdf) by Skape  
        4.2.4 [Corelan: Introduction to Win32 shellcoding](https://www.corelan.be/index.php/2010/02/25/exploit-writing-tutorial-part-9-introduction-to-win32-shellcoding/)  
        4.2.5 [FuzzySecurity:  Writing W32 shellcode](http://fuzzysecurity.com/tutorials/expDev/6.html)   
**    4.3 Stack Based Overflow**  
        4.3.1 Corelan [1 ](https://www.corelan.be/index.php/2009/07/19/exploit-writing-tutorial-part-1-stack-based-overflows/)and [2](https://www.corelan.be/index.php/2009/07/23/writing-buffer-overflow-exploits-a-quick-and-basic-tutorial-part-2/)  
        4.3.2 FuzzySecurity's Exploit Development [1 ](http://fuzzysecurity.com/tutorials/expDev/1.html) and [2](http://fuzzysecurity.com/tutorials/expDev/2.html)   
        4.3.2 Securitysift's Windows Exploit Development [1](http://www.securitysift.com/windows-exploit-development-part-1-basics/), [2](http://www.securitysift.com/windows-exploit-development-part-2-intro-stack-overflow/), [3](http://www.securitysift.com/windows-exploit-development-part-3-changing-offsets-and-rebased-modules/) and [4](http://www.securitysift.com/windows-exploit-development-part-4-locating-shellcode-jumps/)  
**    4.4 SEH**  
        4.4.1 Corelan [3a](https://www.corelan.be/index.php/2009/07/25/writing-buffer-overflow-exploits-a-quick-and-basic-tutorial-part-3-seh/) and [3b](https://www.corelan.be/index.php/2009/07/28/seh-based-exploit-writing-tutorial-continued-just-another-example-part-3b/)   
        4.4.2 FuzzySecurity's Exploit Development [3](http://fuzzysecurity.com/tutorials/expDev/3.html)  
        4.4.3 Securitysift's Windows Exploit Development [6](http://www.securitysift.com/windows-exploit-development-part-6-seh-exploits/)  
        4.4.4 [The need for a POP POP RET instruction sequence ](https://dkalemis.wordpress.com/2010/10/27/the-need-for-a-pop-pop-ret-instruction-sequence/)  
**    4.5 Egghunting**  
        4.5.1 [Skape's Whitepaper on egg-hunting](http://web.archive.org/web/20061010194043/http://www.hick.org/code/skape/papers/egghunt-shellcode.pdf)  
        4.5.2 Corelan [8](https://www.corelan.be/index.php/2010/01/09/exploit-writing-tutorial-part-8-win32-egg-hunting/)  
        4.5.3 FuzzySecurity's Exploit Development [4](http://fuzzysecurity.com/tutorials/expDev/4.html)  
        4.5.4 Securitysift's Windows Exploit Development [5](http://www.securitysift.com/windows-exploit-development-part-5-locating-shellcode-egghunting/)  
        4.5.5 [egg hunter - Exploit-DB](https://www.exploit-db.com/docs/18482.pdf)   
**    4.6 Bypassing ASLR**  
        4.6.1 Corelan Series [6](https://www.corelan.be/index.php/2009/09/21/exploit-writing-tutorial-part-6-bypassing-stack-cookies-safeseh-hw-dep-and-aslr/)  
        4.6.2 [Bypassing ASLR ](https://www.exploit-db.com/docs/18744.pdf)  

#### 5\. Advanced Network Attacks

    5.1 [Bypassing Router’s Access Control List](https://securityshards.wordpress.com/2016/02/05/bypassing-routers-access-control-list-acl/)   
    5.2 [Firewall ACL Bypass](http://www.ipv6.cisco.com/c/en/us/td/docs/ios/sec_data_plane/configuration/guide/12_4/sec_data_plane_12_4_book/sec_fwall_acl_bypass.pdf)   
    5.3 [Hacking networks with SNMP](https://0x41.no/hacking-networks-with-snmp/)   
    5.4 [TCP Session Hijacking](https://www.exploit-db.com/papers/13587/)   
    5.5 [Cisco SNMP configuration attack with a GRE tunnel](https://www.symantec.com/connect/articles/cisco-snmp-configuration-attack-gre-tunnel)  
    5.6 [Exploiting Cisco Routers](https://www.symantec.com/connect/articles/cisco-snmp-configuration-attack-gre-tunnel)   

#### 6\. Study cases:

    6.1 [NNM Exploit](https://www.youtube.com/watch?v=axTthxE-z6A)  
    6.2 [Bypassing Cisco Access Lists using Spoofed SNMP Requests](http://web.archive.org/web/20051024151559/http://new.remote-exploit.org/index.php/SNMP_Spoof)   
  

## Offline Study Resources

  
1\. Hacking: The Art of Exploitation: Chapter 1,2,3 and 5 are relevant to
OSCE.  
2\. Assembly Language Step-by-Step: Programming with Linux  
3\. The Shellcoder's Handbook: Discovering and Exploiting Security Holes  
  

## Practice

1\. http://overthewire.org/wargames/narnia/  
2\. http://www.thegreycorner.com/2010/12/introducing-vulnserver.html  
3\. http://canyouhack.us  
4\. https://holidayhackchallenge.com/2016/  
5\. https://exploit-exercises.com/protostar/  
6\. https://exploit-exercises.com/fusion/  
7\. http://io.netgarage.org:84/ (Thanks WhizzMan!)

