####

_Resources for Forensics challenges [here](http://www.abatchy.com/2017/08/ctf-
resources-forensics.html)._  
_ _  

## Digital Camouflage

  
Let's go over the TCP streams (**Analyze -&gt; Follow -&gt; TCP Streams**),
stream 1 (tcp.stream eq 1) shows login credentials for user mathew.  
  

    
    
    userid=mathewsr&pswrd=aHJLUVNTTFd2Rw%3D%3D  
    

  
Notice that we should decode the values, so pswrd is _aHJLUVNTTFd2Rw==, _this
is possibly a base64 string, let's decode it.__  

    
    
    _abatchy@ubuntu:~/Desktop$ echo aHJLUVNTTFd2Rw==| base64 -d  
    hrKQSSLWvG  
    _

  
**Flag:** hrKQSSLWvG  
  
__Note:_ Encoding a string to base64 format is not a form of encryption_  
  

## Special Agent User  

  
Extremely straight-forward challenge, you're after the User-Agent field for
the browser making requests in the PCAP file. TCP stream 3 contains the
answer.  
  
**Flag:** Chrome 34.0.1847.137  
  
__Note:_ Wget is not the right answer as it's not really a browser._

