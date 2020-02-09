---
layout: post
title:  "Windows Exploitation Part I"
date:   2020-1-1 00:01:00 +0000
categories: Windows-Exploitation
---
# Windows Exploitation Part I - Getting Started with basics

Like anything else, before we start trying to exploit windows, we should do some research into how it works. In this part ill run through what i used to get started with windows based applications.
<br />
Due to how these types of exploits work, we should start by getting familiar with how x86 windows memory works so we can better understand what happens in memory when we launch our exploit. The following is the windows 32 memory mapping and we should get familiar with this as it will help visualise more advanced exploits in future.
<br />
Win32 Memory Map.
```
|-----------------------| > 0x00000000
|			|
|-----------------------| >> When something is pushed to the stack, 
|STACK		^^	| >> .. the stack pointer is decremented.
|			| >> Stack grows upwards to lower addresses
|-----------------------| >> Stack upper limit
|HEAP		vv	| >> Heap grows down to higher addresses
|-----------------------| > 0x0040000
|PROGRAM IMAGE		|
|- PE Header		|
|- .text, rdata		|
|- .data, .rsrc		|
|-----------------------| >> Memory space can be allocated as heap or stack
|			| >> ... for other threads in the process
|-----------------------| 
|DLL			| >> DLLs have a header, .text, .data,
|-----------------------| >> ... .rsrc, .reloc segmentes
|			|
|-----------------------| > 0x7FFDF000
|PEB			|
|-----------------------| > 0x7FFE0000
|SHARED USER PAGE	|
|-----------------------| > 0x7FFE1000
|NO ACCESS / KERNEL LAND|
|-----------------------| > 0x7FFFFFFF
```

# The Stack
The stack is a segment of memory, a data structure that works as Last In First Out (LIFO). The stack gets allocated by the operating system, for each thread. When the thread ends, the stack is cleared.
The size of the stack is defined when it gets created and does not change. It is fast but limited by size. LIFO means the most recent data (result of a PUSH) is the first to be removed again (result of a POP).
When a stack is created, the stack pointer points to the top of the stack (highest point). As information is pushed onto the stack the stack pointer decrements. Thus the stack moving to lower addresses.
The stack contains local variables, function calls and other information that doesnt need to be stored for large amounts of time. Every time a function is called, the function parameters are pushed onto the stack,
as well as the saved values of registers (EBP, EIP). When a function returns, the saved address of EIP is retrieved from the stacck and placed back into EIP so the normal application flow can resume.

# 32 bit Registers
Registers 
```
<--------------------------32 bits>
		<----------16 bits>
	           <8 bits><8 bits>
|---------------------------------|
|EAX		AX |     AH|    AL| < 
|EBX		BX |     BH|    BL| < 
|ECX		CX |     CH|    CL| < General Purpose
|EDX		DX |	 DH|    DL| < 	Registers
|ESI			          | < 
|EDI			          | <
|ESP			          |
|EBP			          |
|---------------------------------|

Data Registers:
EAX = Extended Accumulator Register
EBX = Extended Base Register
ECX = Extended Count Register
EDX = Extended Data Register

Index Registers:
ESI = Extended Source Index
EDI = Extended Destination Index

Pointer Registers:
ESP = Extended Stack Pointer
EBP = Extended Base Pointer
EIP = Extended Instruction Pointer

```
# Hexadecimal
Hexadecimal is something you should know about, almost everything in memory is in hex and it will constantly show up in things related to anything lower level. If youre not familiar, hexadecimal is a base16 number system where 0=0 1=1 ... 9=9 A=10 B=11 C=12 D=13 E=14 F=15 10=16 11=17 and so on.

# Next
For now im going to put off writing about how to get started with C and Assembly, there are loads of articles out there already but if i get around to it i will post something. It is however, something to be somewhat versed with along with a decent scripting language.<br />
The next part will be looking at writing a stock standard windows stack based buffer overflow.
<br />
Windows Exploitation:
[I](/windows-exploitation/2019/12/31/WinExplP1.html)	[II](/windows-exploitation/2019/12/31/WinExplP2.html)	[III](/windows-exploitation/2019/12/31/WinExplP3.html)	[IV](/windows-exploitation/2019/12/31/WinExplP4.html)	[V](/windows-exploitation/2019/12/31/WinExplP5.html)





