#### PowerShell (any version):

  
  

    
    
    (New-Object System.Net.WebClient).DownloadFile("https://example.com/archive.zip", "C:\Windows\Temp\archive.zip")  
    

  
Full documentation [here](https://msdn.microsoft.com/en-
us/library/system.net.webclient\(v=vs.110\).aspx).  
  

#### PowerShell 4.0 &amp; 5.0:

  
  

    
    
    Invoke-WebRequest "https://example.com/archive.zip" -OutFile "C:\Windows\Temp\archive.zip"  
    

  
Full documentation [here](https://technet.microsoft.com/en-
us/library/hh849901.aspx).

