![MSCyberCoursesResized.png](./lab9/MSCyberCoursesResized.png)

<!-- TOC -->
# LAB 9: Memory Dump Analysis Part 2
## Abstract and learning objectives  

This training is designed to make you practice the concepts learned in the lectures.  
Learning objectives:  
- Extract information on loaded drivers
- Analyze IRPs
- Analyze a device's stack

## Overview 

This lab is a very simple environment consisting in a Windows 11 client and Windows Server 2022 servers. Both are members of an Active Directory domain northwindtraders.com. The server provides different services to support lab exercises. 

>[!ALERT] **DISCLAIMER**   
- Unless otherwise noted, the example companies, organizations, products, domain names, e-mail addresses, logos, people, places, and events depicted herein are fictitious, and no association with any real company, organization, product, domain name, e-mail address, logo, person, place or event is intended or should be inferred. Complying with all applicable copyright laws is the responsibility of the user. Without limiting the rights under copyright, no part of this document may be reproduced, stored in or introduced into a retrieval system, or transmitted in any form or by any means (electronic, mechanical, photocopying, recording, or otherwise), or for any purpose, without the express written permission of Microsoft Corporation.   
- Microsoft may have patents, patent applications, trademarks, copyrights, or other intellectual property rights covering subject matter in this document. Except as expressly provided in any written license agreement from Microsoft, the furnishing of this document does not give you any license to these patents, trademarks, copyrights, or other intellectual property.
- The names of manufacturers, products, or URLs are provided for informational purposes only and Microsoft makes no representations and warranties, either expressed, implied, or statutory, regarding these manufacturers or the use of the products with any Microsoft technologies. The inclusion of a manufacturer or product does not imply endorsement of Microsoft of the manufacturer or product. Links may be provided to third party sites. Such sites are not under the control of Microsoft and Microsoft is not responsible for the contents of any linked site or any link contained in a linked site, or any changes or updates to such sites. Microsoft is not responsible for webcasting or any other form of transmission received from any linked site. Microsoft is providing these links to you only as a convenience, and the inclusion of any link does not imply endorsement of Microsoft of the site or the products contained therein.

© 2022 Microsoft Corporation. All rights reserved.

Microsoft and the trademarks listed at <https://www.microsoft.com/en-us/legal/intellectualproperty/Trademarks/Usage/General.aspx> are trademarks of the Microsoft group of companies. All other trademarks are property of their respective owners.

===

## Exercise 1: Devices and drivers

Duration: 1 hour

Synopsis: In this exercise, you will learn how to use the debugger to
extract information about drivers and devices.

### Task 1: List driver modules currently loaded by the system

In this task, you will learn how to extract the list of loaded kernel
modules. This list is also the list of drivers (in the meaning driver
.sys files) leaded by the system. A driver module must not be confused
with a driver object as there is no guaranty of a 1:1 mapping.

1. []Sign in **@lab.VirtualMachine(WIN-CLI2).SelectLink** with following credentials:  
	Username: **+++WIN-CLI2\Karen+++**  
	Password: Please type the password 

1. []Open dump **C:\Lab\Dumps\1\MEMORY.dmp**

1. []Run the command to list all drivers loaded by the system

1. []Using the same command with different arguments, display the details about the `ntfs` module.

---

**Evaluation**

1.  Write down the command you ran.  
    * To list the driver modules loaded by the system:  
    `lm` (other derivatives are possible)
    * To list details about a specific module:  
    `lmvm module_name` (other variations are possible as soone as the `v` flag is present)

1.  Is the appid.sys driver loaded?  
    No

1.  What Is the full path of the ntfs.sys driver?  
    `\SystemRoot\System32\Drivers\Ntfs.sys`

1.  What is the file version of ntfs.sys?  
    10.0.17134.1

---

### Task 2: Navigate the object manager

All drivers and devices are represented internally as objects. The
object manager is responsible for storing objects, keeping reference
counts accurate and organize objects in a tree-like structure.

In this task, you will dump some of the object manager structure in
order to list all driver and device objects.

