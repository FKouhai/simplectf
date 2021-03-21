Welcome to my first write up its about the Try Hack Me box _Simple CTF_

https://www.tryhackme.com/room/easyctf

I want to start by saying im a novice in the CTF environment, so any comment about how to improve my craft will be appretiated, also im gonna use the IP that was given to me for claritie's sake


First I started ennumerating the services exposed in the machine by using

_nmap -sC -sN -oN nmap/nmap.log 10.10.237.53_

And I discovered the following services

 - FTP running on port 21 with anonymous login enabled
 - HTTPD running on port 80 
 - SSH running on port 2222

After discovering all those services, I decided to go access the web server and append an /robots.txt at the end of the URI which led me to discover robots.txt wasn't going to be useful this time, so i decided to use gobuster with my medium sized directory list

_gobuster dir -w directory-list-2.3-medium.txt -u http://10.10.237.53/_

Which discovered the directory /simple, so i also decided to run nikto while im loging in to the machine with FTP
 _nikto -h 10.10.237.53 | tee nikto.log_

ftp 10.10.237.53
user:anonymous
password: 

The anon user didnt have any password and we found an FTP directory inside of it but it wasnt any interesting.

Nikto didnt gave much information either, so I decided to get into that address http://10.10.237.53/simple and got into a CMS called CMS Made Simple, so after poking around it I run msfconsole to check if i had some exploits for and i also run a quick google search to get information about any related CVE to this CMS and low and behold I found the _CVE 2019-9053_.

After some google searches I found a github gist page _"https://gist.github.com/pdelteil/6ebac2290a6fb33eea1af194485a22b1"_
With the exploit written in python so I downloaded the code, also read what it did (its not advisable to run w/e without knowing what you are doing) and so I was ready to go, so the script needs 3 arguments or flags the first one -u for the URI of the server, the second one a -W with the wordlist path and a -c for cracking the password.

_python2 cve-2019-9053.py -u http://10.10.237.53/simple/ -w rockyou.txt -c_

And after some minutes it was done and found the user *mitch* and its password

Right after having this information we ssh'ed into the machine and got the first flag on the user's home directory


I kept investigating a bit more in the machine but the mitch user was banned from running sudo su to get root access

So I decided to scp linpeas to the machine 

_scp -p 2222 linpeas.sh mitch@10.10.237.53:/home/mitch_

Then i gave the script execution permissions 

_chmod +x linpeas.sh_

And then I run it after a few minutes i saw that the user mitch could run vim with sudo and it wouldnt ask for a password, and this is interesting for us since vim is a really powerful editor that's able to invoke a shell so running it with sudo would help us scalating our privilege to get root access and so I did

I run

_sudo vim_

Then in vim you have to enter in  the command mode with : and type in the following
_:set shell=bin/sh_
_:shell_

and so we got our shell to check the user we were running the sh shell, so i run id to check that indeed we were root
so after having a shell as root we pwned the machine so it was time to get that flag


I hope this write up was useful and enterntaining and I hope you liked it :)

GL HF stranger 
