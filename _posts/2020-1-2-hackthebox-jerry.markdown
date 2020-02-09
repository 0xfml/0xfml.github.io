---
layout: post
title:  "HTB Jerry Machine Writeup"
date:   2020-1-2 00:10:00 +0000
categories: hackthebox
---
A very simple box with a big hint.

# Nmap scan
```
8080/tcp open  http    Apache Tomcat/Coyote JSP engine 1.1
|_http-favicon: Apache Tomcat
|_http-server-header: Apache-Coyote/1.1
|_http-title: Apache Tomcat/7.0.88
```

# Exploitation
As the scan shows we have a tomcat box. Tomcat by design is an easy shell once you can login. Luckily someone made some scripts that allow that process to get easier, running a default tomcat login wordlist against the /manager login form gives us access under 'tomcat:secr3t'. We can then upload a .WAR file with metasploits /tomcat_mgr_upload for an easy meterpreter session.
There was no privesc for this box, both root and user flags were available to read at this point. Would have liked to see some method of elevation :/




