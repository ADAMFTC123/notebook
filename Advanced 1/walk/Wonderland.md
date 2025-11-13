

22/tcp open  ssh     OpenSSH 7.6p1 Ubuntu 4ubuntu0.3 (Ubuntu Linux; protocol 2.0)
80/tcp open  http    Golang net/http server (Go-IPFS json-rpc or InfluxDB API)
|_http-title: Follow the white rabbit.

MAC Address: 02:9D:6D:D6:7F:2B (Unknown)
OS: Linux;



==> DIRECTORY: http://10.10.153.137:80/img/                                    
+ http://10.10.153.137:80/index.html                    
==> DIRECTORY: http://10.10.153.137:80/r/                                      
                                                                               
---- Entering directory: http://10.10.153.137:80/img/ ----
+ http://10.10.153.137:80/img/index.html (CODE:301|SIZE:0)                     
                                                                               
---- Entering directory: http://10.10.153.137:80/r/ ----
+ http://10.10.153.137:80/r/a (CODE:301|SIZE:0)                                
+ http://10.10.153.137:80/r/index.html (CODE:301|SIZE:0)                       
                                                                               
-----------------
END_TIME: Mon Nov 10 12:54:02 2025
DOWNLOADED: 13836 - FOUND: 4

 http://10.10.153.137:80/img/ 
http://10.10.153.137:80/r/a/b/b/i/t


wget http://10.10.153.137:80/img/

exiftool image_name.jpg

binwalk image_name.jpg

steghide extract -sf image_name.jpg

stegseek binarycodepixabay easypeasy_1596838725703.txt

file image_name.jpg

exiftool image_name.jpg

strings image_name.jpg 

alice:HowDothTheLittleCrocodileImproveHisShiningTail

#### sudo -l 
alice@wonderland:/$ sudo -l
[sudo] password for alice: 
Matching Defaults entries for alice on wonderland:
    env_reset, mail_badpass,
    secure_path=/usr/local/sbin\:/usr/local/bin\:/usr/sbin\:/usr/bin\:/sbin\:/bin\:/snap/bin

User alice may run the following commands on wonderland:
    (rabbit) /usr/bin/python3.6 /home/alice/walrus_and_the_carpenter.py


one idea:
we know we can run /usr/bin/python3.6 /home/alice/walrus_and_the_carpenter.py
as rabbit so we can change  /home/alice/walrus_and_the_carpenter.py

#### suid

alice@wonderland:~$ find / -type f -perm -04000 -ls 2>/dev/null
   394282     44 -rwsr-xr--   1 root     messagebus    42992 Jun 10  2019 /usr/lib/dbus-1.0/dbus-daemon-launch-helper
   394475     16 -rwsr-xr-x   1 root     root          14328 Mar 27  2019 /usr/lib/policykit-1/polkit-agent-helper-1
   394471    428 -rwsr-xr-x   1 root     root         436552 Mar  4  2019 /usr/lib/openssh/ssh-keysign
   524949    100 -rwsr-xr-x   1 root     root         100760 Nov 23  2018 /usr/lib/x86_64-linux-gnu/lxc/lxc-user-nic
   394289     12 -rwsr-xr-x   1 root     root          10232 Mar 28  2017 /usr/lib/eject/dmcrypt-get-device
   393716     44 -rwsr-xr-x   1 root     root          44528 Mar 22  2019 /usr/bin/chsh
   393920     40 -rwsr-xr-x   1 root     root          37136 Mar 22  2019 /usr/bin/newuidmap
   394097     20 -rwsr-xr-x   1 root     root          18448 Jun 28  2019 /usr/bin/traceroute6.iputils
   393714     76 -rwsr-xr-x   1 root     root          76496 Mar 22  2019 /usr/bin/chfn
   393936     60 -rwsr-xr-x   1 root     root          59640 Mar 22  2019 /usr/bin/passwd
   393809     76 -rwsr-xr-x   1 root     root          75824 Mar 22  2019 /usr/bin/gpasswd
   393919     40 -rwsr-xr-x   1 root     root          40344 Mar 22  2019 /usr/bin/newgrp
   393663     52 -rwsr-sr-x   1 daemon   daemon        51464 Feb 20  2018 /usr/bin/at
   393918     40 -rwsr-xr-x   1 root     root          37136 Mar 22  2019 /usr/bin/newgidmap
   393956     24 -rwsr-xr-x   1 root     root          22520 Mar 27  2019 /usr/bin/pkexec
   394061    148 -rwsr-xr-x   1 root     root         149080 Jan 31  2020 /usr/bin/sudo
   655427     32 -rwsr-xr-x   1 root     root          30800 Aug 11  2016 /bin/fusermount
   655971     28 -rwsr-xr-x   1 root     root          26696 Mar  5  2020 /bin/umount
   655478     64 -rwsr-xr-x   1 root     root          64424 Jun 28  2019 /bin/ping
   655970     44 -rwsr-xr-x   1 root     root          43088 Mar  5  2020 /bin/mount
   655494     44 -rwsr-xr-x   1 root     root          44664 Mar 22  2019 /bin/su


#### capabilities
alice@wonderland:~$ getcap -r / 2>/dev/null
/usr/bin/perl5.26.1 = cap_setuid+ep
/usr/bin/mtr-packet = cap_net_raw+ep
/usr/bin/perl = cap_setuid+ep

you cant use perl

#### crontab
alice@wonderland:~$ cat /etc/crontab

/etc/crontab: system-wide crontab
Unlike any other crontab you don't have to run the `crontab'
command to install the new version when you edit this file
and files in /etc/cron.d. These files also have username fields,
that none of the other crontabs do.

SHELL=/bin/sh
PATH=/usr/local/sbin:/usr/local/bin:/sbin:/bin:/usr/sbin:/usr/bin

17 *	* * *	root    cd / && run-parts --report /etc/cron.hourly
25 6	* * *	root	test -x /usr/sbin/anacron || ( cd / && run-parts --report /etc/cron.daily )
47 6	* * 7	root	test -x /usr/sbin/anacron || ( cd / && run-parts --report /etc/cron.weekly )
52 6	1 * *	root	test -x /usr/sbin/anacron || ( cd / && run-parts --report /etc/cron.monthly )



we know we can run /usr/bin/python3.6 /home/alice/walrus_and_the_carpenter.py
as rabbit so we can change  /home/alice/walrus_and_the_carpenter.py

we know that to edit it you need to be root - (means we wont be using it to find users.txt, but we still maybe use the rabbit shit instead, so we need to find  a way to edit the rule or make a file, delete.)





we know we cant change the command
-maybe we can change what the command mean-alias,env variables
-understand what runs when you run python3.6 /home/alice/walrus_and_the_carpenter.py maybe there a inline script that u can modify

export LOGNAME="alice"

export SHLVL=0


when python imports it first looks at the directory of the executed script


hatter:

hatter :WhyIsARavenLikeAWritingDesk?

alice:

alice:HowDothTheLittleCrocodileImproveHisShiningTail
![[Pasted image 20251113161326.png]]

well create a random.py in the directory of walrus