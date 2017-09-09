---
layout: post
title: "How to prepare for PWK/OSCP, a noob-friendly guide "
date: 2017-03-04 12:00:00
share: true
comments: true
tags: [OSCP Prep]
---

Few months ago, I didn't know what Bash is, who that root guy people were scared of, and definitely never heard of SSH tunneling. I also didn't like paying for the PWK lab time without using it, so I went through a number of resources till I felt ready for starting the course.  
  
_Warning: Don't expect to be spoon-fed if you're doing OSCP, you'll need to spend a lot of time researching, neither the admins or the other students will give you answers easily._  
  
#### 1\. PWK Syllabus
* Linux and Bash  
* Basic tools  
* Passive Recon  
* Active Recon  
* Buffer Overflow  
* Using public exploits  
* File Transfer  
* Privilege Escalation  
* Client Side Attacks  
* Web Application Attacks  
* Password Attacks  
* Port Redirection/Tunneling  
* Metasploit Framework  
* Antivirus Bypassing
    
#### 2\. Wargames
* Over The Wire: Bandit  
* Over The Wire: Natas  
* Root-me.org  

#### 3\. Vulnerable VMs
  
---

## 1\. PWK Syllabus:

Simply the most important reference in the list, it shows the course modules in a detailed way. Entire preparation I did was based on it. Can be found [here](https://www.offensive-security.com/documentation/penetration-testing-with-kali.pdf).

### Linux and Bash:

You don't need to use Kali Linux right away, a good alternative is Ubuntu till you get comfortable with Linux.  

* [`Linux Journey`](https://linuxjourney.com/)
* [`Bash for Beginners`](http://www.tldp.org/LDP/Bash-Beginners-Guide/html/): Best Bash reference IMO.  
* [`OverTheWire: Bandit`](http://overthewire.org/wargames/bandit/): Great start for people who aren't used to using a terminal, aren't familiar with Bash or other *nix in general. Each challenge gives you hints on which commands you can use, you need to research them.  
* [`Explainshell`](http://www.explainshell.com/): Does NOT replace man pages, but breaks down commands easily for new comers.  

### Basic tools:

You will use these tools **a lot**. Make sure you understand what they do and how you can utilize them.  
  
* `Netcat`: Most important tool in the entire course. Understand what it does, what options you have, difference between a reverse shell and a bind shell. Experiment a lot with it.  
* `Ncat`: Netcat's mature brother, supports SSL. Part of Nmap.  
* `Wireshark`: Network analysis tool, play with it while browsing the internet, connecting to FTP, read/write PCAP files.  
* `TCPdump`: Not all machines have that cute GUI, you could be stuck with a terminal.  
  

### Passive Recon:

Read about the following tools/techniques, experiment as much as possible.  
  
* [`Google dorks`](http://whatis.techtarget.com/definition/Google-dork-query)  
* [`Whois`](https://whois.icann.org/en/about-whois)  
* [`Netcraft`](https://searchdns.netcraft.com/)  
* [`Recon-ng`](https://bitbucket.org/LaNMaSteR53/recon-ng): Make sure you check the [Usage guide](https://bitbucket.org/LaNMaSteR53/recon-ng/wiki/Usage%20Guide) to know how it works.  

### Active Recon:

  * Understand what DNS is, how it works, how to perform forward and reverse lookup, what zone transfers are and how to perform them. Great resource [here](http://resources.infosecinstitute.com/dns-hacking/#gref).
  * Nmap: One of the most used tools during the course (if not the most). I'd recommend to start by reading the [man pages](https://nmap.org/book/man.html), understand different scanning techniques and other capabilities it has (scripts, OS detection, Service detection, ...)
  * Services enumeration: [`SMTP`](https://pentestlab.blog/2012/11/20/smtp-user-enumeration/), [`SNMP`](http://carnal0wnage.attackresearch.com/2007/07/over-in-lso-chat-we-were-talking-about.html), SMB, and a lot others. Don't just enumerate them, understand what they're used for and how they work.
  * Great list for [enumeration](http://0daysecurity.com/penetration-testing/enumeration.html) and tools.

### Buffer Overflow:

Most fun part in my opinion. There are countless resources on how to get started, I'd recommend [Corelan's series](https://www.corelan.be/index.php/2009/07/19/exploit-writing-tutorial-part-1-stack-based-overflows/). You probably need the first part only for PWK.  
  

### Using public exploits:

Occasionally, you'll need to use a public exploit, maybe even modify the shellcode or other parts. Just go to Exploit-db and pick one of the older more reliable exploits (FTP ones for example). The vulnerable version is usually present with the exploit code.  
  

### File Transfer:

Not every machine has netcat installed, you'll need to find a way around it to upload exploits or other tools you need. Great post on this is [here](https://blog.ropnop.com/transferring-files-from-kali-to-windows/).  
  

### Privilege Escalation:

A never ending topic, there are a lot of techniques, ranging from having an admin password to kernel exploits. Great way to practice this is by using Vulnhub VMs for practice. Check my OSCP-like VMs list [here](http://www.abatchy.com/2017/02/oscp-like-vulnhub-vms.html).  
  
<center>
<table dir="ltr" style="border-collapse: collapse; border: none; font-family: arial,sans,sans-serif; font-size: 13px; table-layout: fixed;" cellspacing="0" cellpadding="0" border="1"><colgroup><col width="415"></colgroup><tbody>
<tr style="height: 22px;"><td data-sheets-formula="=HYPERLINK(&quot;http://www.greyhathacker.net/?p=738&quot;, &quot;Windows:Elevating privileges by exploiting weak folder permissions&quot;)" data-sheets-value="{&quot;1&quot;:2,&quot;2&quot;:&quot;Windows:Elevating privileges by exploiting weak folder permissions&quot;}" style="background-color: #f4cccc; color: #1155cc; padding: 2px 3px 2px 3px; text-decoration: underline; vertical-align: bottom;"><a class="in-cell-link" href="http://www.greyhathacker.net/?p=738" target="_blank">Windows:Elevating privileges by exploiting weak folder permissions</a></td></tr>
<tr style="height: 21px;"><td data-sheets-formula="=HYPERLINK(&quot;http://www.fuzzysecurity.com/tutorials/16.html&quot;, &quot;Windows: Privilege Escalation Fundamentals&quot;)" data-sheets-value="{&quot;1&quot;:2,&quot;2&quot;:&quot;Windows: Privilege Escalation Fundamentals&quot;}" style="background-color: #f4cccc; color: #1155cc; padding: 2px 3px 2px 3px; text-decoration: underline; vertical-align: bottom;"><a class="in-cell-link" href="http://www.fuzzysecurity.com/tutorials/16.html" target="_blank">Windows: Privilege Escalation Fundamentals</a></td></tr>
<tr style="height: 21px;"><td data-sheets-formula="=HYPERLINK(&quot;https://github.com/GDSSecurity/Windows-Exploit-Suggester&quot;, &quot;Windows: Windows-Exploit-Suggester&quot;)" data-sheets-value="{&quot;1&quot;:2,&quot;2&quot;:&quot;Windows: Windows-Exploit-Suggester&quot;}" style="background-color: #f4cccc; color: #1155cc; padding: 2px 3px 2px 3px; text-decoration: underline; vertical-align: bottom;"><a class="in-cell-link" href="https://github.com/GDSSecurity/Windows-Exploit-Suggester" target="_blank">Windows: Windows-Exploit-Suggester</a></td></tr>
<tr style="height: 21px;"><td data-sheets-formula="=HYPERLINK(&quot;http://pwnwiki.io/#!privesc/windows/index.md&quot;, &quot;Windows: Privilege Escalation Commands&quot;)" data-sheets-value="{&quot;1&quot;:2,&quot;2&quot;:&quot;Windows: Privilege Escalation Commands&quot;}" style="background-color: #f4cccc; color: #1155cc; padding: 2px 3px 2px 3px; text-decoration: underline; vertical-align: bottom;"><a class="in-cell-link" href="http://pwnwiki.io/#!privesc/windows/index.md" target="_blank">Windows: Privilege Escalation Commands</a></td></tr>
<tr style="height: 21px;"><td data-sheets-formula="=HYPERLINK(&quot;https://blog.g0tmi1k.com/2011/08/basic-linux-privilege-escalation/&quot;,&quot;Linux: Basic Linux Privilege Escalation&quot;)" data-sheets-value="{&quot;1&quot;:2,&quot;2&quot;:&quot;Linux: Basic Linux Privilege Escalation&quot;}" style="background-color: #f4cccc; color: #1155cc; padding: 2px 3px 2px 3px; text-decoration: underline; vertical-align: bottom;"><a class="in-cell-link" href="https://blog.g0tmi1k.com/2011/08/basic-linux-privilege-escalation/" target="_blank">Linux: Basic Linux Privilege Escalation</a></td></tr>
<tr style="height: 21px;"><td data-sheets-formula="=HYPERLINK(&quot;http://www.securitysift.com/download/linuxprivchecker.py&quot;, &quot;Linux: linuxprivchecker.py&quot;)" data-sheets-value="{&quot;1&quot;:2,&quot;2&quot;:&quot;Linux: linuxprivchecker.py&quot;}" style="background-color: #f4cccc; color: #1155cc; padding: 2px 3px 2px 3px; text-decoration: underline; vertical-align: bottom;"><a class="in-cell-link" href="http://www.securitysift.com/download/linuxprivchecker.py" target="_blank">Linux: linuxprivchecker.py</a></td></tr>
<tr style="height: 21px;"><td data-sheets-formula="=HYPERLINK(&quot;https://github.com/rebootuser/LinEnum&quot;, &quot;Linux: LinEnum&quot;)
" data-sheets-value="{&quot;1&quot;:2,&quot;2&quot;:&quot;Linux: LinEnum&quot;}" style="background-color: #f4cccc; color: #1155cc; padding: 2px 3px 2px 3px; text-decoration: underline; vertical-align: bottom;"><a class="in-cell-link" href="https://github.com/rebootuser/LinEnum" target="_blank">Linux: LinEnum</a></td></tr>
<tr style="height: 21px;"><td data-sheets-formula="=HYPERLINK(&quot;https://www.youtube.com/watch?v=PC_iMqiuIRQ&quot;,&quot;Practical Windows Privilege Escalation&quot;)" data-sheets-value="{&quot;1&quot;:2,&quot;2&quot;:&quot;Practical Windows Privilege Escalation&quot;}" style="background-color: #f4cccc; color: #1155cc; padding: 2px 3px 2px 3px; text-decoration: underline; vertical-align: bottom;"><a class="in-cell-link" href="https://www.youtube.com/watch?v=PC_iMqiuIRQ" target="_blank">Practical Windows Privilege Escalation</a></td></tr>
<tr style="height: 21px;"><td data-sheets-formula="=HYPERLINK(&quot;https://www.adampalmer.me/iodigitalsec/2013/08/13/mysql-root-to-system-root-with-udf-for-windows-and-linux/&quot;, &quot;MySQL Root to System Root with UDF&quot;)" data-sheets-value="{&quot;1&quot;:2,&quot;2&quot;:&quot;MySQL Root to System Root with UDF&quot;}" style="background-color: #f4cccc; color: #1155cc; padding: 2px 3px 2px 3px; text-decoration: underline; vertical-align: bottom;"><a class="in-cell-link" href="https://www.adampalmer.me/iodigitalsec/2013/08/13/mysql-root-to-system-root-with-udf-for-windows-and-linux/" target="_blank">MySQL Root to System Root with UDF</a></td></tr>
</tbody></table>
</center>  

### Client Side Attacks:

Try out the techniques provided in [Metasploit Unleashed ](https://www.offensive-security.com/metasploit-unleashed/client-side-attacks/)or an IE client side exploit.  

### Web Application Attacks

Another lengthy subject, understand what XSS is, [`SQL injection`](https://www.exploit-db.com/papers/13045/), [`LFI`](https://www.exploit-db.com/docs/40992.pdf), RFI, directory traversal, how to use a proxy like Burp Suite. Solve as much as you can from [`OverTheWire: Natas`](http://overthewire.org/wargames/natas/). It has great examples on Code Injection, Session hijacking and other web vulnerabilities.  
  
Key is research till you feel comfortable.  

### Password Attacks:

Understand the basics of password attacks, difference between online and offline attacks. How to use [`Hydra`](http://sectools.org/tool/hydra/), [`JTR`](https://github.com/magnumripper/JohnTheRipper), [`Medusa`](https://en.kali.tools/?p=200), what rainbow tables are, the list goes on. Excellent post on this topic [here](https://alexandreborgesbrazil.files.wordpress.com/2013/08/introduction_to_password_cracking_part_1.pdf).

### Port redirection/tunneling:

Not all machines are directly accessible, some are dual homed, connected to an internal network. You'll use such techniques a lot in non-public networks. [This post](https://chamibuddhika.wordpress.com/2012/03/21/ssh-tunnelling-explained/) did a great job explaining it.  

### Metasploit Framework:

Decided to skip this part, but if you still want to study it, check out [Metasploit Unleashed](https://www.offensive-security.com/metasploit-unleashed/) course.

### Antivirus Bypassing

Skipped this part too. Pretty basic in OSCP.  
  
---

## 2\. Wargames

Consider these a prep for vulnerable machines.  
  

### OverTheWire: Bandit

Great start for people who aren't familiar with Linux or Bash. Check my walkthroughs [here](http://localhost:4000/tag/bandit/).  
  

### Over The Wire: Natas

Focused on web application, many challenges aren't required for OSCP, but it helps for sure. Check my walkthroughs [here](http://localhost:4000/tag/natas/).  
  

### Root-me.org

Has great challenges on privilege escalation, SQL injection, Javascript obfuscation, password cracking and analyzing PCAP files  
  
---

## 3\. Vulnerable Machines

Boot-to-root VMs are excellent for pentesting, you import a VM, run it and start enumerating from your attacking machine. Most of them result in getting root access. Check my [post](http://www.abatchy.com/2017/02/oscp-like-vulnhub-vms.html) on which machines are the closest to OSCP. Rooting VMs is as important as studying the material. You can't depend on theoretical knowledge only, yet you still need this knowledge to help you tackle harder machines.

---

If you still have questions, feel free to comment below or ask on our [NetSecFocus slack](https://netsecfocus.herokuapp.com/)!
  
\- Abatchy
