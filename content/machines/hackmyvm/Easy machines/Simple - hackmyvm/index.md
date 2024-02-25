+++
title = "Simple"
date = 2024-02-17T18:17:56-06:00
summary="First windows machine"
draft=false
tags=["",""]
series=["Hackmyvm Easy"]
series_order=6
+++

## Foreword
Recently completed the [windows privileges escalation room](https://tryhackme.com/room/windowsprivesc20) in tryhackme.So, I decided to try and solve a windows machine for a change, lets see how it goes.

## Machine info
| Name       | Simple                                                         |
| ---------- | ---------------------------------------------------------------- |
|OS|Windows Server 2019|
| Author     | GatoGamer                                                          |
| Site       | [hackmyvm](https://hackmyvm.eu/machines/machine.php?vm=Simple) | 
| Difficulty | Easy                                                                 |

## Recon and enumeration

We make sure we have communication with the machine:

![](imagenes/Pasted%20image%2020240217182231.png)

Lets run our `nmap` scan now:

```shell-session
nmap -oN simple_scan.txt -vv -sS -sC -sV -T4 -p- 192.168.25.27
Nmap scan report for 192.168.25.27
Host is up, received arp-response (0.00030s latency).
Scanned at 2024-02-17 18:23:05 CST for 101s
Not shown: 65523 closed tcp ports (reset)
PORT      STATE SERVICE       REASON          VERSION
80/tcp    open  http          syn-ack ttl 128 Microsoft IIS httpd 10.0
|_http-server-header: Microsoft-IIS/10.0
|_http-title: Simple
| http-methods: 
|   Supported Methods: OPTIONS TRACE GET HEAD POST
|_  Potentially risky methods: TRACE
135/tcp   open  msrpc         syn-ack ttl 128 Microsoft Windows RPC
139/tcp   open  netbios-ssn   syn-ack ttl 128 Microsoft Windows netbios-ssn
445/tcp   open  microsoft-ds? syn-ack ttl 128
5985/tcp  open  http          syn-ack ttl 128 Microsoft HTTPAPI httpd 2.0 (SSDP/UPnP)
|_http-title: Not Found
|_http-server-header: Microsoft-HTTPAPI/2.0
47001/tcp open  http          syn-ack ttl 128 Microsoft HTTPAPI httpd 2.0 (SSDP/UPnP)
|_http-server-header: Microsoft-HTTPAPI/2.0
|_http-title: Not Found
49664/tcp open  msrpc         syn-ack ttl 128 Microsoft Windows RPC
49665/tcp open  msrpc         syn-ack ttl 128 Microsoft Windows RPC
49666/tcp open  msrpc         syn-ack ttl 128 Microsoft Windows RPC
49667/tcp open  msrpc         syn-ack ttl 128 Microsoft Windows RPC
49668/tcp open  msrpc         syn-ack ttl 128 Microsoft Windows RPC
49669/tcp open  msrpc         syn-ack ttl 128 Microsoft Windows RPC
MAC Address: 08:00:27:57:28:8B (Oracle VirtualBox virtual NIC)
Service Info: OS: Windows; CPE: cpe:/o:microsoft:windows

Host script results:
| nbstat: NetBIOS name: SIMPLE, NetBIOS user: <unknown>, NetBIOS MAC: 08:00:27:57:28:8b (Oracle VirtualBox virtual NIC)
| Names:
|   SIMPLE<00>           Flags: <unique><active>
|   WORKGROUP<00>        Flags: <group><active>
|   SIMPLE<20>           Flags: <unique><active>
| Statistics:
|   08:00:27:57:28:8b:00:00:00:00:00:00:00:00:00:00:00
|   00:00:00:00:00:00:00:00:00:00:00:00:00:00:00:00:00
|_  00:00:00:00:00:00:00:00:00:00:00:00:00:00
| p2p-conficker: 
|   Checking for Conficker.C or higher...
|   Check 1 (port 4952/tcp): CLEAN (Couldn't connect)
|   Check 2 (port 17879/tcp): CLEAN (Couldn't connect)
|   Check 3 (port 64050/udp): CLEAN (Timeout)
|   Check 4 (port 12789/udp): CLEAN (Failed to receive data)
|_  0/4 checks are positive: Host is CLEAN or ports are blocked
|_clock-skew: 0s
| smb2-time: 
|   date: 2024-02-18T00:24:41
|_  start_date: N/A
| smb2-security-mode: 
|   3:1:1: 
|_    Message signing enabled but not required

```

Interesting results, we have a posible workstation name `SIMPLE`, that may be of use later, lets check the webpage:

![](imagenes/Pasted%20image%2020240217183044.png)

We have a couple of potential usernames, lets keep them in mind, I also checked the source code of the page, but didn't find anything interesting, lets run a `gobuster` scan in this webpage:

![](imagenes/Pasted%20image%2020240217183940.png)

In the process of checking  those directories manually, all of them were forbidden. Instead of enumerating each one of them, I move on.
Now, we have this information:
- The following usernames:
	1. ruy
	2. marcos
	3. lander
	4. bogo
	5. vaiper
- The name of the machine : `SIMPLE`

I research how to enumerate certain windows services, and I find the following tools: `crackmapexec`,`smbmap`,`smbclient`. Lets try each one of them and check if we can get any useful info:

First we try with an anonymous login with `smbclient`:

![](imagenes/Pasted%20image%2020240217210521.png)

Nothing, lets try `smbmap`:

![](imagenes/Pasted%20image%2020240217210653.png)

We try again an anonymous login, and we got nothing again. Now, one of the reasons we may prefer `crackmapexec` for this task, is the fact that it lets us use *files* for users and passwords, so, lets do that:

![](imagenes/Pasted%20image%2020240217210931.png)

Interesting, lets see what we can do with that. After a while, it seems, when this machine was created, we could directly access the information with password it had, but due to a windows configuration, the password has expired.So, in order to continue, we have to change the password, I already did it, but didn't take any screenshots, so here is how to do it step-by-step:
1. Go to the windows virtual machine.
2. In the command prompt, you'll see `press ctrl+alt+del o presione ctrl+alt+supr` to continue, depending on your virtual machine manager, you may have to enable keyboard inputs so the machine can pass it through, in virtual box, I had to use this option:
		![](imagenes/Pasted%20image%2020240217211721.png)
		
3. Then, you'll be prompted to enter the admin password, we don't have that. Instead hit `esc` and it'll bring to a user list, in there, select `bogo`, enter password `bogo`, and change the password however you like.
## Initial Access
Now, with our now password, lets try again `crackmapexec`:

![](imagenes/Pasted%20image%2020240217211942.png)

Good, we have read permissions in both `IPC$` and `LOGS` shares. Lets try to log in with `smbclient` into the `LOGS` share:

![](imagenes/Pasted%20image%2020240217212247.png)

We log in, and find a `log file`, lets download it, and check its contents:

![](imagenes/Pasted%20image%2020240217212549.png)

In there, we find some credentials that may allows to do a remote login, lets try that:

![](imagenes/Pasted%20image%2020240217212937.png)

We find the `marcos's` password is expired too, we have to change it, we follow the steps above:

![](imagenes/Pasted%20image%2020240217213601.png)

![](imagenes/Pasted%20image%2020240217213624.png)

![](imagenes/Pasted%20image%2020240217213646.png)

We change the password, and we are logged in as the user marcos, lets check what we can find:

![](imagenes/Pasted%20image%2020240217214719.png)

We find the user flag, now, either we can find a way to get a reverse shell, or continue using the windows machine. Personally I think using the window machine wasn't the way the creator intended to solve the machine. so, I look up how to get a reverseshell in windows. Now, lets remember, we just reset the password for `marcos`, and by the looks of it, he has access to the contents of the web page. we can try to login as `marcos` with `smbclient`, but first, lets take a look at which shares `marcos` has access to:

![](imagenes/Pasted%20image%2020240217220528.png)

This is most likely our ticket to getting `admin` privileges, lets login as `marcos` with `smbclient`:

![](imagenes/Pasted%20image%2020240217220736.png)

Lets try uploading a file with `put`:

![](imagenes/Pasted%20image%2020240217220824.png)

![](imagenes/Pasted%20image%2020240217220854.png)

Alright, this means, we can definitively upload an script that'll allow us to get a reverse shell into our machine, lets look up how to do that.We notice the `asp_client`, after some research, I find [this](https://www.lifewire.com/what-is-an-aspx-file-2619705):

![](imagenes/Pasted%20image%2020240217221818.png)

And in the [hacktricks page](https://book.hacktricks.xyz/generic-methodologies-and-resources/shells/msfvenom#asp-x) luckily for us, there's a cheatsheet where we can find the correct command with `msfvenom`, which will create the payload we will upload to the webpage:

![](imagenes/Pasted%20image%2020240217222025.png)

Lets generate the payload, upload it, set up our listener in netcat, and trigger the payload to see if we get a reverse shell:

![](imagenes/Pasted%20image%2020240217222257.png)

![](imagenes/Pasted%20image%2020240217222306.png)

![](imagenes/Pasted%20image%2020240217222320.png)

*At this point , I realize, with this payload, I need to use meterpreter, since I want to do things manually, I search for different .aspx reverse shell*.

I find [this](https://raw.githubusercontent.com/borjmz/aspx-reverse-shell/master/shell.aspx) script in github, I download it and change its parameters and upload it, and try again:

![](imagenes/Pasted%20image%2020240217223723.png)

![](imagenes/Pasted%20image%2020240217223741.png)

![](imagenes/Pasted%20image%2020240217223752.png)

## Privilege Escalation

![](imagenes/Pasted%20image%2020240217223817.png)

![](imagenes/Pasted%20image%2020240217223826.png)

There we go, now, let see how we can escalate privileges, I run the `whoami /priv` command, to see what privileges I possess with this user:

![](imagenes/Pasted%20image%2020240217224109.png)

Checking my notes, I find the way to escalate privileges using `RogueWinRM`, in the [github repository](https://github.com/antonioCoco/RogueWinRM) explains very well the steps to use this. We are going to use this to get a `admin` cmd shell, first, we upload the `RogueWinRM.exe` to the `C:\ProgramData` hidden folder (*took me a while to find a folder I could download to...*). And then we do the following:

![](imagenes/Pasted%20image%2020240217232535.png)

It seems this one doesn't work. In the hacktricks page, there where another exploits we could use, lets check them. 

After a few hours trying to get elevated privileges, finally, I got elevated privileges and the flag:

![](imagenes/Pasted%20image%2020240218022802.png)

For this, we used the `GodPotato` [exploit](https://book.hacktricks.xyz/windows-hardening/windows-local-privilege-escalation/roguepotato-and-printspoofer#godpotato) paired with `netcat` so we could get a higher privilege session:
![](imagenes/Pasted%20image%2020240218022314.png)

So, you might ask, what took me so long? Well, thing is, the `nc64.exe` that I downloaded earlier wasn't working at all and well, I didn't really think there would other repositories where you could download netcat.I downloaded the first one [here](https://github.com/int0x33/nc.exe/) , which, was updated **7 years ago**. So because of that, I was getting this an error about netcat not being compatible with the current version of windows of the machine. so, in the end I looked up for other version, and indeed found [one](https://github.com/vinsworldcom/NetCat64/releases) which was more recent, well the release date at least. I retried the process, and finally managed to escalate privileges.

Thanks for your time and good luck!

## What I learned from this machine
- The use of `smbclient`,`crackmapexec` and `smbmap` for share enumeration when encountering port `139` and `445`, in those shares, interesting information can be found.
- Enumerate with different tools, the same service, there might be differences in the information found by each tool.
- Privilege escalation with the `SeImpersonatePrivilege` privilege with the `GodPotato` exploit.
- Got stuck up because `netcat` didn't work, but the exploit was working as intended. Should I've had noticed earlier, I would've solved the machine a lot quickly. Have to keep in mind if a program doesn't work for the target machine, look for another version of the program.






















