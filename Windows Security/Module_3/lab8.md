![MSCyberCoursesResized.png](./lab8/MSCyberCoursesResized.png)

<!-- TOC -->
# LAB 8: Memory Dump Analysis Part 1
## Abstract and learning objectives  

This training is designed to make you practice the concepts learned in the lectures.  
Learning objectives:  
- Extraction of information regarding processes and threads
- Extraction of information memory

## Overview 

This lab is a very simple environment consisting in a Windows 11 client and Windows Server 2022 servers. Both are members of an Active Directory domain northwindtraders.com. The server provides different services to support lab exercises. 

>[!ALERT] **DISCLAIMER**   
- Unless otherwise noted, the example companies, organizations, products, domain names, e-mail addresses, logos, people, places, and events depicted herein are fictitious, and no association with any real company, organization, product, domain name, e-mail address, logo, person, place or event is intended or should be inferred. Complying with all applicable copyright laws is the responsibility of the user. Without limiting the rights under copyright, no part of this document may be reproduced, stored in or introduced into a retrieval system, or transmitted in any form or by any means (electronic, mechanical, photocopying, recording, or otherwise), or for any purpose, without the express written permission of Microsoft Corporation.   
- Microsoft may have patents, patent applications, trademarks, copyrights, or other intellectual property rights covering subject matter in this document. Except as expressly provided in any written license agreement from Microsoft, the furnishing of this document does not give you any license to these patents, trademarks, copyrights, or other intellectual property.
- The names of manufacturers, products, or URLs are provided for informational purposes only and Microsoft makes no representations and warranties, either expressed, implied, or statutory, regarding these manufacturers or the use of the products with any Microsoft technologies. The inclusion of a manufacturer or product does not imply endorsement of Microsoft of the manufacturer or product. Links may be provided to third party sites. Such sites are not under the control of Microsoft and Microsoft is not responsible for the contents of any linked site or any link contained in a linked site, or any changes or updates to such sites. Microsoft is not responsible for webcasting or any other form of transmission received from any linked site. Microsoft is providing these links to you only as a convenience, and the inclusion of any link does not imply endorsement of Microsoft of the site or the products contained therein.

© 2022 Microsoft Corporation. All rights reserved.

Microsoft and the trademarks listed at <https://www.microsoft.com/en-us/legal/intellectualproperty/Trademarks/Usage/General.aspx> are trademarks of the Microsoft group of companies. All other trademarks are property of their respective owners.

===

## Exercise 1: Examining system's current activity

Duration: 1 hour

Synopsis: In this exercise, you will learn how to use the debugger to extract information about processes and threads currently running on a system.

### Task 1: Prepare the debugging environment
In this task, you ensure the debugger is able to locate symbols files successfully.

1. []Sign in **@lab.VirtualMachine(WIN-CLI2).SelectLink** with following credentials:  
	Username: **+++WIN-CLI2\Karen+++**  
	Password: Please type the password 

1. []From the **Start** menu, launch **WinDbg Preview**  

1. []Select **File** \\ **Start debugging** \\ **Open dump file** or hit **CTRL-D**

1. []Select the **MEMORY.DMP** file from **C:\\Lab\\Dumps\\1**

1. []In the command section, type the following command:  
    > `.sympath`

1. []Ensure you get something similar to:

    ```
    0: kd\> .sympath

    Symbol search path is:
    SRV*C:\Lab\Symbols*https://msdl.microsoft.com/download/symbols

    Expanded Symbol search path is:
    srv*c:\Lab\symbols*https://msdl.microsoft.com/download/symbols

    ************* Path validation summary **************

    Response Time (ms) Location

    Deferred
    SRV*C:\Lab\Symbols*https://msdl.microsoft.com/download/symbols
    ```

WinDBG is a command-line based debugger. The main window is divided into
2 parts:

