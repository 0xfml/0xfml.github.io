---
layout: post
title:  "HTB Legacy Machine Writeup"
date:   2020-1-2 00:10:00 +0000
categories: hackthebox
---

# Nmap scan
```
PORT     STATE  SERVICE       VERSION
139/tcp  open   netbios-ssn   Microsoft Windows netbios-ssn
445/tcp  open   microsoft-ds  Windows XP microsoft-ds
3389/tcp closed ms-wbt-server
Device type: general purpose|specialized
Running (JUST GUESSING): Microsoft Windows XP|2003|2000|2008 (92%), General Dynamics embedded (89%)
OS CPE: cpe:/o:microsoft:windows_xp cpe:/o:microsoft:windows_server_2003 cpe:/o:microsoft:windows_2000::sp4 cpe:/o:microsoft:windows_server_2008::sp2
Aggressive OS guesses: Microsoft Windows XP SP2 or Windows Small Business Server 2003 (92%), Microsoft Windows 2000 SP4 or Windows XP SP2 or SP3 (92%), Microsoft Windows XP SP2 (92%), Microsoft Windows Server 2003 (90%), Microsoft Windows XP SP3 (90%), Microsoft Windows 2000 SP4 (90%), Microsoft Windows XP Professional SP3 (90%), Microsoft Windows XP SP2 or SP3 (90%), Microsoft Windows XP Professional SP2 (90%), Microsoft Windows 2000 Server (89%)
No exact OS matches for host (test conditions non-ideal).
Network Distance: 2 hops
Service Info: OSs: Windows, Windows XP; CPE: cpe:/o:microsoft:windows, cpe:/o:microsoft:windows_xp

Host script results:
|_clock-skew: mean: -4h01m52s, deviation: 1h24m49s, median: -5h01m51s
|_nbstat: NetBIOS name: LEGACY, NetBIOS user: <unknown>, NetBIOS MAC: 00:50:56:bd:de:4a (VMware)
| smb-os-discovery: 
|   OS: Windows XP (Windows 2000 LAN Manager)
|   OS CPE: cpe:/o:microsoft:windows_xp::-
|   Computer name: legacy
|   NetBIOS computer name: LEGACY\x00
|   Workgroup: HTB\x00
|_  System time: 2018-12-17T08:28:58+02:00
| smb-security-mode: 
|   account_used: guest
|   authentication_level: user
|   challenge_response: supported
|_  message_signing: disabled (dangerous, but default)
|_smb2-time: Protocol negotiation failed (SMB2)
```

# pwning :zzz:
Host is running what looks like xp or 2000, running with netapi enabled luckily theres a metasploit module for that
'smb/ms08_067_netapi' lands us as root ezpz
