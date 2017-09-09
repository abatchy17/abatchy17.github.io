---
layout: post
title: "Port forwarding: A practical hands-on guide"
date: 2017-01-12 12:00:00
share: true
comments: true
tags: [Networking]
---

1. [Introduction](#1-introduction)
2. [Requirements](#2-requirements)
3. [Assumptions](#3-assumptions)
4. [Setup Before Port Forwarding](#4-setup-before-port-forwarding)
    * [Setting up Apache server](#41-setting-up-apache-server)
    * [Configuring iptables](#42-configuring-iptables)
5. [Setup After Port Forwarding](#5-setup-after-port-forwarding)
    * [Dataflow](#51-dataflow)
    * [Rinetd on proxy machine](#52-rinetd-on-proxy-machine)
6. [Did it work?](#6-did-it-work)
7. [No love for Windows?](#7-no-love-for-windows)

## 1\. Introduction

During my preparation for the PWK/OSCP course (starting late January), I had a hard time understanding how port forwarding/tunneling works. Most guides I found were too theoretical for my taste, this guide will allow you to mimic a strict corporate firewall and how to bypass it.

So what is port forwarding? It's a technique that allows you to redirect traffic from one port to another port/IP. What the hell does that mean?

Imagine the following scenario:

Your corp firewall only allows outbound connections to web servers running on standard ports (port 80 for HTTP / port 443 for HTTPS). Your favorite security news website for some reason is run on port 8000. Given the strict corp firewall you won't be able to browse the site any more.

[![](http://i.imgur.com/WaYeE8F.png)](http://i.imgur.com/WaYeE8F.png)

You decide to use your knowledge of port forwarding to still be able to browse your favorite site. We'll be using [rinetd](http://www.lenzg.net/rinetd/rinetd.html) as TCP forwarder.

---

## 2\. Requirements

To follow this guide you'll need the following:

  * Host machine capable of running 2 Linux VMs.
  * LAN network.
  * Some patience.

---
  
## 3\. Assumptions

All machines are on the same network for simplicity, thus no router
configuration is needed.

<center>
<table border="3"><tbody>
<tr>     <th>Machine</th>     <th>IP</th> </tr>
<tr>     <td>abatchy-work</td>     <td>192.168.20.130</td> </tr>
<tr>     <td>abatchy-proxy</td>     <td>192.168.20.131</td> </tr>
<tr>     <td>abatchy-http</td>     <td>192.168.20.132</td> </tr>
</tbody></table>
</center>

  * **192.168.20.130**: Work machine, ideally it's behind a NAT and firewalled. For simplicity it's in the same LAN and iptables is used instead.

  * **192.168.20.131**: Home machine, used as proxy to forward inbound traffic on port 80 to 192.168.20.132 on port 8000.

  * **192.168.20.132:** Web server running your favorite site on non-standard port 8000, inaccessible directly from your work machine because of firewall.

---

## 4\. Setup Before Port Forwarding

We'll be configuring the following setup:


[![](http://i.imgur.com/wKZcLPV.jpg)](http://i.imgur.com/wKZcLPV.jpg)


#### 4.1 Setting up Apache server

We'll start by setting up our Apache server on 192.168.20.132 (Web server)

```console
// Install Apache server
abatchy@abatchy-http:~$ sudo apt-get install apache2

// Change default port Apache is listening on to 8000
abatchy@abatchy-http:~$ sudo nano /etc/apache2/ports.conf
// Change virtual host from
// <VirtualHost *:80>
// to
// <VirtualHost *:8000>
abatchy@abatchy-http:~$ sudo nano /etc/apache2/sites-enabled/000-default

// Restart Apache server
abatchy@abatchy-http:~$ sudo service restart apache2
```

Now let's verify it's working as expected.

```console
//sudo is needed since netstat won't show processes not owned by abatchy
abatchy@abatchy-http:~$ sudo netstat -antp | grep apache2

tcp        0      0 0.0.0.0:8000            0.0.0.0:*               LISTEN      4523/apache2
```

#### 4.2 Configuring iptables


Next, we'll configure iptables on `192.168.20.130` (work machine) to drop any
outgoing traffic except for ones using TCP port 80 and 443.

```console
// Verify that there are no rules defined yet

abatchy@abatchy-work:~$ sudo iptables -L
[sudo] password for abatchy:

Chain INPUT (policy ACCEPT)
target     prot opt source               destination

Chain FORWARD (policy ACCEPT)
target     prot opt source               destination

Chain OUTPUT (policy ACCEPT)
target     prot opt source               destination


// Block all traffic

abatchy@abatchy-work:~$ sudo iptables -I OUTPUT -j DROP

// Allow outbound traffic on port 80

abatchy@abatchy-work:~$ sudo iptables -I OUTPUT -p tcp --dport 80 -j ACCEPT

// Allow outbound traffic on port 443

abatchy@abatchy-work:~$ sudo iptables -I OUTPUT -p tcp --dport 443 -j ACCEPT

// View current rules defined

abatchy@abatchy-work:~$ sudo iptables -L

Chain INPUT (policy ACCEPT)
target     prot opt source               destination

Chain FORWARD (policy ACCEPT)
target     prot opt source               destination

Chain OUTPUT (policy ACCEPT)
target     prot opt source               destination
**ACCEPT     tcp  --  anywhere             anywhere             tcp dpt:https
ACCEPT     tcp  --  anywhere             anywhere             tcp dpt:http**
DROP       all  --  anywhere             anywhere
```


Notice the order the rules were applied. `-I` was used to put the rules on top of the chain, thus overriding the "block all traffic" rule. One more thing is that you can't make DNS requests / resolve URLs since UDP port 53 is included in the rule.

Next, let's ensure that the current rules work properly.
```console
abatchy@abatchy-work:~$ wget http://192.168.20.132:8000
--2017-01-12 03:01:24--  http://192.168.20.132:8000/
Connecting to 192.168.20.132:8000... ^C
```


As expected, we're not able to connect to `http://192.168.20.132:8000`. In case you want to see logs of the dropped packets check [this](http://www.thegeekstuff.com/2012/08/iptables-log-packets/?utm_source=feedburner).

---

## 5\. Setup After Port Forwarding

#### 5.1 Data flow

[![](http://i.imgur.com/6AVblX5.jpg)](http://i.imgur.com/6AVblX5.jpg)

**(1)** Outgoing traffic to `192.168.20.131:80` which our firewall will let through to our switch.
**(2)** On our proxy machine, we will have `rinetd` listening on port 80.


[![](http://i.imgur.com/bnmdfJ7.jpg)](http://i.imgur.com/bnmdfJ7.jpg)



**(3) and (4)** Traffic is forwarded by `rinetd` to our webserver, listening on `192.168.20.132:8000`.
**(5) and (6)** Web server replies back to our proxy machine.
**(7)** Proxy replies back to our work machine, traffic still flowing as it's on port 80.


#### 5.2 Rinetd on proxy machine

Rinetd is a very light-weight, simple-to-use TCP forwarder, we will set it up so it redirects incoming traffic on port 80 to `192.168.20.132:8000` (our web server).

```console
// Install rinetd
abatchy@abatchy-proxy:/home$ sudo apt-get install rinetd

// Add the following rule below the comment
abatchy@abatchy-proxy:/home$ sudo nano /etc/rinetd.conf
# bindadress    bindport  connectaddress  connectport
192.168.20.131  80      192.168.20.132  8000

// Restart the service
abatchy@abatchy-proxy:/home$ sudo service rinetd restart
```

---

## 6\. Did it work?

```console
abatchy@abatchy-work:~$ wget http://192.168.20.131:80
--2017-01-12 19:56:35--  http://192.168.20.131/
Connecting to 192.168.20.131:80... connected.
HTTP request sent, awaiting response... 200 OK
Length: 20 [text/html]
Saving to: ‘index.html’

100%[======================================>] 20          --.-K/s   in 0s

2017-01-12 19:56:35 (6.33 MB/s) - ‘index.html’ saved [20/20]

abatchy@abatchy-work:~$ cat index.html
Super cool website!
```

It worked! A wireshark capture below on the proxy shows the traffic flow.

[![](http://i.imgur.com/q5IJ378.png)](http://i.imgur.com/q5IJ378.png)

**2266-2268:** Three way handshake between Work and Proxy:80.
**2269-2270:** GET request and ACK.
**2271-2273:** Three way handshake between Proxy and WebServer:8000.
**2274-2275:** GET request and ACK
**2276-2277:** WebServer responds and Proxy acknowledges.
**2278-2279:** Proxy forwards received packet back to Work.
**2280-2285:** Work terminates connection with Proxy and Proxy terminates connection with WebServer.

---

## 7\. No love for Windows?

Rinetd supports Windows and is pretty straight-forward as on Linux as well. Binary can be found [here](https://www.boutell.com/rinetd/).

Assuming rinetd is installed in: `C:\rinetd`, just add the same rule to `rinetd.conf`:

```
192.168.20.131  80      192.168.20.132  8000
```

Then run rinetd:

```console
C:\rinetd>rinetd.exe -c rinetd.conf
```

And done.

---

Hopefully this guide sheds some light on port forwarding and more advanced traffic manipulation.
