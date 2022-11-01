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
