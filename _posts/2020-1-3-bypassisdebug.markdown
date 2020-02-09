---
layout: post
title:  "Bypassing IsDebuggerPresent"
date:   2020-1-3 00:11:00 +0000
categories: Windows
---
<h1>Bypassing IsDebuggerPresent in x86 Windows</h1>

The Kernel 32 module 'IsDebuggerPresent' is a common method of detecting usermode debuggers in windows applications. It is also quite common in basic reversing CTF challenges. <br />
For this method, all we need is a standard debugger like Olly or Immunity. For this rundown i will be using references from Immunity as its usually my go to and there may be other ways of bypassing this call but this method seems to work for me. <br />
<br>

Steps to follow:
```
- Load / attach to the application
- Right click within the CPU section
- Select 'Search for'
- Select 'Name in all modules' or 'All intermodular calls'
- Find the kernel32 call for IsDebuggerPresent
- Double click the call to follow in the assembly.
- Add a breakpoint to the kernel call
- Run / restart the application and you should hit the breakpoint
- Once the Process hits, you should see the following assembly;
	- MOV EAX, DWORD PTR FS:[18]
	- MOV EAX, DWORD PTR DS:[EAX+30]
	- MOVZX EAX, BYTE PTR DS:[EAX+2]
	- RETN
- Add a breakpoint, or stop when the process gets to RETN
- Check the current register values.
- EAX should contain 1 at this point (00000001)
- Simple right click on the value and edit EAX to equal 0
- Continue execution whilst being attached to the debugger.
```

