![MSCyberCoursesResized.png](lab3/MSCyberCoursesResized.png)

<!-- TOC -->
# LAB 3: Auditing
## Abstract and learning objectives  

This training is designed to make you practice the concepts learned in the lectures.  
Learning objectives:  
- Implement basic auditing scenarios on logons, resources and privilege use

## Overview 

This lab is a very simple environment consisting in a Windows 11 client and Windows Server 2022 servers. Both are members of an Active Directory domain northwindtraders.com. The server provides different services to support lab exercises. 

>[!ALERT] **DISCLAIMER**   
- Unless otherwise noted, the example companies, organizations, products, domain names, e-mail addresses, logos, people, places, and events depicted herein are fictitious, and no association with any real company, organization, product, domain name, e-mail address, logo, person, place or event is intended or should be inferred. Complying with all applicable copyright laws is the responsibility of the user. Without limiting the rights under copyright, no part of this document may be reproduced, stored in or introduced into a retrieval system, or transmitted in any form or by any means (electronic, mechanical, photocopying, recording, or otherwise), or for any purpose, without the express written permission of Microsoft Corporation.   
- Microsoft may have patents, patent applications, trademarks, copyrights, or other intellectual property rights covering subject matter in this document. Except as expressly provided in any written license agreement from Microsoft, the furnishing of this document does not give you any license to these patents, trademarks, copyrights, or other intellectual property.
- The names of manufacturers, products, or URLs are provided for informational purposes only and Microsoft makes no representations and warranties, either expressed, implied, or statutory, regarding these manufacturers or the use of the products with any Microsoft technologies. The inclusion of a manufacturer or product does not imply endorsement of Microsoft of the manufacturer or product. Links may be provided to third party sites. Such sites are not under the control of Microsoft and Microsoft is not responsible for the contents of any linked site or any link contained in a linked site, or any changes or updates to such sites. Microsoft is not responsible for webcasting or any other form of transmission received from any linked site. Microsoft is providing these links to you only as a convenience, and the inclusion of any link does not imply endorsement of Microsoft of the site or the products contained therein.

© 2022 Microsoft Corporation. All rights reserved.

Microsoft and the trademarks listed at <https://www.microsoft.com/en-us/legal/intellectualproperty/Trademarks/Usage/General.aspx> are trademarks of the Microsoft group of companies. All other trademarks are property of their respective owners.

===

## Exercise 1: Configure auditing

Duration: 15 minutes

Synopsis: In this exercise, you will train yourself in implementing basic auditing scenarios as well as deploying an audit policy.

---

### Task 1: Implement Logon auditing

In this task, you will implement auditing of user logon on server **WIN-SRV1**. You will be using the file sharing services on the machine to trigger some signing events. When a user authenticates to a file share, it authenticates to the server as a Type-3 (Network) logon.

For this first exercise, you will only enable auditing locally, using the `auditpol.exe` command.

1. []Sign in **@lab.VirtualMachine(WIN-SRV1).SelectLink** with following credentials:  
	Username: **+++NORTHWIND\Administrator+++**   
	Password: Please type the password

1. []Right-click the Start menu and select **Windows PowerShell (admin)**

1. []Using the course material and puiblic documentation, find and run the `auditpol.exe` command to enable auditing of Logon events.

1. []In order to end existing file sharing sessions, restart the Server service using the command:

	```Powershell
	Restart-Service Server -Force
	```

1. []Sign in **@lab.VirtualMachine(WIN-CLI1).SelectLink** with following credentials:  
	Username: **+++NORTHWIND\david+++**  
	Password: Please type the password 

1. []Use the file explorer to access `\\WIN-SRV1\Share\Users`

1. []Go back to **@lab.VirtualMachine(WIN-SRV1).SelectLink** and open the event viewer

	>[!hint] the event viewer can easily be opened by right clicking the start menu and selecting Event Viewer.

1. []Open the Security event log in **Event Viewer** \ **Windows Logs** \ **Security**

