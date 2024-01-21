+++
title = 'Ready - Vulnyx'
date = 2024-01-20T16:44:27-06:00
draft = false
+++

| Name       | Ready   |
| ---------- | ------- |
| OS         | Linux   |
| Author     | d4t4s3c |
| Difficulty | Easy    |
| Site           | [vulnyx](https://vulnyx.com)        |

## Enumeration

![](/locoweb/vulnyx/easy/ready/ready%20(1).png)

Checking the web pages:

![](locoweb/ready%20(2).png)

![](vulnyx/easy/ready/ready%20(3).png)

Both are apache default pages, lets enumerate with gobuster just to be sure, while the scans run , lets check what redis is:

![](vulnyx/easy/ready/ready%20(4).png)

Apparently, we are dealing with a database, let's check how we can login with the command *redis-cli*:

![](vulnyx/easy/ready/ready%20(5).png)

Lets try to login without a username:

![](vulnyx/easy/ready/ready%20(6).png)

We are logged in! Lets search for the commons commands for redis:

![](vulnyx/easy/ready/ready%20(7).png)

Before that, we run a nmap enumeration command to see what more info we can find(*it seems, there's no databases in this server...):

![](vulnyx/easy/ready/ready%20(8).png)

*We found nothing of interest in the webpages*

![](vulnyx/easy/ready/ready%20(9).png)

I find this excerpt in the [hacktricks](https://book.hacktricks.xyz/network-services-pentesting/6379-pentesting-redis#ssh) page:

![](vulnyx/easy/ready/ready%20(10).png)

And in redis, I find the following directories:

![](vulnyx/easy/ready/ready%20(11).png)

Let's try doing those steps using the directory /root/.ssh

![](vulnyx/easy/ready/ready%20(12).png)

We export the key to the redis server and follow the steps above:

![](vulnyx/easy/ready/ready%20(13).png)

## Initial acess and privilege escalation
We try to log into the ssh session:

![](vulnyx/easy/ready/ready%20(14).png)

And we have root! We install unzip in this machine to extract the content of the zip:

![](vulnyx/easy/ready/ready%20(15).png)

We set up a simple http server the target machine with the following command `python3 -m http.server -b <TARGET-MACHINE-IP> <PORT>`

We download the file in our machine with `wget <TARGET-MACHINE-IP>:<PORT>/root.zip´

![](vulnyx/easy/ready/ready%20(16).png)

Now we use `zip2john` to print the hash of zip into a file and then crack it:

![](vulnyx/easy/ready/ready%20(17).png)

We extract the contents of the file:

![](vulnyx/easy/ready/ready%20(18).png)

We find the root flag, now me move on to the user flag:

![](vulnyx/easy/ready/ready%20(19).png)

With this, we have pwned the machine.

## What I learned from this machine
- Privesc with redis