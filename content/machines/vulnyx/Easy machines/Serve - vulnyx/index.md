+++
title = "Serve - Vulnyx"
date = 2024-01-21T12:18:15-06:00
summary="Get ready for a long read, despite being an easy machine, took me a while to figure it out."
tags=["python","keepass","pentesting","vulnyx","easy","password bruteforcing","hydra","john"]
series=["Vulnyx Easy"]
series_order=2
+++

## Machine Info
| Name | Serve |
| ---- | ---- |
| OS | Linux Debian |
| Author | d4t4s3c |
| Site | [Vulnyx](https://vulnyx.com) |
| Difficulty | Easy |

## Enumeration
Lets run our nmap scan first:
`sudo nmap -oN serve_scan.txt -vv -sS -sC -sV -T4 -p- 192.168.25.18`

![](imagenes/Pasted%20image%2020240122185116.png)

Both port 22 and 80 open, lets check the webpage first:

![](imagenes/Pasted%20image%2020240122185250.png)

Default apache2 page, lets run our directory enumeration with gobuster, to see if we find some interesting directories:
`gobuster dir -u http://192.168.25.18/ -w /usr/share/wordlists/seclists/Discovery/Web-Content/common.txt -x php,html,txt `

![](imagenes/Pasted%20image%2020240122185506.png)

We find three interesting directories that we can access, lets check each one of them manually, and then run another gobuster scan on them to see if we find anything, first, I'm going to check the notes.txt file:

![](imagenes/Pasted%20image%2020240122185740.png)

We find some critical information,(also, a possible username, *teo*) and if we remember from our gobuster scan further above, we found the `/secrets` directory earlier, so, lets run a scan on that directory, but now, with a different list, so we can be 100% certain that we are going the get something useful:

`gobuster dir -u http://192.168.25.18/secrets/ -w /usr/share/wordlists/seclists/Discovery/Web-Content/directory-list-2.3-small.txt -x php,html,txt`

*Note the different list and the change of directory for the scan.*

While the scan ran, I checked the `/javascript` directory, and, as expected, it's forbidden:

![](imagenes/Pasted%20image%2020240122190229.png)

Now, I wait for the scan to end to see if we find something interesting...

![](imagenes/Pasted%20image%2020240122190402.png)

Nothing... Now, lets remember the message, especially, this part *Change x for your employee number*, now, maybe, if we create our own list with an array of numbers, lets say, 1 through 100, maybe we'll find something? So, I create a simple python script that count until 100 and then stops, and then , transfer that to a text file, there are many ways to do this, you may find a better or optimized way, but anyways, here's how I did it:

![](imagenes/Pasted%20image%2020240122191750.png)

1. Declare a variable, and start the count at 0, this , because the way python counts, rather than starting counting from 1, it starts with 0, so we have to keep that in mind.
2. We use a `while` loop, in simple terms, the loop will the do process that's *indented to it* until it meets the condition it will cause it to break, in this case, it will stop the counter when the number is less than 101. And the `n+=1` is a fancy notation that python has, it's the same as doing this `n=n+1`, which, is what we want:
We try the code and see if it works:

![](imagenes/Pasted%20image%2020240122192324.png)

Great! Off by one number, we simply correct 101 and put 100 in our code and it should work, and now, to get that in a text file we simply use the `>` redirection operator to save it into a text file:

`python3 counter.py > numberlist.txt`

![](imagenes/Pasted%20image%2020240122192522.png)

Now, lets try our gobuster enumeration again:

![](imagenes/Pasted%20image%2020240122192700.png)

Nothing still... Maybe, we can try downloading the `/secrets` directory with `wget`?
Still nothing, it only downloaded an empty html file. I decide to run another gobuster scan, initial ip address, but this time with a bigger list, while the scan was running, I remember that I didn't check the `/secrets` directory, maybe there's something hidden there I didn't notice, I check the source code of the page:

![](imagenes/Pasted%20image%2020240122193721.png)

There's 8 lines, but they're blank. Now, we have a username, maybe we can try bruteforcing a password? Remember the SSH port was open, lets  try that, personally, I use hydra:

`hydra -l teo -P /usr/share/wordlists/rockyou.txt 192.168.25.18 ssh -t 4`

*After a while, it seem that the password cracking was going nowhere, in the case of being an easy to crack password, with this list it should take 60 seconds max, so I decide to terminate the password cracking and keep trying for other ways to find a way in.*

While that's running, I remember I didn't run a scan on the `/javascript` directory, just for the sake of confirmation that there's nothing of interest there, I run a scan:

![](imagenes/Pasted%20image%2020240122194643.png)

We indeed find another directory there, which , we are also forbidden to check, lets run a scan on that directory now:

![](imagenes/Pasted%20image%2020240122195300.png)

Another `/jquery` directory, but look at that a `200` code! We can access this page, we take a look:

![](imagenes/Pasted%20image%2020240122195356.png)

Some javascript code, maybe, it's using components that are within that directory? Lets run another scan:

![](imagenes/Pasted%20image%2020240122200547.png)

Nothing... Lets backtrack, the IT team mentioned a *database*, within the `/secrets` directory, maybe, if we add some common database extension to our scan we'll find something, and, first and foremost, let's check what file extensions could be used to store passwords, and I found one that might be the one we are looking for:

![](imagenes/Pasted%20image%2020240122201844.png)

I add that extension to the gobuster scan, and try again,while the scan was running , I find more info:

![](imagenes/Pasted%20image%2020240122202131.png)

And, it really makes sense, since, storing a password in a plain SQL file is not safe at all, and another thing to note is, in both linux and macOS, the name changes to *KeePassX*, so, maybe, the file extension changes too, lets wait and see if the scan throw any results:

![](imagenes/Pasted%20image%2020240122202847.png)

Nothing, lets try *kdbx*, and see if we find something:

![](imagenes/Pasted%20image%2020240122203004.png)

There we go! We stop the scan, and download the file, and see what we can do with it:

![](imagenes/Pasted%20image%2020240122203155.png)

And, of course, we use `cat` on it,and it throws nothing useful:

![](imagenes/Pasted%20image%2020240122203232.png)

So, time to do some research, lets find out a way to extract the information of this file, and very quickly, I find a [blogpost by "the dutch hacker"](https://www.thedutchhacker.com/how-to-crack-a-keepass-database-file/) on how to crack this type of file, and, it's a tool we've used before, *john*, but the syntax this time is different (*last time, we used john to crack the password of a zip file*):

![](imagenes/Pasted%20image%2020240122203534.png)

So, lets get that hash, and crack the password(s) in this database:

![](imagenes/Pasted%20image%2020240122204033.png)

We find the master password of the *database*, lets download the program *KeePassX*, install it and try open the password database:

Apparently, the version where the database was created is no longer developed, lets find out if we can open this file with the newer version.

![](imagenes/Pasted%20image%2020240122204806.png)

We can, lets download the program, and try to open the file:

![](imagenes/Pasted%20image%2020240122204742.png)

![](imagenes/Pasted%20image%2020240122210253.png)


We have the username : *admin* and the password, and, we see another thing ,a name `webdav`, lets remember our initial gobuster scan, we found a directory with that name. Lets check it:

![](imagenes/Pasted%20image%2020240122210435.png)

Great, lets use the credentials we just found and see the contents of the page, and apparently, *we can't access it*, now, the password ends with *XXX*, and if we remember the message, we have to replace those X with the employee id of *teo*, which apparently, are three numbers, we can brute force this, so, after doing a bit of research I came up with this, I found this in a [reddit post](https://www.reddit.com/r/hacking/comments/ek2b5m/how_to_brute_force_a_popup_box/).

![](imagenes/Pasted%20image%2020240122212314.png)

I have the username *teo(admin too)*, but now, how do I find the password, we know its a combination of a maximum of *three numbers*, so, remember the script we coded earlier, we can adapt it, so we don't type each number manually:

![](imagenes/Pasted%20image%2020240122212750.png)

We try it, and see if it works as intended:

![](imagenes/Pasted%20image%2020240122212835.png)

Great,we export that to a text file.Now lets adapt the hydra command we saw above.

![](imagenes/Pasted%20image%2020240122213538.png)

We get something strange, instead of starting the crack, it did that instead, next up, I found the following in the same post:

![](imagenes/Pasted%20image%2020240122213701.png)

I adapt my command, and it looks like this: `hydra -l admin -P bruteforce_pass.txt -s 80 -f 192.168.25.18 http-get /webdav`, we start the crack and...

![](imagenes/Pasted%20image%2020240123122950.png)

We get a password! Lets try to login again to the webpage:

![](imagenes/Pasted%20image%2020240122214047.png)

Ok, we are in, but theres nothing there... Maybe, if we try to login with SSH?

![](imagenes/Pasted%20image%2020240122214748.png)

Ok, we have and empty directory, maybe, there's some way to upload a file? Via our shell session of course, because the page barely has any features. I find [this](https://reqbin.com/req/c-dot4w5a2/curl-post-file):

![](imagenes/Pasted%20image%2020240122215005.png)

I try uploading a test text file:

![](imagenes/Pasted%20image%2020240122215212.png)

Well, good for us we already have the credentials, lets check which flags we have to include:

![](imagenes/Pasted%20image%2020240122215318.png)

Great, lets add that, and try again:

![](imagenes/Pasted%20image%2020240122215741.png)

In the same page, we look up how to use the `-u` flag:

![](imagenes/Pasted%20image%2020240122215716.png)

Lets try again:

![](imagenes/Pasted%20image%2020240122220004.png)

Same thing, lets check the help page, to see if we are missing something:

![](imagenes/Pasted%20image%2020240122220125.png)

So, what's the difference with the `-d` key?

![](imagenes/Pasted%20image%2020240122220232.png)

So , apparently, we have been sending a `POST` request to the webpage, instead of uploading the file, we change the switch and try again:

![](imagenes/Pasted%20image%2020240122220426.png)

So, in the image above, I saw something that interested me the `--digest` flag, I do some research about what it does. And I find in this [blog](https://www.hackingarticles.in/understanding-http-authentication-basic-digest/) what it is:

![](imagenes/Pasted%20image%2020240122221244.png)

Apparently its the "pop up box", as I called the first time I saw, now I know its proper name, so, I add the flag `--digest` and try again:

![](imagenes/Pasted%20image%2020240122221441.png)

Sucess! Lets see the webpage:

![](imagenes/Pasted%20image%2020240122221522.png)

Great, but, now the question is, can we open this file?

![](imagenes/Pasted%20image%2020240122221549.png)

We can! This is good, it means, if we upload, lets say, a *php* file, it'll run, this will allow us to get a reverse shell, let now upload our *php reverse shell* and set up our netcat listener:

In my case I'm going to use this [php payload](https://pentestmonkey.net/tools/web-shells/php-reverse-shell) , we edit the file with our info, and upload with the command we used earlier:

![](imagenes/Pasted%20image%2020240122221853.png)

![](imagenes/Pasted%20image%2020240122222523.png)

![](imagenes/Pasted%20image%2020240122222449.png)

All is set, we click on the `.php` file and check our listener:

## Initial access

![](imagenes/Pasted%20image%2020240122222607.png)

Great! We are in as the `www-data` user, lets check what can we do:

![](imagenes/Pasted%20image%2020240122222828.png)

We see the user `teo` can run that command as a super user, without a password, lets try to get into that account , lets check how we can get in:
*before hand, we run the following command* `python3 -c 'import pty; pty.spawn("/bin/bash")'` *which will make our session a bit more functional* .
I run a `find / -name "*.txt" 2>/dev/null` command, to see if I find any interesting text files *see this [link](https://explainshell.com/explain?cmd=find+%2F+-name+"*.txt"+2>%2Fdev%2Fnull) if you want to know what this command does:

I find a bunch of text files, but none where interesting, I search in the [hacktricks](https://book.hacktricks.xyz/linux-hardening/privilege-escalation/payloads-to-execute#www-data-to-sudoers) page for some way to escalate privileges or switch users:

![](imagenes/Pasted%20image%2020240122224740.png)

Let's try each one of those:

![](imagenes/Pasted%20image%2020240122225024.png)

None of those worked, lets try something else.

So, we can run the ,	`wget` command as *teo*, if we use the `-u` switch, when we use `sudo`, so maybe, while we can't access data in this machine, maybe we can forward other information? Lets check the man pages of the `wget command`:

![](imagenes/Pasted%20image%2020240122232354.png)

I find this excerpt in the man pages of the wget command, I try it first with a string, I set up my listener:

![](imagenes/Pasted%20image%2020240122232637.png)

![](imagenes/Pasted%20image%2020240122232706.png)

Good! This means, most likely, we can post the contents of a file with `--post-file`. The user *teo*, if we are lucky, most likely has a `/.ssh` directory, with his private ssh `id_rsa` key in it, maybe we can show that data. Lets try that now:

![](imagenes/Pasted%20image%2020240122232950.png)

There we have it! Lets copy that, name it `id_rsa` and give limited privileges with `chmod 400 id_rsa` and then, we try to log in with ssh:

![](imagenes/Pasted%20image%2020240122233232.png)

We are for a passphrase, let's use `ssh2john` and and crack this hash:

![](imagenes/Pasted%20image%2020240122233407.png)

We have a password, lets try to login again:


## Privilege escalation

![](imagenes/Pasted%20image%2020240122233441.png)

We are in, lets look for the user flag:

![](imagenes/Pasted%20image%2020240122233714.png)

Now lets see how we can escalate privileges:

![](imagenes/Pasted%20image%2020240122233911.png)

We check the contents of that file:

![](imagenes/Pasted%20image%2020240122233942.png)

It's a ruby file, lets search how we can escalate privileges with this info,first, lets see what the script does:

![](imagenes/Pasted%20image%2020240123000105.png)

It's a script, we try what the script suggests:

![](imagenes/Pasted%20image%2020240123000218.png)

It takes us to an interactive shell, like **vim**, so, maybe we can escape it, lets try typing `!/bin/bash`:

![](imagenes/Pasted%20image%2020240123000317.png)

We have root, lets look for the root flag, and pwn the machine:

![](imagenes/Pasted%20image%2020240123000407.png)

And with this, we have pwned the machine, quite a long one, but I learned a lot from this one. 

## Things I learned from this machine
- Check for files with other less common extensions, as it was seen in this machine, we found a `.kdbx` file, corresponding to the app *KeePass*, which allowed to gain initial access, keep this fact in mind for future CTF machines.
- Read the man pages of "basic" commands, such as `curl` or `wget`, they can useful in ways a beginner like me wouldn't expect, in this case, we retrieved critical information that allowed us to login as a more priveleged user, a significant improvement compared to the restricted `www-data` user.
- Learned how to brute force an `http digest feed` with hydra. With a **partial password** obtained from the *KeePass* database we downloaded from the webpage.
- If we have, lets say, a custom script, try running it first, and see what it does, and then check the contents of the code. If we have an interactive shell within the script , we can try doing a escape sequence with `!/bin/bash`.