1. []In the security event log, find the security audit event corresponding to the network logon.

	>[!hint] Start by filtering the security event log on the correct event ID then, search for correct Logon type (3) and user's name (NORTHWIND\david).

---

**Evaluation**

1. Copy-paste the auditpol.exe command you ran to enable auditing.

	```Console-nocopy
	auditpol /set /subcategory:Logon /success:enable  
	```

1. Copy-paste the text or a screenshot of the logon event. If there are multiple events for the same user, just capture the first one.

	Event 4624 with logon type 3

1. In this example, you used a network connection (Logon type 3) to exercise the audit subsystem. But there are other types of logon depending how the user authenticates. Using the courser material or public documentation, find the other types of logon.

	| Type 				| Value |
	| ----------------- | -----	|
	| INTERACTIVE		| 2 |
	| NETWORK           | 3 |  
	| BATCH             | 4 |  
	| SERVICE           | 5 |  
	| UNLOCK            | 7 |  
	| NETWORK_CLEARTEXT | 8 |  
	| NEW_CREDENTIALS   | 9 |  

---

You have completed the logon auditing task. You can now move on to auditing resource access.

===

### Task 2: Implement File Access auditing

In this task, you will enable auditing on files and folders. For the purpose of the lab, you will be using the folder hierarchy on **WIN-SRV1** located at `C:\Share`

1. []Sign in **@lab.VirtualMachine(WIN-SRV1).SelectLink** with following credentials   
	Username: **+++NORTHWIND\Administrator+++**   
	Password: Please type the password

1. []Right-click the **Start** menu and select **Windows PowerShell (admin)**

1. []Using the course material and public documentation,

	1. []Find the `auditpol.exe` command to enable auditing of Filesystem events and run it.
	1. []Enable auditing of successful write accesses on `C:\Share\Projects\Project 1\Important Text File.txt`

3. []Modify **Important Text File.txt** contained in the **C:\Share\Projects\Project 1** folder to generate an audit event

---

**Evaluation**

1. Copy-paste the auditpol.exe command you ran to enable auditing.

	`auditpol /set /subcategory:"File System" /success:enable`  
	Or, also acceptable  
	`auditpol /set /subcategory:"File System" /success:enable /failure:enable`

1. Capture a screenshot of the audit permission you set on the `Important Text File.txt` file. The capture must clearly display the permissions you are auditing.

	1. Using file explorer, open the folder **C:\Share\Projects**
	1. Right click folder **Project 1** and select **Properties**
	1. Select the **Security** Tab and click **Advanced**
	1. Select the **Auditing** tab
	1. Click the **Add** button
	1. Click **Select a principal**, type **Everyone** in the text box and click **OK**
	1. In the **Basic permissions** list field, tick the **Write** checkbox and **clear** the other checkboxes.
	1. Click **OK**
	1. In the **Advanced Security Settings** Window, click **Apply** and **OK**

1. Copy-paste the text or screenshot of the security event corresponding to the modification you performed on the file.

	A 4663 event will be written in the Security event log.   From that 4663 event, you can tell who modified the file, what operation was performed, which resource was modified and which process did it.

	```
	An attempt was made to access an object.

	Subject:  
		Security ID:		NORTHWIND\Administrator  
		Account Name:		Administrator  
		Account Domain:		NORTHWIND  
		Logon ID:		0xD0F0B  

	Object:  
		Object Server:		Security  
		Object Type:		File  
		Object Name:		C:\Share\Projects\Project 1\Important Text File.txt  
		Handle ID:		0x570  
		Resource Attributes:	S:AI  

	Process Information:  
		Process ID:		0x1bec  
		Process Name:		C:\Windows\System32\notepad.exe  

	Access Request Information:  
		Accesses:		WriteData (or AddFile)  
					AppendData (or AddSubdirectory or CreatePipeInstance)  
		Access Mask:		0x6 
	``` 

---

