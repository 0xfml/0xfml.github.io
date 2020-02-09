---
layout: post
title:  "HTB Bashed Machine Writeup"
date:   2020-1-2 00:10:00 +0000
categories: hackthebox
---
Bashed was a very easy box, perfect for anyone getting started with htb.

# Nmap scan
```
PORT   STATE SERVICE VERSION
80/tcp open  http    Apache httpd 2.4.18 ((Ubuntu))
|_http-server-header: Apache/2.4.18 (Ubuntu)
|_http-title: Arrexel's Development Site
No exact OS matches for host
```
# User
directory bruteforcing reveals **/dev/phpbash.php** which is by name the program listed on the index. This can also be found by tracing the devs github page.

So were given a bash interface but written in php. We interact with it just the same as a normal bash shell and can already get our user flag.

# Privilege escalation
I chose to upload a shell to the server using a http server and wget on the box, navigating to a php shell gives us our connection. A python pty shell can be spawed for a nicer environment. 
We notice a **scriptmanager** user and **/scripts** directory on the machine. With some su-foo we can move into be scriptmanager. 
```
sudo -u scriptmanager (sudo -u scriptmanager /bin/sh -i)
```
Now we can access the /scripts directory we see a python script we can edit. Changing the script to contain a basic python reverse shell and opening a listener on our host will result in a shell.

