---
layout: post
title: "OSCP-like Vulnhub VMs"
date: 2017-02-23     12:00:00
share: true
comments: true
tags: [Vulnhub Walkthrough, OSCP Prep]
---

Before starting the PWK course I solved little over a dozen of the Vulnhub VMs, mainly so I don't need to start from rock bottom on the PWK lab. Below is a list of machines I rooted, most of them are similar to what you'll be facing in the lab. I've written walkthroughs for a few of them as well, but try harder first ;)  
  
# Linux
  
### Beginner friendly

  * [Kioptrix: Level 1 (#1)](https://www.vulnhub.com/entry/kioptrix-level-1-1,22/) 
  * [Kioptrix: Level 1.1 (#2) ](https://www.vulnhub.com/entry/kioptrix-level-11-2,23/)
  * [Kioptrix: Level 1.2 (#3)](https://www.vulnhub.com/entry/kioptrix-level-12-3,24/) 
  * [Kioptrix: Level 1.3 (#4)](https://www.vulnhub.com/entry/kioptrix-level-13-4,25/) 
  * [FristiLeaks: 1.3 ](https://www.vulnhub.com/entry/fristileaks-13,133/)
  * [Stapler: 1](https://www.vulnhub.com/entry/stapler-1,150/)
  * [PwnLab: init](https://www.vulnhub.com/entry/pwnlab-init,158/)

### Intermediate

  * [Kioptrix: 2014](https://www.vulnhub.com/entry/kioptrix-2014-5,62/)
  * [Brainpan: 1](https://www.vulnhub.com/entry/brainpan-1,51/)
  * [Mr-Robot: 1  ](https://www.vulnhub.com/entry/mr-robot-1,151/)
  * [HackLAB: Vulnix](https://www.vulnhub.com/entry/hacklab-vulnix,48/)

### Not so sure (Didn't solve them yet)

  * [VulnOS: 2](https://www.vulnhub.com/entry/vulnos-2,147/)
  * [SickOs: 1.2](https://www.vulnhub.com/entry/sickos-12,144/)
  * [/dev/random: scream](https://www.vulnhub.com/entry/devrandom-scream,47/) 
  * [pWnOS: 2.0](https://www.vulnhub.com/entry/pwnos-20-pre-release,34/)
  * [SkyTower: 1](https://www.vulnhub.com/entry/skytower-1,96/) 
  * [IMF](https://www.vulnhub.com/entry/imf-1,162/)

---
  
# Windows

There aren't many Windows machines around due to licensing. Few options:  

  * [Hack The Box](https://www.hackthebox.gr/en/login): Got a nice set of Windows machines from Windows 2000 up to Windows 8.1 I believe.
  * [Metasploitable 3](https://github.com/rapid7/metasploitable3/wiki), will download a trial version of Windows Server.
  * <https://github.com/magnetikonline/linuxmicrosoftievirtualmachines> you can download Windows VMs legally then hack your way through them through an unpatched vulnerability or setting up a vulnerable software.
  * Set up your own lab. Default Windows XP SP0 will give you the chance to try out a few remote exploits, or doing some privilege escalation using weak services.
  * [/dev/random: Sleepy](https://www.vulnhub.com/entry/devrandom-sleepy,123/) (Uses VulnInjector, need to provide you own ISO and key.)**[ ](https://www.vulnhub.com/entry/devrandom-sleepy,123/)**
  * [ Bobby: 1](https://www.vulnhub.com/entry/bobby-1,42/) (Uses VulnInjector, need to provide you own ISO and key.)
  
If you think something is worth to be added to this list please mention it in the comments, I do check them ;)

\- Abatchy