You have completed the file system auditing task. You can now move on to auditing registry access.

### Task 3: Enable Registry auditing

In this task, you will learn how to enable audit registry keys and values accesses. You will use the **WIN-SRV1** machine.

1. []Sign in **@lab.VirtualMachine(WIN-SRV1).SelectLink** with following credentials   
	Username: **+++NORTHWIND\Administrator+++**   
	Password: Please type the password

1. []Open registry editor by launching `regedit.exe`

1. []Create this registry key: **HKEY_LOCAL_MACHINE\SOFTWARE\Test**

1. []Right-click the **Start** menu and select **Windows PowerShell (admin)**

1. []Using the course material and public documentation,

	1. []Prepare an `auditpol.exe` command to enable auditing of Registry events and run it.
	1. []Enable auditing of successful registry value modification in `HKEY_LOCAL_MACHINE\SOFTWARE\Test`

1. []In the **HKEY_LOCAL_MACHINE\SOFTWARE\Test** key, create a new DWORD value (you can choose any name) and set a numerical value on it.

---

**Evaluation**

1. Copy-paste the auditpol.exe command you ran to enable auditing.

	`auditpol /set /subcategory:"Registry" /success:enable`  
	Or, also acceptable  
	`auditpol /set /subcategory:"Registry" /success:enable /failure:enable`

1. Capture a screenshot of the audit permission you set on the `Test` registry key. The capture must clearly display the permissions you are auditing.

	1. Right click **Test** key and select **Permissions**
	1. In the **Security** Tab, click **Advanced**
	1. Select the **Auditing** tab
	1. Click the **Add** button
	1. Click **Select a principal**, type **Everyone** in the text box and click **OK**
	1. Click on **Show basic permissions**
	1. In the **Advanced permissions** list field, tick the **Set Value** checkbox and clear the other checkboxes.
	1. Click **OK**
	1. In the **Advanced Security Settings** Window, click **Apply** and **OK**

1. Copy-paste the text or screenshot of the security event corresponding to the modification of the registry value.

	You can observe a 4663 event generated during the value assignment.

	```
	An attempt was made to access an object.

	Subject:  
		Security ID:		NORTHWIND\Administrator  
		Account Name:		Administrator  
		Account Domain:		NORTHWIND  
		Logon ID:		0xD0F0B  

	Object:  
		Object Server:		Security  
		Object Type:		Key  
		Object Name:		\REGISTRY\MACHINE\SOFTWARE\Test  
		Handle ID:		0x5bc  
		Resource Attributes:	-  

	Process Information:  
		Process ID:		0x1374  
		Process Name:		C:\Windows\regedit.exe  

	Access Request Information:  
		Accesses:		Set key value  
					
		Access Mask:		0x2
	```

	Also, a 4657 event which is written when a value is modified. This event type helps keeping track of previous and new value.

	```
	Subject:  
		Security ID:		NORTHWIND\Administrator  
		Account Name:		Administrator  
		Account Domain:		NORTHWIND  
		Logon ID:		0xD0F0B  

	Object:  
		Object Name:		\REGISTRY\MACHINE\SOFTWARE\Test  
		Object Value Name:	test  
		Handle ID:		0x5bc  
		Operation Type:		Existing registry value modified  

	Process Information:  
		Process ID:		0x1374  
		Process Name:		C:\Windows\regedit.exe  

	Change Information:  
		Old Value Type:		REG_SZ  
		Old Value:		  
		New Value Type:		REG_SZ  
		New Value:		rrrr  
	```

---

You have completed this exercise. Congratulations!

===

## Exercise 2: Create an auditing GPO

The common practice for defining an audit policy is to configure a domain GPO - _Group Policy Objects_. 

If you are not familiar with GPOs, the topic is covered more in the Hybrid Identity course. For this exercise, you just have to understand that a GPO is a collection of configuration settings which is forcibly applied on machines which are member of an ActiveDirectory domain.

In this Exerciser, you will learn how auditing settings can be deployed using a GPO.

