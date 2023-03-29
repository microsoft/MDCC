![image](.\IMG\MSCyberCoursesResized.png)

# The foundations of hybrid identity

>Last Update September 2022

# Contents

<!-- TOC -->
 
- [Requirements](#requirements)
- [Azure Portal Navigation Tips](#azure-portal-navigation-tips)
- [MFA Enrollment](#mfa-enrollment)
- [Overview](#overview)
- [Active Directory Services hands-on lab step-by-step](#active-directory-services-hands-on-lab-step-by-step)
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

- [Extending identities to the cloud with Azure AD hands-on lab step-by-step](#extending-identities-to-the-cloud-with-azure-ad-hands-on-lab-step-by-step)
  - [Exercise 1: Integrate an Active Directory forest with an Azure Active Directory tenant](#exercise-1-integrate-an-active-directory-forest-with-an-azure-active-directory-tenant)
        - [Task 1: Create an Azure Active Directory tenant and activate an EMS E5 trial](#task-1-create-an-azure-active-directory-tenant-and-activate-an-ems-e5-trial)
        - [Task 2: Enable EMSE5 Trials](#task-2-enable-ems-e5-trials)
        - [Task 3: Create and configure Azure AD users](#task-3-create-and-configure-azure-ad-users)
        - [Task 4: Install Azure AD Connect](#task-4-install-azure-ad-connect)
        - [Task 5: verify directory synchronization](#task-5-verify-directory-synchronization)
  - [Exercise 2: Enforce MFA for Global Admins](#exercise-2-enforce-mfa-for-global-admins)
        - [Task 1: Enable MFA for the a Global Administrator)](#task-1-enable-mfa-for-the-a-global-administrator)
        - [Task 2: Test the MFA](#task-2-test-the-mfa)
  - [Exercise 3: Overview of Azure AD User & Group objects](#exercise-3-overview-of-azure-ad-user--group-objects)
        - [Task 1: Play with Azure (create cloud user, create cloud security group, create a dynamic group & modify a group)](#task-1-play-with-azure-create-cloud-user-create-cloud-security-group-create-a-dynamic-group--modify-a-group)
  - [Exercise 4: Exercice 4: Enable Self Service Password Reset](#exercice-4-enable-self-service-password-reset)   
        - [Task 1: Assign EMS E5 licenses to Azure AD users](#task-1-assign-ems-e5-licenses-to-azure-ad-users)
        - [Task 2: Enable password writeback and Self-Service Password Reset](#task-2-enable-password-writeback-and-self-service-password-reset)
  - [(OPTIONAL) Exercise 5: Use a Service Principal to access to Microsoft Graph API](#optional-exercise-5-use-a-service-principal-to-access-to-microsoft-graph-api)
        - [Task 1: Create a Service Principal](#task-1-create-a-service-principal)
        - [Task 2: Generate a certificat to secure the Service Principal (SP)](#task-2-generate-a-certificat-to-secure-the-service-principal-sp)
        - [Task 3: Add permissions to allow read Signin logs](#task-3-add-permissions-to-allow-read-signin-logs)
        - [Task 4: Test the SP](#task-4-test-the-sp)  
  - [(OPTIONAL) Exercise Op1: Observe a synchronization round](#optional-exercise-op1-observe-a-synchronization-round)
    
<!-- /TOC -->

>[!ALERT] **DISCLAIMER**   
- Unless otherwise noted, the example companies, organizations, products, domain names, e-mail addresses, logos, people, places, and events depicted herein are fictitious, and no association with any real company, organization, product, domain name, e-mail address, logo, person, place or event is intended or should be inferred. Complying with all applicable copyright laws is the responsibility of the user. Without limiting the rights under copyright, no part of this document may be reproduced, stored in or introduced into a retrieval system, or transmitted in any form or by any means (electronic, mechanical, photocopying, recording, or otherwise), or for any purpose, without the express written permission of Microsoft Corporation.   
- Microsoft may have patents, patent applications, trademarks, copyrights, or other intellectual property rights covering subject matter in this document. Except as expressly provided in any written license agreement from Microsoft, the furnishing of this document does not give you any license to these patents, trademarks, copyrights, or other intellectual property.
- The names of manufacturers, products, or URLs are provided for informational purposes only and Microsoft makes no representations and warranties, either expressed, implied, or statutory, regarding these manufacturers or the use of the products with any Microsoft technologies. The inclusion of a manufacturer or product does not imply endorsement of Microsoft of the manufacturer or product. Links may be provided to third party sites. Such sites are not under the control of Microsoft and Microsoft is not responsible for the contents of any linked site or any link contained in a linked site, or any changes or updates to such sites. Microsoft is not responsible for webcasting or any other form of transmission received from any linked site. Microsoft is providing these links to you only as a convenience, and the inclusion of any link does not imply endorsement of Microsoft of the site or the products contained therein.

© 2022 Microsoft Corporation. All rights reserved.

Microsoft and the trademarks listed at <https://www.microsoft.com/en-us/legal/intellectualproperty/Trademarks/Usage/General.aspx> are trademarks of the Microsoft group of companies. All other trademarks are property of their respective owners.

===
# Requirements
## Attendee’s machine
1. []	Ideal resolution 1920 x 1080 
2. []	An Internet browser
3. []	An internet access without restriction on outbound connections.
4. [] The following TCP port must be accessible : 
    - TCP/80 and TCP/443 to reach Lab On Demand

## Attendee's phone
1. [] A telephone number
1. [] A smartphone with Microsoft Authenticator installed  
or
5. [] A valid email (you can use a existing email or create one at +++https://www.outlook.com+++)

===

# Azure Portal Navigation Tips
You can close blade by using the cross at the upper right  
|||
|:--------|:--------|
|![image](.\IMG\cmwp7y9j.jpg)||  

You can **go back** to the previous Windows by using the navigation bar under the Azure Search bar  
|||
|:--------|:--------|
|![image](.\IMG\dvgy3qcg.jpg)||  

If you use the arrow **Go back** of you browser, you will lose the menu chain.  

# MFA Enrollment
For the MFA Enrollment you must have a Smartphone with at lease 2 of these 3 methods:
- SMS
- Email
- Authenticator (Microsoft, Google or others)

Security informations (means MFA) enrollment occurs after the first Sign-in with Account/Password when MFA is required
|||
|:--------|:--------|
|![image](.\IMG\fs4ni699.jpg)|Select **Next**|
|![image](.\IMG\zcce4vlj.jpg)|Select **Next**|
|  **STOP HERE!** Choose your method||
|![image](.\IMG\qhrpqu9v.jpg)| Select **Next** if you want use Microsoft Authenticator |
|![image](.\IMG\1b9adk10.jpg)| Select **I want to use a different authenticator app** if you want use another Authenticator |
|![image](.\IMG\m9wz590s.jpg)| Select **I want to set up a different method** if you want use Phone or Email instead of a authenticator|
| **If you choose Authenticator** ||
|![image](.\IMG\6bdna63v.jpg)| **Scan the QR code** with your authenticator and follow the wizard|
| **If you choose Phone** ||
|![image](.\IMG\9g3l45it.jpg)| Select France, enter your phone numbre, you will recieve a verified code|
| **If you choose Email** ||
|![image](.\IMG\kjd0quzk.jpg)| Enter your email to received a verification code|
|![image](.\IMG\ljdhqusr.jpg)| You can modify your choice at +++https://aka.ms/setupmfa+++|

===

# Overview

Contoso has asked you to integrate their on-premises Active Directory single-domain forest named contoso.com with Azure AD and implement all necessary prerequisites to allow them to benefit from such Azure AD features as:
- Single sign-on to cloud and on-premises applications,
- Enhanced sign-in security with Multi-Factor Authentication
- automatic enrollment of Windows 10 devices into Microsoft Intune

They want to also provide secure access to their on-premises, Windows Integrated Authentication-based applications from Internet for both organizational users and users who are members of partner organizations, although they also want to be able to loosen restrictions when access originates from Hybrid Azure AD joined computers residing in their on-premises data centers. The same applications also need to be made available to Contoso's business partners. 

In this Lab, the attendees will manage Active Directory using the graphical interfaces and PowerShell. The attendees will also use basic features of AD DS such as GPO and manipulate objects using the default administrative tools.

# Active Directory Services hands-on lab step-by-step 

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
-	What happened?    
    ………………………………………………………………………………………………

-	Why do you have this Behavior?    
    ………………………………………………………………………………………………

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
-	What happened?    
    ………………………………………………………………………………………………

-	How many files do you see?     
    ………………………………………………………………………………………………

-	Why do you have this Behavior?    
    ………………………………………………………………………………………………



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
    ![image](.\IMG\rw4nxk4u.jpg)  

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
    ![image](.\IMG\rw4nxk4u.jpg)
    
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
-	How many users you have?    
    ………………………………………………………………………………………………

-	How many computer(s) you have?    
    ………………………………………………………………………………………………


### Task 5: Experiment LSASS Kill
1. [] Log on to **@lab.VirtualMachine(DC01).SelectLink**
    |||
    |:--------|:--------|
    | Username |**YourInitials-Adm**@contoso.com|
    | Password |**+++NeverTrustAny1!+++**|

2. [] Kill process LSASS.exe by using Task Manager
    
**Question:**
- As LSASS.exe is longer securing the system, what happened?    
    ………………………………………………………………………………………………

**Summary**  
In this exercice, you have try to "hack" LSASS but it's not worked like this !

===
## Exercise 4: Password Policy vs FGPP
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

    - Why? (Ok it's the FGPP but Why?)  
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
-	Who is the originating DC?    
    **IT's DC01**

-	Do Loc USN & Org USN have the same number?    
    **No it's different**

-	Is it normal?    
    **Yes because USN are local for each DC**


### Task 2: Show Metadata for PAR_User2
1. [] Open the DSA.msc console
2. [] Modify password for PAR_User2
3. [] Show the Metadata for this user

**Questions**:  
- What is the version (ver) of the UnicodePwd ("the password" Attribute)?    
    **It's 3**

- What will be the version if I will change 2 times more?    
    **It will be 5, try it**


===
## (OPTIONAL) Exercise 6: RBAC
**Duration**: 30 minutes  
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
    >[!KNOWLEDGE] You will be notified that the file has been downloaded and you will have to press "R". If you want you can resolve it by unblocking the file --> right click properties, unblock.


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
5. [] In Scan Options tab, ensure that HTML is selected
6. [] Click Run Scan
7. [] Compare the results in the html file

    **Questions:**  
    -	What are the changes (in yellow) and what are the implication of these changes?    
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
    >You can open the KB on **@lab.VirtualMachine(SRV01).SelectLink** or on your personal device +++https://docs.microsoft.com/en-us/previous-versions/technet-magazine/ee361593(v=msdn.10)?redirectedfrom=MSDN#id0250006+++  

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
    - What is the AdminCount value?    
    **It's 1**

    - Are the securities changed?    
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

**Summary**
In this Exercice, you have delegating the right to manage Users & Groups for a Group of administrators. You saw also how the delegation is applied on the OU.

===
## (OPTIONAL) Exercise 7: Allow only Administrators to logon CLI01 
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
    |![image](.\IMG\kmksuco8.jpg)|     |

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
    | ![image](.\IMG\q7yhiqmi.jpg) | |

2. [] Select **Next** until **Backup Location**. Select **Browse** & select the folder **GPOs** where you extract the SCT Baseline
    ![image](.\IMG\8sb0p4ri.jpg)

3. [] Select **OK**, Select **Next**
4. [] Select the Backed up GPOs named: **MSFT Windows 11-Computer**
    ![image](.\IMG\7beao4u4.jpg)

5. [] Select **Next** until the final screen & Select ** Finish**

6. [] Right click on the GPO & Select **Edit**

7. [] Go to **Computer Configuration\Policies\Windows Settings\Security Settings\Local Policies\User Rights Assignment**

8. [] Double click on **Allow log on locally** & remove **Users**

9. [] Select **OK** and close the GPO edition (the cross)

### Task 3: Link the GPO to ""deploy" it on client
1. [] Expand OU **Paris\Computers**
2. [] Right click on the OU Computers & select **Link an Existing GPO**
    ![image](.\IMG\219ekvw7.jpg)
3. [] Select **C-Windows 11 Security Baseline**
4. [] Open a DSA.msc
>Be sure that you enable **Advanced Features** in **View**

5. [] Move the computer object **CLI01** under Paris\Computers
6. [] Before testing the impact of this GPO, right click on the OU Computers under the OU Paris (where you link the GPO) & select **Properties**
7. [] Select the Tab **Attribute Editor**
    ![image](.\IMG\lxunz8dt.jpg)
8. [] Go to Attribute **gPLink**

    **Questions**  
    - What is the value of the gPLink attribute?  
    **[LDAP://cn={GUID GPO},cn=policies,.......**  

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

===
# Extending identities to the cloud with Azure AD hands-on lab step-by-step 

## Abstract and learning objectives 

In this hands-on lab you will setup and configure a number of different hybrid identity scenarios. The scenarios involve an Active Directory single-domain forest named contoso.com, which in this lab environment, consists (for simplicity reasons) of :
- Two domain controller named DC01 & DC02
- A single domain member server named SRV01
- A single workstation named CLI01

The intention is to explore Azure AD-related capabilities that allow you to integrate Active Directory with Azure Active Directory, optimize hybrid authentication and authorization, and provide secure access to on-premises resources from Internet for both organizational users and users who are members of partner organizations. 

## Exercise 1: Integrate an Active Directory forest with an Azure Active Directory tenant
**Duration**: 45 minutes  
**Synopsis**:  
In this exercise, you will integrate an Active Directory forest with an Azure Active Directory tenant by:
- Creating an Azure Active Directory tenant and activating an Enterprise Mobility + Security E5 trial,
- Creating and configuring an Azure AD user,
- Installing Azure AD Connect.

### Task 1: Create an Azure Active Directory tenant and activate an EMS E5 trial

In this task, you will create an Azure Active Directory tenant with the following settings: 
-   Organization name: **Contoso**
-   Initial domain name: "School Trigram" + "Student initial" + 2 digits  
    >*Example : **MSCG01***

>[!ALERT] **You DO NOT NEED a credit card for this workshop. If you are asked for one, open a new tab and close the precedent**

>[!ALERT] You will need a valid phone number that can receive text messages.
This phone number can be used for multiple tenants. 
Don't forget to press the send code button to send the validation text.

1. [] Log on to **@lab.VirtualMachine(SRV01).SelectLink**
    |||
    |:--------|:--------|
    | Username | **+++Administrator@contoso.com+++** |
    | Password | Please type the password |

1. [] Open a in-Private/Incognito session & Go to +++https://account.azure.com/organization+++ in a web browser.
    >[!KNOWLEDGE] The In-Private/Incognito session will ensure your current credentials are not used.  

2. [] Enter a valid email. If you don't have email you can create one at +++https://outlook.live.com/owa/+++

3. [] At **Tell us about yourself**, enter:
    - Your First Name, Last Name
    - Your Business Phone (Microsoft use it after to validate your Tenant)
    - Company Name as **CONTOSO**
    - Country as **France**
    ![image](.\IMG\AAD1.png)
4. [] Select Next
6. [] Verify your phone by enter the verification code
7. [] How you will sign in, enter:
    - Username : +++root+++
    - Domain Name : "School Trigram" + "Student initial" + 2 digits  

    >[!HINT] The name supplied under Domain Name will subsequently become the name of your tenant as
    **MSCG01.onmicrosoft.com**  

    - Password : +++NeverTrustAny1!+++
    ![image](.\IMG\AAD3.png)

>[!Alert]**Note**: Register your Tenant information. **Tenant Name, First Global Administrator & password**

8. [] Select **Next**
9. [] Click **Get Started**
    >[!ALERT]**Don't continue to fill the next page !**

    ![image](.\IMG\AADCreation02Resized.png)


6. []  **Open a new tab** and navigate to +++https://portal.azure.com+++ & **close the previous tab**
If prompted, enter the global admin user name and password for your tenant:
    - Username: root@"AzureDomainName".onmicrosoft.com
    - Password: "YourGlobalAdminPassword"

### Task 2: Enable EMS E5 Trials
1. [] Go to Azure licenses center: +++https://portal.azure.com/#blade/Microsoft_AAD_IAM/ActiveDirectoryMenuBlade/Licenses+++
2. [] Select the All Products blade menu item.
3. [] Select the Try/Buy button and enable all of the trials (EMS E5 and AAD P2) in the fly-out that appears.
> **Note**: It may take a few minutes for the licenses to appear in the all products portal; after you activate a trial, you will not need to select it again.

>[!Tip] Activation typically takes about 5 minutes.

### Task 3: Create and configure Azure AD users
In this task, you will configure Azure AD user accounts in the newly created Azure AD tenant with the following settings. This will include assigning EM+S E5 licenses to the user account you are using for this lab as well as creating a new Azure AD user account with the following settings and assigning to it the Global Administrator role as well as the EM+S E5 license.

1. [] Log on to **@lab.VirtualMachine(SRV01).SelectLink**
    |||
    |:--------|:--------|
    | Username | **+++Administrator@contoso.com+++** |
    | Password | Please type the password |

2. [] Open the in the Azure portal & navigate to the Azure AD blade: +++https://portal.azure.com/?feature.msaljs=false#view/Microsoft_AAD_IAM/ActiveDirectoryMenuBlade/~/Overview+++
2. [] Select **Users** under **Manage** in the left navigation.
3. [] On the **Users - All users** blade, select the entry representing your user account.
4. [] On the **Properties** of your user account.
5. [] In the **Settings** section, in the **Usage location** drop-down list, select the **France** entry and select **Save**.
6. [] On the **Profile** blade of your user account, select **Licenses** under **Manage** on the left. 
7. [] On the **Licenses** blade, select **+ Assignments**.
8. [] On the **Update license assignments** blade, enable the **Enterprise Mobility + Security E5** checkbox, ensure that all the corresponding license options are enabled, and select **Save**.

9. [] On the **Users - All users** blade, select **+ New user**.
10. [] On the **New user** blade, ensure that the **Create user** option is selected, specify the following settings, and select **Create**:

    |||
    |:--------|:--------|
    |User name: |+++jdoe+++@*TenantName*.onmicrosoft.com|  
    |Name: |+++John Doe+++|
    |First name: |+++John+++|
    |Last name: |+++Doe+++|
    |Select **Let me create the password**||
    |Password: |+++1LoveSecurity!+++|
    |Groups: |**0 group selected**|
    |Roles: |**Leave blank**|
    |Block sign in: |**No**|
    |Usage location: |**France**|
    |Job title: |**Leave blank**|
    |Department: |**Leave blank**|  

    >[!TIP]Where *TenantName* is the domain name you specified when creating the Contoso Azure AD tenant.  

11. [] On the **Users - All users** blade, select the entry representing the newly created user account.
12. [] On the **john.doe - Profile** blade, select **Licenses** under **Manage** on the left.  
Wait few seconds & Refresh if needed
13. [] On the **john.doe - Licenses** blade, select **+ Assignments**.
14. [] On the **Update license assignments** blade, enable the **Enterprise Mobility + Security E5** checkbox, ensure that all the corresponding license options are enabled, and select **Save**.
15. [] Do the same for the Admin User:

    |||
    |:--------|:--------|
    | Name: | +++(ADM) John Doe+++ |
    | User Name: | +++admaz-jdoe+++@*TenantName*.onmicrosoft.com |
    |Password: |+++NeverTrustAny1!+++|
    | Roles: | **Global Administrator** |

### Task 4: Install Azure AD Connect
1. [] Download Azure AD Connect from +++https://www.microsoft.com/en-us/download/details.aspx?id=47594+++  

>[!KNOWLEDGE]It is a bad practice to access the internet from a Tier 0, or Tier 1 system; additionally a system that can access any of those tiers should follow the same rules because of the Clean Source Principal.

>[!KNOWLEDGE] For the sake of timing, we will ignore this rule for the lab. In a real environment (including dev environments) following strict security processes is crucial for maintaining organization wide security.

2. [] Start the installation in **customize** mode **NOT** use **express settings** 

>[!TIP]No custom options are necessary, just press install.

3. [] Select **Password Hash Synchronization** & select **Enable single sign-on**
4. [] Log in with your Azure AD Global Admin account.
5. [] Add Contoso.com and log in with the Domain admin credentials:
    - Ensure that "Create new AD account" stays selected.
    - Username: +++CONTOSO\Administrator+++
    - Password: +++NeverTrustAny1!+++

>[!KNOWLEDGE]The system will create a new ADDS user account for the AAD Connect utility, you can find it in CN=Users.

6. [] Set AAD Connect to continue without matching all UPN suffixes to verified domains.
![image](.\IMG\AADCInstall2.png)

8. [] Set AAD Connect to sync all domains and OUs 
    >**Note**: In production, you can select some OU insteed All Domains.

    **Question:**
    - What is the goal of the filtering ?  
    ………………………………………………………………………………………………  
 

9. [] "Next" through the rest of the wizard.

10. [] At **Enable single sign-on**, enter your T0 Admin credential

    **Questions:**

    - What is the goal of SSO?  
    ………………………………………………………………………………………………  

    - Why is it Seamless?  
    ………………………………………………………………………………………………  


11. [] At **Ready to configure**, leave by default & press **Install**

>[!KNOWLEDGE]Ordinarily you would register your company’s domain name within Azure AD such that any synchronized users can have that as their suffix in the cloud.   
As this is for testing only all users will assume the <tenantname>.onmicrosoft.com suffix

>[!Alert] If for some reasons the installation of Azure AD Connect crash, Double click on the Azure AD Connect icon on the desktop.

### Task 5: Check directory synchronization
After setting up the user sync, we will want to validate that the sync connection has been established correctly.
1. [] Go to Azure Active Directory in the Azure portal: +++https://portal.azure.com/#blade/Microsoft_AAD_IAM/ActiveDirectoryMenuBlade+++
2. [] In the blade menu, select **Users** → **All Users**.  
You should see lots of users in the directory who have the Directory Synced Column with an attribute of "Yes" or "No" for each users.

### Task 6: Disable security defaults
By default, all new tenants are **Security Defaults** enabled. That's force user & administrators to have MFA. For the rest of the lab, we need to disable this feature, and if you use Conditionnal Access in Production, you will also disable it.
1. [] Go to Azure Active Directory in the Azure portal: +++https://portal.azure.com/#blade/Microsoft_AAD_IAM/ActiveDirectoryMenuBlade+++
2. [] In the blade menu, select **Properties**, **Manage security defaults**
3. [] Select **NO**  
    ![image](.\IMG\SecurityDefaultsResized.png)
4. [] **Save**

**Summary**:  
In this exercise, you integrated an Active Directory forest with an Azure Active Directory tenant by creating an Azure Active Directory tenant and activating an Enterprise Mobility + Security E5 trial, creating and configuring an Azure AD user, and installing Azure AD Connect to configure Hybrid Identity.

===
## Exercise 2: Enforce MFA for Global Administrator
**Duration**:  10 minutes  
**Synopsis**:  
In this exercise you will setup and configure Multi Factor Authentication (MFA) for your admins, this is especially important for cloud accounts which may be susceptible to credential theft attempts.
You can complete this outside of the lab if you wish; Remember to be logged into the correct account if you do this.

### Task 1: Enable MFA for the a Global Administrator
1. [] Log on to **@lab.VirtualMachine(SRV01).SelectLink**
    |||
    |:--------|:--------|
    | Username | **+++Administrator@contoso.com+++** |
    | Password | Please type the password |

2. [] Navigate to +++https://portal.azure.com/#blade/Microsoft_AAD_IAM/UsersManagementMenuBlade/MsGraphUsers+++ and log in with the credentials that correspond with the Azure AD you created in the first Exercice.
3. [] Select the Per-User MFA button at the top menu.
4. [] Select the Root Global Admin account and click Enable.

>[!KNOWLEDGE] In production you can bulk update users using CSV files.  

>[!KNOWLEDGE] MFA can be licensed and used in many ways, here is a breakdown of the options:  
    > - MFA is free for Azure AD users who hold an administrator role  
    > - A subset of MFA features is licensed with Office 365  
    > - Full MFA comes with an Azure AD Premium license  
    > - MFA can be paid for as you go, either per user authentication or per user  
    > - MFA is included with the security defaults  

   **Questions:**  
- Enabling MFA on a per-user basis is not scalable to large environment. In real situations, there are other way to request MFA. Using the reference documentation, find these 2 other deployment options?   
    ………………………………………………………………………………………………


### Task 2: Test the MFA
1. [] Open a new Private/Incognito browsing session and navigate to +++https://myapps.microsoft.com+++
2. [] Log into the service with the user that was just MFA enabled.
3. [] Set up the user's MFA configuration when prompted.
>You may be presented with an app password, these passwords are used for application which can’t or don’t support MFA.

4. [] After the MFA configuration has been completed, log out and log back in to validate that MFA has been set up correctly.
>**Note**: End users can navigate to https://aka.ms/mfasetup to change their MFA settings.

**Summary**:  
In this exercise, you discover Azure AD users & groups.

===
## Exercise 3: Overview of Azure AD User & Group objects
**Duration**: 20 minutes  
**Synopsis**:  
In this exercise, you will discover users & groups in Azure AD

### Task 1: Play with Azure (create cloud user, create cloud security group, create a dynamic group & modify a group)
1. [] Log on to **@lab.VirtualMachine(SRV01).SelectLink**
    |||
    |:--------|:--------|
    | Username | **+++Administrator@contoso.com+++** |
    | Password | Please type the password |

2. [] In the Edge browser window, navigate to the following URL in a tab. 
    +++https://portal.azure.com+++
3. [] Search & Select **Azure Active Directory** in the search bar

3. [] Modify a the Group named +++Paris Prod+++ by adding a member

    **Question**
    - Why can't you change it?  
    ………………………………………………………………………………………………  


4. []Create 2 Cloud only users for emergency (**B** reaking **G** lass **A** ccounts)
    - On the **Users - All users** blade, select **+ New user**.
    - On the **New user** blade, ensure that the **Create user** option is selected, specify the following settings, and select **Create**:

        |||
        |:--------|:--------|
        |User name: |+++admaz-BGA1+++@*TenantName*.onmicrosoft.com|  
        |Name: |+++Breaking Glass Accounts 1+++|
        |Password: |+++NeverTrustAny1!+++|
        |Groups: |**0 group selected**|
        |Roles: |**Global Administrator**|
        |Block sign in: |**No**|
        |Usage location: |**France**|
        |Job title: |**Leave blank**|
        |Department: |**Leave blank**|  

5. [] Create with the same procedure to create Breaking Glass Accounts 2
 

6. [] Find him in ADDS
    -   Open DSA.msc
    -   Right click and select Find. Search your user creation

    **Questions**:  
    -   Do you find it?  
    ………………………………………………………………………………………………  

    -   Why?  
    ………………………………………………………………………………………………  


7. []  In **Azure AD**, Create a **security** Group Named +++SG_Privileged_Accounts+++  
    - Select **Azure AD roles can be aassigned to the group** (at Yes)
    - Add all Global Administrators as members
        - Your first account creating during Tenant creation, 2 BGA & Jdoe admin account   
    >Don't forget to specify the owner (You) 

9. []  Create a Dynamic Group Named +++SG_d_Privileged_Accounts+++
    - Create a new group with the **Membership type** selected to **Dynamic User** 
    - Set this rules : (user.userPrincipalName -startsWith "admaz")
    >[!KNOWLEDGE]The refresh can take 24h, we will review it in the Lab6

    **Questions**:  
    - How many accounts are expected to be member of the dynamic group ?  
    ………………………………………………………………………………………………  
    - Why ?  
    ………………………………………………………………………………………………  


10. []Delete Jdoe Admin account
    - Go to User blade, select Jdoe Admin account and delete it
11. [] Using the documentation located at +++https://docs.microsoft.com/en-us/azure/active-directory/fundamentals/+++, restore it  

    **Questions**:  
    - Does John Doe have their groups membership restored ?  
    ………………………………………………………………………………………………  
    - Does John Doe have their roles membership restored?  
    ………………………………………………………………………………………………  
    - Can you permanently delete a account?  
    ………………………………………………………………………………………………  


**Summary**:  
In this exercise, you discovered & manipulated Azure AD users & groups.

===
## Exercise 4: Enable Self Service Password Reset 
**Duration**: 15 minutes  
**Synopsis**:  
In this exercise, you will enable a powerfull feature to allow users to change themself their password.

### Task 1: Assign EMS E5 licenses to Azure AD users
In this task, you assign a value to the **UsageLocation** attribute of each user account and assign an Azure AD Premium license to each user. This is necessary in order to implement Azure AD-based Multi-Factor Authentication for these users.
1. [] Log on to **@lab.VirtualMachine(SRV01).SelectLink**
    |||
    |:--------|:--------|
    | Username | **+++Administrator@contoso.com+++** |
    | Password | Please type the password |

2. [] On the Script pane of the Windows PowerShell ISE window, run the following to sign into the Contoso Azure AD tenant. When prompted, sign in with the **(ADM) john doe** Azure AD user account, which you created in the previous exercise.  
In a Powershell Cmdlet begin by:    
+++Connect-AzureAD+++

3. [] On the Script pane of the Windows PowerShell ISE window, run the following to set the **Location** attribute to **France** for all Azure AD user accounts with the UPN suffix matching the custom verified domain name of the Contoso Azure AD tenant. 

    +++$domainName = (Get-AzureADDomain | Where-Object IsDefault -eq 'True').Name+++ 

    +++Get-AzureADUser | Where-Object {$_.UserPrincipalName -like "*@$domainName"} | Set-AzureADUser -UsageLocation 'fr'+++

4. [] On the Script pane of the Windows PowerShell ISE window, run the following to assign the EM+S E5 trial licenses to all Azure AD user accounts with the UPN suffix beginning by 'PAR_'. 



    +++$license = New-Object -TypeName Microsoft.Open.AzureAD.Model.AssignedLicense+++  
    +++$license.SkuId = (Get-AzureADSubscribedSku | Where-Object {$_.SkuPartNumber -eq 'EMSPREMIUM'}).SkuId+++  
    +++$licensesToAssign = New-Object -TypeName Microsoft.Open.AzureAD.Model.AssignedLicenses+++  
    +++$licensesToAssign.AddLicenses = $license+++  
    +++$users = Get-AzureADUser | Where-Object {$_.UserPrincipalName -like "PAR_*"}+++  
    +++foreach($user in $users) {+++  
    +++Set-AzureADUserLicense -ObjectId $user.ObjectId -AssignedLicenses $licensesToAssign+++  
    +++}+++  

    >[!KNOWLEDGE]You can use Powershell ISE or Notepad to copy/paste the cmdlet.

    **Questions**:  
    - Does John Doe admin account has a license assigned?  
    ………………………………………………………………………………………………  

    - Does PAR_User3 account has a license assigned?  
    ………………………………………………………………………………………………  

    - Why ?  
    ………………………………………………………………………………………………  

        
### Task 2: Enable password writeback and Self-Service Password Reset

In this task, you will enable password writeback and Self-Service Password Reset (SSPR) for Contoso users that had their accounts synchronized to the Contoso Azure AD tenant.

1. [] On **SRV01** double-click the **Azure AD Connect** desktop shortcut.

2. [] On the **Welcome to Azure AD Connect** page, select **Configure**. 

3. [] Select **Customize synchronization options** and select **Next**.

4. [] On the **Connect to Azure AD** page, sign in by using the credentials of the **admaz-jdoe** account and select **Next**.

5. [] On the **Connect your directories** page, select **Next**.

6. [] On the **Domain and OU filtering** page, select **Next**. 

7. [] On the **Optional features** page, check the **Password writeback** box and select **Next**.

8. [] On the **Enable single sign-on** page, select **Next**.

9. []  On the **Ready to configure** page, ensure that the **Start the synchronization process when configuration completes** checkbox is selected and select **Configure**.

10. [] On the **Configuration complete** page, select **Exit**.

11. [] In the Edge browser window open the **Azure portal**, navigate to the **Contoso - Overview** blade of the Contoso Azure AD tenant.

12. [] On the **Contoso - Overview** blade, select **Password reset** on the left under **Manage**. 

13. [] On the **Password reset - Properties** blade, Under **Self-service password reset enabled** choose **All**.

14. [] Select **Authentication methods** on the left under **Manage**.

15. [] On the **Password reset - Authentication methods** blade, set All **except** **Security questions**. 

      > **Note**: The **Office phone** method is not available in trial subscriptions.

16. [] Select **Save**.

18. [] Select **Registration** on the left and ensure that **Require users to register when signing in** is set to **Yes**

19. [] Select **On-premises integration** on the left and verify that the **Enable password write back for synced users** setting is **checked**.
    >Note that you have the option to **Allow users to unlock accounts without resetting their passwords**.

    **Questions**:

    - What is the goal of the Password Write Back?  
    ………………………………………………………………………………………………  

    - Is it the same functionnality of the Password Hash Sync?  
    ………………………………………………………………………………………………  

    - What is the goal of the SSPR?  
    ………………………………………………………………………………………………  

    - What is the number of days before users are asked to re-confirm their authentication information by default?  
    ………………………………………………………………………………………………  


**Summary**:  
In this exercise, you help employees to reset their passwords themself.

===
## (OPTIONAL) Exercise 5: Use a Service Principal to access to Microsoft Graph API
**Duration**: 15 minutes  
**Synopsis**:  
In this exercise, you will view how to configure and use a Service Principal. A Service Principal is App identity used connecting to the Azure AD or the Microsoft Graph API. 

>[!alert] During this exercice, don't close the Powershell ISE.

### Task 1: Create a Service Principal
 1. [] Log on to **@lab.VirtualMachine(SRV01).SelectLink**
    |||
    |:--------|:--------|
    | Username | **+++Administrator@contoso.com+++** |
    | Password | Please type the password |

 2. [] In a PowerShell ISE, execute the following commands:
![image](.\IMG\dxg2j8jn.jpg)
    ```
    Import-Module AzureADPreview
    Clear-AzContext -Force
    # Singn you with a Global Administrator Account
    $Tenant=Connect-AzureAD

    $AppObjectID = New-AzureADApplication -DisplayName ExportAzureADData

    $AppClientID = $AppObjectID.AppID
    $TenantID = $Tenant.TenantID

    Write-host "The App Client ID is: " $AppClientID
    Write-host "The Tenant ID is:     " $TenantID

    ```
![image](.\IMG\1vakzam3.jpg)

### Task 2: Generate a certificat to secure the Service Principal (SP)
>[!knowledge] You can use as "password" for the SP :
- a secret: generated for you, it's like a password with characters
- a certificat: the public key of a certificat generated by you. (it's the recommended method) 

>**Note**: You can use a certificate from your own PKI or a self-signed certificate.

1. [] Connect to the Microsoft Graph using Powershell. In the **SAME** PowerShell ISE, Open a **New Script** and copy/paste the following command:
    ```
    #Certificate Creation
    $cert = New-SelfSignedCertificate -CertStoreLocation "cert:\CurrentUser\My" `
    -Subject "CN=MSGraph_ReportingAPI" `
    -KeySpec KeyExchange `
    -KeyLength 2048
    $keyValue = [System.Convert]::ToBase64String($cert.GetRawCertData()) 

    #Set Certificate to the Service Principal
    New-AzureADApplicationKeyCredential -ObjectId $AppObjectID.ObjectId -Type AsymmetricX509Cert -Usage Verify -Value $keyValue `
    -EndDate $cert.NotAfter `
    -StartDate $cert.NotBefore  
    ```

Run the script

### Task 3: Add permissions to allow read Signin logs
1. [] In a Browser Open the Azure Portal +++https://portal.azure.com+++ and go to Azure AD Blade
![image](.\IMG\lkanntq5.jpg)  

2. [] Go to **App Registrations** and search your App +++ExportAzureADData+++
![image](.\IMG\5e5wzlpd.jpg)  

3. [] In **API Permission**, click **Add permissions**
![image](.\IMG\k31w2ut4.jpg)

4. [] Click on **Microsoft Graph**
5. [] On the Required permissions page, select **Application Permissions**.  
    - AuditLog.Read.All 

>[!tip] Depending of the information that you want read or write, you should search & checkbox permissions  

![image](.\IMG\30lpzp9d.jpg)

Select **Add permissions**.

>[!knowledge] Application permissions are used by apps that run without a signed-in user present, for example, apps that run as background services or daemons. Only an administrator can consent to application permissions. 

6. [] Click **Grant Admin consent for** & Check the grant
![image](.\IMG\2r9ao4hm.jpg)

### Task 4: Test the SP
1. [] Connect to the Microsoft Graph using Powershell. In the **SAME** PowerShell ISE, Open a **New Script** and copy/paste the following command:
```
$Thumbprint     = (Get-ChildItem cert:\CurrentUser\My\ | Where-Object {$_.Subject -eq "CN=MSGraph_ReportingAPI" }).Thumbprint     

$result=Connect-MgGraph -ClientId $AppClientID -TenantId $tenantId -CertificateThumbprint $Thumbprint

Get-MgAuditLogSignIn -all

```

Try now the command : **Get-MgUser -all**

**Questions**:  
- Why it's not working ? (Note: Insufficient priviledges as a answer is not enought, be more specific)  
    **Because the SP hasn't the right** ***API*** **permissions. (Note:API is the good term here)** 
- Write the solution here:  
    **Add the User.Read.all API Consent on the SP**


>**Note**: If you want test your solution, you must use Disconnect-MgGraph cmdlet and Re Connect-MgGraph to obtain the right permissions.

**Summary**:  
In this exercise, you created and use a Service Principal in a script to export Azure AD Signing logs as you use a service account to export Active Directory logs.

## (OPTIONAL) Exercise Op1: Observe a synchronization round
**Duration**: 15 minutes  
**Synopsis**:  
In this **OPTIONAL** exercise, you will force a synchronization round and observe how data is handled by AAD Connect.
The documentation to use for this task is located at +++https://docs.microsoft.com/en-us/azure/active-directory/hybrid/how-to-connect-sync-whatis+++ 
1. []	Log on to **@lab.VirtualMachine(SRV01).SelectLink**
    |||
    |:--------|:--------|
    | Username | **+++Administrator@contoso.com+++** |
    | Password | Please type the password |

2. []	Create a new user in your Active Directory environment. Username and passwords choice are left to you.  
3. []	Open the start menu and launch Synchronization Service  
    ![image](.\IMG\Picture1.png)
4. []	Focus on the Connector Operations list. Every sync operation adds an entry to the list. By default, operations are sorted in chronological order with the most recent on top.
If no sync operation has been started since you created the user (probability is low), force a sync round by running this Powershell command (launch Powershell as Administrator) :  
    +++Start-ADSyncSyncCycle -PolicyType Delta+++  
5. []	The command output should be similar to:    
    ![image](.\IMG\Picture2.png)  
And 6 additional lines should have appeared in the Synchronization Service Manager as shown in the picture:  
    ![image](.\IMG\Picture3.png)
6. []	Refer to the documentation to understand the role of each of the 6 steps in the synchronization round.  
Reading the article named Azure AD Connect sync: Understanding the architecture is highly recommended.  
7. []	Select the the Delta Import operation related to you Active Directory domain. In the Synchronization Statistics, the Add row should be a clickable link and display 1 as its value.  
    ![image](.\IMG\Picture4.png)
8. []	By clicking the Add link, a window will appear containing the list of new objects imported from Active Directory
    ![image](.\IMG\Picture5.png)
9. []	Double clicking on one of the objects will open a new window displaying attributes details.
10. []	Try to analyze the six steps and understand their meaning. Then, answer the additional questions.  

    **Questions**:  
    - How often does AAD Connect synchronizes your Azure AD tenant and Active Directory domain?  
    **30 minutes**  

    - Is it possible to manually force a synchronization? If yes, how can this be achieved?  
    **Yes, by using the Start-ADSyncSyncCycle cmdlet**  

    - Which synchronization step creates entry in the metaverse for new Active Directory objects?  
    **Delta or Full synchronization on the Active Directory connector (connector named after the on-premises domain)**  

    - Which synchronization step creates new Azure AD objects from data contained in the metaverse? 
    **Export on the Azure AD connector (connector named after the AzureAD tenant)**  


# END