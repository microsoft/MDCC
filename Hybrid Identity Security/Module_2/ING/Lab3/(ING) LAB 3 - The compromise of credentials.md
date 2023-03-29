![image](.\IMG\MSCyberCoursesResized.png)

# Threats targeting the hybrid & cloud identity platform

## Overview

Contoso is a big name and cannot risk being on the front page of all newspapers because of those stupid ransomware attacks that could have been easily prevented.
Thus, they have decided to create a strong security team. The two new employees <font style="color:red;">**Miss Red**</font> and <font style="color:blue;">**Mister Blue**</font> to respectively take care of the Red and Blue team.
The objective for the Miss Red will be to test and detect vulnerable configuration and uncover unsecure practices. The objective for Mr Blue will be to implement proper monitoring and defense mitigations against the attacks detected by the red team. 

===

# The compromise of credentials  

All exercises and tasks must be done in the defined order. They build on each other, if you skip a step, you will no longer be able to continue.

In this lab, your role will alternate between a member of the <font style="color:red;">**Red Team**</font> and a member of the <font style="color:blue;">**Blue Team**</font>. This way you will be able to see what an attacker sees as well as how to configure the environment to mitigate the attacks.

This lab will focus on the compromising on-premises accounts and how to make it harder for attackers. 

You mainly use two accounts:

|Red Team|Blue Team|
|:--------:|:--------:|
|Miss Red|Mister Blue|
|CONTOSO\red|CONTOSO\blue|

<font style="color:red;">**CONTOSO\red**</font> is only a user of **CONTOSO\Domain Users** and does not have any privilege on the domain. However, she is a member of the local **Administrators** group on **CLI01**.

<font style="color:blue;">**CONTOSO\blue**</font> is a domain privileged account member of the **Domain Admins** group.

===

## Exercise 1 - Compromise a local administrator account
We are still working on the same contoso.com environment composed of the following machines:
- **DC01** a domain controller for contoso.com running Windows Server 2016 in the HQ Active Directory site
- **DC02** a domain controller for contoso.com running Windows Server 2022 in the Beijin Active Directory site
- **SRV01** a domain joined server member of the contoso.com domain running Windows Server 2022
- **CLI01** a domain joined client member of the contoso.com domain running Windows 11

In this exercise, <font style="color:red;">**Miss Red**</font> will try to get into SRV01 by guessing the local administrator account's password. And <font style="color:blue;">**Mister Blue**</font> will try to make the environment more resistant to this attack.

### Task 1 - Password spray the admin account of SRV01
Welcome back <font style="color:red;">**Miss Red**</font>! In this exercise you will use **Hydra** to conduct a password spray against the local administrator account of **SRV01**. 


1. [] Log on to **@lab.VirtualMachine(CLI01).SelectLink**. Use the following credentials:
    |||
    |:--------|:--------|
    | Username | `CONTOSO\red` |
    | Password | Please type the password |
    

1. [] Right click on the Start menu ![image](.\IMG\11menu.png) and click on **Windows Terminal (Admin)**.

1. [] Change the current directory by typing `cd \Tools\THC-Hydra` and hit **Enter**.

1. [] Run the following command `Get-Content .\users.lst`.
    This list has all the usernames that will be targeted during the attack.

    📝 What is the first name of this list?

1. [] Run the following command `Get-Content .\passwords.lst`.
    This list has all the passwords that will be attempted during the attack.

1. [] Run **Hydra** by executing `.\hydra.exe -V -F -L .\users.lst -P .\passwords.lst SRV01 rdp`.
   
    >[!knowledge] The **-f** parameter tells **Hydra** to stop the attack as soon as it has found a password.

    And here we go, we found it:
    ![image](.\IMG\FOUND.png)

1. [] Let's put it to the test now and open a **File Explorer** window. In the address bar, type `\\SRV01\C$` and hit **Enter**. You should be presented with an authentication pop-up. Use the freshly guessed credentials:
    |||
    |:--------|:--------|
    | Username | `SRV01\administrator` |
    | Password | Please type the password |

    Note that you were prompted because your current account **CONTOSO\red** doesn't not have the permission to connect to the **C$** of **SRV01**. 

1. [] Close the **File Explorer** window.  

    You are now connected to the administrative share **C$** as the local administrator of **SRV01**. Let's tell  <font style="color:blue;">**Mister Blue**</font> all about it!

1. [] Close the session by right clicking on the Start menu ![image](.\IMG\11menu.png), clicking on **Shut down or sign out** and selecting **Sign out**.

    ⚠️ Do not forget to close <font style="color:red;">**Miss Red**</font>'s session before continuing. 



### Task 2 - Checking the traces left on SRV01

1. [] Log on to **@lab.VirtualMachine(SRV01).SelectLink**. Use the following credentials:
    |||
    |:--------|:--------|
    | Username | `CONTOSO\blue` |
    | Password | Please type the password |

1. [] Right click on the Start menu ![image](.\IMG\2022menu.png) and click on **Event Viewer**.

1. [] In the **Event Viewer** window, navigate to **Event Viewer (Local)** > **Windows Logs** > **Security**.

1. [] On the **Actions** pane, click on **Filter Current Log...** and where you see **<All Event IDs>** type `4625` and click **OK**. Review the failed attempts.

