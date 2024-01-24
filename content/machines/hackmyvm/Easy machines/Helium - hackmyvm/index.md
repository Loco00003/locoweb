+++
title = "Helium - Hackmyvm"
date = 2024-01-23T17:56:40-06:00
summary="Another machine by sml, but this time with a twist, this one also took me a while..."
tags=["pentesting","hackmyvm","steganography"]
series=["Hackmyvm Easy"]
series_order=3
+++

## Machine Info
| Name       | helium       |
| ---------- | ------ |
| OS         | Linux Debian |
| Author     | sml    | 
| Site       | [hackmym](https://hackmyvm.eu)       |
| Difficulty | Easy       |

## Enumeration
I run my usual nmap scan  `sudo nmap -oN helium_scan.txt -vv -sS -sC -sV -T4 -p- 192.168.25.19`

![](imagenes/Pasted%20image%2020240123181823.png)

Both port 22 and 80 open, lets check the webpage:

![](imagenes/Pasted%20image%2020240123181931.png)

Some content in a webpage for a change, we check the source code:

![](imagenes/Pasted%20image%2020240123182004.png)

Nice, we have a directory name and a possible username *paul*, lets check the directory:

![](imagenes/Pasted%20image%2020240123182045.png)

Lets enumerate this directory with gobuster (lets include the `.wav` extension, to see if the audio file is really there). Hmm, nothing apparently, but, the clue is, can we really upload something? Using what we learned the the *Serve machine from vulnyx*, we can try to upload a file with `curl`.

![](imagenes/Pasted%20image%2020240123182925.png)

Nothing, lets backtrack a bit, *paul* has been scolded for uploading *weird files* to the website, so, lets try to download the `.wav` file and run a few tests on it [I took some references from this blog](https://medium.com/analytics-vidhya/get-secret-message-from-audio-file-8769421205c3):

Nothing out place yet... Lets use [binwalk](https://github.com/ReFirmLabs/binwalk)

![](imagenes/Pasted%20image%2020240123185110.png)

![](imagenes/Pasted%20image%2020240123185741.png)

Very interesting, lets use the help pages , and see how we can extract this info:

![](imagenes/Pasted%20image%2020240123185851.png)

We are going to use this flag, lets try the command again, and see if it extracted anything in our directory:

![](imagenes/Pasted%20image%2020240123190211.png)

Nothing, apparently, *index* files are useless, so we move on with the analysis of the file, in the blog I referenced above, the author analyzes the file with a python script using the `wave` library, lets try doing that:

![](imagenes/Pasted%20image%2020240123191633.png)

After trying with that, I did some more research, and found this [tool](https://github.com/ragibson/Steganography#WavSteg):

![](imagenes/Pasted%20image%2020240123192200.png)

I download it and try it:

![](imagenes/Pasted%20image%2020240123195902.png)

Nothing... I try with this [tool](https://github.com/danielcardeenas/AudioStego) too:

![](imagenes/Pasted%20image%2020240123195951.png)

Ok, nothing, we know for sure the key to solving this machine is in a **audio file**, maybe, we got the ***wrong one***? Lets explore the webpage one more time and see if we missed something:

![](imagenes/Pasted%20image%2020240123200102.png)

Indeed I did... I try to download the file with `wget`:

![](imagenes/Pasted%20image%2020240123200204.png)

We got it. I hear it, it sounds like some kind of morse code,lets see if we can find an online tool for it. Finally, I find this [webpage](https://morsecode.world/international/decoder/audio-decoder-expert.html) on it, I upload the file, and select the option that matches the audio I have:

![](imagenes/Pasted%20image%2020240123201347.png)

I play the audio, and check the results:

![](imagenes/Pasted%20image%2020240123201437.png)

And in the *spectogram* section, I find, most likely, the password for the user *paul*, lets try to login with SSH...

## Initial access and privilege escalation

![](imagenes/Pasted%20image%2020240123201606.png)

And, we are in! Lets get that user flag, and find out with `sudo -l` what commands we can run as a super user:

![](imagenes/Pasted%20image%2020240123202348.png)

Lets search that on [gtfobins]():

![](imagenes/Pasted%20image%2020240123202539.png)

Lets try that:

![](imagenes/Pasted%20image%2020240123202630.png)

And we got root, lets get the flag, and finish this machine:

![](imagenes/Pasted%20image%2020240123202738.png)

And with that, we've pwned a machine, took me a while, but pretty interesting overall.

## What I learned from this machine
- If a file contains hidden data, or anything for that matter, try the **easiest method possible** first. Steganography tools, as you can see, can be pretty expansive, in this case, maybe you caught in earlier than me, that I could've just simply used an spectogram, and that was the case. I got too concentrated in finding the right tool, that I didn't consider using an spectogram, good thing to keep in mind in the future if I encounter another audio file in a CTF.
- **ALWAYS**, check **ALL** the webpage contents **MANUALLY**, before moving on to using automated tools, I lost so much time enumerating with gobuster, instead, I could've searched the webpage manually which would've led me to finding that clue earlier.





