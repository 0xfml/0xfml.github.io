---
layout: post
title:  "HTB Optimum Machine Writeup"
date:   2020-1-2 00:10:00 +0000
categories: hackthebox
---
# Description
Optimum was a fairly basic machine that could be easily rooted with the use of public exploits and not much else. 
# Nmap scan
```
PORT STATE SERVICE VERSION
80/tcp open http HttpFileServer http 2.3
|_http-server-header: HFS 2.3
|_http-title: HFS /
```

# Initial shell
Simply searchsploit 'HFS 2.3' or search within the metasploit framework to find the vulnerability for 'Rejetto_hfs_exec'. Set the parameters that would fit the host and execute for a user shell.
# Privilege escalation
Since we have a meterpreter shell, we can leverage the classic 'windows-exploit-suggester' module. We run it and it returns some results. 'Microsoft Windows 8.1 (x64) 'RGNOBJ' Integer Overflow (MS16-098)' is one of them. 
We can download an executable from the offensive security github page and simply transfer and execute the binary on the host to gain SYSTEM permissions.


