+++
title = "Gigachad"
date = 2024-02-24T21:56:29-06:00
summary="Easy machine, a bit of OSINT, and exploit research."
draft=false
tags=["",""]
series=["Hackmyvm Easy"]
series_order=7
+++
## Foreword
As I said in my previous machine post, been a little bit busy with university, but, I'm still learning about ethical hacking in my free time. Anyways, here's this easy machine by tasiyanci.


## Machine info
| Name       | Gigachad                                                         |
| ---------- | ---------------------------------------------------------------- |
| OS         | Linux Debian                                                     |
| Author     | tasiyanci                                                        |
| Site       | [hackmyvm](https://hackmyvm.eu/machines/machine.php?vm=Gigachad) |
| Difficulty | Easy                                                             |

## Recon

We verify our connection with the machine:


![](imagenes/Pasted%20image%2020240224193655.png)

![](imagenes/Pasted%20image%2020240224193743.png)

We run our nmap scan:

```shell-session
Nmap 7.94SVN scan initiated Sat Feb 24 19:35:25 2024 as: nmap -oN gigachad_scan.txt -Pn -vv -sS -sC -sV -T3 -p- 192.168.25.29
Nmap scan report for 192.168.25.29
Host is up, received arp-response (0.00038s latency).
Scanned at 2024-02-24 19:35:25 CST for 11s
Not shown: 65532 closed tcp ports (reset)
PORT   STATE SERVICE REASON         VERSION
21/tcp open  ftp     syn-ack ttl 64 vsftpd 3.0.3
| ftp-anon: Anonymous FTP login allowed (FTP code 230)
|_-r-xr-xr-x    1 1000     1000          297 Feb 07  2021 chadinfo
| ftp-syst: 
|   STAT: 
| FTP server status:
|      Connected to ::ffff:192.168.25.5
|      Logged in as ftp
|      TYPE: ASCII
|      No session bandwidth limit
|      Session timeout in seconds is 300
|      Control connection is plain text
|      Data connections will be plain text
|      At session startup, client count was 2
|      vsFTPd 3.0.3 - secure, fast, stable
|_End of status
22/tcp open  ssh     syn-ack ttl 64 OpenSSH 7.9p1 Debian 10+deb10u2 (protocol 2.0)
| ssh-hostkey: 
|   2048 6a:fe:d6:17:23:cb:90:79:2b:b1:2d:37:53:97:46:58 (RSA)
| ssh-rsa AAAAB3NzaC1yc2EAAAADAQABAAABAQC4uqqKMblsYkzCZ7j1Mn8OX4iKqTf55w3nolFxM6IDIrQ7SV4JthEGqnYsiWFGY0OpwHLJ80/pnc/Ehlnub7RCGyL5gxGkGhZPKYag6RDv0cJNgIHf5oTkJOaFhRhZPDXztGlfafcVVw0Agxg3xweEVfU0GP24cb7jXq8Obu0j4bNsx7L0xbDCB1zxYwiqBRbkvRWpiQXNns/4HKlFzO19D8bCY/GXeX4IekE98kZgcG20x/zoBjMPXWXHUcYKoIVXQCDmBGAnlIdaC7IBJMNc1YbXVv7vhMRtaf/ffTtNDX0sYydBbqbubdZJsjWL0oHHK3Uwf+HlEhkO1jBZw3Aj
|   256 5b:c4:68:d1:89:59:d7:48:b0:96:f3:11:87:1c:08:ac (ECDSA)
| ecdsa-sha2-nistp256 AAAAE2VjZHNhLXNoYTItbmlzdHAyNTYAAAAIbmlzdHAyNTYAAABBBDkds8dHvtrZmMxX2P71ej+q+QDe/MG8OGk7uYjWBT5K/TZR/QUkD9FboGbq1+SpCox5qqIVo8UQ+xvcEDDVKaU=
|   256 61:39:66:88:1d:8f:f1:d0:40:61:1e:99:c5:1a:1f:f4 (ED25519)
|_ssh-ed25519 AAAAC3NzaC1lZDI1NTE5AAAAIIoK0bHJ3ceMQ1mfATBnU9sChixXFA613cXEXeAyl2Y2
80/tcp open  http    syn-ack ttl 64 Apache httpd 2.4.38 ((Debian))
|_http-title: Site doesn't have a title (text/html).
|_http-server-header: Apache/2.4.38 (Debian)
| http-robots.txt: 1 disallowed entry 
|_/kingchad.html
| http-methods: 
|_  Supported Methods: POST OPTIONS HEAD GET
MAC Address: 08:00:27:AC:C9:45 (Oracle VirtualBox virtual NIC)
Service Info: OSs: Unix, Linux; CPE: cpe:/o:linux:linux_kernel

```

| Ports open | 21,22,80     |
| ---------- | ------------ |
| Services   | FTP,SSH,HTTP |
Great, it seems we can connect with FTP, with an anonymous login, lets check:

![](imagenes/Pasted%20image%2020240224194509.png)

We can, lets check what info we can get:

![](imagenes/Pasted%20image%2020240224194634.png)

Lets check the webpage directory:

![](imagenes/Pasted%20image%2020240224194704.png)

An image, we download and save it. But, beforehand, I check the source code of the main page, and find this:

![](imagenes/Pasted%20image%2020240224194752.png)

Appears to be some kind of hash, an `md5` has to be more precise, lets check if that's the case in [hashes.com](https://hashes.com) :

![](imagenes/Pasted%20image%2020240224194856.png)

Hehe, it seems it is, now, maybe the image has some hashes in it? Lets run `exiftool` to find out:

![](imagenes/Pasted%20image%2020240224200120.png)

>I thought `Document ancestors` and `Derived from Original Document ID` were hashes, but they weren't. Now, before overcomplicating things further more, maybe the password is the place in the photo. It seems to me the place is `Istanbul`, and , quickly with google maps, I find the name of the place depicted in the photo:

![](imagenes/Pasted%20image%2020240224201240.png)

![](imagenes/Pasted%20image%2020240224201504.png)

![](imagenes/Pasted%20image%2020240224201649.png)

Now, some possible passwords would be:
- `maidenstower`
- `kizkulesi`
- `istanbul`
- `leanderstower`

## Initial access and privilege escalation.
We can either create a list so we can try to brute-force with `hydra` or just try them manually. After trying manually, I find that the correct password is `maidenstower`:

![](imagenes/Pasted%20image%2020240224202200.png)

I find the flag, and move on find out how to escalate privileges:

![](imagenes/Pasted%20image%2020240224202236.png)

First, lets check the webpage directory, and check if we can find something interesting:

![](imagenes/Pasted%20image%2020240224203540.png)

A `gobuster` scan would've been pointless in this machine... Anyways, lets check the `robots.txt` file:

![](imagenes/Pasted%20image%2020240224203655.png)

Lets check that page:

![](imagenes/Pasted%20image%2020240224203728.png)

Nothing useful. Lets run the `find / -perm -4000 2>/dev/null` command, to see if we see anything out of the ordinary we can exploit:

![](imagenes/Pasted%20image%2020240224212720.png)

This this file is out of the ordinary. I took a look in [gtfobins](https://gtfobins.github.io), but found nothing. But, in [exploit-db](https://www.exploit-db.com/exploits/47172) , I found something interesting, a bash script(which runs a some `c` code inside it) for local privilege escalation:

Here's how it work in a nutshell:

![](imagenes/Pasted%20image%2020240224213022.png)

And here's a successful run of the exploit:

![](imagenes/Pasted%20image%2020240224213109.png)

Since we are already in the `/tmp` directory, lets create a `.sh` file, give it execution binaries, and see if we get root.

![](imagenes/Pasted%20image%2020240224214939.png)

And we get root! Lets find the flag and finish the machine:

![](imagenes/Pasted%20image%2020240224215524.png)

And with that, we are done with this machine, thanks for reading!

