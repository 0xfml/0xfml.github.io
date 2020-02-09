---
layout: post
title:  "HTB Solidstate Machine Writeup"
date:   2020-1-2 00:10:00 +0000
categories: hackthebox
---
Solid state was the first box i did on hackthebox that had the vector of email harvesting, learnt a lot about what not to keep in emails .

# Nmap scan
```
PORT     STATE SERVICE     VERSION
22/tcp   open  ssh         OpenSSH 7.4p1 Debian 10+deb9u1 (protocol 2.0)
| ssh-hostkey: 
|   2048 77:00:84:f5:78:b9:c7:d3:54:cf:71:2e:0d:52:6d:8b (RSA)
|   256 78:b8:3a:f6:60:19:06:91:f5:53:92:1d:3f:48:ed:53 (ECDSA)
|_  256 e4:45:e9:ed:07:4d:73:69:43:5a:12:70:9d:c4:af:76 (ED25519)
25/tcp   open  smtp        JAMES smtpd 2.3.2
|_smtp-commands: solidstate Hello nmap.scanme.org (10.10.14.6 [10.10.14.6]), 
80/tcp   open  http        Apache httpd 2.4.25 ((Debian))
|_http-server-header: Apache/2.4.25 (Debian)
|_http-title: Home - Solid State Security
110/tcp  open  pop3        JAMES pop3d 2.3.2
119/tcp  open  nntp        JAMES nntpd (posting ok)
4555/tcp open  james-admin JAMES Remote Admin 2.3.2
```

# Hello JAMES
The initial webpage yields nothing of interest at all. Enumerating the other running services gives us something. JAMES 2.3.2, a service critically flawed. We can login with root:root and from there we can enumerate users.
```
--------------------------------------------------------
JAMES Remote Administration Tool 2.3.2
Please enter your login and password
Login id:
root
Password:
root
Welcome root. HELP for a list of commands
listusers
Existing accounts 5
user: james
user: thomas
user: john
user: mindy
user: mailadmin
--------------------------------------------------------------
```
Due to pop3 being open on the box, it looks like we will have to harvest emails. JAMES service actually allows us to reset users passwords with 'setpassword'. From there we can use a mail client like thunderbird to login as them and view their mail.
User 'mindy' has an email in her inbox containing ssh credentials.

# Landing root
The initial ssh shell is running rbash, we can escape this simply by typing bash and / or calling a new shell with python.

Looking around freely now, we find a python file in /tmp which runs as root.
{% highlight python %}
#!/usr/bin/env python
import os
import sys
try:
     os.system('rm -r /tmp/*')
except:
     sys.exit()
{% endhighlight %}

As you can see, its running system in python as root. Perfectly secure. Simply change the 'rm' to a shell and get root when the cron fires off.


