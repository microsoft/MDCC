![image](.\IMG\MSCyberCoursesResized.png)

# Hands-on Lab **7: The secure management of hybrid identities Part 1**

>Last Update September 2022

# Contents
  
<!-- TOC -->

- [Requirements](#Requirements)
- [Azure Portal Navigation Tips](#azure-portal-navigation-tips)
- [MFA Enrollment](#mfa-enrollment)
- [Overview](#overview)
- [Hands-on lab step-by-step Part 1](#hands-on-lab-step-by-step-part-1)
    - [Abstract and learning objectives](#abstract-and-learning-objectives)
    - [Exercise 1: Finalize Breaking Glass Accounts creation](#exercise-1-finalize-breaking-glass-accounts-creation)
	    - [Task 1: Complete the BGA creation](#task-1-complete-the-bga-creation)
	    - [Task 2: Test the MFA](#task-2-test-the-mfa)
	    - [Task 3: Create the Conditional Access to force Third Party MFA for BGA 2](#task-3-create-the-conditional-access-to-force-third-party-mfa-for-bga-2)
    - [Exercise 2: Force MFA for all Privileged Administrators Roles, for all Users & block legacy AuthN](#Exercise-2-force-mfa-for-all-privileged-administrators-roles-for-all-users--block-legacy-authn)
	    - [Task 1: Create Conditional Access for our needs](#task-1-create-conditional-access-for-our-needs)
	    - [Task 2: Test the Conditional Access Policy](#task-2-test-the-conditional-access-policy)
    - [Exercise 3: Enable Hybrid Azure AD Join Device](#Exercise-3-enable-hybrid-azure-ad-join-device)
        - [Task 1: Enable Automatic Intune Enrollment](#task-1-enable-automatic-intune-enrollment)
	    - [Task 2: Install Azure AD Connect](#task-2-install-azure-ad-connect)
	    - [Task 3: hybrid Azure Active Directory join implementation](#task-3-hybrid-azure-active-directory-join-implementation)
	    - [Task 4: Joining the workstation to Azure AD – Hybrid Azure AD join](#task-4-joining-the-workstation-to-azure-ad-–-hybrid-azure-ad-join)
	    - [Task 5: Verifying that the workstation is Hybrid Azure AD join in the Portal](#task-5-verifying-that-the-workstation-is-hybrid-azure-ad-join-in-the-portal)
	    - [Task 6: Prepare the workstation for a next Exercise](#task-6-prepare-the-workstation-for-a-next-Exercise)

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
|![image](.\IMG\t8v50bbo.jpg)||  

You can **go back** to the previous Windows by using the navigation bar under the Azure Search bar  
|||
|:--------|:--------|
|![image](.\IMG\kqqp5av5.jpg)||  

If you use the arrow **Go back** of you browser, you will lose the menu chain.  

# MFA Enrollment
For the MFA Enrollment you must have a Smartphone with at lease 2 of these 3 methods:
- SMS
- Email
- Authenticator (Microsoft, Google or others)

Security informations (means MFA) enrollment occurs after the first Sign-in with Account/Password when MFA is required
|||
|:--------|:--------|
|![image](.\IMG\ly9bm4bs.jpg)|Select **Next**|
|![image](.\IMG\zv9iey32.jpg)|Select **Next**|
|  **STOP HERE!** Choose your method||
| ![image](.\IMG\1rgbutm2.jpg) | Select **Next** if you want use Microsoft Authenticator |
| ![image](.\IMG\3bp77qdc.jpg) | Select **I want to use a different authenticator app** if you want use another Authenticator |
| ![image](.\IMG\xnf5u2ja.jpg) | Select **I want to set up a different method** if you want use Phone or Email instead of a authenticator|
| **If you choose Authenticator** ||
| ![image](.\IMG\whr2pgbw.jpg) | **Scan the QR code** with your authenticator and follow the wizard|
| **If you choose Phone** ||
| ![image](.\IMG\66ym6mk6.jpg) | Select France, enter your phone numbre, you will recieve a verified code|
| **If you choose Email** ||
| ![image](.\IMG\kg91j4u0.jpg) | Enter your email to received a verification code|
| ![image](.\IMG\a7elfosz.jpg) | You can modify your choice at +++https://aka.ms/setupmfa+++|
    
===
# Overview

Contoso has resquest you to implement recommended management practices to manage hybrid identities in a secure manner and all necessary prerequisites to allow them to benefit from such Azure AD features as:
- Enhanced sign-in security with Multi-Factor Authentication with conditions
- Self-Service Password Reset  
- Enterprise Access Model with PAW  
- Privileged Identity Management  
- Azure AD Privileged Identity Protection

In summary, Contoso wants implementing Zero Trust or at least beginning.

# Hands-on lab step-by-step Part 1 

## Abstract and learning objectives 

In this hands-on lab you will setup and configure many Zero Trust of identity recommendation, how to:  
- Secure Tenant access by configuring Breaking Glass Accounts  
- Secure Tenant management by configuring basic Conditionnal Access Policy for High Privileged Accounts  
- Enable Hybrid Azure AD joined for Contoso Workstation  

The scenarios involve an Active Directory single-domain forest named contoso.com, which in this lab environment, consists (for simplicity reasons) of:
- 1 domain controller named DC01 (1 for LAB, in production 2 is mandatory)
- A single domain member server named SRV01
- A single workstation named CLI01
And an Azure AD (created by you in the LAB1) synced with your Active Directory to have Hybrid Identities used for cloud Apps. 

===
## Exercise 1: Finalize Breaking Glass Accounts creation
**Duration**: 30 minutes  
**Synopsis**: 
In this exercise you will finish the configuration of the Braking Glass Accounts. You will use Conditional Access to provide additional security controls for the Azure Portal.
In the below scenario, you want to ensure the BGA use MFA when they log in and access Azure Portal & all other applications.
This exercise can be completed on your normal machine, they do not need to be completed in the lab environment.

>[!Alert]We will reuse the tenant created in the previous LAB (**Extending identities to the cloud with Azure AD**) 

### Task 1: Complete the BGA creation
1. [] Log on to **@lab.VirtualMachine(SRV01).SelectLink**
    |||
    |:--------|:--------|
    | Username | **+++Administrator@contoso.com+++** |
    | Password | Please type the password |

2. [] Navigate to/search for the Azure AD Conditional Access settings: 
    +++https://portal.azure.com/#view/Microsoft_AAD_IAM/ActiveDirectoryMenuBlade/~/Overview+++  

    Signin with **Root** account

3. [] Find the +++Breaking Glass Account 1+++, Go to Properties and:
    - Edit **Contact Information** & Add a **other emails**
        - Enter your own email address or your school address.  
    >This will send you some interesting mails from the Tenant  

    - **Save** your changes

4. [] Select **Authentication methods**, on the left under **Manage**

5. [] Enter your phone number in **Phone** field (Format: +33 123456789) 
    - **Save** your changes

4. [] Do the same modification for the +++Breaking Glass Account 2+++

5. [] Navigate to **All Users** and Select the **Per-User MFA** button at the top menu.

6. [] Select your BGA 1 accounts and click Enable.

7. [] Re-Select your BGA 1 accounts and click Enforce

7. []  In **Azure AD**, Create a **security** Group Named +++SG_ExcludedFromAllCA+++  
    - Group Description: +++Members of this group will be excluded from all Conditional Access+++
    - Select **Azure AD roles can be assigned to the group** (at Yes)
    - Add **Breaking Glass Account 1**, **Breaking Glass Account 2** & **ALL** accounts beginning by +++Sync_+++

    **Questions**:  
    - Can a Group Administrator change member of this group?  
        ............................................... 
    - Who can also manage groups with this option (**Azure AD roles can be aassigned to the group**)?  
        ...............................................

    <details><summary>➡️**Note**: See a hint ?</summary>
    - Click to the **i** after **Azure AD roles can be aassigned to the group** to learn more
    </details>

### Task 2: Test the MFA
1. [] Open a new Private/Incognito browsing session and navigate to +++https://myapps.microsoft.com+++
2. [] Log into the service with the BGA 1 that was just MFA enabled.
    - you will be prompt for changing your current password: +++NeverTrustAny1!+++
    - I recommend you to reuse the same password + **!**
    ![image](.\IMG\5eqm6au1.jpg)

3. [] Set up the user's MFA configuration when prompted.

    **Questions**:  
    - Why this time your are directly prompt for MFA and not to enroll MFA?  
        ............................................... 

    >[!tip] During the Lab1, you test the MFA from the beginning. If you don't provide authnetication phone number, if account information (Login/password) are stolen and security information are not set, an attacker can register this security information from his location instead of the user. 

### Task 3: Create the Conditional Access to force Third Party MFA for BGA 2
in this task, you will create a Custom Control to simulate a third party MFA
1. [] Navigate to/search for the Azure AD Conditional Access settings: 
    +++https://portal.azure.com/#blade/Microsoft_AAD_IAM/ConditionalAccessBlade/Policies+++  
2. [] On **Custom control (Preview)**, on the left under **Manage**, add a **New custom control**

3. [] Replace the current json by this one:
+++{
  "Name": "Azure AD Contoso",
  "AppId": "bcdc5543-465a-4c20-82fc-adc2bee22451",
  "ClientId": "Contoso",
  "DiscoveryUrl": "https://connect.customMFA.com/bcdc5543-465a-4c20-82fc-adc2bee2245/.well-known/openid-configuration",
  "Controls": [
    {
      "Id": "RequireCustomMfa",
      "Name": "RequireCustomMfa",
      "ClaimsRequested": [
        {
          "Type": "CustomMfa",
          "Value": "MfaDone",
          "Values": null
        }
      ],
      "Claims": null
    }
  ]
}+++

    >**Note**: Remove all extra "]" & "}" created automatically if you use the automatic Copy/Paste feature to reflect the Json  

    ![image](.\IMG\CustomControl1_R.png)  

    >[!alert]It's an example, this custom control will not work, it's just to have an example when creating the Conditional Access.
4. [] Save the Custom Control

3. [] Go to **Overview** and Create a **new policy** not *New Policy from template*
    - You can name it: +++CA001-BGA2-Force 3rd party MFA for All Apps+++
    - In **Users or workload identities**
        - Include **Breaking Glass Account 2**  
        ![image](.\IMG\BGACA1_R.png)
    - In **Cloud apps or actions**
        - Select **All cloud apps**  
        ![image](.\IMG\BGACA2_R.png)
    - In **Grant**
        - Select your Custom control created **RequireCstomMFA**  
        ![image](.\IMG\BGACA3_R.png)

5. [] Select that the policy is enforced **on**
6. [] Create the policy.

    **Questions**:  
    If i used Report Only:
    - Will the CA be applied?  
        ...............................................
    - If i'm BGA2, Will I be prompt for MFA?  
        ............................................... 

### Final Task:
1. **Close all Internet browsers**

===
## Exercise 2: Force MFA for all Privileged Administrators Roles, for all Users & block legacy AuthN
**Duration**: 45 minutes  
**Synopsis**:  
In this exercise you will force MFA (and SIF) for All Administrators Roles, MFA for all Users and block legacy Authentification to remove all apps that does not satisfy MFA.

### Task 1: Create Conditional Access for our needs
1. [] Log on to **@lab.VirtualMachine(SRV01).SelectLink**
    |||
    |:--------|:--------|
    | Username | **+++Administrator@contoso.com+++** |
    | Password | Please type the password |   

1. [] Navigate to/search for the Azure AD Conditional Access settings: 
    +++https://portal.azure.com/#blade/Microsoft_AAD_IAM/ConditionalAccessBlade/Policies+++  
2. [] Create **New Policy from template**, and choose **Identity**
3. [] Select **Require multifactor authentication for all users**
    - Name Your policy: +++CA010-All-Require MFA for all apps anywhere+++
    - Select Report-Only  

    >This allow CA modification before enforcing it
4. [] Create Policy
5. [] Click on the **CA010** to modify the **Users or workload identities**
6. [] Exclude the group +++SG_ExcludedFromAllCA+++ and only this group, remove all other account
7. [] Select that the policy is enforced **on**
    
    >[!alert]During CA creation you will have this **alert**:![image](.\IMG\1ygzt02y.jpg)  
    **Select I understand that my account will be impacted....**

8. [] Save the policy.

9. [] Create a new CA from template **Require multifactor authentication for admins**
    - Name Your policy: +++CA020-Admins-Require MFA for all apps anywhere+++
    - Select Report-Only 
4. [] Create Policy
5. [] Click on the **CA020** to modify the **Users or workload identities**
6. [] Exclude the group +++SG_ExcludedFromAllCA+++ and only this group, remove all other account
1. [] In **Session**, select:
    - **Sign-in Frequency**: **10h**
    - **Persistent browser session** to **Never Persistent**
7. [] Select that the policy is enforced **on**
8. [] Save the policy.

    **Questions**:  
    - What is the role of the SIF, here 10h?  
        ............................................... 

2. [] Create **New Policy from template**
3. [] Select **Block legacy authentication**
    - Name Your policy: +++CA011-All-Block legacy authentication+++
    - Select Report-Only  
4. [] Create Policy
5. [] Click on the **CA011** to modify the **Users or workload identities**
6. [] Exclude the group +++SG_ExcludedFromAllCA+++ and only this group, remove all other account
7. [] Select that the policy is enforced **on**
8. [] Save the policy.

    **Questions**:  
    - What is Legacy AuthN?  
    ...............................................   
    - What other CA could we have done?  
    ............................................... 

    >[!knowledge]We haven't incremented the CA. Why this naming convention ? Because we dedicated 2nd Digit to a category:
    - 10-19 for Users
    - 20-29 for Admins
    You can do this for Guest, for Service Principal, ......

### Task 2: Test the Conditional Access Policy
In this task, we will sign-in to the **Entra Portal** (the identity portal, now in preview) as a member of the Global Administrator to verify our Conditional Access Policy is working as planned.


1. [] Open a new Private/Incognito browsing session and navigate to +++https://entra.microsoft.com+++.
2. [] Log in as +++admaz-jdoe+++ / Password: +++NeverTrustAny1!+++  
    >If the conditional access policy was created successfully, this user should be prompted to set up MFA on their account.  

3. [] Complete the MFA registration
4. [] It's working ! :)

    **Questions**:  
    - What is solutions to be not prompt for the MFA? (You are 5 solutions, enumerate them. Note:It's not recommended but it's for a brainstorming :) )  
    ............................................... 

### Final Task:
1. **Close all Internet browsers**

===
## Exercise 3: Enable Hybrid Azure AD Join Device
**Duration**: 30 minutes  
**Synopsis**:  
In this exercise you will Configure the Hybrid Azure AD Join & the Intune Enrollment to have a modern device management. Hybrid Azure AD join is mandatory for Intune enrollment, for Passwordless, for some CA restriction, and for many other Azure features.

### Task 1: Enable Automatic Intune Enrollment
In this task, you will enable automatic enrollment of hybrid Azure AD devices into Intune. 

1. [] Log on to **@lab.VirtualMachine(SRV01).SelectLink**
    |||
    |:--------|:--------|
    | Username | **+++Administrator@contoso.com+++** |
    | Password | Please type the password |

2. []  Navigate to/search for the Intune portal:  
    +++https://endpoint.microsoft.com+++

2. [] On the **Microsoft Endpoint Manager admin center** page, select **Devices** on the left navigation.
3. [] On the **Devices** blade, select **Enrolled devices** under **Device enrollment** on the left.
4. [] On the **Windows enrollment** blade, select **Automatic Enrollment**.
5. [] On the **Configure** blade, set **MDM user scope** to **All** and select **Save**.


### Task 2: Install Azure AD Connect
>[!alert]Due to the lab restriction, you must re-install Azure AD Connect.  

1. [] Download Azure AD Connect from +++https://www.microsoft.com/en-us/download/details.aspx?id=47594+++  
2. [] Start the installation in **customize** mode **NOT** use **express settings** 
    >[!TIP]No custom options are necessary, just press install.

3. [] Select:
    - [] **Password Hash Synchronization**
    - [] **Enable single sign-on**

4. [] Log in with your Azure AD Global Admin account.
5. [] Add Contoso.com and log in with the Domain admin credentials:
    - Ensure that "Create new AD account" stays selected.
    - Username: +++CONTOSO\Administrator+++
    - Password: +++NeverTrustAny1!+++
    >[!KNOWLEDGE]The system will create a new local user account for the AAD Connect utility, you can find it in CN=Users.

6. [] Set AAD Connect to continue without matching all UPN suffixes to verified domains.
    ![image](.\IMG\AADCInstall2.png)

8. [] Set AAD Connect to sync all domains and OUs 
    >**Note**: In production, you can select some OU insteed All Domains.

9. [] click "Next" **until Optional features** .
10. [] At **Optional features**, select:
    - [] **Password write back**
10. [] At **Enable single sign-on**, enter your T0 Admin credential
11. [] At **Ready to configure**, leave by default & press **Install**

## Task 3: hybrid Azure Active Directory join implementation
In this task, you will configuring hybrid Azure Active Directory join implementation.
The reference documentation to use is located at +++https://docs.microsoft.com/en-us/azure/active-directory/devices/hybrid-azuread-join-managed-domains+++
>[!tip]Here, we will configure hybrid Azure AD join by using Azure AD Connect but you could configure it by using a GPO, see +++https://docs.microsoft.com/en-us/azure/active-directory/devices/hybrid-azuread-join-control+++

1.	[] On the desktop, Launch **Azure AD Connect**
3.	[] Select **Configure**.
4.	[] At Additional tasks, select **Configure device options**
5.	[] At Overview, select **Next**.
6.	[] At Connect to Azure AD, **enter the credentials** of a global administrator for your Azure AD tenant.
7.	[] Select Configure Hybrid Azure AD join, and then select Next.
8.	[] Select **Windows 10 or later domain-joined devices**, and then select Next.

    **Questions**:  
    - What is the device registering Process for the downlevel Windows System?  
        ...............................................  

9.	[] At **SCP configuration**, for each forest where you want Azure AD Connect to configure the SCP, complete the following steps, and then select Next.
    ![image](.\IMG\Hybrid2.png)  

    >If you click on Download ConfigureSCP.ps1 Button, you should view the PowerShell code to configure the Service Connection Point (SCP).  

    **Questions**:  
    - By using the script, what is the DN of the SCP?  
        ............................................... 
	- Where can i find the SCP, in which naming context?  
        ...............................................  
	 
10.	[] At Ready to configure, select **Configure**.
11.	[] At Configuration complete, select **Exit**.
  
### Task 4: Joining the workstation to Azure AD – Hybrid Azure AD join
Let’s now register the client as a hybrid Azure AD Join device.

1.	[] Log on to **@lab.VirtualMachine(CLI01).SelectLink**
    |||
    |:--------|:--------|
    | Username | **+++David@contoso.com+++** |
    | Password | **+++1LoveSecurity!+++**  |

2.	[] Reboot the workstation
3.	[] As soon as the device has rebooted, open a session with +++David@contoso.com+++
4.	[] Open an **ELEVATED** command prompt (with administrative rights) and type: (use local administrator +++.\administrator+++/+++NeverTrustAny1!+++) 

    +++dsregcmd /join+++

5.	[]	Wait a few second and open a command prompt (**with user rights**) then evaluate the join status by typing
    **dsregcmd /(find the right setting)**

    **Questions**:  
    - What is the command to view the status?  
        ............................................... 
 
6.	[]	Review the following fields and make sure that they have the expected values (if not read the alert):
    - AzureAdJoined : **Yes**
    - DomainJoined : **Yes**

     >[!alert]If the device doesn't show as Azure AD-joined yet might be because the computer object hasn't been synced to Azure AD yet. You need to wait for the Azure AD Connect sync to complete and the next join attempt after sync completion will resolve the issue. The default synchronization frequency of Azure AD connect is 30 minutes.  
    But you are lucky, you can speed up the process, by running the following commands on the @lab.VirtualMachine(SRV01).SelectLink.  
    - Open an ELEVATED PowerShell (Admin) console.
    - Force Azure AD Sync to perform synchronization  
    +++cd "C:\Program Files\Microsoft Azure AD Sync\bin\ADSync"+++  
    +++Import-Module .\ADSync.psd1+++  
    +++Start-ADSyncSyncCycle -PolicyType Delta+++  
    - After the Sync, wait at least 1 min
    - Re-try from the point 4.

    >[!tip]- **AzureAdJoined**: This field indicates whether the device is joined. The value will be YES if the device is either an Azure AD joined device, or a hybrid Azure AD joined device.  If the value is NO, the join to Azure AD has not completed yet.  
    - **DomainJoined**: This field indicates whether the device is joined to an on-premises Active Directory or not. If the value is NO, the device cannot perform a hybrid Azure AD join. 

**Questions**:  
    - Copy/Paste a screenshot of your result.  
        ............................................... 

### Task 5: Verifying that the workstation is Hybrid Azure AD join in the Portal
1. [] Log on to **@lab.VirtualMachine(SRV01).SelectLink**
    |||
    |:--------|:--------|
    | Username | **+++Administrator@contoso.com+++** |
    | Password | Please type the password |

1.	Open a browsing session and navigate to +++https://entra.microsoft.com+++ 
2.	Sign in with your Global Admin Azure AD account, you’ve created before
3.	Deploy **Azure Active Directory**, **Devices** & click on **All devices**
4.	Verify if the Workstation that the Join Type is Hybrid Azure AD joined
    ![image](.\IMG\dsregcmd2.png)
5.	Close the browser

### Task 6: Prepare the workstation for a next Exercise
1.	[] Log on to **@lab.VirtualMachine(CLI01).SelectLink**
    |||
    |:--------|:--------|
    | Username | **+++David@contoso.com+++** |
    | Password | **+++1LoveSecurity!+++**  |

2. [] Open a **elevated** Powershell prompt, enter:  
    +++Remove-Computer -restart+++
    ![image](.\IMG\uzpyom91.jpg)  
    Enter **Y** to continue 

3. [] Done !

**Summary**  
In this exercise, you configure and performed Hybrid Azure AD join

===

# End