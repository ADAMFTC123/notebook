
![[Pasted image 20251029162833.png]]

#### script to enumerate smb shares on the computer system:
nmap -p 445 --script=smb-enum-shares.nse,smb-enum-users.nse 10.10.90.98



Using nmap we can enumerate a machine for SMB shares.

Nmap has the ability to run to automate a wide variety of networking tasks. There is a script to enumerate shares!

nmap -p 445 --script=smb-enum-shares.nse,smb-enum-users.nse 10.10.4.26

On most distributions of Linux smbclient is already installed. Lets inspect one of the shares.

smbclient //10.10.4.26/anonymous

You can recursively download the SMB share too. Submit the username and password as nothing.

smbget -R smb://10.10.4.26/anonymous

 port 111 running the service rpcbind. This is just a server that converts remote procedure call (RPC) program number into universal addresses. When an RPC service is started, it tells rpcbind the address at which it is listening and the RPC program number its prepared to serve. 

In our case, port 111 is access to a network file system. Lets use nmap to enumerate this.

nmap -p 111 --script=nfs-ls,nfs-statfs,nfs-showmount 10.10.4.26

find exploits with search sploit



#understand how we manipulated menu and the ssh connection with the key
