---
layout: post
title: "[DRAFT] Tips on designing boot2root challenges"
date: 2019-02-18 10:00:00
share: true
comments: true
description: Tips on how to create and properly configure/test an intentionally vulnerable VM, also known as boot2roots.
tags: [boot2root]
---

*During the past couple of years I published a couple of boot2root challenges on [Vulnhub](https://www.vulnhub.com/author/abatchy,393/) and had some asks on how to properly create such a challenge. I hope this quick post gives you some guidance.*

---

**1. Use the official ISOs to create the VM:** Avoid using pre-created VMs, many times they aren't ported properly to be distributed and/or contain unwanted bloatware. Using the official ISOs gives you flexibility on creating the VM hypvervisor-agnostic, meaning it should have no dependencies on whether you created them on VMWare/VirtualBox, so don't install the guest additions.

**2. Configure the network:** Let the VM rely on the DHCP server, take a look at the Vulnhub "Isolating the lab" section [here](https://www.vulnhub.com/lab/network/). Don't assign it a static IP unless needed, and make sure the player knows about that and any setup required.

**3. Patch it up:**: You normally want players to find the challenges you implemented, not the ones that result from an outdated vulnerable service/kernel. Updating it will reduce the odds of this happening, [hopefully](https://dirtycow.ninja/).

**4. Choose a theme:** Deciding on a theme turns out to be quite important as some people prefer realistic challenges; would I find this in a production environment? Dealing with unrealistic challenges without expecting them could be frustrating. Yet don't be afraid to stray from that, there are lots of great unrealistic boot2roots on vulnhub.

**5. Create the attack surface:** This is where the real work begins. You'll want to craft a path (or more) for players to discover. Since players attack the VM from an attacker machine, avoid them being able to go from attacking remotely directly to root. `remote -> root` single challenges, at the very least it should go `remote -> local -> root`. Even more fun is to create multiple scenarios.

**6. Take snapshots and automate:** Taking snapshots will give you the flexibility on dismissing corrupted iterations. Automating various tasks should be a good ROI, like deleting temporary files and such.

**7. Remove unwanted services/files:** Removing the GUI layer will save well on space.

**8. Have test bunnies:** Many people would love to help you test the challenges, another pair of eyes does wonders.

**9. `Update /etc/motd`:** This is your chance to put a note for the player, personalize the VM with a name or even some ASCII artwork.

**10. Export to OVA format:** Avoid uploading the disk file as many times it might have dependencies on the HW you created it on or misconfigured. Instead export the VMs to OVF format so the hypervisor properly mounts them. Both [VMWare](https://docs.vmware.com/en/VMware-Fusion/11/com.vmware.fusion.using.doc/GUID-16E390B1-829D-4289-8442-270A474C106A.html) and [VirtualBox](https://docs.oracle.com/cd/E26217_01/E26796/html/qs-import-vm.html) support this.

*\-Abatchy*