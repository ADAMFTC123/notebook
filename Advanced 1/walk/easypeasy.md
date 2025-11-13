# walkthroug

##### How many ports are open?
nmap -sS -sV -p- ipaddress
##### What is the version of nginx?
nmap -sS -sV -p- ipaddress
##### What is running on the highest port?
nmap -sS -sV -p- ipaddress

##### Using GoBuster, find flag 1.:
gobuster dir -u http://ipaddress -w /usr/share/wordlists/dirbuster/directory-list-2.3-medium.txt

found /hidden
surfed to it  -> view page source

![[Pasted image 20251104091127.png]]

it uses an image so i surfed to it and  downloaded it 

it had nothing

i did gobuster again
found the path hidden/whatever
surfed to it  -> view page source

![[Pasted image 20251104092118.png]]
we see == so itried base 64

flag{f1rs7_fl4g}

##### Further enumerate the machine, what is flag 2?

on port 65524 i did the same stages 

gobuster dir -u http://ipaddress -w /usr/share/wordlists/dirbuster/directory-list-2.3-medium.txt

found /robots.txt
surfed to it  -> 

![[Pasted image 20251104092619.png]]

we need to identify the hash
we can use hash -identifier

![[Pasted image 20251104095701.png]]
![[Pasted image 20251104095720.png]]
or use this site- https://www.dcode.fr/hash-identifier
adn well decode with - https://md5hashing.net/hash/md5/a18672860d0510e5ab6699730763b250
flag{1m_s3c0nd_fl4g}

##### Crack the hash with easypeasy.txt, What is the flag 3?

on port 65524 i obiously checked the page source and found:

![[Pasted image 20251104100756.png]]

##### What is the hidden directory?
on port 65524 i obiously checked the page source and found:
![[Pasted image 20251104100301.png]]

i checked these links and found nothing
http://10.10.204.5:65524/#flag

![[Pasted image 20251104095953.png]]
encoding analyzer - https://www.dcode.fr/cipher-identifier
![[Pasted image 20251104100056.png]]
![[Pasted image 20251104100144.png]]

##### Using the wordlist that provided to you in this task crack the hash what is the password?

i surfed to the directory and cheked the source page 
![[Pasted image 20251104100518.png]]

![[Pasted image 20251104095701.png]]
![[Pasted image 20251104095720.png]]
or use this site- https://www.dcode.fr/hash-identifier
![[Pasted image 20251104100616.png]]
adn well decode with - https://md5hashing.net/hash/md5/a18672860d0510e5ab6699730763b250


![[Pasted image 20251104100842.png]]

![[Pasted image 20251104101056.png]]


##### What is the password to login to the machine via SSH?
what do we do when we see a image file on the web???
we fuck it!

on  10.10.204.5:65524/n0th1ng3ls3m4tt3r/  we see:
![[Pasted image 20251104101210.png]]

i donwloaded the image and did:

![[Pasted image 20251104101607.png]]
with the password we found from the hash
![[Pasted image 20251104101700.png]]

![[Pasted image 20251104101734.png]]


##### What is the user flag?
logged in:
ssh -p 6498  boring@10.10.204.5

found flag:
![[Pasted image 20251104101956.png]]

decoded ceaser cipher with - https://cryptii.com/pipes/caesar-cipher
![[Pasted image 20251104102055.png]]

##### What is the root flag?
to get root flag well use a pe technice:
 **Check for SUID binaries:** Run `find / -perm -4000 -type f 2>/dev/null` to locate SUID files that may allow you to execute commands with elevated privileges.
 
 `suid -l`  to check rule book for which suid process dont require password
    
2. **Look for misconfigurations:** Check the `/etc/passwd` and `/etc/shadow` files for any abnormal entries.
    
3. **Search for writable files:** Use `find / -writable -type f 2>/dev/null` to find files you can modify, which may assist in privilege escalation.
    
4. **Inspect cron jobs:** List scheduled tasks with `cat /etc/crontab` to see if any can be exploited.

![[Pasted image 20251104102208.png]]
we can see that every minute a bash script with root privs runs
lets check if we can change the scrript:
![[Pasted image 20251104102248.png]]
we have write privs
![[Pasted image 20251104102918.png]]

![[Pasted image 20251104102942.png]]
![[Pasted image 20251104103647.png]]
# TIPS


https://medium.com/@kirimichris7/easy-peasy-tryhackme-detailed-walk-through-f2ab3ff401ae

-if youre stuck on fuinding directories and shit, open the ones you found and;

inspect them
steghide their images

-any hash or wierd text you see might just be an encoded flag


 ceaser cipher with - https://cryptii.com/pipes/caesar-cipher
 
hash analyzer - https://www.tunnelsup.com/

hash cracker - https://md5hashing.net/hash

-if you find a image download with wget and do this to it:

exiftool image_name.jpg

binwalk image_name.jpg

steghide extract -sf image_name.jpg

stegseek binarycodepixabay easypeasy_1596838725703.txt

file image_name.jpg

exiftool image_name.jpg

strings image_name.jpg 

-when dinding a directory keep using gobuster inside it 
-in dirbuster we can use a recursive mode

-if there is a lot of hash options use john