-   **The command text box**. This is basically the location where you will type all the commands to the debugger. It is located at the very bottom of the main window. Most of the time, the text location is prefixed with   
***n*: kd\>**   
where n is the current processor ID. On a 2-processor machine, **n** will be either 0 or 1.

- **The output panel**. It is basically where the result of the commands you type is printed on. It takes all the upper region of the main widow, sitting just above the command text box. All the commands you type will be copied in the output panel prior to
display the command result.

There are 3 types of commands in WinDBG:

-   **Native commands**: they do not have any prefix character in their name.

-   **Meta-commands**: they all start with a '**.'** character.

-   **Extension commands**: they all start with a '**!'** character

At any time, you can consult the documentation of any command by hitting the **F1** key or opening the **Help** \\ **Window F1** menu item. Use the **Index** tab an type the command full name to display the help content for that particular command.

===

### Task 2: List currently running processes

In this Task, you will display the list of currently running processes by leveraging the `!process` WinDBG command.

2.  Determine the proper parameter list to display all the running
    processes with minimal details for each one.

3.  Run your command.

    If you ran the correct command, you should see an output similar to:

    ```
    **** NT ACTIVE PROCESS DUMP ****
    PROCESS ffffa7043f68e440
    SessionId: none Cid: 0004 Peb: 00000000 ParentCid: 0000
    DirBase: 001ca000 ObjectTable: ffff960b7e615000 HandleCount: 2036.
    Image: System

    PROCESS ffffa7043f7a6040
    SessionId: none Cid: 0058 Peb: 00000000 ParentCid: 0004
    DirBase: 00200000 ObjectTable: ffff960b7e63d000 HandleCount: 0.
    Image: Registry
    ```

---

**Evaluation**

1.  Write down the command you used:

    `!process 0 0`  
    Or,   
    `!process 0 0 _name_of_exe_file_`

1.  What is the address of the following processes?
    1.  Lsass.exe:  
        0xffffa7044232d080

    1.  Winlogon.exe:  
        0xffffa70442304400

    1.  Services.exe:  
        0xffffa70441ffe440

1.  What is the PID of the following processes?
    1.  Wininit.exe:  
        0x01fc

    1.  Explorer.exe:  
        0x0d04

    1.  Powershell.exe:
        0x0d5c

1.  For each of the following processes, give the name of their
    respective parent process:
    1.  Powershell.exe:  
        explorer.exe

    1.  Onedrive.exe:  
        explorer.exe

    1.  Spoolsv.exe:  
        services.exe

---

You have completed the process enumeration task, you can move on to the next step.

===

### Task 3: Get details about a running process -- part 1
In this task, you will focus on a single process and attempt to get
information about it. One location where you can find some interesting
data is the PEB *-- Process Environment Block*. The PEB is a subset of
Windows's internal control structure for processes which is available in
user mode. The process chosen for study is winlogon.exe but, the
principles applies to any other process.

1. []Run **!process ffffa70442304400 0** in order to get the general
    information regarding winlogon.exe\
    *Note: ffffa70442304400 is the address of winlogon.exe*

1. []Copy the address displayed right after **Peb:**, this is the virtual
    address of the PEB.

1. []Run !peb addr where addr is the virtual address of the PEB.\
    **!peb e934519000**

1. []You should see an error similar to :

    ```
    0: kd\> !peb e934519000
    PEB at 000000e934519000
    error 1 InitTypeRead( nt!\_PEB at 000000e934519000)\...
    ```

    This is an error triggered by the debugger because it is unable to read
    the PEB structure from target's memory.

    You must fix the issue by setting the current process context.

1. []Run **.process ffffa70442304400** in order to set the correct
    process context

1. []Run again **!peb e934519000**. You should obtain an output similar
    to:

    ```
    0: kd\> !peb e934519000
    PEB at 000000e934519000
    InheritedAddressSpace: No
    ReadImageFileExecOptions: No
    BeingDebugged: No
    ImageBaseAddress: 00007ff705190000
    ```

