![image](.\IMG\MSCyberCoursesResized.png)

# Threats targeting the hybrid & cloud identity platform

## Overview

Contoso is a big name and cannot risk being on the front page of all newspapers because of those stupid ransomware attacks that could have been easily prevented.
Thus, they have decided to create a strong security team. They hired two new employees <font style="color:red;">**Miss Red**</font> and <font style="color:blue;">**Mister Blue**</font> to respectively take care of the Red and Blue team.
The objective for Miss Red will be to test and detect vulnerable configuration and uncover unsecure practices. The objective for Mr Blue will be to implement proper monitoring and defense mitigations against the attacks detected by the red team. 

<details><summary>🖱️️ ️Never heard of a red/blue team? Click **here**.</summary>

A group of people authorized and organized to emulate a potential adversary’s attack or exploitation capabilities against an enterprise’s security posture. The <font style="color:red;">**Red Team**</font>’s objective is to improve enterprise Information Assurance by demonstrating the impacts of successful attacks and by demonstrating what works for the defenders (i.e., the Blue Team) in an operational environment. The group responsible for defending an enterprise’s use of information systems by maintaining its security posture against a group of mock attackers (i.e., the Red Team). Typically, the <font style="color:blue;">**Blue Team**</font> and its supporters must defend against real or simulated attacks […]
🔗 [Source](https://csrc.nist.gov/glossary/term/red_team_blue_team_approach)
</details> 

===

>[!ALERT] **DISCLAIMER**   
- Unless otherwise noted, the example companies, organizations, products, domain names, e-mail addresses, logos, people, places, and events depicted herein are fictitious, and no association with any real company, organization, product, domain name, e-mail address, logo, person, place or event is intended or should be inferred. Complying with all applicable copyright laws is the responsibility of the user. Without limiting the rights under copyright, no part of this document may be reproduced, stored in or introduced into a retrieval system, or transmitted in any form or by any means (electronic, mechanical, photocopying, recording, or otherwise), or for any purpose, without the express written permission of Microsoft Corporation.   
- Microsoft may have patents, patent applications, trademarks, copyrights, or other intellectual property rights covering subject matter in this document. Except as expressly provided in any written license agreement from Microsoft, the furnishing of this document does not give you any license to these patents, trademarks, copyrights, or other intellectual property.
- The names of manufacturers, products, or URLs are provided for informational purposes only and Microsoft makes no representations and warranties, either expressed, implied, or statutory, regarding these manufacturers or the use of the products with any Microsoft technologies. The inclusion of a manufacturer or product does not imply endorsement of Microsoft of the manufacturer or product. Links may be provided to third party sites. Such sites are not under the control of Microsoft and Microsoft is not responsible for the contents of any linked site or any link contained in a linked site, or any changes or updates to such sites. Microsoft is not responsible for webcasting or any other form of transmission received from any linked site. Microsoft is providing these links to you only as a convenience, and the inclusion of any link does not imply endorsement of Microsoft of the site or the products contained therein.

© 2022 Microsoft Corporation. All rights reserved.

Microsoft and the trademarks listed at <https://www.microsoft.com/en-us/legal/intellectualproperty/Trademarks/Usage/General.aspx> are trademarks of the Microsoft group of companies. All other trademarks are property of their respective owners.
===
# Reconnaissance 
All exercises and tasks must be done in the defined order. They build on each other, if you skip a step, you will no longer be able to continue.

In this lab, your role will alternate between a member of the <font style="color:red;">**Red Team**</font> and a member of the <font style="color:blue;">**Blue Team**</font>. This way you will be able to see what an attacker sees as well as how to configure the environment to mitigate the attacks. This lab will focus on the reconnaissance operations that an attacker can perform to gather intel about the environment. 

You mainly use two accounts:

|Red Team|Blue Team|
|:--------:|:--------:|
|Miss Red|Mister Blue|
|CONTOSO\red|CONTOSO\blue|

<font style="color:red;">**CONTOSO\red**</font> is only a user of **CONTOSO\Domain Users** and does not have any privilege on the domain. However, she is a member of the local **Administrators** group on **CLI01**.

<font style="color:blue;">**CONTOSO\blue**</font> is a domain privileged account member of the **Domain Admins** group.

===

## Exercise 1 - Prepare the environment
You will probably find the environment familiar as it is the continuity of the previous labs. We have the following machines:
- **DC01** a domain controller for contoso.com running Windows Server 2016 in the HQ Active Directory site
- **DC02** a domain controller for contoso.com running Windows Server 2022 in the Beijin Active Directory site
- **SRV01** a domain joined server member of the contoso.com domain running Windows Server 2022
- **CLI01** a domain joined client member of the contoso.com domain running Windows 11

### Task 1 - Create some activities on SRV01
1. [] Log on to **@lab.VirtualMachine(SRV01).SelectLink**. 
Use the following credentials:
    |||
    |:--------|:--------|
    | Username | `CONTOSO\lee.mendoza.adm` |
    | Password | Please type the password |
    

1. [] Right click on the Start menu ![image](.\IMG\2022menu.png) and click on **Run**.
1. [] In the **Run** window, type `\\DC01\SYSVOL` and click **OK**.

    📝 What types of files can we find in this folder?

This will open an explorer window and will maintain an SMB connection between SRV01 and DC01. Leave it open like this and carry on. 

### Task 2 - Create some activities from DC02
1. [] Log on to **@lab.VirtualMachine(DC02).SelectLink**. 
Use the following credentials:
    |||
    |:--------|:--------|
    | Username | `CONTOSO\v.fergusson.adm` |
    | Password | Please type the password |
    

1. [] Right click on the Start menu ![image](.\IMG\2022menu.png) and click on **Run**.
1. [] In the **Run** window, type `\\DC01\C$` and click **OK**.

This will open an explorer window and will maintain an SMB connection between DC02 and SRV01. Leave it open like this and carry on. 

===

## Exercise 2 - Use BloodHound for recon

Hello <font style="color:red;">**Miss Red**</font>! In this exercise you will use BloodHound and its collection script SharpHound to enumerate objects in Active Directory. Note that although BloodHound could be use for Azure AD recon, we will focus only on on-premises Active Directory in this exercise. 

### Task 1 - Run SharpHound

1. [] Log on to **@lab.VirtualMachine(CLI01).SelectLink**. Use the following credentials:
    |||
    |:--------|:--------|
    | Username | `CONTOSO\red` |
    | Password | Please type the password |
    

1. [] Right click on the Start menu ![image](.\IMG\11menu.png) and click on **Windows Terminal (Admin)**.

1. [] Change the current directory by typing `cd \Tools\Scripts` and hit **Enter**.

    >[!knowledge] **cd** is the DOS command to change directory, in PowerShell, **cd** is an alias for the **Set-Location** commandLet. You can type `Get-Alias cd` to confirm it.

    📝 ls is an alias for what PowerShell command?

1. [] Display the help menu for **SharpHound** by typing `.\sharphound.exe --?` and hit **Enter**.
    
    >[!knowledge] Get familiar with the available options. Specifically the **--collectionmethods**. Each of this collection method will bring a particular type of information. Not all methods are relevant depending of the type of recon you want to perfom. We'll get to that later.

1. [] Run **SharpHound** by executing `.\sharphound.exe --collectionmethods All --skippasswordcheck`

    THe collection will take a minute or two.
   
    >[!knowledge] The collection should take a minute or so as there are not a lot of objects in your environment. In a production environment, this can take hours and should be optimized for performance and stealth. 

1. [] Now we also want to enumerate as many **SRV01**. We are going to create a file that will contain our long list of computer to scan... Well, just one. Execute the following `Write-Output "SRV01.contoso.com" | Out-File computers.lst`

1. [] Run **SharpHound** again but this time by executing `.\sharphound.exe --collectionmethods All --computerfile computers.lst --skippasswordcheck`

1. [] You can see the collection files you have created by running the following command `dir *.zip`. 

    📝 How many zip files do you see?

### Task 2 - Run BloodHound

1. [] You are still connected to **@lab.VirtualMachine(CLI01).SelectLink**. Open the **File Explorer** and navigate to `C:\Tools\BloodHound`. Then double click on the **BloodHound.exe** icon.

1. [] You should see the BloodHound login prompt with the default bolt path `bolt://localhost:7687`. Use the following credentials:
    |||
    |:--------|:--------|
    | Neo4j Username | `neo4j` |
    | Neo4j Password | `NeverTrustAny1!` |

1. [] You are welcomed with a message telling you there's nothing in the database. Well, let's change that and import the zip files you have created. Click on the **Upload Data** button on the right side ![image](.\IMG\BH1.png). Navigate to the `C:\Tools\Script` folder, select the first zip file and click **Open**. The import should take a few seconds. 

1. [] Click on the **Upload Data** button again and select the second zip file and click **Open**. Then click the small white X next to the popup title **Upload Progress**.

1. [] Click the burger menu on the top left corner ![image](.\IMG\BH2.png) then click on the **Analysis** tab. Scroll down to the **Shortest Paths** section and click **Find Shortest Path do Domain Admins**. In the popup, click on **DOMAIN ADMIN@CONTOSO.COM**.

    You should see something like this:

    ![image](.\IMG\BH3.png)

    Spend some time exploring the graph. Note the different types of edges. Some of them were obtained by performing LDAP queries:
    - MemberOf
    - Contains
    - GenericAll
    - GPLink
    - ...
    
    Some others by SMB enumerations against the DCs
    - HasSession

    📝 Can you see who has a session and from where?

1. [] On the **Search for a node** on the top left corner, type `SRV01.CONTOSO.COM`. You should see a machine icon in the center of the window. Right click on it and select **Shortest Paths to Here**.

    You should see something like this:

    ![image](.\IMG\BH4.png)
    
    This time you can see another type of edge collected by a SAM-R enumeration:
    - CanRDP

    <details><summary>❓ Can you guess what it means? **Click here to see the answer**.</summary>
    In this case, it means that the user Abigail Storey is a member of the local group **Remote Desktop Users** on **SRV01**. 
    </details> 

Good job <font style="color:red;">**Miss Red**</font>! 
You are now sharing your findings with <font style="color:blue;">**Mister Blue**</font>.

===

## Exercise 3 - Enable LDAP logging

Hello <font style="color:blue;">**Mister Blue**</font>! You just received some nice screenshots of identified paths to get to your admins. That was clearly too easy to get that data. You are already working at reducing the number of domain admins to the strict minimum. And you will soon be deploying Microsoft Defender for Identity... But what can you do in the meantime to get some visibility on what is happening? Well, let's enable LDAP logging to see what it looks like.

In this Exercise you will enable logging on your domain controllers to start seeing when SharpHound is used.  

### Task 1 - Enable LDAP logging on DC01

1. [] Log on to **@lab.VirtualMachine(DC01).SelectLink**. Use the following credentials:
    |||
    |:--------|:--------|
    | Username | `CONTOSO\blue` |
    | Password | Please type the password |

1. [] Right click on the Start menu ![image](.\IMG\2022menu.png) and click on **Run**.

1. [] In the **Run** window, type `regedit` and click **OK**.

1. [] In the **Registry Editor**, navigate to **HKEY_LOCAL_MACHINE** > **SYSTEM** > **CurrentControlSet** > **Services** > **NTDS** > **Diagnostics**. Locate the value **15 Field Engineering**, double click on it, set it to `5` and click **OK**.

    📝 What is the default value of all diagnostics configurations? 

1. [] Now, navigate to **HKEY_LOCAL_MACHINE** > **SYSTEM** > **CurrentControlSet** > **Services** > **NTDS** > **Parameters**. In this location, create the following DWORD values:

    |Value|Type|Data|
    |:--------:|:--------:|:--------:|
    | `Expensive Search Results Threshold` |REG_DWORD| `1` |
    | `Inefficient Search Results Threshold` |REG_DWORD| `1` |
    | `Search Time Threshold (msecs)` |REG_DWORD| `1` |

    It should look like this:
    ![image](.\IMG\LDAPREG1.png)

1. [] Let's open the logs, right click on the Start menu ![image](.\IMG\2022menu.png) and click on **Event Viewer**.

1. [] In the **Event Viewer**, navigate **Event Viewer (Local)** > **Applications and Services Logs** > **Directory Service**.

Now we are going to ask <font style="color:red;">**Miss Red**</font> to run SharpHound again to see what we can log.

### Task 2 - Run SharpHound again

1. [] Go back to **@lab.VirtualMachine(CLI01).SelectLink**. You should still be logged in the following credentials:
    |||
    |:--------|:--------|
    | Username | `CONTOSO\red` |
    | Password | Please type the password |
    

1. [] If you closed the terminal before, reopen it. Right click on the Start menu ![image](.\IMG\11menu.png) and click on **Windows Terminal (Admin)**.

1. [] Make sure the current directory is `C:\Tools\Scripts`.

1. [] Run **SharpHound** by executing `.\sharphound.exe --collectionmethods All`.

### Task 3 - View the LDAP logs

1. [] Go back on to **@lab.VirtualMachine(DC01).SelectLink** with the following credentials:
    |||
    |:--------|:--------|
    | Username | `CONTOSO\blue` |
    | Password | Please type the password |

1. [] In the **Event Viewer**, you should still be in **Event Viewer (Local)** > **Applications and Services Logs** > **Directory Service**. In the **Actions** pane, click **Refresh**.

    You should see a lot of **1644** events corresponding to SharpHound execution (CLI01's IP address is 192.168.1.31). For example:
    ![image](.\IMG\1664.png)

    >[!knowledge] Although your lab is pretty quiet, you can see the ton of events it has generated. This is why this solution doesn't scale well on large environment and might affect the overall performance of domain controllers. You will need a SIEM to collect and parse this. Also note the maximum size of the Directory Service event log by default. Right click on the name of the eventlog in the console tree and click **Properties**. You see that the default size is **1,028Kb**. You should consider make it larger, probably something like 1Gb as long as you have enough space on the disk.

    You have configured it on DC01 only. SharpHound can very well use DC02. So to enable this type of logging consistently accross all domain controllers, it is better to use a group policy. We will skip that for now.
  
===

## Exercise 4 - Restrict SAM-R enumeration on a member server

Dear <font style="color:blue;">**Mister Blue**</font>. In the data <font style="color:red;">**Miss Red**</font> shared with you, you could see paths such as **CanRDP** to **SRV01**. This means that your member server is not protected against remote SAM enumeration. That's odd because **SRV01** is running Windows Server 2022 and by default only the members of the local adminstrators group can perform this type of enumeration and Red's account isn't a part of it. Maybe someone messed with the default configuration, that would not be the first time... Let's fix this!

### Task 1 - Confirm local group membership

1. [] Log on to **@lab.VirtualMachine(SRV01).SelectLink**. Use the following credentials:
    |||
    |:--------|:--------|
    | Username | `CONTOSO\blue` |
    | Password | Please type the password |

    You can use the **Switch User** button on the bottom if necessary. 

1. [] Right click on the Start menu ![image](.\IMG\2022menu.png) and click on **Run**.

1. [] In the **Run** window, type `cmd` and click **OK**.

1. [] In the command prompt window, type `net localgroup "Remote Desktop Users"` and hit **Enter**.

    📝 Who is a member of the group? 

### Task 2 - Correct SRV01 SAM-R configuration 

1. [] Still on **@lab.VirtualMachine(SRV01).SelectLink**, right click on the Start menu ![image](.\IMG\2022menu.png) and click on **Run**.

1. [] In the **Run** window, type `gpedit.msc` and click **OK**.

1. [] In the **Local Group Policy Editor** window, navigate to **Local Computer Policy** > **Computer Configuration** > **Windows Settings** > **Security Settings** > **Local Policies** > **Security Options**. Double click on the **Policy** called **Network access: Restrict clients allowed to make remote calls to SAM**. Click on **Edit Security...** and note the current security principals.

    This is not the default configuration for Windows Server 2022. Starting Windows Server 2016, only the local Administrators group should be here (when the setting isn't configured, the default applies). 

1. [] Select **Authenticated Users**, then click **Remove** and **OK**.

    >[!knowledge] It might be a good idea to consider using a group policy to enforce the default settings.

    📝 Why is using a group policy recommended to use a group policy for these settings?

===

## Exercise 5 - Enumerate domain users and group anonymously
Hello <font style="color:red;">**Miss Red**</font>! Your collegue and almost nemessis by now <font style="color:blue;">**Mister Blue**</font> is hardening the environment based on your input. Let's show him thatou can do even better )or worse) than BloodHound and let's gather intel without using an account.

### Task 1 - Use anonymous SAM-R enumeration

1. [] Log on to **@lab.VirtualMachine(CLI01).SelectLink**. Use the following credentials:
    |||
    |:--------|:--------|
    | Username | `CONTOSO\red` |
    | Password | Please type the password |
    
1. [] If you don't have a **Windows Terminal** already opened, open a new one by right clicking on the Start menu ![image](.\IMG\11menu.png) and clicking on **Windows Terminal (Admin)**. Else you can use an existing one.

1. [] In the terminal, open a new tab by clicking on the chevron and selecting **Command Prompt** like shown in this screenshot:  ![image](.\IMG\TERM1.png)

1. [] In this new tab, run the following command `nmap --script smb-enum-users -p 445 DC01`. This is essentially doing a SAM-R call and as you can see, we did not specify a user.

    >[!knowledge] You can run the same command with `-d` to use the debug mode and see in the ouput that we did not authenticate.

You send the output to <font style="color:blue;">**Mister Blue**</font> with a nice encouragement message "Do better".

### Task 2 - Block anonymous SAM-R enumeration

1. [] Log on to **@lab.VirtualMachine(DC01).SelectLink**. Use the following credentials:
    |||
    |:--------|:--------|
    | Username | `CONTOSO\blue` |
    | Password | Please type the password |

1. [] Right click on the Start menu ![image](.\IMG\2022menu.png) and click on **Run**.

1. [] In the **Run** window, type `dsa.msc` and click **OK**.

1. [] In the **Active Directory Users and Computers** window, navigate to **contoso.com** > **Builtin** and double click on the group **Pre-Windows 2000 Compatible Access**. Select the **Members** tab.

    >[!knowledge] The presence of the security principal **ANONYMOUS LOGON** is the reason why Red's SAM-R enumeration worked without authentication.

   📝 Who else is a member of this group in the lab?

1. [] Select **ANONYMOUS LOGON**, click **Remove** and **OK**. If there is a confirmation popup, confirm by clicking **Yes**. 

1. [] Right click on the Start menu ![image](.\IMG\2022menu.png) and click on **Run**.

1. [] In the **Run** window, type `services.msc` and click **OK**.

1. [] In the **Services** console, right click on the **Server** service and click **Restart**. A pop-up will ask you if you want to restart the dependencies, click **Yes**.

### Task 3 - Check that anonymous SAM-R enumeration is disabled

1. [] Go back to **@lab.VirtualMachine(CLI01).SelectLink** using the following credentials:
    |||
    |:--------|:--------|
    | Username | `CONTOSO\red` |
    | Password | Please type the password |
    
1. [] In the command prompt tab, run the following command `nmap --script smb-enum-users -p 445 DC01 -d`. 

    📝 What is the error message?

===

## Exercise 6 - Restrict SMB enumeration [optional]

My dear <font style="color:blue;">**Mister Blue**</font>. In the data <font style="color:red;">**Miss Red**</font> shared with you, you could see paths such as **HasSession** which were telling you where users were connected from. This was made possible because of SMB enumerations. Well good news, that's an easy fix.

### Task 1 - Check SMB enumeration again

SMB enumeration has very volatile output. Users and machines aren't always connected to domain controllers and attackers have to run it multiple times. Last time <font style="color:red;">**Miss Red**</font> ran it as a part of SharpHound. This time, let's isolate the test.

1. [] Log on to **@lab.VirtualMachine(CLI01).SelectLink**. Use the following credentials:
    |||
    |:--------|:--------|
    | Username | `CONTOSO\red` |
    | Password | Please type the password |


1. [] If you don't have a **Windows Terminal** already opened, open a new one by right clicking on the Start menu ![image](.\IMG\11menu.png) and clicking on **Windows Terminal (Admin)**. Else you can use an existing one.

1. [] In the terminal, make sure you are in a PowerShell tab and in the directory **C:\Tools\Scripts**. Run the following script `.\Invoke-NetSessionEnum.ps1 -Hostname DC01`
    The output should look like this:
    ![image](.\IMG\NETSESSION1.png)
    If you do not see Lee Mendoza's connection, you can log back in **@lab.VirtualMachine(SRV01).SelectLink** with Lee's account and refresh the Explorer window connected to `\\DC01\SYSVOL`.

### Task 2 - Restrict SMB Enumeration with NetCease

Now that you have the confirmation it is open, let's restrict it.

1. [] Log on to **@lab.VirtualMachine(DC01).SelectLink**. Use the following credentials:
    |||
    |:--------|:--------|
    | Username | `CONTOSO\blue` |
    | Password | Please type the password |

1. [] Right click on the Start menu ![image](.\IMG\2022menu.png) and click on **Run**.

1. [] In the **Run** window, type `powershell` and click **OK**.

1. [] In the PowerShell prompt, run the following cmdLet `Get-Command -Module NetCease` and note the output.

    >[!knowledge] The module was pre-installed in the lab. If you want to install it on another machine, you can use the cmdLet `Install-Module NetCease`.

1. [] In the same prompt, run the following cmdLet `Get-NetSessionEnumPermission | Out-GridView -Title "SMB permissions"` and note the output. You should see the **NT AUTHORITY\Authentication Users** security principal. That explains why Red's account, although she isn't a privileged account can enumerate sessions. You need to remove **Authenticated Users**. Close the **SMB Permissions** window.

    📝 Try to run the previous command without the "| Out-GridView -Title "SMB permissions". What is the difference?

1. [] In the same prompt, run the following cmdLet `Set-NetSessionEnumPermission` and then run `Get-NetSessionEnumPermission | Out-GridView -Title "SMB permissions"`. You should see the difference.
>Note: If you have an error maybe it's a problem with how you execute the PowerShell! Try to run it as administrator.

1. [] Right click on the Start menu ![image](.\IMG\2022menu.png) and click on **Run**.

1. [] In the **Run** window, type `services.msc` and click **OK**.

1. [] In the **Services** console, right click on the **Server** service and click **Restart**. A pop-up will ask you if you want to restart the dependencies, click **Yes**.

    >[!knowledge] In a production environment, you can't just restart those services at any time, you'll need to plan that carefully to avoid application outages. 


### Task 3 - Check SMB enumeration one last time

1. [] Log back on to **@lab.VirtualMachine(CLI01).SelectLink** using the following credentials:
    |||
    |:--------|:--------|
    | Username | `CONTOSO\red` |
    | Password | Please type the password |

1. [] In the terminal, run the following script `.\Invoke-NetSessionEnum.ps1 -Hostname DC01`. 

    📝 Do you still see connections?

===

Good job to you two <font style="color:red;">**Miss Red**</font> and <font style="color:blue;">**Mister Blue**</font>. Your efforts allowed Contoso's environment to be more resistant to reconnaissance actions.

Remember that in an assumed breach world, you need to attack the economic model of the malicious actors! Making recon noisy and slow it definitely going to help!

🍾 Congratulations! You have completed the labs for this chapter.

**Make sure you have noted all the answers to the questions marked with 📝.** 