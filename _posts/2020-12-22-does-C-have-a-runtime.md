---
layout: post
title:  "Does C have a runtime ?"
date:   2020-12-22 14:29:37 -0700
categories: c
---




![bubble](/public/images/speech_bubble1.png "Image source : https://pixabay.com/illustrations/speech-bubble-thinking-talk-5775045/"){: height="auto" width="30%"}

Java has a runtime. We have seen countless times to install or update JRE - Java Runtime environment. Python too has runtime. It is provided by the python executable. But what about C ? Does it have a runtime ? I have never come across a C runtime installer. How about C++ ?  
The short answer is **YES**.   
C has a runtime, albeit a tiny one and C++ also has a runtime. We don’t notice it because it is usually provided by the OS itself and with every new OS update, it gets updated. There is a small exception with that (VC runtime library on Windows) but we will talk about it later. Let’s get to the basics.

## What is C runtime (CRT) ? 
Whenever we refer to C runtime, we usually mean one or both of these things:
1. *Bootstrap and Runtime environment*  : It sets up the necessary bootstrapping code which tells the OS how to run this binary code, followed by code for setting up the runtime environment for executing user code. It is included as inline assembly in the compiled binary itself. 
2. *Runtime library* : it contains functions to interact with the OS; functions to allocate memory, read and write to files, network etc.


### Bootstrap and C runtime environment
As the name suggests, this is the code which sets up the execution point for the program for the OS to call. It defines the structure of functions, setup call stack and its calling convention (_stdcall, __cdecl ). This is the most important and necessary code to run our program in the OS.  It is specific to the OS version and doesn’t usually change but gets updated by the OS itself during its update. Among those dozens of Windows update we receive regularly, some of them update the CRT too.    

![winupdate](https://www.howtogeek.com/wp-content/uploads/2018/06/img_5b2d86d471d92.png "Please wait while Windows is updating ..."){: height="auto" width="50%"}



[`crt0`](https://en.wikipedia.org/wiki/Crt0) is a special file which gets linked to every program during the linking phase of compilation. It has basic program loading routines which instruct the compiler on how to call this program’s `main()` function and even exit the running program. Like everything else in the C and C++ ecosystem, there is a flexibility of choice on including it. If one wants to write his/her own startup code, they can avoid linking to crt0 altogether ([nostartfiles](https://gcc.gnu.org/onlinedocs/gcc/Link-Options.html) in GCC) and just provide their code in `_start` label in assembly ([`_start` calls `main()` of user code](https://stackoverflow.com/questions/29694564/what-is-the-use-of-start-in-c)). 
In case of C++, `crt1` can be linked alternatively,  which provides some extra functionality for constructor and destructors.


### C runtime library
The runtime library or standard library (`stdlib`) contains a series of useful functions like `malloc`, `free`, `printf`, `memcpy` ([MSVC](https://docs.microsoft.com/en-us/cpp/c-runtime-library/reference/crt-alphabetical-function-reference?view=msvc-160), [GCC](https://gcc.gnu.org/onlinedocs/gcc/Other-Builtins.html#Other-Builtins) ) for interacting with the OS which almost every program needs. This part can be statically or dynamically linked. Windows provides libcmt.lib for C and libcpmt.lib for C++. 
If one chooses to link it statically, only the code called by the user's program gets added to its binary, increasing the size. Removing stdlib can significantly reduce the binary size, which could be as significant as 90% in tiny programs. If one has to create the C code binary with smallest size, this would be the key step in reducing binary size to ~ 1 kb.

If one dynamically links to CRT, this size bump is not there (it is standard for programs to dynamically link to this library for multiple reasons like compatibility with other dll, apart from small size saving). Every OS provides an implementation. `UCRT.dll` on Windows and `libc.so` (GLibc) on Linux. 

>Normal dynamic loading of dll rule applies : If the program loads two different versions of CRT dll, they won’t share global variables. Not taking care of this fact could cause very subtle bugs in multi-dll codebases,  in calling runtime functions which maintain any state.

A user code cannot be faster than its runtime library’s performance.
Runtime libraries keep getting improvements through updates and one can receive those fixes as part of VC++Runtime updates on Windows or libc updates on Linux. Programs compiled with a specific CRT lib may expect that specific or compatible CRT dll. Thus, it is possible to keep side by side copies of different versions of CRT dlls ([DLL hell ?](https://en.wikipedia.org/wiki/DLL_Hell)). 

## All you need is C
C runtime is an interesting piece of software - it depends not only on OS and architecture but also on compiler. To give an example, runtime code generated on 64 bit windows by GCC would be different from that generated by MSVC. 
If you want to port C to a brand new OS,  all you need apart from the compiler is a working C runtime and that’s it, C gets ported.

Thanks for reading this far. For the last couple of months, I have been interested in inner workings of C and C++ ecosystem. This is an article in that direction. Tweet any suggestions or corrections to me at [@pratikone](https://twitter.com/pratikone)

## References
[https://stackoverflow.com/questions/2709998/crt0-o-and-crt1-o-whats-the-difference](https://stackoverflow.com/questions/2709998/crt0-o-and-crt1-o-whats-the-difference)

[http://www.vishalchovatiya.com/crt-run-time-before-starting-main](http://www.vishalchovatiya.com/crt-run-time-before-starting-main)

[https://blogs.oracle.com/linux/hello-from-a-libc-free-world-part-1-v2](https://blogs.oracle.com/linux/hello-from-a-libc-free-world-part-1-v2)

[https://docs.microsoft.com/en-us/cpp/c-runtime-library/c-run-time-library-reference?view=msvc-160](https://docs.microsoft.com/en-us/cpp/c-runtime-library/c-run-time-library-reference?view=msvc-160)

[https://gcc.gnu.org/onlinedocs/gcc/Other-Builtins.html#Other-Builtins](https://gcc.gnu.org/onlinedocs/gcc/Other-Builtins.html#Other-Builtins)