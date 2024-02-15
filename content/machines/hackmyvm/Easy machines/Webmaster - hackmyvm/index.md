+++
title = "Webmaster"
date = 2024-01-28T22:57:32-06:00
summary="Another machine by sml, and pretty interesting too, introduced me to the use of the dig command."
tags=["hackmyvm","pentesting","nginx","dns","dig"]
series=["Hackmyvm Easy"]
series_order=4
draft= false
+++
## Machine Info
| Name       | webmaster |
| ---------- | --------- |
| OS         | Linux     | 
| Author     | sml          |
| Site       | [hackymyvm](https://hackmyvm.eu)          |
| Difficulty | Easy          |

## Recon and enumeration
We start with our service recon with nmap:

![](imagenes/Pasted%20image%2020240128185639.png)

3 ports open. The usual , port 80 and 22, and a first encounter with port *53*, which appears to be a *DNS* server, lets take a look at the webpage now:

![](imagenes/Pasted%20image%2020240128185754.png)

First clue, a password *might* be in a *.txt file*, lets take a look at the webpage source code:

![](imagenes/Pasted%20image%2020240128185847.png)

A comment *webmaster.hmv*, it may have a meaning in pwning this machine ,or not... Well, since we can't do anything else manually in this page, lets run our usual gobuster directory enumeration:

![](imagenes/Pasted%20image%2020240128201038.png)

*I did some more scans, but found nothing...* 

Lets backtrack a bit, we found a comment, *webmaster.hmv*, now, maybe, that's a **domain name**? We add that domain to our `/etc/hosts`, like this:

![](imagenes/Pasted%20image%2020240128201308.png)

We try viewing the webpage again:

![](imagenes/Pasted%20image%2020240128201329.png)

It works, good. Now, we have a *domain name*, and we have *port 53* open, lets consult [hacktricks](https://book.hacktricks.xyz) to see what we can find. Eventually,I find something in this [section](https://book.hacktricks.xyz/network-services-pentesting/pentesting-dns#zone-transfer).

![](imagenes/Pasted%20image%2020240128201505.png)

Didn't really know what `AXFT` was, so I [googled](https://www.briskinfosec.com/blogs/blogsdetail/DNS-Zone-Transfer#:~:text=AXFR%20(Asynchronous%20Full%20Transfer%20Zone,DNS%20server%20is%20well%20synced.)) it:

![](imagenes/Pasted%20image%2020240128202047.png)

Since we have both `IP` and `domain name`, lets try with the second command of the `zone transfer section`.

![](imagenes/Pasted%20image%2020240128202241.png)

## Initial Access
And, we most likely have found some `SSH` login credentials, lets try to login as a user:

![](imagenes/Pasted%20image%2020240128202414.png)

We get the user flag, now then, and we have a script in the home directory, lets check what it does:

![](imagenes/Pasted%20image%2020240128203911.png)

Mm, I try the script, and it shows this:

![](imagenes/Pasted%20image%2020240128203946.png)

Of course, it isn't going to show to root flag, because it isn't in the directory, let's try to add `/root/` before `root.txt`:

![](imagenes/Pasted%20image%2020240128204437.png)

No change... Lets see what else we can do:

![](imagenes/Pasted%20image%2020240128204536.png)

Mmm, we have super user privileges with the *nginx web server*, lets do some research on how we can exploit that. So, after a while, I *got the root flag*, but didn't escalate privileges, anyways, here's what I did:

I found this script from this [blog](https://darrenmartynie.wordpress.com/2021/10/25/zimbra-nginx-local-root-exploit/) by darrenmarty:

![](imagenes/Pasted%20image%2020240128211456.png)

In a nutshell, what it does, it creates a *nginx.conf* file in the `/tmp` directory, sets the `user` as `root`, establishes some capabilities of the server, then, **starts an http server on the root directory**. After that, it launches *nginx* with `sudo` privileges with the config file it just created and shows the contents of the `/etc/passwd` directory. 

Now, I adapted this script for what I need from this machine, so, I did this:

![](imagenes/Pasted%20image%2020240128211843.png)

You may ask, *why set up the listener to the target machine ip*? Because of this:

![](imagenes/Pasted%20image%2020240128211941.png)

The script successfully starts the http server with the `config` file the script created, but, in this machine, `curl` is not installed or the user isn't allowed to use it. But, the http server is running, so, we can download try to download the file with `wget`. 

![](imagenes/Pasted%20image%2020240128212212.png)

## Privilege Escalation
And we have the root flag, but, **we didn't escalate privileges to super user**. Checking the blog even further, the author did escalate privileges, here's how he did it:

![](imagenes/Pasted%20image%2020240128214019.png)

I don't know what *dynamic linker* means, so I google it:

![](imagenes/Pasted%20image%2020240128214352.png)

![](imagenes/Pasted%20image%2020240128214439.png)

So, I use the [modified script](https://github.com/darrenmartyn/zimbra-hinginx/blob/main/hinginx.sh) adapt it to my needs, and see it we get a root shell:

![](imagenes/Pasted%20image%2020240128220024.png)

Well, that's not good, maybe there's an easier way. Now, I decide to consult another writeups, to see if someone got a root session with a different method. Consulting the writeup of the hackmyvm user [ARZ](https://arz101.medium.com/hackmyvm-webmaster-6f5862000c0a). It seems, in the *html directory* we have **unrestricted privileges**, which allows us to write a php script, what will allow us to execute a reverse shell to our machine. Good thing to keep in mind in the future.

![](imagenes/Pasted%20image%2020240128222557.png)

![](imagenes/Pasted%20image%2020240128222531.png)

![](imagenes/Pasted%20image%2020240128222758.png)

![](imagenes/Pasted%20image%2020240128222620.png)

## Things I learned from this machine
- If there are domains involved, add them to my `/etc/hosts` file, and run related domain commands(`dig` or `nslookup`) to obtain information about the machine.
- **Always** check before hand the content of the directory where the web page assets are when logged in as a user, if we are lucky, we could have unrestricted privileges on that directory, and write a simple php code to **enable remote code execution, and get a reverse shell on our machine**.