### Task 1: Create the auditing GPO

In this task, you will create the auditing GPO on the domain controller of your lab, **WIN-DC1**

1. []Sign in **@lab.VirtualMachine(WIN-DC1).SelectLink** with following credentials   
	Username: **+++NORTHWIND\Administrator+++**   
	Password: Please type the password

1. []Open the Group policy management console (From the **Server Manager** application, select the **Tools** menu, then **Group Policy Management**)

1. []In the **Group Policy Management** console, unfold the nodes: **Group Policy Management\Forest:northwindtraders.com\Domains\northwindtraders.com\Group Policy Objects**

1. []Right click **Group Policy Objects** and select **New**

1. []Give a name to your GPO. For instance, **Domain Auditing Policy**

1. []Right click the new GPO and select **Edit**

1. []In the Group Policy Management Editor console, expand **Computer Configuration\Policies\Windows Settings\Security Settings\Advanced Audit Policy Configuration\Audit policies**

1. []Using the course material and public documentation, enable auditing of sensitive privilege use for success and failures.

1. []WHen done, close the **Group Policy Management Editor**

1. []Link your GPO to the domain.

	1. From the **Group Policy Management** console, right click the **northwindtraders.com** node and select **Link an existing GPO**.
	1. Choose your auditing GPO and click **OK**.

--- 

**Evaluation**

1. Explain how you enabled the auditing of privilege use or, copy-paste a screenshot of the auditing settinng. In the latter case, the specific subcategory must be clearly visible with its settings.

	1. Open the **Privilege Use** Category
	1. Double-click on **Audit Sensitive Privilege Use**. Tick the **Configure the following audit events**. Then tick **failure**  and  **success** boxes.
	1. Click **Apply** and **OK** to close the dialog.

1. The Windows audit subsystem makes a difference between sensitive and non-sensitive privileges. List all privileges which are considered sensitive.

	- Act as part of the operating system.
	- Back up files and directories.
	- Create a token object.
	- Debug programs.
	- Enable computer and user accounts to be trusted for delegation.
	- Generate security audits.
	- Impersonate a client after authentication.
	- Load and unload device drivers.
	- Manage auditing and security log.
	- Modify firmware environment values.
	- Replace a process-level token.
	- Restore files and directories.
	- Take ownership of files or other objects.
---

You have completed the GPO creation task. You can now move on to exercise it.

===

### Task 2: Exercise the audit policy

In this task, you will exercise the audit policy you have just created by leveraging one the sensitive privilege.

1. []Sign in **@lab.VirtualMachine(WIN-SRV1).SelectLink** with following credentials   
	Username: **+++NORTHWIND\Administrator+++**   
	Password: Please type the password

1. []Right-click the **Start** menu and select **Windows PowerShell (admin)**

1. []Run **gpupdate** to ensure GPO gets refreshed

1. []Run **takeown.exe /F C:\Windows\notepad.exe**

---

**Evaluation**

1. Which privilege has been activated by the `takeown.exe` command?

	Activated privilege is **SeTakeOwnershipPrivilege**

1. Copy-paste the text or capture a screenshot of the security event corresponding to the use of the privilege by the `takeown.exe` command.

	You can observe a 4674 event is written in the Security event logs.
	```
	An operation was attempted on a privileged object.

	Subject:  
		Security ID:		NORTHWIND\Administrator  
		Account Name:		Administrator  
		Account Domain:		NORTHWIND  
		Logon ID:		0xD0F0B  

	Object:  
		Object Server:	Security  
		Object Type:	File  
		Object Name:	C:\Windows\notepad.exe  
		Object Handle:	0x114  

	Process Information:  
		Process ID:	0x1894  
		Process Name:	C:\Windows\System32\takeown.exe  

	Requested Operation:  
		Desired Access:	WRITE_OWNER  
		Privileges:		SeTakeOwnershipPrivilege
	```

---

You have completed this exercise. Congratulations!