---
layout: post
title:  "Anatomy of Windows Hello World program"
date:   2020-06-07 14:29:37 -0700
categories: c++ windows-hello-world
---

# Anatomy of Windows Hello World program

Windows programming is in state of constant change since the dawn of Win32 api (even before that) - whether it is move to .NET, again to WPF, introduction of Modern Apps which then got changed to UWP and now finally Project Reunion (merger of Windows apis announced in Build 2020). One thing that hasn’t changed at all in this is how to write a hello world in Windows. Since Win95, it has largely remained the same. However, a lot has changed internally. Windows does a lot of work under the hood to make sure the simple hello world continues to work even when the kernel changes (Win 98 -> Win XP)  and large scale OS changes (XP -> Vista, Win7 -> Windows 8/10). This blog attempts to go deeper into the hello world to showcase the rich technical background of windows hello world and a little bit of history.
Intro
Win32 api are the system level api for programming Windows at the lowest level. When Windows transitioned to 32-bits OS with Win95, they wanted a way to distinguish between old WinApi for 16-bit Win3.1 OS and the newer 32-bit apis. Those 32-bit api got the name Win3
 Win32. Such is the price of popularity. It will cause a whole lot more confusion and change of documents to remove 32 from the name. So, Windows is continuing with it. <https://en.wikipedia.org/wiki/Windows_API>
The hello world code for Win32 has largely unchanged since its inception except when the infamous hello world program got replaced with a shorter version because it was intimidating to beginners. <http://www.charlespetzold.com/blog/2014/12/The-Infamous-Windows-Hello-World-Program.html>. This anatomy is not of that code but the successor which is decent sized and captures a lot of Windows functionality in itself. Let’s start by comparing this hello world with its console cousin.

Console hello world
======================
```c++
#include <iostream>

int main()
{
    std::cout << "Hello World!\n";
}
```

That is pretty straight-forward. You call the IO header, entry point is main() and it prints hello world. Compare it to win32 hello world.

Windows hello world

```c++
#ifndef UNICODE
#define UNICODE
#endif

#include <windows.h>

LRESULT CALLBACK WindowProc(HWND hwnd, UINT uMsg, WPARAM wParam, LPARAM lParam);

int WINAPI wWinMain(HINSTANCE hInstance, HINSTANCE, PWSTR pCmdLine, int nCmdShow)
{
    // Register the window class.
    const wchar_t CLASS_NAME[]  = L"Sample Window Class";
    
    WNDCLASS wc = { };

    wc.lpfnWndProc   = WindowProc;
    wc.hInstance     = hInstance;
    wc.lpszClassName = CLASS_NAME;

    RegisterClass(&wc);

    // Create the window.

    HWND hwnd = CreateWindowEx(
        0,                              // Optional window styles.
        CLASS_NAME,                     // Window class
        L"Learn to Program Windows",    // Window text
        WS_OVERLAPPEDWINDOW,            // Window style

        // Size and position
        CW_USEDEFAULT, CW_USEDEFAULT, CW_USEDEFAULT, CW_USEDEFAULT,

        NULL,       // Parent window    
        NULL,       // Menu
        hInstance,  // Instance handle
        NULL        // Additional application data
        );

    if (hwnd == NULL)
    {
        return 0;
    }

    ShowWindow(hwnd, nCmdShow);

    // Run the message loop.

    MSG msg = { };
    while (GetMessage(&msg, NULL, 0, 0))
    {
        TranslateMessage(&msg);
        DispatchMessage(&msg);
    }

    return 0;
}

LRESULT CALLBACK WindowProc(HWND hwnd, UINT uMsg, WPARAM wParam, LPARAM lParam)
{
    switch (uMsg)
    {
    case WM_DESTROY:
        PostQuitMessage(0);
        return 0;

    case WM_PAINT:
        {
            PAINTSTRUCT ps;
            HDC hdc = BeginPaint(hwnd, &ps);



            FillRect(hdc, &ps.rcPaint, (HBRUSH) (COLOR_WINDOW+1));

            EndPaint(hwnd, &ps);
        }
        return 0;

    }
    return DefWindowProc(hwnd, uMsg, wParam, lParam);
}
``` 
That is way bigger, but it also does a lot of things. It launches a typical rectangular UI window filled with a background color. Now, that may not seem like much but we are operating at syscall level for Windows. This much code achieves a lot including functional UI, proper keyboard and mouse support and event processing.  Unlike other hello world programs for other platforms, this hello world is the foundation of any Win32 giant codebase out there. They have to implement all of this somewhere in the code whether it is Photoshop or Firefox for Windows etc
Now that is out of the way, let’s follow this code block by block.

