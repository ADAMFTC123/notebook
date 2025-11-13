 
#### **port scans** 
nmap -sT   (TCP connect scan)
`tcp.flags.syn==1 and tcp.flags.ack==0 and tcp.window_size > 1024`
![[Pasted image 20250929141710.png]]

nmap -sS` (TCP SYN scan)
`tcp.flags.syn==1 and tcp.flags.ack==0 and tcp.window_size <= 1024`
![[Pasted image 20250929141730.png]]

nmap -sU
`icmp.type==3 and icmp.code==3`
![[Pasted image 20250929141738.png]]

##### ARP SPOOFING DETECTION 
![[Pasted image 20250929145130.png]]

#### host and user identification in sniff
![[Pasted image 20250929150455.png]]
- `dhcp.option.hostname contains "keyword"`
- `dhcp.option.domain_name contains "keyword"`
-  `nbns.name contains "keyword"`
-  `kerberos.SNameString == "krbtg"`
- `kerberos.CNameString and !(kerberos.CNameString contains "$" )` - if it has a dollar its hostname if not its username
- also exists in http get
![[Pasted image 20250930104242.png]]
![[Pasted image 20250929174817.png]]
שאני מסיים bloodhound מקס קונה לי אייס קפה 
![[Pasted image 20250930091404.png]]