1. []Run command .reload /f in order to ensure symbols are loaded.

1. []Using the WinDBG help content, find a command to dump objects.
    Familiarize yourself with command's options.

1. []Dump the list of all objects in the driver namespace \\Driver

1. []Find a command specially designed to dump the details of a driver
    object and dump the details of the HTTP driver, including driver's
    dispatch routines and device objects owned by the driver.

1. []Find a command specially designed to dump the details of a device
    and dump the details of device 0xffffa704428b1c20

1. []Dump the security descriptor of this device

---

**Evaluation**

1.  What is the address of the HTTP driver object?  
    ffffa70442864e60

2.  How many devices are owned by this driver?  
    3

3.  What is the name of the HTTP driver's routine handling IRP_MJ_CREATE IRPs?  
    `HTTP!UxCreate`

4.  Regarding the security descriptor of device 0xffffa704428b1c20, which principal is the owner of this device?  
    S-1-5-32-544 (Alias: BUILTIN\Administrators)

---

### Task 3: I/O Request Packets

In this task, you'll learn how the object manager and driver works
together to access a file in a storage location. You will start with the
C: drive which is the usual location where Windows and applications are
installed. The true object name for the C: drive is \\Global??\\C:

1. []Using learnings from previous task, dump the \\Global??\\C: object

1. []Note this object is like a redirector to another object. Dump the
    true target object.

1. []As this object is a device, use the device specific command to dump
    the device object.

1. []When an application opens a file, the device which handles the
    request is generally an instance of the volmgr driver (volmgr stands
    for Volume Manager). So, opening a file means asking the volume
    manager to open the file from the particular volume. But, volmgr
    does not have the knowledge of the layout of filesystems. It must
    request some help from a filesystem driver. The link between a
    specific volume device and the filesystem device is stored in the
    VPB -- *Volume Parameter Block*.\
    Using the help content of WinDBG, find the command which can dump
    the content of a VPB. Then, use the result from previous step to
    dump the VPB of the volume. The address of the filesystem device is
    specified as the DeviceObject field.

1. []Now, you have the address of the Ntfs device which handles the C:
    volume. We want to know all I/O requests which are processed by this
    volume.\
    Using the help content of WinDBG, find a command which can search
    IRP using search criterias. Then, run a command to list all IRP
    which are related to the Ntfs device.

1. []We are interested by the IRP which has its minor and major codes
    equals to 0. This one should appear as (0, 0) in the IRP list.\
    Using the help content of WinDBG, find a way to dump the content of
    the IRP.

1. []In the IRP stack, you can find the address of the file object which
    is manipulated by the IRP. Use the !fileobj command to dump this
    file object.

1. []The IRP also contains the address of the thread which is currently
    processing the IRP. The thread address is displayed at the very top
    of the irp details.\
    Using command from previous exercises, find the process which this
    thread belongs to.

---

**Evaluation**

1.  Write down the command and resulting output from step #1.

    ```
    0: kd> !object \Global??\C:
    Object: ffff960b7e728bb0  Type: (ffffa7043f65e390) SymbolicLink
    ObjectHeader: ffff960b7e728b80 (new version)
    HandleCount: 0  PointerCount: 1
    Directory Object: ffff960b7e6049f0  Name: C:
    Flags: 00000000 ( Local )
    Target String is '\Device\HarddiskVolume3'
    Drive Letter Index is 3 (C:)
    ```

2.  What is the exact object type of `\Global??\C:`?  

    `\Global??\C:` is a symbolic link.

3.  Write down the command and resulting output from step #2

    ```
    0: kd> !object \Device\HarddiskVolume3
    Object: ffffa70440bef280  Type: (ffffa7043f717dc0) Device
    ObjectHeader: ffffa70440bef250 (new version)
    HandleCount: 0  PointerCount: 11
    Directory Object: ffff960b7e62eea0  Name: HarddiskVolume3
    ```

