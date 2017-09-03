Few months ago, I didn't know what Bash is, who that root guy people were
scared of, and definitely never heard of SSH tunneling. I also didn't like
paying for the PWK lab time without using it, so I went through a number of
resources till I felt ready for starting the course.  
  
_Warning: Don't expect to be spoon-fed if you're doing OSCP, you'll need to
spend a lot of time researching, neither the admins or the other students will
give you answers easily._  
  
**1\. PWK Syllabus**  
    1.1 *nix and Bash  
    1.2 Basic tools  
    1.3 Passive Recon  
    1.4 Active Recon  
    1.5 Buffer Overflow  
    1.6 Using public exploits  
    1.7 File Transfer  
    1.8 Privilege Escalation  
    1.9 Client Side Attacks  
    1.10 Web Application Attacks  
    1.11 Password Attacks  
    1.12 Port Redirection/Tunneling  
    1.13 Metasploit Framework  
    1.14 Antivirus Bypassing  
**2\. Wargames**  
    2.1 Over The Wire: Bandit  
    2.2 Over The Wire: Natas  
    2.3 Root-me.org  
**3\. Vulnerable VMs**  
  

## 1\. PWK Syllabus:

Simply the most important reference in the list, it shows the course modules
in a detailed way. Entire preparation I did was based on it. Can be found
[here](https://www.offensive-security.com/documentation/penetration-testing-
with-kali.pdf).  

###  

### 1.1 *nix and Bash:

You don't need to use Kali Linux right away, a good alternative is Ubuntu till
you get comfortable with Linux.  
  
