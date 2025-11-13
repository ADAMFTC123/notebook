#### RECON
![[Pasted image 20251029141245.png]]One of the more interesting ports that is open is Microsoft Remote Desktop (MSRDP) - 3389
 also we can see there is an icecast service -8000
 hostname - DARK-PC

#### GAIN ACCESS
ice cast vulnerability:
 https://www.cvedetails.com/cve/CVE-2004-1561/
well find an exploit with metasploit:
![[Pasted image 20251029130453.png]]
now we have a shell.
we'll use "ps" to see what user is running the icecast service
![[Pasted image 20251029130759.png]]we'll use sysinfo to see what version of windows is running:
![[Pasted image 20251029141723.png]]
athering local system information using `sysinfo`, `getuid`, and `getsystem` commands. Search for potential local exploits with Metasploit's `local exploit suggester`#
### 
PRIVELEGE ESCALATION
well run post/windows/gather/enum_logged_on_users:
*i dont know why*

![[Pasted image 20251029142918.png]]



we will background our session and use "post/multi/recon/local_exploit_suggester" and we will set the session to be the meterpreter shell:


![[Pasted image 20251029145255.png]]


well use the exploit, set the session to our meterpreter shell again and run it:
now we can see we have high privileges:
![[Pasted image 20251029145747.png]]
SeTakeOwnershipPrivilege - allows us to take ownership of filess

now we want to migrate to a process that has the permissionss t ointeract with the lsass sevice
we need to be 'living in' a process that is the same architecture as the lsass service (x64 in the case of this machine)


the term 'living in' a process. Often when we take over a running program we ultimately load another shared library into the program (a dll) which includes our malicious code. From this, we can spawn a new thread that hosts our shell.

#### MIGRATE AND GET SYSTEM
migrate-works with create remote thread
getsystem;
![[Pasted image 20251029153107.png]]

One elevation technique is the `Named Pipe Impersonation (In Memory/Admin)`
Pipes are part of Windows OS to help communication between processes. In this technique, Meterpreter creates a named pipe. Then a cmd.exe is created under the local system that connects to the Meterpreter named pipe. Meterpreter can then impersonate the local security privileges, in this case SYSTEM. This makes you the SYSTEM administrator.
There are a few things to keep in mind when using this technique:

- `ELEVATE_TECHNIQUE_SERVICE_NAMEDPIPE` only works on native Windows Meterpreter.
- The account type used must be an administrator.
- The session can’t be a user account control or it will fail, even if you are using an administrator account.

To learn more about Impersonation in Windows, see Impersonating a Named Pipe Client on Microsoft:[https://docs.microsoft.com/en-us/windows/win32/ipc/impersonating-a-named-pipe-client?redirectedfrom=MSDN](https://docs.microsoft.com/en-us/windows/win32/ipc/impersonating-a-named-pipe-client?redirectedfrom=MSDN) .

#### password dumping
load kiwi
creds_all

timestomp-To complicate forensics efforts we can modify timestamps of files on the system. What command allows us to do this? Don't ever do this on a pentest unless you're explicitly allowed to do so! This is not beneficial to the defending team as they try to breakdown the events of the pentest after the fact.