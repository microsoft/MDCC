![image](.\IMG\MSCyberCoursesResized.png)

# The foundations of hybrid identity - Extending identities to the cloud with Azure AD

>Last Update September 2022

# Contents

<!-- TOC -->

- [Requirements](#requirements)
- [Azure Portal Navigation Tips](#azure-portal-navigation-tips)
- [MFA Enrollment](#mfa-enrollment)
- [Overview](#overview)
- [Hands-on lab step-by-step](#hands-on-lab-step-by-step)
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
  - [(OPTIONAL) Exercice 5: Use a Service Principal to access to Microsoft Graph API](#optional-exercice-5-use-a-service-principal-to-access-to-microsoft-graph-api)
        - [Task 1: Create a Service Principal](#task-1-create-a-service-principal)
        - [Task 2: Generate a certificat to secure the Service Principal (SP)](#task-2-generate-a-certificat-to-secure-the-service-principal-sp)
        - [Task 3: Add permissions to allow read Signin logs](#task-3-add-permissions-to-allow-read-signin-logs)
        - [Task 4: Test the SP](#task-4-test-the-sp)  
  - [(OPTIONAL) Exercice Op1: Observe a synchronization round](#optional-exercice-op1-observe-a-synchronization-round)

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
|![image](.\IMG\zxikevod.jpg)|You can **go back** to the previous Windows by using the navigation bar under the Azure Search bar |
|![image](.\IMG\9ag2up2j.jpg)|If you use the arrow **Go back** of you browser, you will lose the menu chain|  

# MFA Enrollment
For the MFA Enrollment you must have a Smartphone with at lease 2 of these 3 methods:
- SMS
- Email
- Authenticator (Microsoft, Google or others)

Security informations (means MFA) enrollment occurs after the first Sign-in with Account/Password when MFA is required
|||
|:--------|:--------|
|![image](.\IMG\jo80clem.jpg)|Select **Next**|
|![image](.\IMG\vsj1pd15.jpg)|Select **Next**|
|  **STOP HERE!** Choose your method||
|![image](.\IMG\odii03cu.jpg)| Select **Next** if you want use Microsoft Authenticator |
|![image](.\IMG\sebnmlp6.jpg)| Select **I want to use a different authenticator app** if you want use another Authenticator |
|![image](.\IMG\ascpsoy2.jpg)| Select **I want to set up a different method** if you want use Phone or Email instead of a authenticator|
| **If you choose Authenticator** ||
|![image](.\IMG\wbu8l84j.jpg)| **Scan the QR code** with your authenticator and follow the wizard|
| **If you choose Phone** ||
|![image](.\IMG\4ur49n7b.jpg)| Select France, enter your phone numbre, you will recieve a verified code|
| **If you choose Email** ||
|![image](.\IMG\lkgkbzgs.jpg)| Enter your email to received a verification code|
|![image](.\IMG\sabloawp.jpg)| You can modify your choice at +++https://aka.ms/setupmfa+++|
    
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

>[!KNOWLEDGE]The system will create a new Domain user account for the AAD Connect utility, you can find it in CN=Users.

6. [] Set AAD Connect to continue without matching all UPN suffixes to verified domains.
![image](.\IMG\AADCInstall2.png)

8. [] Set AAD Connect to sync all domains and OUs 
    >**Note**: In production, you can select some OU insteed All Domains.

    **Questions:**

    - What is the goal of the filtering  ?  
    .......................................................  

9. [] "Next" through the rest of the wizard.

10. [] At **Enable single sign-on**, enter your T0 Admin credential

    **Questions:**

    - What is the goal of SSO ?  
    ..............................................
    - Why is it Seamless ?  
    ............................................

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
- Enabling MFA on a per-user basis is not scalable to large environment. In real situations, there are other way to request MFA. Using the reference documentation, find these 2 other deployment options ?  
    .............................................. 

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
    - Why can't you change it ?  
    ..................................... 

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
    -   Do you find it ?  
    ...............................................
    -   Why ?  
    ............................................  

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
        ............................................... 
        - Why ?  
        ............................................ 

10. []Delete Jdoe Admin account
    - Go to User blade, select Jdoe Admin account and delete it
11. [] Using the documentation located at +++https://docs.microsoft.com/en-us/azure/active-directory/fundamentals/+++, restore it  

    **Questions**:  
        - Does John Doe have their groups membership restored ?  
        ...............................................   
        - Does John Doe have their roles membership restored ?  
        ...............................................  
        - Can you permanently delete a account ?  
        ...............................................

**Summary**:  
In this exercise, you discovered & manipulated Azure AD users & groups.

===
## Exercice 4: Enable Self Service Password Reset 
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
        ...............................................   
        - Does PAR_User3 account has a license assigned?  
        ...............................................  
        - Why ?  
        ...............................................
        
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

    - What is the goal of the Password Write Back ?  
    ..........................................
    - Is it the same functionnality of the Password Hash Sync ?  
    .........................................
    - What is the goal of the SSPR ?  
    ................................
    - What is the number of days before users are asked to re-confirm their authentication information by default?  
    ................................

**Summary**:  
In this exercise, you help employees to reset their passwords themself.

===
## (OPTIONAL) Exercice 5: Use a Service Principal to access to Microsoft Graph API
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
![image](.\IMG\nmkd3ojo.jpg)
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
![image](.\IMG\3xlwtg9d.jpg)


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
![image](.\IMG\ni9gdt4d.jpg)

2. [] Go to **App Registrations** and search your App +++ExportAzureADData+++
![image](.\IMG\k77emvr5.jpg)

3. [] In **API Permission**, click **Add permissions**
![image](.\IMG\qqbm0eyz.jpg)

4. [] Click on **Microsoft Graph**
5. [] On the Required permissions page, select **Application Permissions**.  
    - AuditLog.Read.All 

>[!tip] Depending of the information that you want read or write, you should search & checkbox permissions  

![image](.\IMG\6ks1yshr.jpg)

Select **Add permissions**.

>[!knowledge] Application permissions are used by apps that run without a signed-in user present, for example, apps that run as background services or daemons. Only an administrator can consent to application permissions. 

6. [] Click **Grant Admin consent for** & Check the grant
![image](.\IMG\ithmk77v.jpg)

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

## (OPTIONAL) Exercice OP1: Observe a synchronization round
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
    ![image](.\IMG\Picture16.png)
4. []	Focus on the Connector Operations list. Every sync operation adds an entry to the list. By default, operations are sorted in chronological order with the most recent on top.
If no sync operation has been started since you created the user (probability is low), force a sync round by running this Powershell command (launch Powershell as Administrator) :  
    +++Start-ADSyncSyncCycle -PolicyType Delta+++  
5. []	The command output should be similar to:    
    ![image](.\IMG\Picture17.png)

And 6 additional lines should have appeared in the Synchronization Service Manager as shown in the picture:  
    ![image](.\IMG\Picture18.png)

6. []	Refer to the documentation to understand the role of each of the 6 steps in the synchronization round.  
Reading the article named Azure AD Connect sync: Understanding the architecture is highly recommended.  
7. []	Select the the Delta Import operation related to you Active Directory domain. In the Synchronization Statistics, the Add row should be a clickable link and display 1 as its value.  
    ![image](.\IMG\Picture19.png)

8. []	By clicking the Add link, a window will appear containing the list of new objects imported from Active Directory
    ![image](.\IMG\Picture20.png)

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