---
layout: post
title:  "HTB Valentine Machine Writeup"
date:   2020-1-2 00:10:00 +0000
categories: hackthebox
---
This was actually my first ever rooted box on hackthebox!

# Nmap scan
```
PORT    STATE SERVICE  VERSION
22/tcp  open  ssh      OpenSSH 5.9p1 Debian 5ubuntu1.10 (Ubuntu Linux; protocol 2.0)
| ssh-hostkey: 
|   1024 96:4c:51:42:3c:ba:22:49:20:4d:3e:ec:90:cc:fd:0e (DSA)
|   2048 46:bf:1f:cc:92:4f:1d:a0:42:b3:d2:16:a8:58:31:33 (RSA)
|_  256 e6:2b:25:19:cb:7e:54:cb:0a:b9:ac:16:98:c6:7d:a9 (ECDSA)
80/tcp  open  http     Apache httpd 2.2.22 ((Ubuntu))
|_http-server-header: Apache/2.2.22 (Ubuntu)
|_http-title: Site doesn't have a title (text/html).
443/tcp open  ssl/http Apache httpd 2.2.22 ((Ubuntu))
|_http-title: Site doesn't have a title (text/html).
| ssl-cert: Subject: commonName=valentine.htb/organizationName=valentine.htb/stateOrProvinceName=FL/countryName=US
| Not valid before: 2018-02-06T00:45:25
|_Not valid after:  2019-02-06T00:45:25
|_ssl-date: 2018-12-04T12:02:14+00:00; -1m32s from scanner time.
No exact OS matches for host
```
 
# pwning user.
The index page for the website shows an image of a bleeding heart. Hinting at heartbleed and hence the ssl site. Poking around the site we find a encode.php and decode.php which seems to just do hex en/decoding.
Directory bruteforcing also reveals a /dev/ directory which contains a private key when decoded.

Running a heartbleed script from github will leak a base64 string eventually (may take some attempts). I also tried using a metasploit module but it kept throwing a random md5 string which led me down a rabbithole.
```
aGVhcnRibGVlZGJlbGlldmV0aGVoeXBlCg==
=> heartbleedbelievethehype
```

Once we chmod 600 the private key we can try login to the box as user 'hype' and once we supply the key and password we get a ssh session

# Getting root
Checking the system version was the first thing i did on the box.
```
Linux Valentine 3.2.0-23-generic #36-Ubuntu SMP Tue Apr 10 20:39:51 UTC 2012 x86_64 x86_64 x86_64 GNU/Linux
```
This kernel looks old. It also fits the timeframe for dirtycow to work. Downloading the exploit from github editing it to change the username, uploading it to the box with wget and simplehttpserver and running it eventually will do what it has to.
Once it has finished, log back into ssh as hype and the password supplied in the exploit. Should be root.
```
hype@Valentine:~# id -a
uid=0(hype) gid=0(root) groups=0(root),24(cdrom),30(dip),46(plugdev),124(sambashare)
```
Can now grab that root flag.
