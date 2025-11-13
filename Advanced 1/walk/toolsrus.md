
nmap -sS -sV -p-
dirb http://10.10.164.239:8000/ /usr/share/dirb/wordlists/common.txt



id did the base64 alone

ORT     STATE SERVICE VERSION

80/tcp   open  http    Apache httpd 2.4.18 ((Ubuntu))
|_http-server-header: Apache/2.4.18 (Ubuntu)
|_http-title: Site doesn't have a title (text/html).

1234/tcp open  http    Apache Tomcat/Coyote JSP engine 1.1
|_http-favicon: Apache Tomcat
|_http-server-header: Apache-Coyote/1.1
|_http-title: Apache Tomcat/7.0.88

8009/tcp open  ajp13   Apache Jserv (Protocol v1.3)
|_ajp-methods: Failed to get a valid response for the OPTION request
MAC Address: 02:1F:24:FE:BD:C1 (Unknown)
No exact OS matches for host (If you know what OS is running on it, see https://nmap.org/submit/ ).


if you found creds,  the exploit in msfconsole youre lookingfor probly requires creds, so you can narrow it down like that 

#understand how could you have looked for that exploit
you searched for cves in context of  Apache-Coyote/1.1,Tomcat/7.0.88 you serched for the names Tomcat,Coyote, you found one for Tomcat 7.0.88, maybe there is a couple of paylods who are right

u used tomcat_mgr_upload