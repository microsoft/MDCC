![image](.\IMG\MSCyberCoursesResized.png)MSCyberCoursesResized.png)

# The foundations of hybrid identity - Active Directory Services

>Last Update September 2022

# Contents

<!-- TOC -->

- [Requirements](#requirements)
- [Overview](#overview)
- [Hands-on lab step-by-step](#hands-on-lab-step-by-step)
  - [Abstract and learning objectives](#abstract-and-learning-objectives)
  - [Exercise 1: Manage access without Active Directory](#exercise-1-manage-access-without-active-directory)
        - [Task 1: Open remote share folder](#task-1-open-remote-share-folder)
	    - [Task 2: Reach remote public folder on DC01](#task-2-reach-remote-public-folder-on-DC01)
  - [Exercise 2: Add a new computer to the domain](#exercise-2-add-a-new-computer-to-the-domain)
        - [Task 1: Add CLI01 to the Domain Contoso.com](#task-1-add-CLI01-to-the-domain-contosocom)
        - [Task 2: Reach remote public folder on DC01](#task-2-reach-remote-public-folder-on-DC01-1)
  - [Exercise 3: Discovery of the environment](#exercise-3-discovery-of-the-environment)
        - [Task 1: Create a new OU](#task-1-create-a-new-ou)
        - [Task 2: Create new users](#task-2-create-new-users)
        - [Task 3: Create a new group](#task-3-create-a-new-group)
        - [Task 4: Enumerate & find objects in ADDS](#task-4-enumerate--find-objects-in-adds)
        - [Task 5: Experiment LSASS Kill](#task-5-experiment-lsass-kill)
  - [Exercise 4: Password Policy vs FGPP](#exercise-4-password-policy-vs-fgpp)
        - [Task 1: Configure FGPP](#task-1-configure-fgpp)
        - [Task 2: Verify that the Admin-FGPP is applied to Administrators](#task-2-verify-that-the-admin-fgpp-is-applied-to-administrators)
  - [(OPTIONAL) Exercise 5: Replication concept](#exercise-5-replication-concept)
        - [Task 1: Show Metadata for a user](#task-1-show-metadata-for-a-user)
        - [Task 2: Show Metadata for PAR_User2](#task-2-show-metadata-for-paruser2)
  - [(OPTIONAL) Exercise 6: RBAC](#exercise-5-rbac)
        - [Task 1: Run AD ACL Scanner on OU Paris](#task-1-run-ad-acl-scanner-on-ou-paris)
        - [Task 2: Delegate Management of an OU to a group of users](#task-2-delegate-management-of-an-ou-to-a-group-of-users)
        - [Task 3: Compare Result of delegation with AD ACL Scanner on OU Paris](#task-3-compare-result-of-delegation-with-ad-acl-scanner-on-ou-paris)
        - [Task 4 Manage OU](#task-4-manage-ou)
        - [Task 5: Change the Frequency for the execution of the AdminSDHolder](#task-5-change-the-frequency-for-the-execution-of-the-adminsdholder)
        - [Task 6: Review the AdminSDHolder’s permissions](#task-6-review-the-adminsdholders-permissions)
  - [(OPTIONAL) Exercise 7: Allow only Administrators to logon CLI01](#optional-exercise-7-allow-only-administrators-to-logon-cli01)
        - [Task 1: Create a GPO](#task-1-create-a-gpo)
        - [Task 2: Import the Windows 11 Security Baseline](#task-2-import-the-windows-11-security-baseline)
        - [Task 3: Link the GPO to "deploy" it on client](#task-3-link-the-gpo-to-deploy-it-on-client)
        - [Task 4: Test the ](#task-4-test-the-gpo)

<!-- /TOC -->

>[!ALERT] **DISCLAIMER**   
- Unless otherwise noted, the example companies, organizations, products, domain names, e-mail addresses, logos, people, places, and events depicted herein are fictitious, and no association with any real company, organization, product, domain name, e-mail address, logo, person, place or event is intended or should be inferred. Complying with all applicable copyright laws is the responsibility of the user. Without limiting the rights under copyright, no part of this document may be reproduced, stored in or introduced into a retrieval system, or transmitted in any form or by any means (electronic, mechanical, photocopying, recording, or otherwise), or for any purpose, without the express written permission of Microsoft Corporation.   
- Microsoft may have patents, patent applications, trademarks, copyrights, or other intellectual property rights covering subject matter in this document. Except as expressly provided in any written license agreement from Microsoft, the furnishing of this document does not give you any license to these patents, trademarks, copyrights, or other intellectual property.
- The names of manufacturers, products, or URLs are provided for informational purposes only and Microsoft makes no representations and warranties, either expressed, implied, or statutory, regarding these manufacturers or the use of the products with any Microsoft technologies. The inclusion of a manufacturer or product does not imply endorsement of Microsoft of the manufacturer or product. Links may be provided to third party sites. Such sites are not under the control of Microsoft and Microsoft is not responsible for the contents of any linked site or any link contained in a linked site, or any changes or updates to such sites. Microsoft is not responsible for webcasting or any other form of transmission received from any linked site. Microsoft is providing these links to you only as a convenience, and the inclusion of any link does not imply endorsement of Microsoft of the site or the products contained therein.

© 2022 Microsoft Corporation. All rights reserved.

Microsoft and the trademarks listed at <https://www.microsoft.com/en-us/legal/intellectualproperty/Trademarks/Usage/General.aspx> are trademarks of the Microsoft group of companies. All other trademarks are property of their respective owners.

===

# Requirements
**Attendee’s machine**:
1. [] Ideal resolution 1920 x 1080 
2. [] An Internet browser
3. [] An internet access without restriction on outbound connections.
4. [] The following TCP port must be accessible : 
    - TCP/80 and TCP/443 to reach Lab On Demand

===

# Overview

Contoso has asked you to integrate their on-premises Active Directory single-domain forest named contoso.com with Azure AD and implement all necessary prerequisites to allow them to benefit from such Azure AD features as:
- Single sign-on to cloud and on-premises applications,
- Enhanced sign-in security with Multi-Factor Authentication
- automatic enrollment of Windows 10 devices into Microsoft Intune

They want to also provide secure access to their on-premises, Windows Integrated Authentication-based applications from Internet for both organizational users and users who are members of partner organizations, although they also want to be able to loosen restrictions when access originates from Hybrid Azure AD joined computers residing in their on-premises data centers. The same applications also need to be made available to Contoso's business partners. 

In this Lab, the attendees will manage Active Directory using the graphical interfaces and PowerShell. The attendees will also use basic features of AD DS such as GPO and manipulate objects using the default administrative tools.

# Hands-on lab step-by-step 

## Abstract and learning objectives 

In this hands-on lab you will setup and configure a number of different Active Directory aspects. The scenarios involve an Active Directory single-domain forest named contoso.com, which in this lab environment, consists (for simplicity reasons) of :
- Two domain controller named DC01 & DC02
- A single domain member server named SRV01
- A single workstation named CLI01

The intention is to explore Active Directory related capabilities that allow you to know how ADDS works and to understand authentication and authorization.

## Exercise 1: Manage access without Active Directory
**Duration**: 10 minutes  
**Synopsis**: 
In this exercise, you will try to reach a remote folder to DC01 to experiment difference between local user & domain user

### Task 1: Open remote share folder
1. [] Log on to **@lab.VirtualMachine(CLI01).SelectLink**.  
Use the following credentials:
    |||
    |:--------|:--------|
    | Username | **+++.\David+++** |
    | Password | Please type the password |
    
	>[!TIP] Click the **+++Type Text+++** icon to enter the associated text into the virtual machine.

### Task 2: Reach remote public folder on DC01
1. [] In Explorer open the remote Share Folder **+++\\\SRV01.contoso.com\public+++**   
    >[!ALERT] (Don't try to enter credential)  

    **Questions**:
    - What happened ?   
    ..................................................................
    - Why do you have this Behavior ?  
    ..................................................................

===
## Exercise 2: Add a new computer to the domain
**Duration**: 15 minutes

**Synopsis**: 
In this exercise, you will join new computers to the domain & re-try to open the public folder on DC01

### Task 1: Add CLI01 to the Domain Contoso.com
1. [] Log on to **@lab.VirtualMachine(CLI01).SelectLink**
   

    |||
    |:--------|:--------|
    | Username | **+++.\David+++** |
    | Password | Please type the password |
    
	>[!TIP] Click the **+++Type Text+++** icon to enter the associated text into the virtual machine.

2. [] Join CLI01  to the domain Contoso.com by using the command Powershell **with Administrator Rights**:
+++Add-Computer+++

<details><summary>➡️**Note**: You must find the right commande. it's not complicated, see a hint</summary>
Cmdlet: [add-computer](https://docs.microsoft.com/en-us/powershell/module/microsoft.powershell.management/add-computer?view=powershell-5.1).
    
|Parameter field|Parameter value|
|--|--|
|-DomainCredential|Domain Admins Cred|
|-DomainName|contoso|
|-Force||
|-Restart||  

</details>  


> Use **Administrator / NeverTrustAny1!** as admin account to add the Workstation to the domain

3. Reboot the computer

### Task 2: Reach remote public folder on DC01
1. [] Log on to **@lab.VirtualMachine(CLI01).SelectLink**
    |||
    |:--------|:--------|
    | Username | **+++David@contoso.com+++** |
    | Password | Please type the password |
    
	 >[!ALERT] Select Other User, not David. David is the local account  

2. In Explorater open the remote Share Folder **+++\\\SRV01.contoso.com\public+++**

    **Questions**:
    - What happened ?  
    ..................................................................
    - How many files do you see ?   
    ..................................................................
    - Why do you have this Behavior ?  
    ..................................................................

===
## Exercise 3: Discovery of the environment
**Duration**: 20 minutes  
**Synopsis**:  
In this exercise, attendees will discover the basic management of Active Directory by creating organizational unit, users, groups.

### Task 1: Create a new OU
1. [] Log on to **@lab.VirtualMachine(DC01).SelectLink**
    |||
    |:--------|:--------|
    | Username | **+++Administrator@contoso.com+++** |
    | Password | Please type the password |
    
	>[!TIP] Click the **+++Type Text+++** icon to enter the associated text into the virtual machine.


2. [] Click on Windows button on the taskbar, click on Windows Administrative Tools and choose Active Directory Administrative Center
3. [] Create the following  Organizational Unit structure:

    |||
    |:--------|:--------|
    | Paris ||
    ||Users|
    ||Groups|
    ||Computers|
    |IT||
    ||Users|
    ||Groups|
    ||Computers|

### Task 2: Create new users
1. [] Under the container Users, create a new user with **YourInitials-Adm** and add it to the “**Domain Admins**” group
    |||
    |:--------|:--------|
    | Full name  | **(ADM) Your full Name** |
    | User logon name | yourInitials-Adm |
    | Password | Please type the password |
  
    >Select **Other password options**
    ![image](.\IMG\oepisy9d.jpg)

2. [] Close the session and open the session with the your Domain Admins account
3. [] Under the OU **Paris/Users**, create 5 users : 
    |||
    |:--------|:--------|
    | | +++PAR_User1+++ |
    | | +++PAR_User2+++ |
    | | +++PAR_User3+++ |
    | | +++PAR_User4+++ |
    | | +++PAR_User5+++ |  

    - Password: +++1LoveSecurity!+++
    >Select **Other password options**
    ![image](.\IMG\oepisy9d.jpg)
    
>[!tip]*Note that you can technically create an account without setting a password. But the account will have to be disabled and you will need to set a password to enable the account.*

4. [] Add **PAR_User1** to the group **Account Operators**
    - open the user and select **member of** tab

### Task 3: Create a new group

1. [] Under the OU Paris/Group, create a group named : +++Paris Prod+++
2. [] Add All PAR_ Users to **Paris Prod**
2. [] Click OK

>*All actions performed with the console can be done with PowerShell.*

### Task 4: Enumerate & find objects in ADDS
1. [] Open a Powershell CLI **with Administrator Rights**
2. [] Use the Powershell command **+++Get-ADUser+++** to list all users in this ADDS
<details><summary>➡️**Note**: You must find the right commande. it's not complicated, see a hint</summary>
Cmdlet: [Get-ADUser](https://docs.microsoft.com/en-us/powershell/module/activedirectory/get-aduser?view=windowsserver2022-ps).
    
    |Parameter field|Parameter value|
    |--|--|
    |-filter|??|

</details> 

3. [] Use a Powershell command to list all computer objects
<details><summary>➡️**Note**: You must find the right commande. it's not complicated, see a hint</summary>
Cmdlet: [Get-ADUComputer](https://docs.microsoft.com/en-us/powershell/module/activedirectory/get-adcomputer?view=windowsserver2022-ps).
</details> 

**Questions:**  
- How many users you have ?  
..........................................................
<details><summary>➡️**Note**: See a hint ?</summary>
Cmdlet: (xxx).count
</details> 
- How many computer(s) you have ?  
..........................................................

### Task 5: Experiment LSASS Kill
1. [] Log on to **@lab.VirtualMachine(DC01).SelectLink**
    |||
    |:--------|:--------|
    | Username |**YourInitials-Adm**@contoso.com|
    | Password |**+++NeverTrustAny1!+++**|

2. [] Kill process LSASS.exe by using Task Manager
    
    **Question:**
    - As LSASS.exe is longer securing the system, What append ?  
    ..........................................................
    

**Summary**  
In this exercice, you have try to "hack" LSASS but it's not worked like this !

===
## Exercice 4: Password Policy vs FGPP
**Duration**: 10 minutes  
**Synopsis**:  
In this exercise, attendees will learn how to create Fine-Grained Password Policies and how to attach it to a group.
Although the Fine Grained Password Policies (FGPP) are called policies, they are not group policies (GPO)! Therefore, they do not apply to the domain or to an OU but to a user or the member of a group.

### Task 1: Configure FGPP
1. []  Log on to **@lab.VirtualMachine(SRV01).SelectLink**
    |||
    |:--------|:--------|
    | Username |**YourInitials-Adm**@contoso.com|
    | Password |**+++NeverTrustAny1!+++**|

2. [] Launch Active Directory Administrative Center
    >**Note**: We installed in advance the RSAT to manage the ADDS from this server

3. [] Go to System/Password Settings Container
4. [] Create a new Password Settings
5. [] Name : +++Admin-FGPP+++
6. [] Precedence : 5
7. [] Enforce  minimum password length : **15**
8. [] Add it to **Domain Admins** Group

### Task 2: Verify that the Admin-FGPP is applied to Administrators
1. [] Click **Global Search**

    <details><summary>➡️**Note**: See a hint ?</summary>
    - At your left, last entry
    </details>

2. [] In the Search Zone, Enter Administrator
3. [] Click Search
4. [] Right click on the Administrator and Select View resultant password setting
5. [] The **Admin-FGPP** should be displayed
6. [] Change the password by entering +++NewPassword22!+++

    **Questions**  
    - Do you success to change the password?  
    ………………………………………………………………………………………………
    - Why? Ok it's the FGPP but Why ?  
    ………………………………………………………………………………………………

**Summary**  
In this exercice, you have applying FGPP to have a different password policy for all Domain Administrators.

===
## (OPTIONAL) Exercise 5: Replication concept
**Duration**: 15 minutes  
**Synopsis**:  
In this exercise, attendees will discover the basic of replication process.

### Task 1: Show Metadata for a user
1. [] Log on to **@lab.VirtualMachine(DC02).SelectLink**
    |||
    |:--------|:--------|
    | Username |**YourInitials-Adm**@contoso.com|
    | Password |**+++NeverTrustAny1!+++**|

2. [] Open a command line (CLI) with administrator rights
    - Execute the command **Repadmin /showobjmeta DC02** to find the metada for a PAR_User2  
    >[!tip] Use Repadmin /help:showobjmeta for .... help  

    <details><summary>➡️**Note**: See a hint ?</summary>
    In a PowerShell cmdlet with administrative rights, find the DistinguishedName of PAR_User1 with the cmdlet +++get-aduser PAR_User1+++
    </details> 


    >[!KNOWLEDGE] This indicate replication change for a object  
    > The Origination DSA indicate on which DC changed the object attribute.

    **Questions:**
    - Who is the originating DC ?  
    **IT's DC01**
    - Do Loc USN & Org USN have the same number ?  
    **No it's different**
    - Is it normal ?  
    **Yes because USN are local for each DC**

### Task 2: Show Metadata for PAR_User2
1. [] Open the DSA.msc console
2. [] Modify password for PAR_User2
3. [] Show the Metadata for this user

    **Questions**:  
    - What is the version (ver) of the UnicodePwd ("the password" Attribute)?  
    **It's 3** 
    - What will be the version if I will change 2 times more ?  
    **it will be 5** 

===
## (OPTIONAL) Exercise 6: RBAC
**Duration**: 50 minutes  
**Synopsis**:  
In this exercise, attendees will set the OU’s permissions to an administrator and test the delegation. The permissions on the OU before and after the delegation will also be compared. Attendees will see how the AdminSDHolder works and the steps to perform when a user is not more depending of the AdminSDHolder.

### Task 1: Run AD ACL Scanner on OU Paris
1. []  Log on to **@lab.VirtualMachine(DC01).SelectLink**
    |||
    |:--------|:--------|
    | Username |**YourInitials-Adm**@contoso.com|
    | Password |**+++NeverTrustAny1!+++**|

2. [] Check if ADACLScanner is present in the folder **c:\Tools\ADACLScanner-master**
    - If not Download it on +++https://github.com/canix1/ADACLScanner+++ et unzip it in cd \ADACLScanner 
3. [] In PowerShell with administrator rights, run ADACLScan.ps1
    >[!KNOWLEDGE] you will be notified that the file has been downloaded and you will have to press "R". If you want you can resolve it by unblocking the file --> right click properties, unblock.


    - **Connect** and Select **OU Paris**
    - Output Options section, select **CSV Template**

    ![image](.\IMG\ADACLScan_resized2.png)

4. [] Click Run Scan
5. [] Close AD ACL Scanner

### Task 2: Delegate Management of an OU to a group of users
1. [] Open Active Directory Users and Computers
2. [] On OU IT\users
3. [] Create a user 
    - Username: +++Admin_Par+++ 
    - Password : +++NeverTrustAny1!+++
    >Uncheck **User must change password at next logon**

4. [] Go to OU IT\Groups
    - Create a global group named +++Paris-Admin+++
    - Add **Admin_Par** to the new created group
5. [] Right click on **Paris OU** and select **Delegate Control** to the group **Paris-Admin**:
    - Create, delete and manager User accounts
    - Create, delete and manager groups

### Task 3: Compare Result of delegation with AD ACL Scanner on OU Paris
1. [] In PowerShell, run ADACLScan.ps1 
    - Select OU Paris
2. [] In Compare tab, click on Enable compare
3. [] Click on Select Template
4. [] Select the CSV file generate during the first run
5. [] In Output Options, ensure that HTML is selected
6. [] Click Run Scan
7. [] Compare the results in the html file

    **Questions:**  
    - What are the changes (in yellow) and what are the implication of these changes?  
    **We see Paris-Admin has Full control on descendant User Objects and Groups and users in the group can create/Delete user & Group in the OU Paris and all child OUs**

8. [] Close AD ACL Scanner
 
### Task 4 Manage OU
1. [] Log on to **@lab.VirtualMachine(SRV01).SelectLink**
    |||
    |:--------|:--------|
    | Username | +++Admin_Par+++ |
    | Password | Please type the password | 

1. [] Click on the Windows button on the taskbar, click on Windows Administrative Tools and choose Active Directory Administrative Center
2. [] Go to IT\Users
3. [] Create a new user 
    - Username: +++User10+++
    - Password : +++1LoveSecurity!+++

    **Questions:**  
    - Were you able to create the user?  
    **No**
    - Why?  
    **Because I have no delegation on the OU IT\Users, so no rights**

4. [] Go to Paris/Users
5. [] Create a new user 
    - Username: +++Par_User10+++
    - Password : +++1LoveSecurity!+++
    - Click on **Other password options**

    **Questions:**  
    - Were you able to create the user?  
    **Yes**
    - Why?  
    **Because I have the good ACL on this OU, I have the delegation to create a User**

### Task 5: Change the Frequency for the execution of the AdminSDHolder
1. []  Log on to **@lab.VirtualMachine(DC01).SelectLink**
    |||
    |:--------|:--------|
    | Username |**YourInitials-Adm**@contoso.com|
    | Password |**+++NeverTrustAny1!+++**|

2. [] Using the registry, change the frequency of the execution of the AdminSDHolder to 2 minutes (120)
    >+++https://docs.microsoft.com/en-us/previous-versions/technet-magazine/ee361593(v=msdn.10)?redirectedfrom=MSDN#id0250006+++  

    <details><summary>➡️**Note**: See a hint ?</summary>
    - Search +++AdminSDProtectFrequency+++ in the article
    - Choose **Dword** when creation the value  
    - Choose decimal when you set the value  
    - And 2 mins is xx .... in seconds
    </details> 

3. [] Restart the machine and wait until it goes back online before continuing the labs  

    > **Note** That changing this value is not particularly recommended. We are changing the default value from 1 hour to 2 minutes to avoid waiting too long to see the results in our labs.  

### Task 6: Review the AdminSDHolder’s permissions
1. [] Log on to **@lab.VirtualMachine(DC01).SelectLink**
    |||
    |:--------|:--------|
    | Username |**YourInitials-Adm**@contoso.com|
    | Password |**+++NeverTrustAny1!+++**|

1. [] Using “Active Directory User and Computers” review the permissions of the AdminSDHolder container located on in System
    >[!Alert]Enable Advanced Feature for the Active Driectory Users & Computers MMC

    ![image](.\IMG\DSA_AdvView.png)    

2. [] Expand the OU PARIS\Users
3. [] Review the permission of the account for **PAR_User1** and choose **Properties**
4. [] Select the tab **Attribute Editor**
5. [] Click on **Filter** and choose **Show only attributes that have values**
6. [] Check the value of **AdminCount**
    > As the user is member of the group **Account Operators**, it depends on the AdminSDHolder, so its adminCount value is 1  

    **Questions**:  
    - What is the AdminCount value ?  
    **It's 1**
    - Are the securities changed ?  
    **Yes It's the same as the AdminSDHolder**
    - Why ?  
    **That simply setting the adminCount attribute to 0 does nothing as the attribute is just written by the process and not read. And the opposite is also true. If you set the adminCount attribute of an account to 1 whereas this account is not in the scope of the adminSDHolder, the account does not become protected.**

8. [] Remove **PAR_User1** from the group **Account Operators**
9. [] Wait 2 minutes
10. [] Right click on **PAR_User1** and choose Properties
12.	[] Select the tab Attribute Editor
12. [] Review the adminCount
> The value should still be set to 1. **The protection mechanism does not unprotect accounts**. This has to be done manually.

13. [] Click on the **Security tab**
14. [] Click **Advanced**
15. [] The **Inheritance** is still **disabled**
16. [] Click on **Enable Inheritance**
17. [] Click **OK**
18. [] Click **Yes** on the warning
19. [] Select the tab **Attribute Editor**
20.	[] Set the **adminCount** to **0**
21. [] Wait 2 mins

===
## (OPTIONAL) Exercice 7: Allow only Administrators to logon CLI01 
**Duration**: 15 minutes  
**Synopsis**:  
In this exercise, attendees will learn how to create a GPO, Import a GPO backuped and how to applied it.

### Task 1: Create a GPO
1. [] Log on to **@lab.VirtualMachine(SRV01).SelectLink**
    |||
    |:--------|:--------|
    | Username | **+++Administrator@contoso.com+++** |
    | Password | Please type the password |  

2. [] Open a Navigator and open **Security Compliance Tool Kit** Download page: +++https://www.microsoft.com/en-us/download/details.aspx?id=55319+++
3. [] Select **Download** & select **Windows 11 Security Baseline.zip**. Download it
4. [] Extract the archive **Windows 11 Security Baseline.zip**
    |||
    |:--------|:--------|
    |![image](.\IMG\0axs5jpw.jpg)|     |

5. [] Open the GPMC.msc  

>[!knowledge]GPMC.msc is the MMC to manage all GPOs for the forest.

6. [] Expand the Forest & the Domain **Contoso**, select **Group Policy Objects**
7. [] Right Click on it & select **New**
Name it: +++C-Windows 11 Security Baseline+++  

>[!tip]It's important to have a naming convention. IT's especially right for GPO. You can use for example:  
- U for GPO applied on Users
- C for GPO applied on Conputers
- A for GPO applied on Uses & Computers 

8. [] Select **OK**

### Task 2: Import the Windows 11 Security Baseline
1. [] Right Click on the GPO created & select **Import Settings**
    |||
    |:--------|:--------|
    | ![image](.\IMG\fh5aitwd.jpg) | |

2. [] Select **Next** until **Backup Location**. Select **Browse** & select the folder **GPOs** where you extract the SCT Baseline
    ![image](.\IMG\1c3qlkks.jpg)

3. [] Select **OK**, Select **Next**
4. [] Select the Backed up GPOs named: **MSFT Windows 11-Computer**
    ![image](.\IMG\0wsyorts.jpg)

5. [] Select **Next** until the final screen & Select ** Finish**

6. [] Right click on the GPO & Select **Edit**

7. [] Go to **Computer Configuration\Policies\Windows Settings\Security Settings\Local Policies\User Rights Assignment**

8. [] Double click on **Allow log on locally** & remove **Users**

9. [] Select **OK** and close the GPO edition (the cross)

### Task 3: Link the GPO to ""deploy" it on client
1. [] Expand OU **Paris\Computers**
2. [] Right click on the OU Computers & select **Link an Existing GPO**
    ![image](.\IMG\6vd9mzjb.jpg)
3. [] Select **C-Windows 11 Security Baseline**
4. [] Open a DSA.msc
>Be sure that you enable **Advanced Features** in **View**

5. [] Move the computer object **CLI01** under Paris\Computers
6. [] Before testing the impact of this GPO, right click on the OU Computers under the OU Paris (where you link the GPO) & select **Properties**
7. [] Select the Tab **Attribute Editor**
    ![image](.\IMG\wu6fid6t.jpg)
8. [] Go to Attribute **gPLink**

    **Questions**  
    - What is the value of the gPLink attribute?  
    **[LDAP://cn={GUID GPO},cn=policies,......**

    >[!Knowledge]gPLink attribute contains the DN of all GPOs linked on the OU. It's like that that the client find which GPO applied.

### Task 4: Test the GPO
1. [] Switch to **@lab.VirtualMachine(CLI01).SelectLink**

>[!alert] Don't log on the station

Reboot the client

>[!knowledge]If the session is already open, you can, in a command prompt, execute the cmdlet +++gpupdate+++ to refresh and apply GPO

2. [] After the reboot, log on with:
    - Username: +++Par_User10+++
    - Password : +++1LoveSecurity!+++

    **Questions**  
    - What's Happend?  
    **The user cannot log on**
    - Why?  
    **Because we removed User from the URA allow Logon and Par_User10 is not Administrators**

3. [] Switch to **@lab.VirtualMachine(SRV01).SelectLink**

4. [] In the GPMC.msc, right click on the GPO **C-Windows 11 Security Baseline** on \Paris\Computers and click on **Delete**

5. [] Switch to **@lab.VirtualMachine(ClI01).SelectLink**

6. [] Logon with
    |||
    |:--------|:--------|
    | Username | +++.\administrator+++ |
    | Password | Please type the password |

7. [] Open a command prompt and execute +++Gpupdate+++

8. [] Wait the completion & logoff

2. [] Log on with:
    - Username: +++Par_User10+++
    - Password : +++1LoveSecurity!+++

    **Questions**  
    - What's Happend?  
    **The user can log on**

**Summary**
In this exercise, you created a GPO to apply a Security Baseline from Microsoft Recommendations (SCT) to CLI01 & allow only Administrators to log on the workstation. 



# End 


