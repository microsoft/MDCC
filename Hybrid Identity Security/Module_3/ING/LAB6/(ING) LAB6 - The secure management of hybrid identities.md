![image](.\IMG\MSCyberCoursesResized.png)

# Hands-on Lab **6: The secure management of hybrid identities**

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
- [Hands-on lab step-by-step Part 2](#hands-on-lab-step-by-step-part-2)
    - [Exercise 4: Use the least priviledge](#exercise-4-use-the-least-priviledge)
	    - [Task 1: Enable Privileged Identity Management for Global Administrators](#task-1-enable-privileged-identity-management-for-global-administrators)
	    - [Task 2: Enable Privileged Identity Management for User Administrators](#task-2-enable-privileged-identity-management-for-user-administrators)
	    - [Task 3: Complete PIM Assignment](#task-3-Complete-pim-assignment)
	    - [Task 4: Use the User Administrator role](#task-4-use-the-user-administrator-role)
    - [Exercise 5: Identity Protection](#exercise-5-identity-protection)
	    - [Task 1: Use Identity Protection to set user risk policy](#task-1-use-identity-protection-to-set-user-risk-policy)
	    - [Task 2: Block Administrators if Signin risk is high](#task-2-block-administrators-if-signin-risk-is-high)
	    - [Task 3: Simulate a Sign-in Risk High](#task-3-simulate-a-sign-in-risk-high)
    - [(OPTIONAL) Exercise 6: Explore Azure AD Logs](#optional-exercise-6-explore-azure-ad-logs)
	    - [Task 1: Explore Signin Logs](#task-1-explore-signin-logs)
	    - [Task 2: Explore Audit Logs](#task-2-explore-audit-logs)
    - [(OPTIONAL) Exercise 7: Windows Hello for Business](#optional-exercise-7-windows-hello-for-business)
	    - [Task 1: Deploy Azure AD kerberos](#task-1-deploy-azure-ad-kerberos)
	    - [Task 2:Create & link the Windows Hello for Business Group Policy object](#task-2-create--link-the-windows-hello-for-business-group-policy-object)
	    - [Task 3: Test Windows Hello for Business](#task-3-test-windows-hello-for-business)

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
|![image](.\IMG\t8v50bbo.jpg)|You can **go back** to the previous Windows by using the navigation bar under the Azure Search bar |  
|![image](.\IMG\kqqp5av5.jpg)|If you use the arrow **Go back** of you browser, you will lose the menu chain|  

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

5. [] Enter your phone number in **Phone** field (Format: +33 12345678)  
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
    ………………………………………………………………………………………………
    - Who can also manage groups with this option (**Azure AD roles can be assigned to the group**)?      
    ………………………………………………………………………………………………

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
    - Why this time you are directly prompt for MFA and not to enroll MFA?      
    ………………………………………………………………………………………………

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
    ………………………………………………………………………………………………
    - If i'm BGA2, Will I be prompt for MFA?      
    ………………………………………………………………………………………………

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

    **Question**:  
    - What is the role of the SIF, here 10h?      
    ………………………………………………………………………………………………

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
    ………………………………………………………………………………………………
    - What other CA could we have done?      
    ………………………………………………………………………………………………

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
    - What are solutions to be not prompt for the MFA? (You are 5 solutions, enumerate them.     
    ………………………………………………………………………………………………  
    >Note: It's not recommended but it's for a brainstorming :) ) 

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

    **Question**:  
    - What is the device registering Process for the downlevel Windows System?      
    ………………………………………………………………………………………………  

9.	[] At **SCP configuration**, for each forest where you want Azure AD Connect to configure the SCP, complete the following steps, **CLICK ADD** and then select Next.
    ![image](.\IMG\Hybrid2.png)  

    >Note: If you click on Download ConfigureSCP.ps1 Button, you should view the PowerShell code to configure the Service Connection Point (SCP).  

    **Questions**:  
    - By using the script, what is the DN of the SCP?      
    ………………………………………………………………………………………………
	- Where can i find the SCP, in which naming context?      
    ……………………………………………………………………………………………… 
	 
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

5. [] Wait a few second and open a command prompt (**with user rights**) then evaluate the join status by typing
    **dsregcmd /(find the right setting)**

    **Question**:  
    - What is the command to view the status?      
    ………………………………………………………………………………………………
 
6. [] Review the following fields and make sure that they have the expected values (if not read the alert):
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

**Question**:  
    - Copy/Paste a screenshot of your result.      
    ………………………………………………………………………………………………

### Task 5: Verifying that the workstation is Hybrid Azure AD join in the Portal
1. [] Log on to **@lab.VirtualMachine(SRV01).SelectLink**
    |||
    |:--------|:--------|
    | Username | **+++Administrator@contoso.com+++** |
    | Password | Please type the password |

1. [] Open a browsing session and navigate to +++https://entra.microsoft.com+++ 
2. []	Sign in with your Global Admin Azure AD account, you’ve created before
3. []	Deploy **Azure Active Directory**, **Devices** & click on **All devices**
4. []	Verify if the Workstation that the Join Type is Hybrid Azure AD joined
    ![image](.\IMG\dsregcmd2.png)
5. []	Close the browser

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
# Hands-on lab step-by-step Part 2

In this hands-on lab you will setup and configure many Zero Trust of identity recommendation, how to:  
- Secure Tenant access by configuring Breaking Glass Accounts  
- Secure Tenant management by configuring basic Conditionnal Access Policy for High Privileged Accounts  
- Enable Hybrid Azure AD joined for Contoso Workstation  
- Implement least privileged management with PIM  
- Use Identity Protection  
- Exploit Azure AD Logs  
- Enable Windows Hello for Business  

The scenarios involve an Active Directory single-domain forest named contoso.com, which in this lab environment, consists (for simplicity reasons) of:
- 1 domain controller named DC01 (1 for LAB, in production 2 is mandatory)
- A single domain member server named SRV01
- A single workstation named CLI01

===
## Exercise 4: Use the least priviledge
**Duration**: 40 minutes  
**Synopsis**:  
In this exercise you will configure Privileged Identity Management for Global Administrator and User Administrator Roles. This is the base for using least priviledge in Azure.

### Task 1: Enable Privileged Identity Management for Global Administrators
1. [] Log on to **@lab.VirtualMachine(SRV01).SelectLink**
    |||
    |:--------|:--------|
    | Username | **+++Administrator@contoso.com+++** |
    | Password | Please type the password |

1.	[] Open a browsing session and navigate to +++https://portal.azure.com+++
2. [] In **Azure AD**, Create a **security** Group Named +++SG_GlobalAdmins+++  
    - Group Description: +++This group is used to provide Global Administrator role+++
    - Select **Azure AD roles can be aassigned to the group** (at **Yes**)
    - Add **admaz-jdoe** & **root** accounts as Members
3. [] Create a **security** Group Named +++SG_UserAdmins+++  
    - Group Description: +++This group is used to provide User Administrator role+++
    - Select **Azure AD roles can be aassigned to the group** (at **Yes**)
    - Add **Admin_Par** as Member
4. [] Create a **security** Group Named +++SG_PAR_UserAdmins+++  
    - Group Description: +++This group is used to provide Paris User Administrator role+++
    - Select **Azure AD roles can be aassigned to the group** (at **Yes**)
    - Add **Admin_Par** as **Owner**
    - Add **Admin_Par** as Member

5. [] Search PIM in the Azure portal search bar & Open **Azure AD Privileged Identity Management** or use +++https://portal.azure.com/#view/Microsoft_Azure_PIMCommon/CommonMenuBlade/~/quickStart+++
6. [] Select **Azure AD roles** on the left 
7. [] Select **Roles** on the left & select **Global Administrator**
    ![image](.\IMG\PIM1.png)
8. [] Select **Role settings** on the left & select **Edit**
9. [] Configure the Activation maximum duration (hours) at 4h

    **Questions**:  
    - By default, the configuration of "On activation, require Azure MFA" is it at the good configuration?      
    ………………………………………………………………………………………………
    - What is the difference between this setting and a conditional access that force MFA?      
    ………………………………………………………………………………………………

10. [] Select **Next: Assignment**, take some secondes to view options
11. [] Select **Next: Notification**, take some secondes to view options

    **Questions**:  
    - When I add someone in the GA role:
        - All accounts member of this role will be notify?      
    ………………………………………………………………………………………………
        - Is it possible to notify the Security Team?      
    ………………………………………………………………………………………………
        - Is it possible to notify manager of each account automatically? Manually?      
    ………………………………………………………………………………………………
    - When I active my GA role:
        - All accounts member of this role will be notify?      
    ………………………………………………………………………………………………
        - Is it possible to notify the Security Team?      
    ………………………………………………………………………………………………
        - Is it possible to delegate the approbation to active my role?      
    ………………………………………………………………………………………………

12. [] Select Update & re-select **Global Administrator**
13. [] Add assignments & select +++SG_GlobalAdmins+++
14. [] Let's by default the Setting

    **Questions**:  
    - What is the difference between Eligible & Active?      
    ………………………………………………………………………………………………
    - What is the impact if I uncheck **Permanently eligible** setting?      
    ………………………………………………………………………………………………

15. [] Select **Assign**
16. [] After few minutes and if you assigned a alternate email to BGA accounts, you will receive email from PIM because you **ADD** someone as GA Role
    ![image](.\IMG\PIMGA1.png)
    ![image](.\IMG\PIMGA2.png)
12. [] Select Update & re-select **Global Reader** **NOT** *Global Administrator**
13. [] Add assignments & select +++SG_GlobalAdmins+++
20. [] At **Setting**, Select **Active** & provide a justification: **For Test LAB PIM**
15. [] Select **Assign**

### Task 2: Enable Privileged Identity Management for User Administrators
1. [] Select **Roles** on the left & select **User Administrator**
2. [] Select **Role settings** on the left & select **Edit**
3. [] Configure the Activation maximum duration (hours) at 10h

    **Questions**:  
    - Do you have a idea, why we set GA at 4h and UA at 10h?      
    ………………………………………………………………………………………………
    - If I enable my User Administrator role at 9h00, I close my browser at 13h to lunch and Open a new session at 14h, do I keep my User Administrator role or do I need to re-grant me in PIM?      
    ………………………………………………………………………………………………

4. [] Require that **Root** account **approve** the grant (activation) in **selecte approvers**
    ![image](.\IMG\PIMUA1.png)


5. [] Select **Update** & re-select **User Administrator**
6. [] Add assignments & select +++SG_UserAdmins+++
7. [] Let's by default the Setting
8. [] Select **Assign**
9. [] After few minutes and if you assigned a alternate email to BGA accounts, you will receive email from PIM because you **ADD** someone as GA Role
10. [] Go to Azure Active Directory user settings blade: +++https://portal.azure.com/#view/Microsoft_AAD_IAM/ActiveDirectoryMenuBlade/~/UserSettings+++
11. [] At Administration portal, select **Yes** to **Restrict access to Azure AD administration portal**  
12. [] Select **Save**  

    **Questions**:  
    - What is the impact of this setting?      
    ………………………………………………………………………………………………
    - is it the best solution?      
    ………………………………………………………………………………………………
    <details><summary>➡️**Note**: See a hint ?</summary>
    - Click on **learn more**
    </details>

## Task 3: Complete PIM Assignment
1. [] Navigate to PIM or use +++https://portal.azure.com/#view/Microsoft_Azure_PIMCommon/CommonMenuBlade/~/quickStart+++
2. [] Select **My roles** at the left
3. [] Select **Active assignments**  

    **Question**:  
    - What do you see?      
    ………………………………………………………………………………………………

4. [] **Remove admaz-jdoe** from the **Active assignment** in the GA role using **PIM** +++https://portal.azure.com/#view/Microsoft_Azure_PIMCommon/ResourceMenuBlade/~/roles/resourceId//resourceType/tenant/provider/aadroles+++

    >**Note**: You cannot remove yourself from GA, you should request this change from another GA or Privileged Role Administrators

5. [] Sign out **Root** account, click on the face icon at the upper right  
    ![image](.\IMG\Signout1.png)
6. [] Signin with +++admaz-jdoe@+++**YourTenantName**.onmicrosoft.com
7. [] Remove **Root** from the Active assignment for GA role using **PIM**

    **Question**:  
    - is it working?      
    ………………………………………………………………………………………………

8. [] Close Global Administrator assignments panel and navigate to +++https://portal.azure.com/#view/Microsoft_Azure_PIMCommon/CommonMenuBlade/~/quickStart+++
9. [] Select **My Roles** and select Activate to active your GA Role
10. [] Follow the Wizard

    **Question**:  
    - We selected prompt for MFA when we activate the role! Why I didn't have a MFA Prompt ?      
    ………………………………………………………………………………………………

11. [] If you select Acitve assignments, you will see your role Activated
12. [] Now navigate to the GA role. **PIM Quick start** --> **Azure AD Roles** --> **Roles** in the Manage section and Remove **Root** from the Active Assignment

    **Questions**:  
    - How many accounts are in active assignment permanently ?      
    ………………………………………………………………………………………………
    - How could you stop your active assignment if you finished your task before the 4h ?      
    ………………………………………………………………………………………………

### Task 4: Use the User Administrator role
In this task you will use the User Administrator role to understand how roles works & find them.  
1. [] Open a InPrivate browsing session, and navigate to +++https://portal.azure.com+++  
2. [] Signin with +++Admin_PAR@+++yourTenantName.onmicrosoft.com / **Password**: +++NeverTrustAny1!+++
3. [] Finish the MFA Enrollment by providing your security information
3. [] Go to Azure **Active Directory**

    **Questions**:  
    - Why do you have an Access Denied ? I'm a User Administrator, no ??      
    ………………………………………………………………………………………………

4. [] Resolve this normal behaivior with **PIM**  

    >You must approved the request from Admin_PAR  
    ![image](.\IMG\r7t1u18g.jpg)  

5. [] On your "normal" browser, Sign out **admaz-jdoe** session & open a session with **Root** account
6. [] Go to **PIM**, select **Approve request** at the left
7. [] Select Admin_PAR request & select **approve**
    ![image](.\IMG\d6ig1q3o.jpg)  

8. [] Return to your inprivate browser session   
5. [] Now go to **Azure Active Directory**
6. [] Select **Roles & Administrators**
7. [] Search & select **User Administrator**
8. [] Select **Description** at the left & review permissions of this role

    **Questions**:  
    If i'm User Administrator, can I
    - Create User?      
    ………………………………………………………………………………………………  
    - Delete a Group?      
    ………………………………………………………………………………………………
    - Modify members of a Group?      
    ………………………………………………………………………………………………
    - Delete a Device?      
    ………………………………………………………………………………………………

9. [] Return to **Contoso Overview page** by using the menu navigation bar
10. [] Select Groups & search +++SG_UserAdmins+++
11. [] Add a user to this group

    **Questions**:  
    - Does it work?      
    ………………………………………………………………………………………………  
    - Why? You are a User Administrator?      
    ………………………………………………………………………………………………

12. [] Select Groups & search +++SG_PAR_UserAdmins+++
13. [] Add a user to this group

    **Questions**:  
    - Does it work?      
    ………………………………………………………………………………………………  
    - Why? It's the same kind of group than SG_UserAdmins?      
    ………………………………………………………………………………………………  
    - Is it possible to restore a security group when I a User Administrator?      
    ………………………………………………………………………………………………  
    <details><summary>➡️**Note**: See a hint ?</summary>
    The answer is in the first paragraph : +++https://docs.microsoft.com/en-us/azure/active-directory/enterprise-users/groups-restore-deleted+++
    </details>

**Summary**  
In this exercise, you configure and use Privileged Identity Management to have the least priviledge and grant you in a role.

>[!tip] When you use Directory role as User Administrator your scope is the Directory, for this role, all users of the directory. If you want restrict a piece of users, groups or devices, you must use Administrative Unit and scope the role for this Administrative Unit. For more detail view this article +++https://docs.microsoft.com/en-us/azure/active-directory/roles/administrative-units+++

===
## Exercise 5: Identity Protection
**Duration**: 20 minutes  
**Synopsis**:  
In this exercise you will configure Identity Protection to block Signin Risk High for all Administrators. You will explore how to use it.

### Task 1: Use Identity Protection to set user risk policy
1. [] Log on to **@lab.VirtualMachine(SRV01).SelectLink**
    |||
    |:--------|:--------|
    | Username | **+++Administrator@contoso.com+++** |
    | Password | Please type the password |

    >[!Knowledge]You have 2 way to use Identity Protection:
    - Using Identity Protection policies
    - Using Conditionnal Access (recommended method)

2. [] Go to Azure Active Directory: +++https://portal.azure.com/#view/Microsoft_AAD_IAM/ActiveDirectoryMenuBlade/~/Overview+++

3. [] Select **Security** at the left

4. [] Select **Identity Protection** at the left

5. [] Select **User risk policy** at the left

6. [] In a ideal world, we will select All users but for our lab we will select only in the **Assignments**:  

    >[!Alert]Now PIM is activated, if you cannot make modification, Go to PIM to grant you to GA role.

    - All PAR_Users (6 users)
    - User risk: **Medium**
    - Controls: **Allow access** + check **Require password change**
    - Enforce policy: **On**

7. [] Click **Save**

### Task 2: Block Administrators if Signin risk is high
1. [] Go to Azure Active Directory: +++https://portal.azure.com/#view/Microsoft_AAD_IAM/ActiveDirectoryMenuBlade/~/Overview+++

3. [] Select **Security** at the left

3. [] Select **Conditional Access** at the left

4. [] Create new CA by choosing **New Policy** 

3. [] Select **Require multifactor authentication for all users**
    - Name Your policy: +++CA021-Admins-Block if signin risk is high from anywhere+++
    - At **Users or workload identities**,
        - Include:  Directory Role: Global Administrator
                    Users & Groups : +++SG_Privileged_Accounts+++
        - Exclude: Users & Groups : +++SG_ExcludedFromAllCA+++
    - At **Cloud apps or actions, select **All cloud apps**
    - At **Conditions**, select **Sign-in risk level**
        - Select *Configure* at **Yes**
        - Check only **High**
    - At **Grant**, select **Block**  

    >[!alert]During CA creation you will have this **alert**:![image](.\IMG\1ygzt02y.jpg)  
    **Select I understand that my account will be impacted....**
 
    ![image](.\IMG\tbugm127.jpg)  

7. [] Select that the policy is enforced **on**

8. [] Save the policy.

### Task 3: Simulate a Sign-in Risk High
1. [] Select **What if** near *New policy*
    >What if is to test your Conditional Access Policies

2. [] Select the user +++**admaz-jdoe**+++
3. [] Select Sign-in risk: **High**
4. [] Select **What if**

    **Questions**:  
    - How are policies are applied? The first win, the one with the smallest number or something else ?      
    ………………………………………………………………………………………………
    - Which policy(ies) will be applied?      
    ………………………………………………………………………………………………
    - What will be the result for admaz-Jdoe? Can he access to a app? if yes with which condition (MFA,.....)?      
    ………………………………………………………………………………………………*  

5. [] Close all your Internet browsers
6. [] Use TOR browser: navigate to c:\tools\Tor Browser
    >The browser can be really slow! if very slow, close the browser and retry

7. [] Select **Connect**


7. [] Navigate to +++https://portal.azure.com/+++ & sign-in with **admaz-jdoe** / Password: maybe +++NeverTrustAny1!+++ + **!** ??

    **Question**:  
    - It's working? What is the error message?      
    ………………………………………………………………………………………………

8. [] Open Edge and navigate to +++https://portal.azure.com/+++ & sign-in with **admaz-jdoe**

    **Questions**:  
    - it's working?      
    ………………………………………………………………………………………………
    **Questions**:  
    - Why it's doesn't work with the Tor Browser but it's work with Edge, on the same server, the same location, the same apps?      
    ………………………………………………………………………………………………

9. [] Do the same experience, with **Par_User4@'YourTenantDomainName'.onmicrosoft.com** / Password: +++1LoveSecurity!+++
    - Begin with the TOR browser
    - Continue in inprivate with Edge

    **Questions**:  
    - What's happen? why this behavior? It's the same test, the same server, the same location, the same apps!      
    ………………………………………………………………………………………………
    ![image](.\IMG\r4184nrc.jpg)



9. [] With Edge Navigate to **Identity Protection** blade +++https://portal.azure.com/#view/Microsoft_AAD_IAM/IdentityProtectionMenuBlade/~/Overview+++

10. [] Select **Risk Detections**, Explore **Risky Users** at left 

    >These menus show you all users at risk.  

    ![image](.\IMG\wikbg0l2.jpg)

    **Questions**:  
    - What's the risk for admaz-jdoe?      
    ………………………………………………………………………………………………
    - what's the Detection timing for this risk?      
    ………………………………………………………………………………………………

    >[!Knowledge]With this configuration, Admaz-Jdoe can work from a safe location, but Par_User4 not anymore!  

**Summary**  
In this exercise, you configure Identity Protection to block Risky sign-in for a group of privileged Accounts with Conditional Access. This configuration don't totaly block the account, only risky sign-in. You also configure a password change requirement for Risky Users with Identity Protection policies.

===
## (OPTIONAL) Exercise 6: Explore Azure AD Logs
**Duration**: 15 minutes  
**Synopsis**:  
In this exercise you will explore the Azure AD Signin Logs and Audit Logs. A wealth of information!

### Task 1: Explore Signin Logs
1. [] Log on to **@lab.VirtualMachine(SRV01).SelectLink**
    |||
    |:--------|:--------|
    | Username | **+++Administrator@contoso.com+++** |
    | Password | Please type the password |

2. [] Go to Azure Active Directory: +++https://portal.azure.com/#view/Microsoft_AAD_IAM/ActiveDirectoryMenuBlade/~/Overview+++

3. [] Click on Sign-in logs 
    ![image](.\IMG\bjbfuhnf.jpg)

4. [] Apply a filter to only display the username admaz-jdoe

5. [] Select the first ligne where the status is **Failure**
   
    **Question**:  
    - What's the failure reason?      
    <font style="color:blue;">Access has been blocked by Conditional Access Policies </font>  


6. [] Navigate under:
    - **Location**: You can see your location & IP Address
    - **Device info**: You can see your Browser & mostly if the device is compliant and the join type
    - **Authentication Details**: You can see which kind of method the account used for the AuthN
   
    **Question**:  
    - What's the Authentication method use by Jdoe?      
    <font style="color:blue;">Password in the cloud, ndlr: PHS </font>


7. [] Select **Conditional Access**
  
    **Question**:  
    - See the result of CA021? Why is it Failure? Do that mean the failed to be applied? what does it mean?      
    <font style="color:blue;">No, it means that the CA was applied but the result has been failed to logon </font>

8. [] Click on the CA21 Policy Name  

    >You can review the result of the CA processing.  
    You can see that the **Sign-in risk High** had **Matched** and the **Grand Controls** applied has been **Block**

>[!Tip]Use **the cross** at **upper right** to close the windows and return to the logs

9. [] Do the same with the user PAR_User4, apply Username filter and find the first **Failure status**.
    
    **Questions**:  
    - What's the Failure reason?      
    <font style="color:blue;">Password Change is required due to account risk </font>
      - See the result of CA021? Why this result?      
    <font style="color:blue;">It's not applied because the User Not Matched </font>
    
    >If you click on the Policy Name you can open it and view it into read only mode.

### Task 2: Explore Audit Logs
1. [] Go to Azure Active Directory: +++https://portal.azure.com/#view/Microsoft_AAD_IAM/ActiveDirectoryMenuBlade/~/Overview+++

1. [] Click on **Audit logs** under Sign-in logs

    >[!knowledge]These logs show you all the change made in Azure AD

2. [] In the filter select **Service**: **PIM**
3. [] In the filter select **Target**: +++admaz-jdoe+++
4. [] Select one audit & review the Activity tab
5. [] Select **Modified Properties**
   
    **Question**:  
    - What's the target of Modified Properties?      
    <font style="color:blue;">Global Administrator</font>

**Questions**:
- Write the filter to found who add admaz-jdoe to Global Administrator?  
    >Not all filters will be used, don't forget we don't assign admaz-jdoe directly, we used a gr...  
    >Some filters are case sensitive      
    
    <font style="color:blue;">Service: ...............**PIM** </font>   
    <font style="color:blue;">Category: ...............**RoleManagement** </font>    
    <font style="color:blue;">Activity: ...............**Add eligible member to role in PIM completed (permanent)** </font>    
    <font style="color:blue;">Target: ...............**SG_GlobalAdmins**  </font>  
    <font style="color:blue;">Initiated by: ...............**None**  </font>  
    <font style="color:blue;">User Agent: ...............**None**  </font>  
    <font style="color:blue;">Status: ...............**Success**</font>  
  
- Write the filter to found if Admin_Par have used the User Administrator Role?      
    <font style="color:blue;">Service: ...............**PIM**  </font>  
    <font style="color:blue;">Category: ...............**RoleManagement**  </font>  
    <font style="color:blue;">Activity: ...............**Add member to role completed (PIM activation)** </font>   
    <font style="color:blue;">Target: ...............**Admin_Par**  </font>  
    <font style="color:blue;">Initiated by: ...............**None**  </font>  
    <font style="color:blue;">User Agent: ...............**None**  </font>  
    <font style="color:blue;">Status: ...............**Success** </font>  
  

>[!knowledge]You can export all these logs, if your are an Azure Subscription or by using the Microsoft Graph API (via PowerShell or a RestAPI)

### Task 3:View which AuthN methodes are used
1. [] Select **Usage & Insights** at the left
2. [] Select **Authentication methods** at the left & Explore the dashboard

    >As you can see, you can view Account AuthN Statistic. A powerfull information to know how your users are authenticated by Azure AD. Which AuthN method (Mobile,Email, Password,...)

    **Questions**:  
    - Can you see how many users doesn't **register** MFA? if yes how many?      
    <font style="color:blue;">Yes, around 90% </font>
    - Can you know if user is authenticated in single factor (without MFA)? if yes, where?      
    <font style="color:blue;">Yes, in the usage Tab </font>

**Summary**  
In this exercise, you explore Azure AD logs to know when, where, how, why a account sign-in and where, why, what a account modified the Azure AD, in summary the sign-in & activity logs. 

===
## (OPTIONAL) Exercise 7: Windows Hello for Business
**Duration**: 30 minutes  
**Synopsis**:  
In this exercise you will configure Azure AD Kerberos to enable hybrid authentication & enable Windows Hello for Business Cloud Trust for all users.

>[!Alert]Because we haven't custom domain enable, and so no routable domain, we cannot have PRT. I don't know if we can try Hybrid Cloud trust if not we will test only Azure Active Directory join cloud only deployment. 

### Task 1: Deploy Azure AD kerberos
>[!Tip]Azure AD Kerberos is also used to enable FIDO2 support in Hybrid environment  

1. [] Log on to **@lab.VirtualMachine(SRV01).SelectLink**
    |||
    |:--------|:--------|
    | Username | **+++Administrator@contoso.com+++** |
    | Password | Please type the password |

2.	[] Install the Module *AzureADHybridAuthenticationManagement* , Open an **ELEVATED** PowerShell prompt and type:  
    +++Install-Module -Name AzureADHybridAuthenticationManagement -AllowClobber+++

    >[!Tip]If the AzureAD PowerShell module is already installed (our case) on your local computer, the installation described here might fail because of conflict. To prevent any conflicts during installation, be sure to include the "-AllowClobber" option flag.

3. [] Run the following PowerShell commands to create a new Azure AD Kerberos Server object both in your on-premises Active Directory domain and in your Azure Active Directory tenant **(You will be prompt 2 times)**:  

    ```
    $domain = "contoso.com"

    # Enter an Azure Active Directory global administrator username and password.
    $cloudCred = Get-Credential -Message 'An Active Directory user who is a member of the Global Administrators group for Azure AD.'
    Connect-AzureAD

    # Create the new Azure AD Kerberos Server object in Active Directory
    # and then publish it to Azure Active Directory.
    Set-AzureADKerberosServer -Domain $domain -CloudCredential $cloudCred

    ```
    >[!Alert]If you have this error: ![image](.\IMG\ovtl275m.jpg), you are not GA, use PIM to grant you  

    **Questions**:  
    - This command created a AzureADKerberos Object in Active Directory. Where?      
    <font style="color:blue;">In the Domain Controller OU </font>  
    - What is the DC type of this object?      
    <font style="color:blue;">It's a Read-only Domain Controller </font>
    - Do you sync Active Directory Object with this RODC?      
    <font style="color:blue;">No Only a key is sync between this RODC and Azure AD </font>

4. [] View and verify the Azure AD Kerberos Server,
    +++Get-AzureADKerberosServer -Domain $domain -CloudCredential $cloudCred+++

### Task 2: Create & link the Windows Hello for Business Group Policy object
1. [] In command prompt, start the Group Policy Management Console (+++gpmc.msc+++).
2. [] Expand the domain and select the **Group Policy Object** node in the navigation pane.
3. [] Right-click Group Policy object and select **New**.
4. [] Type +++C-Enable Windows Hello for Business+++ in the name box and click OK.
5. [] In the content pane, right-click the **C-Enable Windows Hello for Business** Group Policy object and click **Edit**.
6. [] In the navigation pane, expand **Policies** under **Computer** Configuration.
7. [] Expand **Administrative Templates** > **Windows Component**, and select **Windows Hello for Business**.
8. [] In the content pane, double-click **Use Windows Hello for Business**. Click **Enable** and click **OK**.
9. [] In the content pane, double-click **Use cloud trust for on-premises authentication**. Click **Enable** and click **OK**.
> Optional but recommended in production environment: In the content pane, double-click **Use a hardware security device**. Click **Enable** and click **OK**.  

Close the GPO Edition console
11. [] Link the GPO at the top level  

    ![image](.\IMG\n5bfcmfg.jpg)  

    >[!Knowledge]In production environment, this group policy should be targeted at the computer group that you've created for that you want or link to an OU to use Windows Hello for Business.  

    >You could use Intune to deploy this configuration.

### Task 3: Test Windows Hello for Business
>[!alert]Due to lab restriction, you cannot continue a real test of Hybrid Windows Hello For Business Cloud Trust. To do this, you should add a custom domain. We cannot do that in this lab but we will simulate it by experimenting Windows Hello for Business Cloud Only.

1. [] Log on to **@lab.VirtualMachine(CLI01).SelectLink**
    |||
    |:--------|:--------|
    | Username | **+++.\David+++** |
    | Password | **+++1LoveSecurity!+++**  |  

4. [] Click **Start**,  type and select **Settings**
5. [] Select **Accounts** and select **Access work or school**
    ![image](.\IMG\c86g44sl.jpg)
6. [] Click **Connect**
7. [] Click **Join this device to Azure Active Directory**
    ![image](.\IMG\b3mgi4xv.jpg)  
    - Authenticate with +++root@+++YourTenantDomainName.onmicrosoft.com
    - (If your Password is **+++NeverTrustAny1!+++**)
    - Press **Join**
8. [] After completion, reboot **CLI01**
9. [] Log on with **David@YourTenantDomainName.onmicrosoft.com** / Password: +++1LoveSecurity!+++    

    >[!Knowledge]**Why @YourTenantDomainName.onmicrosoft.com ?**  
    Because you didn't bought a custom domain.  
    In our case Contoso.com. That why Azure AD convert your Onpremise user *David@contoso.com* to *David@YourTenantDomainName.onmicrosoft.com*.  

    >[!Knowledge]It's a good exemple to explain that. When we say Hybrid Identity, in fact it's 2 identities synced with Azure AD Connect to have a SSO experience and many other experience (FIDO2, ....) --> 1 in Azure AD, 1 in Active Directory.

10. [] You will be prompt to configure Windows Hello for Business
    ![image](.\IMG\ef9jmh0n.jpg)
11. [] Enter a PIN code
    ![image](.\IMG\rntv15xh.jpg)

    **Questions**:  
    - By default how many characters?      
    <font style="color:blue;"> 6 </font>

12. [] At the desktop, try a logoff and a logon with you PIN. Well done this Exercise is finished !

**Summary**  
In this exercise, you set Azure AD Kerberos to enable Hybrid AuthN like Windows Hello for Business or FIDO2 AuthN. You also implement Windows Hello For Business Cloud Trust.

===
 
# End