---

**Evaluation**

1.  Explain the failure: 

    The command failed because the current process context of the debugger is not the one of winlogon.exe. Virtual Addresses have sense only in the context of their owning process. We don’t know what the current process context is. In the current context, 0xe934519000 might simply not be a valid virtual address. To make the command work, the debugger must be first instructed to set its process context to the one of winlogon.exe

1.  Based on the output of the **!peb** command:

    1.  What is the value of the **USERNAME** environment variable?  
        SYSTEM

    1.  How many modules (main binary + loaded DLLs) were loaded by
        winlogon.exe?  
        41 (counting the DLLs + the EXE)

    1.  What is the command line which was used to launch the
        application?  
        `winlogon.exe`

1.  Use the same technique to analyze the svchost.exe process which PID
    is 0x0174 (PID is in hexadecimal form) and answer the following
    questions, based on the output of the **!peb** command:

    1.  What is the value of the **USERNAME** environment variable?  
        DESKTOP-RDFVC7F$

    1.  Did svchost.exe load WinSCard.dll?  
        No

    1.  What is the command line which was used to launch this svchost.exe process?  
        `C:\Windows\System32\svchost.exe -k LocalSystemNetworkRestricted -p`

---

### Task 4: Get details about a running process -- part 2

In this task, you will display more details about a running process by
dumping the content of Kernel-mode structures. The process studied in
this exercise is an instance of powershell.exe.

One of the additional entries brought by **!process** with greater
detail level is the address of the process's token. Process's token
represents the identity of the process when accessing resources and is
generally inherited from the user or account which has started it.

Assignments:

1. []Run **!process ffffa704432a1580 1** in order to get details about
    the PowerShell process

1. []Get the address of the token object

1. []Run **!token -n *addr*** where *addr* is the address of the token
    object\
    **!token -n ffff960b821f2720**

---

**Evaluation**

1.  Based on the output of the !token command, answer following
    questions:

    a.  What is the SID of the user?  
        `S-1-5-21-201441023-2640717268-4266869512-1001`

    b.  Does the user belong to the local Administrators group?  
        Yes

    c.  What is the Mandatory Integrity Level of that user?  
        High or 12288

    d.  Does the user is granted the SeTcb privilege?  
        No

    e.  Does the user is granted the SeDebug privilege?  
        Yes

2.  Based on the output of the !process command output, how long does
    the process have been running for?  
    00:16:46.364 or 00 hours, 16 minutes, 46 seconds, 364 msecs

---

You have completed the process details gathering task. You can move on to the process activity analysis task.

===

### Task 5: Explore a process activity

As you dig deeper in your analysis, you may want to understand what a
process is doing. Windows does not schedule processes. Processes are
mainly a container and an isolation layer between applications. Windows
schedule threads. In this task, you will learn

Assignments:

1. []Run .process ffffa70442304400 to set the current process to
    winlogon.exe

1. []Run .reload /f /user in order to load winlogon's symbols\
    Note: Symbols are files which help the debugger to map memory areas
    with functions and variables names in the source code.

1. []Run !process ffffa70442304400 7 in order to list all threads running
    inside the process.

You should get an output like:

```
0: kd\> !process ffffa70442304400 7

PROCESS ffffa70442304400
SessionId: 1 Cid: 0248 Peb: e934519000 ParentCid: 01e0
DirBase: 08f80000 ObjectTable: ffff960b7f991b80 HandleCount: 256.
Image: winlogon.exe
VadRoot ffffa70442303f70 Vads 77 Clone 0 Private 392. Modified 1786.
Locked 0.
DeviceMap ffff960b7e61fd90

THREAD ffffa70441ffd080 Cid 0248.024c Teb: 000000e93451a000 Win32Thread:
ffffa70441e01520 WAIT: (UserRequest) UserMode Non-Alertable

ffffa7044272cf80 SynchronizationEvent

Not impersonating
```