4.  Write down the command and resulting output from step #3

    ```
    0: kd> !devobj ffffa70440bef280
    Device object (ffffa70440bef280) is for:
    HarddiskVolume3 \Driver\volmgr DriverObject ffffa704405de840
    Current Irp 00000000 RefCount 3976 Type 00000007 Flags 00001150
    Vpb ffffa70440bef1c0 SecurityDescriptor ffff960b7e7e7060 DevExt ffffa70440bef3d0 DevObjExt ffffa70440bef5a0 Dope ffffa70440bb3150 DevNode ffffa70440bf1d20 
    ExtensionFlags (0x00000800)  DOE_DEFAULT_SD_PRESENT
    Characteristics (0x00020000)  FILE_DEVICE_ALLOW_APPCONTAINER_TRAVERSAL
    AttachedDevice (Upper) ffffa70440c29030 \Driver\fvevol
    Device queue is not busy.
    ```

5.  Write down the command and resulting output from step #4

    ```
    0: kd> !vpb 0xffffa70440bef1c0
    Vpb at        0xffffa70440bef1c0
    Flags:        0x1 mounted 
    DeviceObject: 0xffffa70440c30030 (dt nt!DEVICE_OBJECT)
    RealDevice:   0xffffa70440bef280 (dt nt!DEVICE_OBJECT)
    RefCount:     3973
    Volume Label: ""
    ```

6.  Write down the command you used for step #5

    ``` 
    !irpfind 0 0 device ffffa70440c30030
    ```

7.  What is the functional description of an IRP with major code equals
    to 0?

    It is a CREATE operation.    

8.  Which file is being accessed?

    ```
    C:\Windows\explorer.exe
    ```

9.  Which application is making the call?

    ```
    Powershell.exe
    ```

---

You have completed this exercise. Congratulations.

===

## Exercise 2: Anatomy of a keylogger

Duration: 1 hour

Synopsis: In this exercise, you will analyze a memory dump from a system which was infected by kernel-mode key logger.

### Task 1: Locate the keyboard device stack.
In this task, you start by displaying the device stack of the keyboard.

The dump was captued on a Hyper-V virtual machine, we know that the keyboard driver is `hyperkbd`.

1. []Sign in **@lab.VirtualMachine(WIN-CLI2).SelectLink** with following credentials:  
	Username: **+++WIN-CLI2\Karen+++**  
	Password: Please type the password 

1. []Open dump **C:\Lab\Dumps\2\MEMORY.dmp**

1. []Using the **!devnode** command, display the entire device tree.  
    >Refer the the course material or WinDbg documentation to understand how you need to call the command.

1. []Locate the only device node corresponding to the **hyperkbd** driver.
    >The **!devnode** displays the address of the PDO corresponding to the keyboard.

1. []Using the **!devstack** command, dump the device stack of the PDO.
    >Refer the the course material or WinDbg documentation to understand how you need to call the command.

You should obtain a result similar to
```
  !DevObj           !DrvObj            !DevExt           ObjectName
  ffffdc84a94cede0  \Driver\pcie       ffffdc84a94cef30  
  ffffdc84a3b04aa0  \Driver\kbdclass   ffffdc84a3b04bf0  KeyboardClass0
  ffffdc84a36288d0  \Driver\hyperkbd   ffffdc84a363ee30  
> ffffdc84a2bb2c90  \Driver\vmbus      ffffdc84a2bba320  0000001e
```

---

**Evaluation**

1. What is the value of the PDO address?

    ffffdc84a2bb2c90

1. What are the drivers present in the keyboard's device stack?

    * `\Driver\pcie`
    * `\Driver\kbdclass`
    * `\Driver\hyperkbd`
    * `\Driver\vmbus`

1. Is there any suspicious driver? If yes, what is it and why is it suspicious?

    Suspicious driver is pcie. Valid reasons:
    * It does not exist in Windows
    * It is at the top of the stack
    * The name does not match a valid usage with the keyboard

---

You have completed the device stack analysis task. You can move on to the pcie driver analysis.

===

### Task 2: Anaylyze the pcie.sys driver
In this task, you will analyze what the driver pcie.sys does and what are its interractions with the keyboard.

