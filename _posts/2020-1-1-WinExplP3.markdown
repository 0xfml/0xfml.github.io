---
layout: post
title:  "Windows Exploitation Part III"
date:   2020-1-1 00:03:00 +0000
categories: Windows-Exploitation
---
# Windows Exploitation Part III - Structured Exception Handler
'Structured exception handling is a mechanism for handling both hardware and software exceptions.' - MSDN. Basically, its a protection method against standard stack based buffer overflows but that said, its still flawed and we will see why.


{% highlight c++ %}
try {
//run stuff. If exception occurs, go to code.
}
catch {
//run stuff when exception occurs
}
{% endhighlight %}

When we put this in terms of our memory model we should get something like this:
```
|-----------------------| > Top of the stack
|			|
|-----------------------|<==\
|	Local Vars	|   |
|-----------------------|   |
|	Saved EBP	|   | This frame contains the exception handling (try{})
|-----------------------|   |
|	Saved EIP	|   |
|-----------------------|   |
|	Params		|   |
|-----------------------|   |
|	Address of EH	|<<===== This frame contains the exception handler code (catch{})
|-----------------------|<==/
|			|
|			| >> More frames
|			|
|-----------------------| > Bottom of stack
```


# POP POP RET?
When dealing with SEH youre going to see the assembly instruction sequence of 'pop pop ret' in your exploits. In the exploits we overwrite the handler with this instruction sequence.
<br>
**breakdown**
```
pop - take 4 bytes from the stack
pop - take another 4 bytes from the stack
ret - take current value from ESP (nseh value(was ESP+8)) and put into EIP 

|---------------| > top of stack
| ESP		| > gets popped
|---------------|
| ESP+4 	| > gets popped
|---------------|
| ESP+8 (nseh)	| > put into EIP
|---------------| 
```


# Next
Next we will expand on this with whats called egghunting.
<br>
Windows Exploitation:
[I](/windows-exploitation/2019/12/31/WinExplP1.html)	[II](/windows-exploitation/2019/12/31/WinExplP2.html)	[III](/windows-exploitation/2019/12/31/WinExplP3.html)	[IV](/windows-exploitation/2019/12/31/WinExplP4.html)	[V](/windows-exploitation/2019/12/31/WinExplP5.html)