1. [] Double click on one of the events and look for the **Logon Type** property.

    Here are the most common Logon Types
    |Logon Type|Title|Description|
    |:--------|:--------|:--------|
    | 2 | Interactive |A user logged on to this computer.|
    | 3 | Network |	A user or computer logged on to this computer from the network.|
    |10|RemoteInteractive|A user logged on to this computer remotely using Terminal Services or Remote Desktop.|

    >[!knowledge] The event **4625** is generated when a failed authentication takes place on the system. It tells you information about:
    - The type of logon that was attempted
    - The account for which the authentication was
    - The reason for the failure
    - The source IP of the authentication attempt
    - The authentication protocol for the attempt
    The detail of all errors codes for the event 4625 can be found in the 🔗 [security event documentation](https://docs.microsoft.com/en-us/windows/security/threat-protection/auditing/event-4625).

1. [] On the **Actions** pane, click again on **Filter Current Log...** and where you see **<All Event IDs>** type `4624` and click **OK**. Review the successful connections.

    >[!knowledge] The event **4624** is generated when a successful connection takes place on the system. Like for the 4625, it has a lot of interesting details 🔗 [security event documentation](https://docs.microsoft.com/en-us/windows/security/threat-protection/auditing/event-4624).

1. [] On the **Actions** pane, click again on **Filter Current Log...**. In the **Filter Current Log** window, select the **XML** tab and click the **Edit query manually** check box. Acknowledge the pop-up without reading it, I mean, who has time to read pop-up... Just click **Yes**.

1. [] Select the XML filter in the window and delete it.

1. [] Type the following filter instead:
    `
    <QueryList>
    <Query Id="0" Path="Security">
    <Select Path="Security">
    Event[
    System[
    (EventID=4624) or (EventID=4625) 
    ] and
    EventData[
    Data[@Name="LogonType"]=3
        and
    Data[@Name="IpAddress"]="192.168.1.31"
    ]
    ]
    </Select>
    </Query>
    </QueryList>
    `
    And click **OK**.

    This filter is looking for both the events 4624 and 4625, for network logon only and from the IP address of **CLI01**. You should see the following pattern:
    ![image](.\IMG\EVENTS1.png)
    Multiple failures followed by a success.

1. [] Note that we see those events because the Windows audit policy is configured to log them. To verify that, you right click on the Start menu ![image](.\IMG\2022menu.png) and click on **Windows PowerShell (Admin)**. 

1. [] Run the following command `auditpol /get /subcategory:Logon`. You should see the following output:
![image](.\IMG\audit1.png)

    📝 What is the command to list all audit categories?

Well, that's nice to see all that... Let's make it harder for <font style="color:red;">**Miss Red**</font>.

===

## Exercise 2 - Secure the local administrator account
In this exercise, you will see different strategies to secure the local administrator account of a Windows machine. Note that those strategies help address attack against credentials as well as lateral movement. The biggest protection for the local administrator account is [LAPS](https://support.microsoft.com/en-us/topic/microsoft-security-advisory-local-administrator-password-solution-laps-now-available-may-1-2015-404369c3-ea1e-80ff-1e14-5caafb832f53). But we are going to see that one in the next lab. In this exercise, <font style="color:blue;">**Mister Blue**</font> will try to make it harder to guess and use le local administrator account.  

### Task 1 - Password policy for member servers
All Windows machines apply at least the **Default Domain Policy**. This group policy contains a security section called **Password Policy** which when applied on a domain controller governs the password policies for domain users, but when applied a member governs the password policy for local users. Unlike for domain users, local users on member machines do not have **Fine Grained Password Policies**. It means that all local users apply the winning password policy on the local system. Also, since the password policies are evaluated at the time the password was set, if the password of a local account was not reset since the local system joined the domain, it is possible that it has a weaker policy than what the **Default Domain Policy** dictates.

Let's see the password policy on **SRV01**.

1. [] If that's not the case already, log on to **@lab.VirtualMachine(SRV01).SelectLink** using the following credentials:
    |||
    |:--------|:--------|
    | Username | `CONTOSO\blue` |
    | Password | Please type the password |

1. [] You should still have a PowerShell console opened, if that's not the case right click on the Start menu ![image](.\IMG\2022menu.png) and click on **Windows PowerShell (Admin)**. Execute the following command in the console: `net accounts`. Review the policy settings.

    Note that the local administrator account, the one with the SID finishing in **-500**, can still be used when it is locked out.
    
    Let's increase the minimum password lenght from 7 to 14.

1. [] Right click on the Start menu ![image](.\IMG\2022menu.png) and click on **Run**.

1. [] In the **Run** window, type `gpedit.msc` and click **OK**.

1. [] In the **Local Group Policy Editor** window, navigate to **Local Computer Policy** > **Computer Configuration** > **Windows Settings** > **Security Settings** > **Account Policies** > **Password Policy**. Can you modify any of the settings?

    <details><summary>❓ Do you know why you cannot modify those settings? **Click here to see the answer**.</summary>
    They are currently set through the **Default Domain Policy**. To modify them you need to modify the domain group policy.
    </details> 

    Close the **Local Group Policy Editor** window.

1. [] Right click on the Start menu ![image](.\IMG\2022menu.png) and click on **Run**.

1. [] In the **Run** window, type `gpmc.msc` and click **OK**.

1. [] In the **Group Policy Management** window, navigate to **Group Policy Management** > **Forest: contoso.com** > **Domains** > **contoso.com**. Right click on the **Default Domain Policy** and click **Edit**.

1. [] In the **Group Policy Management Editor** window, navigate to **Default Domain Policy [DC01.CONTOSO.COM]** > **Computer Configuration** > **Windows Settings** > **Security Settings** > **Account Policies** > **Password Policy**. Double click on Minimum password length and type 14 instead of 7 characters. Click **OK**.

1. [] Close the **Group Policy Management Editor** window.

1. [] Go back to your PowerShell console and type `Invoke-GPUpdate -RandomDelayInMinutes:$false` then execute `net accounts`. Note that now the minimum password lenght is 14.

    This does not affect the existing accounts. If you want to make sure your local administrator account for which the password was found is using a 14 characters long password, you will need to reset it.

1. [] Still in the console, execute `net user administrator LongPassword!`. This should fail because the password is only 13 characters long. Now execute `net user administrator LongPassword1!`, this should work.

    You effectively changed the local administrator password to `LongPassword1!`.

### Task 2 - Restrict the local administrator account usage

Well, now we hope that the password spray will fail... One way to make things even more complicated for the attacker is to restrict from where we can use local administrator accounts. Then, even if <font style="color:red;">**Miss Red**</font> finds the password, she will not be able to use the account unless she is already connected directly to **SRV01**.

1. [] We stay on **@lab.VirtualMachine(SRV01).SelectLink** with the following credentials:
    |||
    |:--------|:--------|
    | Username | `CONTOSO\blue` |
    | Password | Please type the password |

1. [] Right click on the Start menu ![image](.\IMG\2022menu.png) and click on **Run**.

1. [] In the **Run** window, type `gpedit.msc` and click **OK**.

1. [] In the **Local Group Policy Editor** window, navigate to **Local Computer Policy** > **Computer Configuration** > **Windows Settings** > **Security Settings** > **Local Policies** > **User Right Assignment**. Double click on **Deny access to this computer from the network**. Click **Add User or Group...**. In the popup, click the **Locations...** button and select **SRV01**. Now in the **Enter the object names to select ** section, type `Local account and member of Administrators group`, click **Check Names** and click **OK**. Click **OK** to close the **Deny access to this computer from the network Properties** window. 

1. [] Close the **Local Group Policy Editor** window.

Let's tell <font style="color:red;">**Miss Red**</font> to try again to use the account.

### Task 3 - Verify local administrator login restriction 

1. [] Log on to **@lab.VirtualMachine(CLI01).SelectLink**. Use the following credentials:
    |||
    |:--------|:--------|
    | Username | `CONTOSO\red` |
    | Password | Please type the password |
    
1. [] If you already have **File Explorer** windows opened, close them. Open a new **File Explorer** window and type the following in the address bar: `\\SRV01\C$`.

1. [] In the authentication prompt, try the following credentials

    |||
    |:--------|:--------|
    | Username | `SRV01\administrator` |
    | Password | Please type the password |

    This should fail as <font style="color:blue;">**Mister Blue**</font> changed the password.
    But eh, <font style="color:red;">**Miss Red**</font> was kind enough to give you the new password.

1. [] In the authentication prompt, try the new password:

    |||
    |:--------|:--------|
    | Username | `SRV01\administrator` |
    | Password | Please type the password |

1. [] This time it should fail with a different error message:
![image](.\IMG\URAPOP1.png)

### Task 4 - Check the security log on SRV01 

1. [] Go back on **@lab.VirtualMachine(SRV01).SelectLink** with the following credentials:
    |||
    |:--------|:--------|
    | Username | `CONTOSO\blue` |
    | Password | Please type the password |

1. [] Right click on the Start menu ![image](.\IMG\2022menu.png) and click on **Event Viewer**.

1. [] In the **Event Viewer** window, navigate to **Event Viewer (Local)** > **Windows Logs** > **Security**.

1. [] On the **Actions** pane, click on **Filter Current Log...** and where you see **<All Event IDs>** type `4625` and click **OK**. Review the last failed attempt.
![image](.\IMG\15B.png)

    📝 What is the status code in the failure information section of the event?

    >[!knowledge] Although that looks like a great feature, it can be tricky to deploy in production as local accounts are sometimes expected to be used by helpdesk members and server operators. But they should not. Local accounts often don't have a centralized audit log (unless there is a SIEM collecting all machines security logs). Local accounts are often use for persistence, so even if you can't fully restrict them, you should closely monitor all local account management and activities.

===

## Exercise 3 - Attack domain accounts' passwords

<font style="color:blue;">**Mister Blue**</font> has locked down remote access for local accounts on **SRV01**. Well, not a problem for <font style="color:red;">**Miss Red**</font>, let's attack domain accounts instead.

### Task 1 - Password spray domain accounts through SRV01
Welcome back <font style="color:red;">**Miss Red**</font>! In this exercise you will use **Hydra** to conduct a password spray against the domain administrator account through **SRV01**.

1. [] Log on to **@lab.VirtualMachine(CLI01).SelectLink**. Use the following credentials:
    |||
    |:--------|:--------|
    | Username | `CONTOSO\red` |
    | Password | Please type the password |
    

1. [] Right click on the Start menu ![image](.\IMG\11menu.png) and click on **Windows Terminal (Admin)**.

1. [] Change the current directory by typing `cd \Tools\THC-Hydra` and hit **Enter**.

1. [] Run the following command `Get-Content .\domainusers.lst`.
    This list has all the usernames that will be targeted during the attack. Which in our case is just two **administrator** and **pierre**.

1. [] Run the following command `Get-Content .\passwords.lst`.
    This list has all the passwords that will be attempted during the attack.

    📝 How many passwords are in this list?

1. [] Run **Hydra** by executing `.\hydra.exe -V -F -L .\domainusers.lst -P .\passwords.lst SRV01 rdp CONTOSO`. Note the results.
   
    >[!knowledge] The **CONTOSO** string at the end of the command tells **Hydra** to use the CONTOSO domain for the account instead of a local account database.

It is time to report to <font style="color:blue;">**Mister Blue**</font> that we have found something.

<details><summary>❓ What should the blue team do if the red team confirmed a password was guess during a red team exercise? **Click here to see the answer**.</summary>
You should consider resetting the password twice in a row. It is not very well known, but if you reset the password of a user, you can still use the previous password for NTLM network authentication up to one hour after. When you reset a password for security reason, always reset it twice. Check this out for more details: 🔗 [New setting modifies NTLM network authentication behavior](https://learn.microsoft.com/en-US/troubleshoot/windows-server/windows-security/new-setting-modifies-ntlm-network-authentication)
</details> 

### Task 2 - Check the traces left of SRV01
Welcome back <font style="color:blue;">**Mister Blue**</font> let check what we can on **SRV01** on these attempts.

1. [] If that's not the case already, log on to **@lab.VirtualMachine(SRV01).SelectLink** using the following credentials:
    |||
    |:--------|:--------|
    | Username | `CONTOSO\blue` |
    | Password | Please type the password |

1. [] If the **Event Viewer** is not already open, right click on the Start menu ![image](.\IMG\2022menu.png) and click on **Event Viewer**. Then navigate to **Event Viewer (Local)** > **Windows Logs** > **Security**.

1. [] On the **Actions** pane, click again on **Filter Current Log...**. In the **Filter Current Log** window, select the **XML** tab and click the **Edit query manually** check box. Acknowledge the pop-up by clicking **Yes**.

1. [] Select the XML filter in the window and delete it.

1. [] Type the following filter instead:
    `
    <QueryList>
    <Query Id="0" Path="Security">
    <Select Path="Security">
    Event[
    System[
    (EventID=4624) or (EventID=4625) 
    ] and
    EventData[
    Data[@Name="LogonType"]=3
        and
    Data[@Name="TargetUserName"]="pierre"
    ]
    ]
    </Select>
    </Query>
    </QueryList>
    `
    And click **OK**.

    You should see a series of failed logon (event ID **4625**) followed by a success (event ID **4624**)


### Task 3 - Check the traces left of DC01

Regular member servers are not always covered by a SIEM or auditing reporting solution. For domain controllers, that's another story. They are often covered by such solution. So, let's check what we see on them. 

1. [] Log on to **@lab.VirtualMachine(DC01).SelectLink** using the following credentials:
    |||
    |:--------|:--------|
    | Username | `CONTOSO\blue` |
    | Password | Please type the password |

1. [] Right click on the Start menu ![image](.\IMG\2022menu.png) and click on **Event Viewer**.

1. [] In the **Event Viewer** window, navigate to **Event Viewer (Local)** > **Windows Logs** > **Security**.

1. [] On the **Actions** pane, click on **Filter Current Log...** and where you see **<All Event IDs>** type `4776` and click **OK**. 

1. [] Open any of the event **4776**. They should look like this:
![image](.\IMG\ID4776.png)
You can see the account name, the client where it comes from (here that's the real name of the workstation **CLI01**, but that can be spoofed as it is at the discretion of the client to provide this string during authentication).

    📝 What is the error code for wrong password?

    >[!knowledge] The event 4776 is what a domain controller logs when it deals with an NTLM passthrough authentication. Detail about the error code is available in the 🔗 [event 4776 documentation](https://docs.microsoft.com/en-us/windows/security/threat-protection/auditing/event-4776).

    Note that it does not tell you through which server the authentication went through. In our case we know since we did the attack. In a real environment, if server targeted during the attack is not covered by some sort of monitoring solution, you would not know where it is coming from. To correct this, we are going to use NTLM auditing.

1. [] Right click on the Start menu ![image](.\IMG\2022menu.png) and click on **Run**.

1. [] In the **Run** window, type `gpmc.msc` and click **OK**.

1. [] In the **Group Policy Management** window, navigate to **Group Policy Management** > **Forest: contoso.com** > **Domains** > **contoso.com** and expand **Domain controllers**. Right click on the **Default Domain Controller Policy** and click **Edit**. 

1. [] In the **Group Policy Management Editor** window, navigate to **Default Domain Controller Policy [DC01.CONTOSO.COM]** > **Computer Configuration** > **Windows Settings** > **Security Settings** > **Local Policies** > **Security option**. Change the following settings according to this table:

    |Setting|Value|
    |:--------|:--------|
    | Network security: Restrict NTLM: Outgoing NTLM traffic to remote servers | **Audit all** |
    | Network security: Restrict NTLM: Audit NTLM authentication in this domain | **Enable all** |
    | Network security: Restrict NTLM: Audit Incoming NTLM Traffic | **Enable auditing for all accounts** |

    Make sure you pick the right setting and the correct value in the different drop down menus.

1. [] Close the **Group Policy Management Editor** window.

1. [] Right click on the Start menu ![image](.\IMG\2022menu.png) and click on **Command Prompt (Admin)**.

1. [] In the command prompt window, execute `gpupdate`. 

    >[!knowledge] We used the PowerShell cmdLet **Invoke-GPUpdate** earlier in this lab. **gpupdate** is the command line version of it.

    Now let's ask <font style="color:red;">**Miss Red**</font> to do it again.

1. [] Go back on **@lab.VirtualMachine(CLI01).SelectLink** with the following credentials:
    |||
    |:--------|:--------|
    | Username | `CONTOSO\red` |
    | Password | Please type the password |
    

1. [] If you don't already have a **Windows Terminal** window opened on **C:\Tools\THC-Hydra**, right click on the Start menu ![image](.\IMG\11menu.png) and click on **Windows Terminal (Admin)**. In the console, change the current directory by typing `cd \Tools\THC-Hydra` and hit **Enter**.

1. [] Run **Hydra** by executing `.\hydra.exe -V -F -L .\domainusers.lst -P .\passwords.lst SRV01 rdp CONTOSO`. 

    Now let's see what we see on the domain controller.

1. [] Log back on **@lab.VirtualMachine(DC01).SelectLink** using the following credentials:
    |||
    |:--------|:--------|
    | Username | `CONTOSO\blue` |
    | Password | Please type the password |

1. [] In the **Event Viewer**, navigate to **Event Viewer (Local)** > **Windows Logs** > **Applications and Services Logs** > **Microsoft** > **Windows**  > **NTLM** > **Operational**. You should see events **8004** which will tell you the IP address through which the NTLM authentication has been through. Here we can see **SRV01**:
![image](.\IMG\NTLM8004.png)

===

## Exercise 4 - Protect domain accounts' passwords

Hello again <font style="color:blue;">**Mister Blue**</font>. You can't play [Whac-A-Mole](https://en.wikipedia.org/wiki/Whac-A-Mole) with <font style="color:red;">**Miss Red**</font> forever with passwords. You need to make sure passwords will not be easily guessable.

Now is the moment to bring your **Azure AD** and **AD DS** together with **Azure AD Password Protection for On-Premises Active Directory servers**.

**⚠️ You cannot perform this lab if you do not already have an Azure AD tenant.**

Please enter your global admin user principal name here: @lab.TextBox(UPN)

Do not enter your password. Just your user principal name (something like user@something.onmicrosoft.com)

🔗 Documentation [Plan and deploy on-premises Azure Active Directory Password Protection](https://docs.microsoft.com/en-us/azure/active-directory/authentication/howto-password-ban-bad-on-premises-deploy)

### Task 1 - Check the status of the solution in Azure AD

The feature is enabled by default in Azure AD. But only in audit mode. Let's have a look and change what needs to be changed.

1. [] Log on to **@lab.VirtualMachine(SRV01).SelectLink** using the following credentials:
    |||
    |:--------|:--------|
    | Username | `CONTOSO\blue` |
    | Password | Please type the password |

1. [] Launch **Edge** by clicking on the icon ![image](.\IMG\EDGE.png) in the taskbar.

1. [] Navigate to `https://aad.portal.azure.com` and sign in with a global administrator account of the tenant you created in the previous labs.

1. [] Click the burger menu on the top left ![image](.\IMG\AADBURGER.png) and then click on **Azure AD Security** ![image](.\IMG\AADSECURITY.png).

1. [] On the **Security | Getting started** page, click on the **Authentication Methods** blade. 

1. [] On the **Authentication Methods | Policies** page, click on the **Password protection** blade. 

1. [] Make sure the **Enforce custom list** feature is set to **Yes**.

1. [] Make sure the **Enable password protection on Windows Server Active Directory** feature is set to **Yes** and the **Mode** set to **Enforce**. 

1. [] On the **Authentication Methods | Password protection** page, add the following words to the list of **Custom banned password list**:
    - `cyber`
    - `school` 

    This should be like this:
![image](.\IMG\AADPORTALS1.png)

1. [] Click on the Save button ![image](.\IMG\AADSAVE.png) on the top.

### Task 2 - Deploy the Azure AD Password Protection proxy service

1. [] Still on to **@lab.VirtualMachine(SRV01).SelectLink** using the following credentials:
    |||
    |:--------|:--------|
    | Username | `CONTOSO\blue` |
    | Password | Please type the password |

1. [] If not already running, launch **Edge** by click on the icon ![image](.\IMG\EDGE.png) in the taskbar.

1. [] Navigate to `https://www.microsoft.com/en-us/download/details.aspx?id=57071` and click **Download**.

1. [] Check both **AzureADPasswordProtectionProxySetup.exe** and **AzureADPasswordProtectionDCAgentSetup.msi** and click **Next**. You will see a pop-up on the top of the browser asking you to allow multiple downloads:
![image](.\IMG\EDGEPOP.png) click **Allow**.

1. [] Open a **File Explorer** window and navigate to the **Downloads** folder. Right click on **AzureADPasswordProtectionProxySetup.exe** and click **Properties**. At the bottom of **AzureADPasswordProtectionProxySetup Properties** window, check **Unblock** and click **OK**.

1. [] Double click on **AzureADPasswordProtectionProxySetup.exe** to start the installation. Check the box **I agree to the license terms and conditions** and click **Install**.
The installation should take only few seconds. Once it is done, click **Close**.

    👏 The installation is done. Now we need to register the proxy.

1. [] You should still have a PowerShell console opened, if that's not the case right click on the Start menu ![image](.\IMG\2022menu.png) and click on **Windows PowerShell (Admin)**.

1. [] In the PowerShell console, execute the following cmdLet `Import-Module AzureADPasswordProtection` then run `Register-AzureADPasswordProtectionProxy -AccountUpn @lab.Variable(UPN)`

    At this point, an authentication pop-up will prompt you for your password. Finish the authentication.

1. [] In the same PowerShell console, execute the following cmdLet `Register-AzureADPasswordProtectionForest -AccountUpn @lab.Variable(UPN)` and follow through with the authentication.

1. [] Go back to you **File Explorer** window open in the **Downloads** folder. Right click on **AzureADPasswordProtectionDCAgentSetup.msi** and click **Properties**. At the bottom of **AzureADPasswordProtectionDCAgentSetup Properties** window, check **Unblock** and click **OK**. Then right click on that file again and click **Copy**. Then in the address bar, type `\\DC01\NETLOGON`. Right click in the middle of the window and select **Paste**.

### Task 3 - Azure AD Password Protection DC agent

1. [] Log on to **@lab.VirtualMachine(DC01).SelectLink** using the following credentials:
    |||
    |:--------|:--------|
    | Username | `CONTOSO\blue` |
    | Password | Please type the password |

1. [] Open a **File Explorer** window and navigate to `C:\Windows\SYSVOL\domain\scripts`. Double click on **AzureADPasswordProtectionDCAgentSetup.msi** and click **Run**. 

    >[!knowledge] The **NETLOGON** share points to the **scripts** folder in SYSVOL. The agent requires **.Net Framework 4.7.2** but this has been pre-installed in your machines already. 

1. [] On the installation wizard, read carefully the entire license agreement, or just check the box **I accept the terms in the License Agreement** click **Install**. Another pop-up yeah... Let's not read it and click **Yes**.

    <details><summary>❓ Do you know what is pop-up about? **Click here to see the answer**.</summary>
    It is a part of the Windows protection feature called **User Account Control**. You will cover it the Windows Security class. If you can't wait and want to know more about it, check this out: 🔗 [How User Account Control works](https://docs.microsoft.com/en-us/windows/security/identity-protection/user-account-control/how-user-account-control-works)
    
    Note that it is disabled on the other machines, that's why you didn't get that pop-up for a while. 
    </details> 

1. [] Once the installation is over, click **Finish**. A pop-up asks you to reboot, click **Yes** and go grab a coffee or something ☕.

### Task 4 - Test Azure AD Password Protection on DC01

1. [] Connect back to  **@lab.VirtualMachine(DC01).SelectLink** with the following credentials:
    |||
    |:--------|:--------|
    | Username | `CONTOSO\blue` |
    | Password | Please type the password |

1. [] Open a **File Explorer** window and navigate to `C:\Windows\SYSVOL\domain\AzureADPasswordProtection\PasswordPolicies`
    You should see a **.ppe** file.

    >[!knowledge] This is the encrypted banned password list. It contains both the **Microsoft managed list** as well as your **custom list** composed of the words **cyber** and **school**.

1. [] Right click on the Start menu ![image](.\IMG\2022menu.png) and click on **Command Prompt (Admin)** anc click **Yes**.

1. [] In the command prompt, let's reset Pierre's password with the command `net user Pierre CyberSchoolPa$$w0rd`. This should fail with the following error message:
![image](.\IMG\PIERREERR.png)

    >[!knowledge] Note that the error message is generic and doesn't tell the end user that the password was rejected because it uses some of the banned words. When deploying this feature in production, both users and helpdesk services need to be aware that it is rolling out to have the appropriate guidance when it comes to choose a correct password.

    >[!help] If the password is accepted, it might be that the list has not been fully downloaded yet. You can wait and try again (it's a one time issue). Or go ahead with the exercises. Up to you!

    Let's see what we have in the DC's eventlog.

1. [] Right click on the Start menu ![image](.\IMG\2022menu.png) and click on **Event Viewer**. Then navigate to **Event Viewer (Local)** > **Applications and Services Logs** > **Microsoft** > **AzureADPasswordProtection** > **DCAgent** > **Admin**. Here you should see an event **30027**:
![image](.\IMG\30027.png)

    📝 Can you see the password in the event 30027 or any other event in this event log?

    Here are some of the events and their meaning:

    |Event ID|Description|
    |:--------|:--------|
    | 10014 | Password was reset successfully|
    | 10015 | Password was changed successfully|
    | 30003 | Password reset failed because we found words from the custom list|
    | 30005 | Password reset failed because we found words from the Microsoft list|
    | 30027 | Password reset failed because we found words from both lists|

    The full reference is available here: 🔗[Monitor and review logs for on-premises Azure AD Password Protection environments](https://docs.microsoft.com/en-us/azure/active-directory/authentication/howto-password-ban-bad-on-premises-monitor)


    📝 What is the difference between a password change and a password reset?

Of course, you will need to install the agent on all domain controllers. But eh, you get the gist of it.

Note that the users will be protected only after they update their password or until an operator reset their password. It means that if you want to leverage this feature for accounts for which the password does not expire, you need to manually set the password.

===

## Exercise 5 - Roast Kerberos service tickets [optional]

Well well <font style="color:red;">**Miss Red**</font> your last attacks left quite the traces in the logs. Let's try to be more discreet this time. Let's attack a user's password without making a million entry in the logs.
It is time for you to switch gear and use Kerberos roasting attacks.

### Task 1 - Detect potential Kerberos roastable users

For this, you can use BloodHound as it pre-chewed the recon for you.

1. [] Log back on **@lab.VirtualMachine(CLI01).SelectLink** using the following credentials:
    |||
    |:--------|:--------|
    | Username | `CONTOSO\red` |
    | Password | Please type the password |
    
1. [] Open a new **File Explorer** window and navigate to `C:\Tools\BloodHound`.

1. [] Double click on **BloodHound.exe** and log in with these credentials:
    |||
    |:--------|:--------|
    | Neo4j Username | `neo4j` |
    | Neo4j Password | `NeverTrustAny1!` |
    
1. [] Click on the burger menu on the top left ![image](.\IMG\BHBURGER.png) and then click on the **Analysis** tab. In the **Kerberos Interaction** section, click on **List all Kerberoastable Accounts**. Then search the account **SVC-SQL@CONTOSO.COM** in the graph.

1. [] Once located, click on **SVC-SQL@CONTOSO.COM**. The menu of the left will switch to the **Node Info** tab. Scroll down and look at the information in the **NODE PROPERTIES** section. 

    📝 When was the password last set?     

    📝 When does the password expire?

    This looks like a prime target for Kerberos roasting 🍗

### Task 2 - Rquest for a ticket and save it on the disk

1. [] You are still on **@lab.VirtualMachine(CLI01).SelectLink**. If you don't have a **Windows Terminal** already open, then right click on the Start menu ![image](.\IMG\11menu.png) and click on **Windows Terminal (Admin)**.

1. [] Change the current directory by typing `cd \Tools\Ghostpack` and hit **Enter**.

1. [] Execute the following command `.\Rubeus.exe kerberoast /outfile:svc-sql.ticket`.

    >[!knowledge] This will request for a ticket for all enabled user accounts with a servicePrincipalName. In our case, it will only be **svc-sql**.

At this point we have the ticket saved locally **C:\Tools\Ghostpack\svc-sql.ticket**. Now we need to roast it 🔥

### Task 3 - Roast the ticket

In our case we will perform a dictionary attack against the ticket. We don't have the performance in our lab to use GPU and try a full-on rainbow attack on the ticket.

1. [] You are still on **@lab.VirtualMachine(CLI01).SelectLink**. If you don't have a **Windows Terminal** already open, then right click on the Start menu ![image](.\IMG\11menu.png) and click on **Windows Terminal (Admin)**.

1. [] Change the current directory by typing `cd \Tools\hashcat` and hit **Enter**.

1. [] Execute the following command `.\hashcat32.exe -a 0 -m 13100 C:\Tools\Ghostpack\svc-sql.ticket passwords.lst -o svc-sql.txt -O`
    Here are some details about the parameters:
    - **-a 0** is for dictionary attack, on our case the dictionary is the file **passwords.lst**
    - **-m 13100** is to tell hashcat that we are doing a TGS Kerberos ticket, in our case saved in this location  **C:\Tools\Ghostpack\svc-sql.ticket**
    - **-o svc-sql.txt** will save the result with the actual password in a local txt file
    - **-O** is to run hashcat is optimized kernel mode as we don't have a lot of resources on our virtual machine

    ![image](.\IMG\HASHCATOUTPUT.png)

1. [] Open a **File Explorer** window and navigate to `C:\Tools\hashcat` and open the file **svc-sql.txt** with **Notepad**.

1. [] On the **Notepad** window, click **View** and check **Word wrap**. You should see the password at the very end of the file.

    📝 What the password for the **svc-sql** account?

1. [] At this point it is possible that the **Windows Terminal** console is hanging. Close the window.

Time to bring the news to <font style="color:blue;">**Mister Blue**</font>.

===

## Exercise 6 - Detect Kerberos roasting [optional]

<font style="color:red;">**Miss Red**</font> came up strong this time... She found the password of a service account which is also a privileged account. Let see if we could have avoided it...

### Task 1 - Check the logs on your domain controller

Welcome back <font style="color:blue;">**Mister Blue**</font>, let's dig into our DC logs.

1. [] Connect back to  **@lab.VirtualMachine(DC01).SelectLink** with the following credentials:
    |||
    |:--------|:--------|
    | Username | `CONTOSO\blue` |
    | Password | Please type the password |

1. [] If the **Event Viewer** is not already open, right click on the Start menu ![image](.\IMG\2022menu.png) and click on **Event Viewer**. Then navigate to **Event Viewer (Local)** > **Windows Logs** > **Security**.

1. [] On the **Actions** pane, click on **Filter Current Log...** and where you see **<All Event IDs>** type `4769` and click **OK**. 

    📝 What is the **Task Category** of the event **4769**?

1. [] On the **Actions** pane, click on **Find...**, in the **Find** window type `svc-sql` and click **Find Next**.

1. [] Here is the only trace that this attack left on the environment. And as you can see, it is not even a failed logon or anything super suspicious:
![image](.\IMG\4769.png)

    >[!knowledge] The event **4769** is generated when a failed authentication takes place on the sytem. It tells you information about:
    - The account for tp whom the ticket was issued, here **red@CONTOSO.COM**
    - The account for which the ticket was issued, here **svc-sql**, note that we do not see the servicePrincipalName
    - The source IP of the authentication attempt
    - The encryption type used for the ticket, here **0x17** means **RC4-HMAC**
    - The error code if the ticket issuance failed, here it is **0x0** because it was a success
    All the information about the event 4769 can be found in the 🔗 [security event documentation](https://docs.microsoft.com/en-us/windows/security/threat-protection/auditing/event-4769).

    📝 What is the **Ticket Encryption Type** in plain text?

### Task 2 - Identify ideal target for roasting

You do not need to wait for <font style="color:red;">**Miss Red**</font> report. You can identify yummy kerberos roasting targets with a simple LDAP filter:

1. [] You are still connected on  **@lab.VirtualMachine(DC01).SelectLink** with the following credentials:
    |||
    |:--------|:--------|
    | Username | `CONTOSO\blue` |
    | Password | Please type the password |

1. [] Right click on the Start menu ![image](.\IMG\2022menu.png) and click on **run**.

1. [] In the **Run** window, type `dsac.exe` and click **Run**.

1. [] In the **Active Directory Administrative Center** window, click on **Global Search** in the left.

1. [] In the **GLOBAL SEARCH** section, click on **Convert to LDAP** and type the following in the text area `(&(objectCategory=person)(servicePrincipalName=*))` and click **Apply**. You can disregard the **krbtgt** account. You san see that you also found the **svc-sql**.
  
    >[!knowledge] You could refine the filter to include only **enabled** accounts for which the **password never expired ** as they are definitely the best targets.

### Task 3 - Enable AES256 encryption type

Changing the encryption type doesn't make the account uncrackable using Kerberos Roasting tools and techniques. But it makes things slightly more complicated as:
- AES256 hash are salted, so users with the same password will not have the same AES256 hash. This forces an attacker to calibrate a rainbow attack specifically for that ticket and is time consuming.
- it also gives you a detection opportunity as attackers might explicitly ask for lower encryption and make it easier for you to spot suspicious ticket issuance in the logs

1. [] You are still connected on  **@lab.VirtualMachine(DC01).SelectLink** with the following credentials:
    |||
    |:--------|:--------|
    | Username | `CONTOSO\blue` |
    | Password | Please type the password |

1. [] You should still have the **Active Directory Administrative Center** open in the search result page, double click on the **svc-sql** user.

1. [] In the **svc-sql** window, expand the **Encryption options** on the right side. Select **Other encryption options**, tick the checkbox **This account supports Kerberos AES256 bit encryption** and click **OK**.

    >[!knowledge] Tickets for this account should now be using AES256. Note that if that ticket needs to be used on system which do not support AES256 for Kerberos encryption, this might break workload. Always make sure your systems are up-to-date and support the latest encryption.

    Note that the AES keys for this account might not exist. If the account is old and the password was changed at a time AES256 was not supported by domain controllers (like the Windows Server 2003 era) the DCs will still issue RC4-HMAC ticket for the account until the password has be reset twice.

    **Assumed breach!** If you find an ideal account for roasting because the password has not changed in a long time and AES256 was never enabled, just pretend the account is already compromised. Enable AES256 and change the password twice in a row. Or event better... Replace the account with a gMSA account!

    🔗 [Group Managed Service Accounts Overview](https://docs.microsoft.com/en-us/windows-server/security/group-managed-service-accounts/group-managed-service-accounts-overview)

===

## Exercise 7 - Roast Kerberos TGT [optional]

What if we could roast users even if they don't have a servicePrincipalName? Well, no problem! If an admin has disabled Kerberos pre-authentication on a user account (they sometimes do it by mistake during troubleshooting and forget to add it back), then we can ask for a TGT for that account and try to roast it.

### Task 1 - Identify accounts without Kerberos pre-authentication 

1. [] Log back on **@lab.VirtualMachine(CLI01).SelectLink** using the following credentials:
    |||
    |:--------|:--------|
    | Username | `CONTOSO\red` |
    | Password | Please type the password |

1. [] right click on the Start menu ![image](.\IMG\11menu.png) and click on **Windows Terminal (Admin)**.

1. [] In the **Windows Terminal** window, change the current directory by typing `cd \Tools\Ghostpack` and hit **Enter**.

1. [] Execute the following command `.\Rubeus.exe asproast /enc:RC4 /outfile:users.tgt`.

    >[!knowledge] This will request a TGT (encrypted with RC4-HMAC) for all enabled user accounts with the flag "Kerberos pre-authentication not required". 

At this point we have the TGT of **Connie.Flores** saved locally **C:\Tools\Ghostpack\users.tgt**. Now you just need to roast it using your favorit tool. 

### Task 2 - Enable Kerberos pre-authentication 

Hello again <font style="color:blue;">**Mister Blue**</font>! Words on the street are that <font style="color:red;">**Miss Red**</font> was able to crack a TGT... Well, probably due to the misconfiguration of some accounts in your domain. Let's identify which one and correct it.

1. [] Go back on **@lab.VirtualMachine(DC01).SelectLink** with the following credentials:
    |||
    |:--------|:--------|
    | Username | `CONTOSO\blue` |
    | Password | Please type the password |

1. [] You should still have the **Active Directory Administrative Center**. Click on **Global Search** in the left.

1. [] In the **GLOBAL SEARCH** section, click on **Convert to LDAP** and type the following in the text area `(&(objectClass=user)(userAccountControl:1.2.840.113556.1.4.803:=4194304))` and click **Apply**. 

    >[!knowledge] The **userAccountControl** attribute is well documented here: 🔗 [Use the UserAccountControl flags to manipulate user account properties](https://docs.microsoft.com/en-us/troubleshoot/windows-server/identity/useraccountcontrol-manipulate-account-properties)

    📝 What is the default userAccountControl value for a regular user account?
    
    📝 What is the userAccountControl flag for a disabled account?

1. [] Double click on **Connie Flores** account. In the **Account** section, expand **Other options** and uncheck **Do not require Kerberos pre-authentication**.

    This is the option on a user account's properties page in the **dsa.msc** console:
    ![image](.\IMG\CONNIE1.png)

    Ideally you will need to reset the password twice in a row... Again that's the assumed breach mindset.
===

## Exercise 8 - Visualize path to domain admins 

Hello <font style="color:red;">**Miss Red**</font>, you have found credentials during this second attack phase.
Now you own:
- **svc-sql**
- **Pierre**
- **Connie Flores**
- **SRV01** (through the knowledge of the local admin password)

It is time to update BloodHound with your updated knowlegde.

### Task 1 - Udpate BloodHound

1. [] Log back on **@lab.VirtualMachine(CLI01).SelectLink** using the following credentials:
    |||
    |:--------|:--------|
    | Username | `CONTOSO\red` |
    | Password | Please type the password |

1. [] If **BloodHound** is not running already, open a **File Explorer**, navigate to `C:\Tools\BloodHound` and double click on **BloodHound.exe**. Using the following credentials:
    |||
    |:--------|:--------|
    | Neo4j Username | `neo4j` |
    | Neo4j Password | `NeverTrustAny1!` |
    
1. [] In the **Search of a node** field on the top left, type `SVC-SQL@CONTOSO.COM` and click on the suggestion. In the graph, right click on the gree user icon and in the contextual menu, click **Mark User as Owned**.

1. [] Repeat the same operation for `PIERRE@CONTOSO.COM`, `CONNIE.FLORES@CONTOSO.COM` and `SRV01@CONTOSO.COM`. Now, you should own four identities.

1. [] Click on the burger menu on the top left ![image](.\IMG\BHBURGER.png) and then click on the **Analysis** tab. In the **Shortest Paths** section, click on **Shorest Paths to Domain Admins from Owned Principals**. Select **DOMAIN ADMINS@CONTOSO.COM** and explore the results.

    📝 Why does SRV01 has a $ sign at the end of its name?

Hey <font style="color:blue;">**Mister Blue**</font>! This is the type of view the attacker have of your environment. Far from the consoles you manage it with right?

===

🍾 Congratulations! You have completed the labs for this chapter.


**Make sure you have noted all the answers to the questions marked with 📝.** 