1\. [Bash for Beginners](http://www.tldp.org/LDP/Bash-Beginners-Guide/html/):
Best Bash reference IMO.  
2\. [Bandit on Over The Wire](http://overthewire.org/wargames/bandit/): Great
start for people who aren't used to using a terminal, aren't familiar with
Bash or other *nix in general. Each challenge gives you hints on which
commands you can use, you need to research them.  
3.  [Explainshell](http://www.explainshell.com/): Does NOT replace man pages, but breaks down commands easily for new comers.  

###  

### 1.2 Basic tools:

You will use these tools **a lot**. Make sure you understand what they do and
how you can utilize them.  
  
**1\. Netcat:** Most important tool in the entire course. Understand what it does, what options you have, difference between a reverse shell and a bind shell. Experiment **a lot** with it.  
**2\. Ncat**: Netcat's mature brother, supports SSL. Part of Nmap.  
**3\. Wireshark:** Network analysis tool, play with it while browsing the internet, connecting to FTP, read/write PCAP files.  
**4\. TCPdump:** Not all machines have that cute GUI, you could be stuck with a terminal.  
  

### 1.3 Passive Recon:

Read about the following tools/techniques, experiment as much as possible.  
  
1\. [Google dorks](http://whatis.techtarget.com/definition/Google-dork-query)  
2\. [Whois](https://whois.icann.org/en/about-whois)  
3\. [Netcraft](https://searchdns.netcraft.com/)  
4\. [Recon-ng](https://bitbucket.org/LaNMaSteR53/recon-ng): Make sure you
check the [Usage guide](https://bitbucket.org/LaNMaSteR53/recon-
ng/wiki/Usage%20Guide) to know how it works.  
  

### 1.4 Active Recon:

  * Understand what DNS is, how it works, how to perform forward and reverse lookup, what zone transfers are and how to perform them. Great resource [here](http://resources.infosecinstitute.com/dns-hacking/#gref).
  * Nmap: One of the most used tools during the course (if not the most). I'd recommend to start by reading the [man pages](https://nmap.org/book/man.html), understand different scanning techniques and other capabilities it has (scripts, OS detection, Service detection, ...)
  * Services enumeration: [SMTP](https://pentestlab.blog/2012/11/20/smtp-user-enumeration/), [SNMP](http://carnal0wnage.attackresearch.com/2007/07/over-in-lso-chat-we-were-talking-about.html), SMB, and a lot others. Don't just enumerate them, understand what they're used for and how they work.
  * Great list for [enumeration](http://0daysecurity.com/penetration-testing/enumeration.html) and tools.

### 1.5 Buffer Overflow:

Most fun part in my opinion. There are countless resources on how to get
started, I'd recommend [Corelan's
series](https://www.corelan.be/index.php/2009/07/19/exploit-writing-tutorial-
part-1-stack-based-overflows/). You probably need the first part only for PWK.  
  

### 1.6 Using public exploits:

Occasionally, you'll need to use a public exploit, maybe even modify the
shellcode or other parts. Just go to Exploit-db and pick one of the older more
reliable exploits (FTP ones for example). The vulnerable version is usually
present with the exploit code.  
  

### 1.7 File Transfer:

Not every machine has netcat installed, you'll need to find a way around it to
upload exploits or other tools you need. Great post on this is
[here](https://blog.ropnop.com/transferring-files-from-kali-to-windows/).  
  

### **1.8 Privilege Escalation:**

A never ending topic, there are a lot of techniques, ranging from having an
admin password to kernel exploits. Great way to practice this is by using
Vulnhub VMs for practice. Check my OSCP-like VMs list
[here](http://www.abatchy.com/2017/02/oscp-like-vulnhub-vms.html).  
  
  
[Windows:Elevating privileges by exploiting weak folder
permissions](http://www.greyhathacker.net/?p=738)  
---  
[Windows: Privilege Escalation
Fundamentals](http://www.fuzzysecurity.com/tutorials/16.html)  
[Windows: Windows-Exploit-Suggester](https://github.com/GDSSecurity/Windows-
Exploit-Suggester)  
[Windows: Privilege Escalation
Commands](http://pwnwiki.io/#!privesc/windows/index.md)  
[Linux: Basic Linux Privilege Escalation](https://blog.g0tmi1k.com/2011/08
/basic-linux-privilege-escalation/)  
[Linux:
linuxprivchecker.py](http://www.securitysift.com/download/linuxprivchecker.py)  
[Linux: LinEnum](https://github.com/rebootuser/LinEnum)  
[Practical Windows Privilege
Escalation](https://www.youtube.com/watch?v=PC_iMqiuIRQ)  
[MySQL Root to System Root with
UDF](https://www.adampalmer.me/iodigitalsec/2013/08/13/mysql-root-to-system-
root-with-udf-for-windows-and-linux/)  
  
###  

### 1.9 Client Side Attacks:

Try out the techniques provided in [Metasploit Unleashed ](https://www
.offensive-security.com/metasploit-unleashed/client-side-attacks/)or an IE
client side exploit.  

###  

### 1.10 Web Application Attacks

Another lengthy subject, understand what XSS is, [SQL injection](https://www
.exploit-db.com/papers/13045/), LFI, RFI, directory traversal, how to use a
proxy like Burp Suite. Solve as much as you can from [Natas
](http://overthewire.org/wargames/natas/)on Over The Wire. It has great
examples on Code Injection, Session hijacking and other web vulnerabilities.  
  
Key is research till you feel comfortable.  

###  

### 1.11 Password Attacks:

Understand the basics of password attacks, difference between online and
offline attacks. How to use Hydra, JTR, Medusa, what rainbow tables are, the
list goes on. Excellent post on this topic
[here](https://alexandreborgesbrazil.files.wordpress.com/2013/08/introduction_to_password_cracking_part_1.pdf).  

###  

### 1.12 Port redirection/tunneling:

Not all machines are directly accessible, some are dual homed, connected to an
internal network. You'll use such techniques a lot in non-public networks.
[This post](https://chamibuddhika.wordpress.com/2012/03/21/ssh-tunnelling-
explained/) did a great job explaining it.  
  

### **1.13 Metasploit Framework:**

Decided to skip this part, but if you still want to study it, check out
[Metasploit Unleashed](https://www.offensive-security.com/metasploit-
unleashed/) course.** **  

### ** **

### **1.14 Antivirus Bypassing:**

Skipped this part too. Pretty basic in OSCP.  
  

## 2\. Wargames

Consider these a prep for vulnerable machines.  
  

### 2.1 Over The Wire: Bandit

Great start for people who aren't familiar with Linux or Bash. Check my
walkthroughs
[here](http://www.abatchy.com/search/label/OverTheWire%3A%20Bandit%200%20%E2%86%92%2027).  
  

### 2.2 Over The Wire: Natas

Focused on web application, many challenges aren't required for OSCP, but it
helps for sure. Check my walkthroughs
[here](http://www.abatchy.com/search/label/OverTheWire%3A%20Natas%200%20%E2%86%92%2033).  
  

### 2.3 Root-me.org

Has great challenges on privilege escalation, SQL injection, Javascript
obfuscation, password cracking and analyzing PCAP files  
  

## 3\. Vulnerable Machines

Boot-to-root VMs are excellent for pentesting, you import a VM, run it and
start enumerating from your attacking machine. Most of them result in getting
root access. Check my [post ](http://www.abatchy.com/2017/02/oscp-like-
vulnhub-vms.html)on which machines are the closest to OSCP. Rooting VMs is as
important as studying the material. You can't depend on theoretical knowledge
only, yet you still need this knowledge to help you tackle harder machines.  
  
If you still have questions, feel free to comment below or ask on our
[NetSecFocus slack](https://netsecfocus.herokuapp.com/)!  
  
\- Abatchy