---

**Evaluation**

1.  How many threads does the winlogon process own?  
    6

2.  List all thread IDs in hexadecimal form  
    24c, 268, 554, dc0, 250, ae8 

3.  In that instance of winlogon.exe, the thread 0x24c is currently in
    the WAIT state meaning it iss not running but rather waiting on a
    resource to be signaled or available. How long this thread has been
    waiting for?  
    50.562s or 50 seconds and 562 millisecs

---

===

## Exercise 2: Analyzing memory usage

Duration: 1 hour

Synopsis: In this exercise, you will learn how to use the debugger to
extract information about how memory is used by the system.

### Task 1: Display general memory usage

In this task, you will discover how to get information about general
memory usage on the system.

1. []Sign in **@lab.VirtualMachine(WIN-CLI2).SelectLink** with following credentials:  
	Username: **+++WIN-CLI2\Karen+++**  
	Password: Please type the password 

1. []Open the same dump as in previous exercise.  
    _C:\Lab\Dumps\1\MEMORY.DMP_

1. []Run the command which displays general virtual memory information.

---

**Evaluation**

1.  How much physical memory is installed on this system?  
    524038 pages or 2096152 KB

2.  How much memory is used by the Non Paged pool (normal and NX)?  
    * 12790 pages or 51160 KB (using !vm which is the preferred way)
	* Also accepted: 1040Kb or 260 pages
	* Other possible answer: 50511440 bytes (using !poolused)

3.  How much memory is used by the Paged Pool?
    * 15945 pages or 63780 KB
	* Other possible answer: 64206880 (using !poolused)

4.  How much memory is Committed by the powershell.exe program?  
    Possible answers : 59312 Kb or 14828 pages or 60735488 bytes

---

### Task 2: Get insights about pools consumption

In this task, you will learn basic commands to visualize the Paged and
NonPaged pools allocations

1. []Run the command: !poolused 1

---

**Evaluation**

1.  Briefly explain what the !poolused 1 command does.  
    The command summarizes non paged pool and paged pool allocations for each tag

2.  How much memory is allocated under the Toke tag?  
    2831840 bytes

3.  What kind of kernel object does the Toke tag relate to?  
    Token objects

4.  Find the WinDBG command to list all allocations for a given tag.  
    !poolfind

---

===

### Task 3: Examine the memory layout of an application

In this task, you will learn how to display the memory layout of an
application. Windows Memory Manager keeps internal structure to describe
the virtual address space of a process. These structure are called VAD
*-- Virtual Address Descriptors* and describes the current status of a
memory region in the process's address space. Any valid virtual address
belongs to a VAD. In this example, the application is winlogon.exe

1. []Run .process ffffa70442304400 to change the process context to
    Winlogon.exe

1. []Run !process ffffa70442304400 1 to display detail process
    information:

1. []Retrieve the address of the root VAD. This address is located right
    after the VadRoot tag

In the case of winlogon.exe, root VAD is located at address
0xffffa70442303f70

4.  Run !vad ffffa70442303f70

You should get an output like:

```
0: kd\> !vad ffffa70442303f70
VAD Level Start End Commit
ffffa70441fe4980 6 7ffe0 7ffe0 1 Private READONLY
ffffa70442304150 5 7ffef 7ffef 1 Private READONLY
ffffa70441ffbce0 6 e934400 e9345ff 13 Private READWRITE
ffffa70441ffdbe0 4 e934600 e93467f 17 Private READWRITE
ffffa7044318a450 6 e934680 e9346ff 17 Private READWRITE
```

---

**Evaluation**

1.  What is the address of the VAD which correspond to the mapping of
    profext.dll in winlogon's virtual address space?  
    0xffffa704423630b0

2.  What is the start address of profext.dll?  
    0x7ff8d4540000

3.  What is the physical address which corresponds to the virtual
    address found in previous question?  
    0x147d5000

---