1. []Using the **!drvobj** command, display the dispatch routines of the driver
    
    _Refer the the course material or WinDbg documentation to understand how you need to call the command._

    >From the list, you can observe there are only 2 dispatch routines supported by this driver :
    * pcie!KLDvDispatchRead for READ IRPs.
    * pcie!KLDvDispatchPassThrough for other IRPs.
    When Windows reads the state of the keyboard (key pressed), it dispatches a READ IRP to the keyboard device stack.    

1. []Using the **u** command family, unassemble pcie!KLDvDispatchRead

    >The keylogger example is simple enough we can understand what it does by looking at the disassembled code and focus on the **call** instructions. This instructions denote a call to another function, most of them being public kernel API.

    >When in KLDvDispatchRead, the keylogger does not know yet which keys are pressed. Keylogger is at the top of the stack. The READ order hasn't been sent yet to the keyboard. The keylogger will do most of its work after the keys are read. To perform activities after the IRP has been completed, a driver can set a completion routine which will be executed after IRP is completed.

1. []Using disassembly output and public Windows API documentation, find the name of the completion routine set by the KLDvDispatchRead dispatch routine.

    >[!hint]In x64 calling convention, the second argument of a function is in the RDX register.

---

**Evaluation**

1. What **!drvobj** command did you run?

    `!drvobj \Driver\pcie 3`

1. What is the dispatch routine which will be executed when the IRP is completed?

    `pcie!OnReadCompletion`

---

You have completed the analysis of the driver's dispatch routines. You can move on to a deeper analysis of pcie!pcie!KLDvDispatchRead.

===

## Task 3: Anaylyze the completion routine
In this task, you will analyze what is the completion routine doing.

1. []Using the **u** command family, unassemble the completion routine.

1. []Read the public documentation for the routines called by the 3 first **call** instructions.

    >[!hint]Remove the \_imp\_ prefix.

---

**Evaluation**

1. What are the 3 first API called by the **call** instructions and what are their respective functional description (1 sentence for each)?

    API called:
    * ExAllocatePoolWithTag : allocates paged pool memory with a specified Tag.
    * ExInterlockedInsertTailList : atomically inserts an entry at the end of a doubly linked list
    * KeReleaseSemaphore : releases the specified semaphore object

1. What is a semaphore?

    A semaphore is a synchornization mechanism which helps controling how many threads can access a specific resource or memory area.

1. Based on the API called by the completion routine, make an argumented guess on what that routine does.

    The completion routine is called after the I/O request is completed. It contains the list of key statuses. The completion routine is probably enqueuing the data and signaling a semaphore so that another thread can pick up the key pressed events and process them asynchronously.

---

You have completed the completion routine analysis task. You can move on to the next step.

===

### Task 4: Analysis of the processing routine.
In this task, you will analyze how the key pressed events are processed by the keylogger.

1. []Using the **!process** command, try to find another thread running the keylogger's code.

    >[!hint]Kernel threads belong to the **System** process.

1. []Dump the details of the thread and understand what this thread is waiting for.

1. []Using the **u** command family, unassemble the keylogger routine.

1. []Read the public documentation for the routines called by the 2 first **call** instructions.

    >[!hint]Remove the \_imp\_ prefix.

1. []Using API documentation and the disassembly output, locate the path of the file where the keylogger writes data.

    >[!hint]Read the documentation for the called APIs. Use disassembly output to understand how data flows between memory and registers.  
    The first four integer or pointer parameters are passed in the rcx, rdx, r8, and r9 registers. 

---

**Evaluation**

1. What is the thread waiting for?

    Thread is waiting on a semaphore. The steps are not sufficient to ensure this is the same semaphore as the one signaled by the completion routine. But, we can make a guess here.

1. What are the first 2 Windows API called?

    API called are:
    * RtlInitUnicodeString : initializes a UNICODE_STRING struct.
    * ZwCreateFile : creates a new file or opens an existing file.

1. What is the full path of the file?

    `\DosDevices\C:\klog.txt`

---

You have completed this exercise. Congratulations.