

# walkthroug:


##### What is the version of the Apache server?
nmap -sS -sV -p- ipaddress
##### What is the port number of the FTP service?
nmap -sS -sV -p- ipaddress


##### What is the FQDN for the website hosted using a self-signed certificate and contains critical server information as the homepage?

the email address is in the ssl certificate 

the signature is singed by the guy listed in the "issued by" field in the ssl cert


##### What is the value of the **PHP Extension Build** on the server?

see whats running php nmap -p all ports that are running web interfaces -A ipaddress
surf to the ports running php 

What is the banner for the FTP service?
ftp ip port

##### What software is used for managing the database on the server?

when there is php enabled on the server most of the time 
the software usesd for managing is phpmyadmin

##### What is the Content Management System (CMS) hosted on the server?
go to the source of the webinterface and search for file sand directorys that indicate for a specific cms
for example wp-somthing means there is wordpress


##### What is the version number of the CMS hosted on the server?
normally listed on the bottom of the main page of the web
heare we see:
![[Pasted image 20251104083802.png]]
so we use wpscan
-i also looked for "version" with ctrl+f in source file of the main page

##### During vulnerability scanning, **OSVDB-3092** detects a file that may be used to identify the blogging site software. What is the name of the file?
nikto -h https://10.10...:9007 -nossl


##### What is the username for the admin panel of the CMS?
find with wpscan


##### What is the name of the software being used on the standard HTTP port?
nmap -sS -sV -p- ipaddress
# TIPS

when we dont find meanigful contetn on web page we will use gobuster

blank page is to make it not do directory listing so we would still do it using gobuster,dirbuster


if website uses ssl use --disable-tls-checks in wpscan

we can see the root of the webpage that presents phpinfo() in the  Document Root  field
**`DOCUMENT_ROOT`**: `/var/www/info`
    
This means that any file accessed via `https://10.10.164.239:1443/` is served from the `/var/www/info` directory on the server's file system.

Most web applications, especially CMS like WordPress, are typically installed in directories that reflect their purpose, such as:

- `/var/www/html/` (General default)
    
- `/var/www/wordpress/`
    
- `/var/www/blog/`
-

Since the Document Root is `/var/www/info`, it must be a minimal, dedicated location for the file(s) on port 1443.


 If the web application has a working, official **phpMyAdmin** page, it **confirms with 100% certainty** that the backend database is either **MySQL** or **MariaDB**.
phpMyAdmin supports MySQL-compatible databases.

    MySQL 5.5 or newer
    MariaDB 5.5 or newer


The core files for the WordPress administration area are almost always found relative to the site's root directory:

1. **Login File:** The most common path, leading to the login form:
    
    - `http://[Target_IP]:[Port]/wp-login.php`
        
2. **Admin Directory:** If you are already logged in, this leads to the main dashboard:
    
    - `http://[Target_IP]:[Port]/wp-admin/`
### Other Common CMS (For future reference)

If you were unsure of the CMS, you'd check these standard paths:

- **Joomla!:** `/administrator/`
    
- **Drupal:** `/user/` or `/admin/`
    
- **Default:** `/login/` or `/admin/`


you can idetify specific database software:
nmap -p 1433,3306,5432,1521,27017,6379,9042 -sV 10.10.164.239

when you see a ftp service open grab the banner
if anonymous login is enabled you can login to ftp service using
username:anonymous
password:anonymous

or ftp,ftp


we can see what webservers are running php using
nmap -p <all ports that are running web interfaces> -A <ipaddress>