## Anatomy
Header file windows.h
Over the years, Windows programming has undergone large changes, like move from 16-bit to 32-bit api in Win95, move to NT kernel with XP and later changes for Vista and Windows 8. Any app whether the one developed in 1995 or in 2008 which calls windows.h must continue to work in the latest version of Windows.  Windows.h and Win32 apis do a lot of heavy duty work to ensure that all these apps remain compatible even when they are all including the same header.  For example, if you are developing a new app which uses a legacy component (from Windows 95 era referring to windows.h header files of that era ),  it is possible that your new component and legacy component will consume the same windows.h (from latest Windows SDK) and yet, pick up the era-appropriate functions.
It is a catch-all header for most common windows system calls. Windows.h, a small file on its own, carries forward declaration for multiple headers. For this hello world code, the apis are in winuser.h and linked using user32.dll. For any other functionality, some other header might be needed. Windows.h makes it easy by acting as a “router” for all these headers. You only need to include windows.h header and you get all this functionality. As MSDN page mentions, you can carefully select a smaller subset of functionality by defining this macros. (https://docs.microsoft.com/en-us/windows/win32/winprog/using-the-windows-headers)

### Main function
int WINAPI wWinMain(HINSTANCE hInstance, HINSTANCE, PWSTR pCmdLine, int nCmdShow)
 Is the entry point to the program. (explain basic args). Comparing against WinMain of Win95
 On a quick glance,  there are a lot of 0s and NULLs passed as arguments in functions. Many of these functions have existed since the inception of Win32 apis and have changed in their behavior. Thus, many of the arguments no longer have any effect and are left for compatibility sake. Thise arguments are always NULL or 0. For the rest of the arguments, flags can be combined with logical OR |.  
Another interesting artefacts of the past are wParam and lParam parameters. Before Win95, Windows was 16-bit OS. wParam was WORD param which was 16-bit and lparam was LONG param which was 32-bit. With Win95,Windows moved to 32-bit era and both WORD and LONG are now 32-bit so there is no difference between wParam and lParam.

### Window of Windows - Hwnd (pronounced as h-wind)

#### What is an Hwnd ?
<pictures of hwnd : https://docs.microsoft.com/en-us/windows/win32/learnwin32/images/window01.png>

“Obviously, windows are central to Windows. They are so important that they named the operating system after them.”  ((https://docs.microsoft.com/en-us/windows/win32/learnwin32/what-is-a-window-))
A window is a rectangular area on the screen that receives user input and displays output in the form of text and graphics. (https://www.amazon.com/Programming-Windows%C2%AE-Fifth-Developer-Reference/dp/157231995X )

In Win32 programming convention, a window is referenced by HWND - handle to a window. A handle is a numerical value associated with that window. In the kernel, an object is created for every window with an unique id. This id is stored in that window’s handle. For easy understanding, assume hwnd and its window are one and the same thing. If a statement says, change the size of hwnd, that is referring to the window referenced by that hwnd. If a window is a person, hwnd is its name. ;]

#### Everything is an hwnd
In a true sense of the movie Inception, every UI element in a typical hwnd is a hwnd itself. (<show picture of hwnd with buttons, menu and controls>).  This is achieved using child windows/owned windows (more in CreateWindow). This leads to a lot of HWNDs in a moderately complex app. Frameworks like XAML(WinUI), WPF have found non-HWND ways to create UI elements but it is still pervasive enough. (talk of GDI ?) 
Spy++, bundled with Visual Studio, is a good tool to visualize that. (picture of Spy++).

https://docs.microsoft.com/en-us/windows/win32/learnwin32/images/window03.png

https://docs.microsoft.com/en-us/windows/win32/learnwin32/images/window04.png

Modern Windows HWNDs are hardware accelerated i.e. take help of graphics pipelines like Direct2D to draw pixels faster, using a GPU. Comparing with running same hello world in win95 vs win 10
  Good read : https://en.wikipedia.org/wiki/Desktop_Window_Manager (reference).


Hwnd registration and window messaging system are the few pieces of code here which are reminiscent of very 80s Object Oriented (OO) design. Microsoft was on-board OO train very early on, even when it was not totally accepted by the industry. It was when Microsoft had been using only C for programming Windows. OO in C requires a lot of weird design choices and because of its great backward compatibility, a lot of it has stayed.

#### Register the window class with OS

// Register the window class.
    const wchar_t CLASS_NAME[]  = L"Sample Window Class";
    
    WNDCLASS wc = { };

    wc.lpfnWndProc   = WindowProc;
    wc.hInstance     = hInstance;
    wc.lpszClassName = CLASS_NAME;

    RegisterClass(&wc);

Windows has a weird way of doing inheritance (reminder of 80s origin of Windows architecture design). You will have to register your newly created hwnd object with OS for it to start communicating with it. 

#### Create the aforementioned Window
// Create the window.
    HWND hwnd = CreateWindowEx(
        0,                              // Optional window styles.
        CLASS_NAME,                     // Window class
        L"Learn to Program Windows",    // Window text
        WS_OVERLAPPEDWINDOW,            // Window style

        // Size and position
        CW_USEDEFAULT, CW_USEDEFAULT, CW_USEDEFAULT, CW_USEDEFAULT,

        NULL,       // Parent window    
        NULL,       // Menu
        hInstance,  // Instance handle
        NULL        // Additional application data
        );

    if (hwnd == NULL)
    {
        return 0;
    }

This code creates the HWND. We provide the window classes (we just registered) to be associated with this hwnd. Window creation api is very rich https://docs.microsoft.com/en-us/windows/win32/apiindex/windows-api-list and can create 100s windows with different configurations and behaviors. One good example is creation of child windows. Since everything is a window. A button within a UI window is a child to that hwnd. That means the parent window can handle processing of messages for that child window. This kind of hierarchy is extremely useful. It is an implementation of inheritance concept from the OO paradigm. In this way, you can add a new window as child window of parent window and you don’t have to write any additional code for its basic operations like resize, maximize, close etc.

#### Show the god-damn window
ShowWindow(hwnd, nCmdShow);
Shows the window on the screen. What is the point of explicitly calling show window ? There are many cases where a window is not shown directly. It can be created with the above command and updated using UpdateWindow and when ready, shown to the screen. There is also AnimateWindow for doing 90s PPT slide like transition animations which nobody should use in modern day and age.
Windows messaging system
Windows messaging system is event-driven achieved using dynamic polymorphism. The OS communicates to the program by passing messages and it will call a special function in your program to give you an option to deal with those messages. These messages can be keyboard, mouse, touch events generated on interacting with your program, or OS created events like when your program is minimized, maximized or closed. 
For this message passing model, Windows creates a message queue for a thread which handles all the messages for HWNDs created on that thread.

### Message loop

// Run the message loop.

    MSG msg = { };
    while (GetMessage(&msg, NULL, 0, 0))
    {
        TranslateMessage(&msg);
        DispatchMessage(&msg);
    }

Message loop code is what is responsible for filing up this message queue.  There has to be only one message loop per thread. This message queue is hidden and not accessible by your code. It is handled entirely by the OS. All your code can do is to remove the topmost message from this queue using GetMessage() api call. This message is then translated for keyboard input so that it can handle shortcut keys and do other keyboard input processing (https://stackoverflow.com/questions/12581889/why-exactly-translatemessage) and then it is dispatched to the handler function WndProc (discussed below).  GetMessage is a blocking function so it will wait if the loop is empty. But this doesn’t mean your UI will be unresponsive. An alternative to that is PeekMessage function which can peek and tell if there is a message on top of the queue. Since it won’t block, it is good for certain scenarios to do a “Peek” before a “Get”.



#### WndProc - the most important function in your code
LRESULT CALLBACK WindowProc(HWND hwnd, UINT uMsg, WPARAM wParam, LPARAM lParam)
{
    switch (uMsg)
    {
    case WM_DESTROY:
        PostQuitMessage(0);
        return 0;

    case WM_PAINT:
        {
            PAINTSTRUCT ps;
            HDC hdc = BeginPaint(hwnd, &ps);



            FillRect(hdc, &ps.rcPaint, (HBRUSH) (COLOR_WINDOW+1));

            EndPaint(hwnd, &ps);
        }
        return 0;

    }
    return DefWindowProc(hwnd, uMsg, wParam, lParam);
}

WndProc is the special function which has to be present in every Win32 program code, either directly or indirectly.  WndProc is the function which Windows OS calls whenever it has to communicate anything with your running code. 
WndProc usually has a giant switch statement for handling window messages like WM_DESTROY (what to do when a user clicks on the small x on the top-right of the window). It can choose to ignore it and it won’t close the window. Thankfully, there are other ways to close a window. This shows the level of control and flexibility Windows OS provides to developers which can be beneficial but can also be misused and the recent Windows programming model has evolved to counter that. 
PostQuitMessage adds a WM_QUIT message to the message queue which causes GetMessage() to false, exiting the loop and exiting the program.
Your program doesn’t have to handle all the messages. It can handle a few special messages of interest and then call DefWindowProc - OS provided default handler to deal with the rest. WndProc can choose to handle messages for all windows in a thread or it can defer them to respective WndProc to handle.  See Subclassing as example of dynamic polymorphism to achieve that https://docs.microsoft.com/en-us/windows/win32/api/commctrl/nf-commctrl-defsubclassproc 

#### Painting the window
The code within case WM_PAINT  is the boilerplate code for drawing anything in a window. Windows Graphics Driver Interface (GDI) is immediate mode (link to it). A lot of newer UI libraries like WPF and WinUI are retained mode GUI frameworks because of memory and performance reasons. MSDN’s  Painting the Window - Win32 apps does a very good job of explaining this code. 

Compilation on terminal, without Visual Studio
With newer msbuild tools unbundled from Visual Studio, one can easily compile them from terminal only, without needing to open Visual Studio. Make sure you have installed Microsoft C++ Build Tools 
The functions exported by windows.h have their definition in user32.lib and user32.dll. You only need those two for compilation. Every windows OS installation has these and they are included in the system path, so no extra steps needed.

```
> set cl & set link
CL=/EHsc /GR /FI"iso646.h" /Zc:strictStrings /we4627 /we4927 /wd4351 /W4 /D"_CRT_SECURE_NO_WARNINGS" /nologo
LINK=/entry:mainCRTStartup /nologo


> cl winhello.cpp /Fe"hello" /link /subsystem:windows user32.lib
winhello.cpp
````



Ending notes
All the above functions make up for the basics of every Win32 application program written out there whether it is a giant complex program like Photoshop or Firefox.  
Future scope 


References
MSDN
Charles Petzold book
https://stackoverflow.com/questions/20778319/command-line-to-compile-win32-hello-world-in-visual-studio-2005

Programming Windows 1998
WinRT, WinUI3

