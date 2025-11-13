
##### have credentials - 
smb open -> psexec->using exploit/windows/smb/psexec

ssh open

#####
-couldnt upload a php file, tried .phtml and it works


-the attackbox has the mfsconsole version you need to run the room

-לקפוץ session
if you have system you cna open task manager and see what active sessions are there, if a user has an active session you can jump to it and act as the user

have admin on a endpoint

get there with psexec cmd - or just enter your creds 
open cmd as system -   
''psexec -i -s cmd.exe''
open taskmgr

jump to session


-hashcracking - https://md5hashing.net/hash

-the shit is in phazes and each question is based on the situation you got to from the last answer

if you had a question "qhat is the hidden directrory"
and you found it, THAN THE ANSWER TO THE LAST Q will probly be in that directory big bro

-if you find a image download with wget and do this to it:

exiftool image_name.jpg

binwalk image_name.jpg

steghide extract -sf image_name.jpg

stegseek binarycodepixabay easypeasy_1596838725703.txt

file image_name.jpg

exiftool image_name.jpg

strings image_name.jpg 


-from revshell to interactive shell:

https://fahmifj.medium.com/get-a-fully-interactive-reverse-shell-b7e8d6f5b1c1

revshells- https://www.revshells.com/

revshells- https://github.com/swisskyrepo/PayloadsAllTheThings/blob/master/Methodology%20and%20Resources/Reverse%20Shell%20Cheatsheet.md

revshells- https://web.archive.org/web/20200901140719/http://pentestmonkey.net/cheat-sheet/shells/reverse-shell-cheat-sheet

revshells-  `/usr/share/webshells`.  [SecLists repo](https://github.com/danielmiessler/SecLists)

 the infamous [PentestMonkey php-reverse-shell](https://raw.githubusercontent.com/pentestmonkey/php-reverse-shell/master/php-reverse-shell.php)

hash anaylyzer -  https://www.dcode.fr/hash-identifier

encoding analyzer - https://www.dcode.fr/cipher-identifier


#### revshell on windwos server
on attacker maching:
sudo nc -lvnp 443

on windows server:
![[Pasted image 20251103143948.png]]located in https://tryhackme.com/room/introtoshells

לשמוע את השיר נובמבר מירי מסיקה ב-1 בנובמבר


When the server receives a request for a directory (like `http://...:4321/`) and doesn't find a default index file (like `index.html` or `index.php`) in that location, a secure server should return a **403 Forbidden** error.

- **What happened:** In this case, the server is configured to **automatically generate an HTML page listing all the contents** of that directory. This feature is called **Directory Indexing** (or `Indexes` in Apache).
![[Pasted image 20251104132301.png]]


user@machine$ hydra -l admin -P 500-worst-passwords.txt 10.10.x.x http-post-form "/login-get/index.php:username=^USER^&password=^PASS^:F=Login failed" -f

when http is mad annoying check what you get back when its failed
when checking for login try to do http-post

-when i did get with the right creds i still got the login failed page because it didnt really post and submitted the creds


#### Verify the string Hydra will look for (local check)

send a manual POST and show a short snippet

curl -s -X POST 'http://10.10.105.184/login-post/index.php' \
  -d 'username=burgess&password=wrong' -i | sed -n '1,120p'     # h
