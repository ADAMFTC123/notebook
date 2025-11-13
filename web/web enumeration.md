
https://elhacker.info/Cursos/Certified%20Ethical%20Hacker-CEHv12-Practical%20hands%20on%20Labs/3.%20Footprinting%20and%20Reconnaissance/1.1%20Directory%20Busting%20and%20VHOST%20Enumeration.pdf

https://hacknseek.gitlab.io/cheatsheets/common-wordlists/

**Using our Browsers Developer Console**



 This suite includes a range of tools including:

- Viewing page source code
- Finding assets
- Debugging & executing code such as javascript on the client-side (our Browser)

**Inspector** - HTML source code of the webpage we have loaded in our browser

This often contains things such as developer comments, and the name to certain aspects of web page features including forms and the likes.
![[Pasted image 20251030102713.png]]

#### dirbuster

dirb http://10.10.164.239:8000/ /usr/share/dirb/wordlists/common.txt

#### GoBuster
##### dir mode  -  allows the user to enumerate website directories.
  
![[Pasted image 20251030103459.png]]

gobuster dir -u http://10.10.10.10 -w /usr/share/wordlists/dirbuster/directory-list-2.3-medium.txt

The URL is going to be the base path where Gobuster starts looking from
![[Pasted image 20251030103439.png]]
`gobuster dir -t 64 -u http://10.10.252.123/myfolder -w /usr/share/wordlists/dirbuster/directory-list-2.3-medium.txt -x.html,.css,.js`


 **.conf** or **.config** files usually contain configurations for the application - including sensitive info such as database credentials.
 
if https is enabled you may encounter this error:
![[Pasted image 20251030103758.png]]
add the `-k` flag to your scan and it will bypass this invalid certification and continue scanning and deliver the goods!

##### dns mode - This allows Gobuster to brute-force subdomains.
 For example, if State Farm owns statefarm.com and mobile.statefarm.com, there may be a hole in mobile.statefarm.com that is not present in statefarm.com. This is why it is important to search for subdomains too!
 
  
gobuster dns -d mydomain.thm -w /usr/share/wordlists/SecLists/Discovery/DNS/subdomains-top1million-5000.txt

This tells Gobuster to do a sub-domain scan on the domain "mydomain.thm"
![[Pasted image 20251030104304.png]]

##### "vhost" Mode 
The last and final mode we'll focus on is the "vhost" mode. This allows Gobuster to brute-force virtual hosts. Virtual hosts are different websites on the same machine. In some instances, they can appear to look like sub-domains, but don't be deceived! Virtual Hosts are IP based and are running on the same server. This is not usually apparent to the end-user. On an engagement, it may be worthwhile to just run Gobuster in this mode to see if it comes up with anything. You never know, it might just find something! While participating in rooms on TryHackMe, virtual hosts would be a good way to hide a completely different website if nothing turned up on your main port 80/443 scan.

`gobuster vhost -u http://example.com -w /usr/share/wordlists/SecLists/Discovery/DNS/subdomains-top1million-5000.txt`

in host discovery we will first learn the domain name with nmap and than add the:
target ip
domain name 
to our /etc/host - so when we access the ip it automatically directs to the domain

echo "10.10.187.225 webenum.thm" >> /etc/hosts
or:
echo "10.10.187.225 webenum.thm" | sudo tee -a /etc/hosts




You will also need to add any virtual hosts that you discover through the same way, before you can visit them in your browser i.e.:

`echo "10.10.187.225 mysubdomain.webenum.thm" >> /etc/hosts`
##### and than we will look in this virtual host like this:
gobuster dir -t 64 -u http://products.webenum.thm -w wordlist.txt -x.html,.css,.js,.php,.txt

##### how is it that in etc/host we configure them to have the same ip but we get diffrenet result when scanning?

The web server (like Apache or Nginx) running on `10.10.3.61` receives the connection and the HTTP request. Since the **IP address is the same** for both requests, the server relies entirely on the value of the **`Host:` header** to decide which website's files to serve:

- **For the request with `Host: webenum.thm`:** The server looks at its configuration and routes the request to the document root (the directory containing web files) configured for the **`webenum.thm` virtual host**.
    
