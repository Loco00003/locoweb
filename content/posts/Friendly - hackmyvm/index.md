+++
title = "Friendly - Hackmyvm"
date = 2024-01-09T20:12:37-06:00
summary="My first machine, despite the easy difficulty, it took me quite some time"
draft = false
tags=["pentesting","hackmyvm","ftp"]
series=["Hackmyvm"]
series_order=2
+++

***
| Name  | Friendly  |
|---|---|
|  OS | Linux  |
|Author   |RiJaba1   |
|Difficulty   |Easy   |
|Site   |[hackmyvm](https://hackmyvm.eu)   |

***



Beforehand, we set up in our virtual box a NAT network, so our machines can communicate with each other, we assign the ip we desire to the network,and then in each machine check if both are on the same network:
![](imagenes/Pasted%20image%2020240109202337.png)
![](imagenes/Pasted%20image%2020240109213033.png)
![](imagenes/Pasted%20image%2020240109213047.png)


## Enumeration

We run an nmap scan on the machine:
```
Nmap 7.94SVN scan initiated Tue Jan  9 20:20:06 2024 as: nmap -oN friendly_scan.txt -vv -sS -sV -sC -T4 -p 0-9999 192.168.25.4
Warning: Hit PCRE_ERROR_MATCHLIMIT when probing for service http with the regex '^HTTP/1\.1 \d\d\d (?:[^\r\n]*\r\n(?!\r\n))*?.*\r\nServer: Virata-EmWeb/R([\d_]+)\r\nContent-Type: text/html; ?charset=UTF-8\r\nExpires: .*<title>HP (Color |)LaserJet ([\w._ -]+)&nbsp;&nbsp;&nbsp;'
Nmap scan report for 192.168.25.4
Host is up, received arp-response (0.00015s latency).
Scanned at 2024-01-09 20:20:06 CST for 14s
Not shown: 9998 closed tcp ports (reset)
PORT   STATE SERVICE REASON         VERSION
21/tcp open  ftp     syn-ack ttl 64 ProFTPD
| ftp-anon: Anonymous FTP login allowed (FTP code 230)
|_-rw-r--r--   1 root     root        10725 Feb 23  2023 index.html
80/tcp open  http    syn-ack ttl 64 Apache httpd 2.4.54 ((Debian))
| http-methods: 
|_  Supported Methods: HEAD GET POST OPTIONS
|_http-title: Apache2 Debian Default Page: It works
|_http-server-header: Apache/2.4.54 (Debian)
MAC Address: 08:00:27:A2:9F:C0 (Oracle VirtualBox virtual NIC)

Read data files from: /usr/bin/../share/nmap
Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
# Nmap done at Tue Jan  9 20:20:20 2024 -- 1 IP address (1 host up) scanned in 14.18 seconds
```

ftp port and http port open, we go to the page to see what info we can gather;
![](imagenes/Pasted%20image%2020240109202627.png)
It appears to be the default page of apache2, we run a directory enumeration scan to see if there's something of interest:
![](imagenes/Pasted%20image%2020240109202929.png)
As we can see, with both the small and medium list of dirbuster, we didnt find anything of interest on the web page, we move on to ftp. We try an *anonymous login and see it works*:
![](imagenes/Pasted%20image%2020240109203206.png)
We are in the ftp server, lets check for interesting files.
![](imagenes/Pasted%20image%2020240109204803.png)

We find the index.html file, which is nothing more than the page we saw before, and we can see we have read and write privileges over it, maybe we can upload something? I'll try with the text file of my nmap scan:
![](imagenes/Pasted%20image%2020240109205653.png)
And it uploaded correctly, if we check the page, we should see our scan:
![](imagenes/Pasted%20image%2020240109205725.png)
Now, we are gonna try uploading a php reverse shell, but first, we have to modify it's content, for the listener we are gonna run in our machine:
![](imagenes/Pasted%20image%2020240109210055.png)
We upload the exploit to the ftp server:
![](imagenes/Pasted%20image%2020240109210147.png)
We set up our netcat listener, and then on the webpage, we should go to the url bar and type this `http://192.168.25.4/php-reverse-shell.php` and hit enter, and we should get a shell on our listener:
![](imagenes/Pasted%20image%2020240109210347.png)

## Initial access and privilege escalation
We have the user shell, we list the directory contents:
![](imagenes/Pasted%20image%2020240109210430.png)
We go to the home directory to se if we find something there:
We find a posible *username: RiJaba1*
Inside the directory we find the user flag:
![](imagenes/Pasted%20image%2020240109210733.png)

Now, we need to do some privelege escalation, we run a sudo -l
![](imagenes/Pasted%20image%2020240109210836.png)
Interesting, we search gtfo bins for a way to exploit this:
https://gtfobins.github.io/gtfobins/vim/
We find the following for sudo:
### Sudo
If the binary is allowed to run as superuser by `sudo`, it does not drop the elevated privileges and may be used to access the file system, escalate or maintain privileged access.

A. `sudo vim -c ':!/bin/sh'`
B. This requires that `vim` is compiled with Python support. Prepend `:py3` for Python 3.
	`sudo vim -c ':py import os; os.execl("/bin/sh", "sh", "-c", "reset; exec sh")'`

C. This requires that `vim` is compiled with Lua support.
`sudo vim -c ':lua os.execute("reset; exec sh")'`

We are going to try with method A

![](imagenes/Pasted%20image%2020240109211405.png)

We run the command, and now we have root privileges, lets check what we can find:

![](imagenes/Pasted%20image%2020240109211436.png)
We go to the root directory and find the flag, we run an ls -al , and we can see we only have read permissions as a user, we change it with chmod 777 and try to discover it with cat:
![](imagenes/Pasted%20image%2020240109211740.png)
Apparently, the flag we found isnt the correct one, we run a `find / -name root.txt` , to see if we can find the correct one:
![](imagenes/Pasted%20image%2020240109212033.png)
And we found the one that's most likely the one we are looking for, which would be the one that's in the apache2 folder, which makes sense, since it's this machines main service(ignore the the one in the home folder, I copied it when I was trying something else), we cat that text file, and we should get the flag string:
![](imagenes/Pasted%20image%2020240109212322.png)
And with this, we are finished with the machine.


## Observations
Didn't realize try at first that I could upload files to the ftp server.

## Thing I learned from this machines
- Always check if I can upload files to if the services allows to.