- **For the request with `Host: products.webenum.thm`:** The server routes the request to a **different** document root (a different directory on the server's file system) configured for the **`products.webenum.thm` virtual host**.
- 
##### wordlists:
- /usr/share/wordlists/dirbuster/directory-list-2.3-*.txt
- /usr/share/wordlists/dirbuster/directory-list-1.0.txt
- /usr/share/wordlists/dirb/big.txt
- /usr/share/wordlists/dirb/common.txt
- /usr/share/wordlists/dirb/small.txt
- /usr/share/wordlists/dirb/extensions_common.txt - Useful for when fuzzing for files!
-
Daniel Miessler has created an amazing GitHub repo called [SecLists](https://github.com/danielmiessler/SecLists). It compiles many different lists used for many different things. The best part is, it's in apt! You can `sudo apt install seclists`

##### fuzzing files:
gobuster dir -u http://10.10.225.212:80/Changes -w /usr/share/wordlists/dirb/big.txt -x /usr/share/wordlists/dirb/extensions_common.txt



what -x will do is, for every name we iterate through we will also iterate through the 
/usr/share/wordlists/dirb/extensions_common.txt
and try many versions of the name 
for example if there is the name ''video'' it will try also video.asp,video.html, and iterate through the extensions listed in the -x 


#### WPScan

 wpscan --update

we can look at the assets our web browser loads and then looks for the location of these on the webserver. Using the "Network" tab in your web browsers developer tools, you can see what files are loaded when you visit a webpage.

Take the screenshot below, we can see many assets are loaded, some of these will be scripts & the stylings of the theme that determines how the browser renders the website.

![[Pasted image 20251102100901.png]]


 We can take a pretty good guess that the name of the current theme is "twentytwentyone". After inspecting the source code of the website, we can note additional references to "twentytwentyone"

![[Pasted image 20251102101108.png]]

However, let's use WPScan to speed this process up by using the `--enumerate` flag with the `t` argument like so:

`wpscan --url http://cmnatics.playground/ --enumerate t`
![[Pasted image 20251102101300.png]]The great thing about WPScan is that the tool lets you know how it determined the results it has got. In this case, we're told that the "twentytwenty" theme was confirmed by scanning "_Known Locations_". The "twentytwenty" theme is the default WordPress theme for WordPress versions in 2020.

##### **Enumerating for Installed Plugins**
`wpscan --url http://cmnatics.playground/ --enumerate p`

##### **Enumerating for Users**
`wpscan --url http://cmnatics.playground/ --enumerate u`

##### **The "Vulnerable" Flag**
In the commands so far, we have only enumerated WordPress to discover what themes, plugins and users are present. At the moment, we'd have to look at the output and use sites such as MITRE, NVD and CVEDetails to look up the names of these plugins and the version numbers to determine any vulnerabilities.

wpscan --url http://cmnatics.playground/ --enumerate vp

**Note, that this requires setting up WPScan to use the WPVulnDB API which is out-of-scope for this room.**


##### **Performing a Password Attack**
After determining a list of possible usernames on the WordPress install, we can use WPScan to perform a bruteforcing technique against the username we specify and a password list that we provide. Simply, we use the output of our username enumeration to build a command like so:

wpscan –-url http://cmnatics.playground –-passwords rockyou.txt –-usernames cmnatic

![[Pasted image 20251102102230.png]]
##### **Adjusting WPScan's Aggressiveness (WAF)**

Unless specified, WPScan will try to be as least "noisy" as possible. Lots of requests to a web server can trigger things such as firewalls and ultimately result in you being blocked by the server.

  

This means that some plugins and themes may be missed by our WPScan. Luckily, we can use arguments such as `--plugins-detection` and an aggressiveness profile (passive/aggressive) to specify this. For example: `--plugins-detection aggressive`

##### walkthrough

echo "10.10.69.238 **wpscan.thm**" >> /etc/host

Enumerate the site, what is the name of the theme that is detected as running?

wget -O- 10.10.69.238 | grep theme

Enumerate the site, what is the name of the plugin that WPScan has found?

wpscan --url 10.10.69.238 --enumerate p
wpscan — url   10.10.69.238 -e ap

Enumerate the site, what username can WPScan find?

wpscan --url 10.10.69.238 --enumerate u

Construct a WPScan command to brute-force the site with this username, using the rockyou wordlist as the password list. What is the password to this user?

wpscan –-url 10.10.69.238 –-passwords rockyou.txt –-usernames phreakazoid

#### Nikto

sudo apt update && sudo apt install nikto


Nikto can be used to discover possible vulnerabilities including:

- Sensitive files
- Outdated servers and programs (i.e. [vulnerable web server installs](https://httpd.apache.org/security/vulnerabilities_24.html))
- Common server and software misconfigurations (Directory indexing, cgi scripts, x-ss protections)

**nikto -h vulnerable_ip**

![[Pasted image 20251102104858.png]]
Note a few interesting things are given to us in this example:

- Nikto has identified that the application is Apache Tomcat using the favicon and the presence of "_/examples/servlets/index.html_" which is the location for the default Apache Tomcat application.
- HTTP Methods "_PUT_" and "_DELETE_" can be performed by clients - we may be able to leverage these to exploit the application by uploading or deleting files.
-

we can scan 172.16.0.0/24 (subnet mask 255.255.255.0, resulting in 254 possible hosts) with Nmap (using the default web port of 80) and parse the output to Nikto like so:
 We must instruct Nmap to output a scan into a format that is friendly for Nikto to read using Nmap's  `-oG`  flags
 
`nmap -p80 172.16.0.0/24 -oG - | nikto -h -`

##### **scanning multiple ports on one specific host:**

nikto -h 10.10.10.1 -p 80,8000,8080

nikto plugins:

you can use: --list-plugins

https://github.com/sullo/nikto/wiki/Plugin-list

![[Pasted image 20251102105506.png]]

##### verbosity

We can increase the verbosity of our Nikto scan by providing the following arguments with the `-Display` flag
![[Pasted image 20251102110011.png]]

 We can use the `-Tuning` to specify the vulnerabilities categories we test
 ![[Pasted image 20251102105702.png]]
##### output to a file:
-o file.html


Where to go from here (recommended rooms)

GoBuster:

- [OWASP Top 10 (Walkthrough)](https://tryhackme.com/room/owasptop10)
- [EasyPeasyCTF (Challenge)](https://tryhackme.com/room/easypeasyctf)

WPScan:

- [Blog (Challenge)](https://tryhackme.com/room/blog)

Nikto:

- [OWASP Top 10 (Walkthrough)](https://tryhackme.com/room/owasptop10)  
    
- [ToolsRUs (Walkthrough)](https://tryhackme.com/room/toolsrus)
- [EasyCTF (Challenge)](https://tryhackme.com/room/easyctf)



