![MSCyberCoursesResized.png](lab6/MSCyberCoursesResized.png)

<!-- TOC -->
# LAB 6: Windows Corruption
## Abstract and learning objectives  

This training is designed to make you practice the concepts learned in the lectures.  
Learning objectives:  
- Understand the risks of an unprotected boot sequence  
- Deploy Virtualization Based Security 
- Develop and deploy a WDAC policy

## Overview 

This lab is a very simple environment consisting in a Windows 11 client and Windows Server 2022 servers. Both are members of an Active Directory domain northwindtraders.com. The server provides different services to support lab exercises. 

>[!ALERT] **DISCLAIMER**   
- Unless otherwise noted, the example companies, organizations, products, domain names, e-mail addresses, logos, people, places, and events depicted herein are fictitious, and no association with any real company, organization, product, domain name, e-mail address, logo, person, place or event is intended or should be inferred. Complying with all applicable copyright laws is the responsibility of the user. Without limiting the rights under copyright, no part of this document may be reproduced, stored in or introduced into a retrieval system, or transmitted in any form or by any means (electronic, mechanical, photocopying, recording, or otherwise), or for any purpose, without the express written permission of Microsoft Corporation.   
- Microsoft may have patents, patent applications, trademarks, copyrights, or other intellectual property rights covering subject matter in this document. Except as expressly provided in any written license agreement from Microsoft, the furnishing of this document does not give you any license to these patents, trademarks, copyrights, or other intellectual property.
- The names of manufacturers, products, or URLs are provided for informational purposes only and Microsoft makes no representations and warranties, either expressed, implied, or statutory, regarding these manufacturers or the use of the products with any Microsoft technologies. The inclusion of a manufacturer or product does not imply endorsement of Microsoft of the manufacturer or product. Links may be provided to third party sites. Such sites are not under the control of Microsoft and Microsoft is not responsible for the contents of any linked site or any link contained in a linked site, or any changes or updates to such sites. Microsoft is not responsible for webcasting or any other form of transmission received from any linked site. Microsoft is providing these links to you only as a convenience, and the inclusion of any link does not imply endorsement of Microsoft of the site or the products contained therein.

© 2022 Microsoft Corporation. All rights reserved.

Microsoft and the trademarks listed at <https://www.microsoft.com/en-us/legal/intellectualproperty/Trademarks/Usage/General.aspx> are trademarks of the Microsoft group of companies. All other trademarks are property of their respective owners.

===

# Exercise 1: Effects of unsecured start-up sequence

Duration: 30 minutes 

In this exercise, you will experiment with the security issues implied by not protecting the startup sequence of a Windows system. This lab demonstrates a kind of offline attack where the disk of the victim's machine is accessed directly from another system. 

Synopsis: You will work with a nested VM of the **WIN-CLI1** machine and try to add a new privileged local user account. Then, you will deploy the built-in defenses to protect the VM from this kind attacks.

## Task 1: Add the new account
In this task, you will mount the virtual machine's disk and manually create a local GPO so that next time the VM starts, a startup script will add a new user account with a password set to a value you control.

1. []Sign in **@lab.VirtualMachine(WIN-CLI1).SelectLink** with following credentials:  
	Username: **+++WIN-CLI1\karen+++**  
	Password: Please type the password 

1. []Start **Hyper-V Manager** and ensure nested VM **WIN-CLI2** is stopped.

1. []Browse `C:\Lab\VM\WIN-CLI2\Virtual Disks` and double-click on the `.vhdx` file to mount it locally.  
    *The newly attached virtual disk will appear as a new drive in the file explorer.*

1. []Create a new text file named `gpt.ini`  

    Add the following lines into the `gpt.ini` file you created:

    ```ini
    [General]
    gPCFunctionalityVersion=2
    gPCMachineExtensionNames=[{42B5FAAE-6536-11D2-AE5A-0000F87571E3}{40B6664F-4972-11D1-A7CA-0000F87571E3}]
    Version=5
    ```

1. []Copy `gpt.ini` in `\Windows\System32\GroupPolicy` on the drive you attached (if gpt.ini exists, rename to gpt.ini.bak)

1. []Create a new text file named `scripts.ini` 

   Add the following lines the `scripts.ini` file you created:

    ```ini
    [Startup]
    0CmdLine=AddAccount.cmd
    0Parameters=
    ```

1. []Move `scripts.ini` in `\Windows\System32\GroupPolicy\Machine\Scripts\`. Make sure hidden folders and file name extensions are shown. If needed, create the `Machine` or `Scripts` folders.

1. []Create a new text file named `AddAccount.cmd` with the following contents:

    ```console
    net user rogueAdmin MyStr0ng!Mdp /add /Y
    net localgroup administrators rogueAdmin /add
    net localgroup "remote desktop users" rogueAdmin /add
    ```

1. []Move `AddAccount.cmd` in `\Windows\System32\GroupPolicy\Machine\Scripts\Startup\`. If needed, create the `Startup` folder.

1. []Unmount and detach the nest VM OS disk from the explorer by right-clicking on the drive and by selecting **Eject**

1. []From the **Hyper-V Manager** console, start the **WIN-CLI2** VM.

1. []When VM **WIN-CLI2** is ready, you will be able to sign-in using the credentials you have specified in the `AddAccount.cmd` script:  
    Username: **+++WIN-CLI2\rogueAdmin+++**  
    Password: **+++MyStr0ng!Mdp+++**

    This can be further verified by opening a `cmd.exe` command prompt as Administrator and running `whoami /all`command.

1. []From **WIN-CLI2**, remove the following files to clean up the environment:

    * From %windir%\System32\GroupPolicy\Machine\Scripts\Startup
      * remove AddAccount.cmd
    * From %windir%\System32\GroupPolicy\Machine\Scripts
      * remove scripts.ini
    * From %windir%\System32\GroupPolicy
      * remove gpt.ini

## Task 2: Enable the requirements for BitLocker and Measured Boot
In this task, you will start the remediation process by enabling the platform requirements for BitLocker and Measured Boot.

1. []If not already, sign in **WIN-CLI1** with following credentials  
	Username: WIN-CLI1\Karen  
	Password: 1LoveSecurity!  

1. []Start the **Hyper-V Manager** console

1. []From the console, stop the **WIN-CLI2** VM.

1. []Right-click the **WIN-CLI2** VM and select **Settings**.

1. []In the **Settings for WIN-CLI2 on WIN-CLI1** window, select the **Security** tab.

1. []Select the **Secure Boot** and **Enable  Trusted Platform Module** check-boxes.

1. []Click **Apply** and **OK** to save the new configuration and exit the settings window.

> **Note:** From that point, you enabled the vTPM and SecureBoot features on the host. Strickly speaking, SecureBoot is not required for Measured Boot but you will need it in the next exercises. Enabling this feature now will also add it in the reference profile when you will enable BitLocker.

## Task 3: Encrypt system drive of the WIN-CLI2 VM
In this task, you will deploy a mitigation for the vulnerability you previously exploited. You will encrypt the system drive using BitLocker and seal the VMK with the virtual TPM (vTPM).

1. []Ensure you sill have the **Hyper-V Manager** console opened on **WIN-CLI1**.

1. []Start the **WIN-CLI2** VM

1. []Sign in **WIN-CLI2** with following credentials  
	Username: **+++WIN-CLI2\Karen+++**  
	Password: Please type the password 

1. []Right-click the **Start** menu and select **Command Prompt (Admin)**.

1. []Using the course material and public documentation, find the `manage-bde` commands to perform the following:
    - Add a recovery password protector
    - Add a TPM protector.
    - Start encryption **of used disk space only**.

    >[!hint] Use manage-bde documentation here:   
    https://learn.microsoft.com/en-us/windows-server/administration/windows-commands/manage-bde  
    
    > **Important:** After running commands, BitLocker will output the recovery key for the volume. It is important you keep a copy of the key.

1. []As a safety measure, BitLocker encryption won't start until a reboot is performed. Now, it is time to reboot **WIN-CLI2** to start the encryption process.

1. []When **WIN-CLI2** is ready, sign in with following credentials  
	Username: **+++WIN-CLI2\Karen+++**  
	Password: Please type the password 

1. []Right-click the **Start** menu and select **Command Prompt (Admin)**.

1. []Run the following command to check the encryption status:

    ```console
    manage-bde C: -status
    ```

    Wait until encryption is completed before running other tasks.

---

**Evaluation**

1. What `manage-bde` commands did you run? For each command you ran, explain the purpose of it.

    ```console
    ::Add a recovery password protector (RP)
    manage-bde C: -protectors -add -RP

    ::Add a TPM protector (TPM)
    manage-bde C: -protectors -add -TPM

    ::Start encryption but only on used disk space.
    manage-bde C: -on -UsedSpaceOnly
    ```

---

You have completed the Bitlocker activation task, you can move on to assessing the new configuration.

===

## Task 4: Assess the new configuration (1/2)
In this task, you will attempt to access the VM's drive using the same steps as in the beginning of the exercise to observe the changes.

1. []Ensure you sill have the **Hyper-V Manager** console opened on **WIN-CLI1**.

1. []Shutdown the **WIN-CLI2** VM

1. []Browse `C:\Lab\VM\WIN-CLI2\Virtual Disks` and double-click on the `.vhdx` file to mount it locally. 
    *The newly attached virtual disk will now appear as a new drive in Explorer.*

    > This time, you will notice Windows prompts you for a BitLocker recovery key.

1. []Unmount and detach the nest VM OS disk from the explorer by right-clicking on the drive and by selecting **Eject**.

---

**Evaluation**

1. Explain why you are not able to access the VM's drive anymore?

    This is the expected behvior as the volume was encrypted in the previous step. Windows is able to mount the virtual disk but it cannot mount the NTFS system volume as it is encrypted.

1. Immagine you are an attacker trying to corrupt the **WIN-CLI2** virtual machine. What are the two options to access the drive's content offline? For each option, state what would be needed and the difficulty.

    As there are 2 BitLocker protectors, there are 2 options to unlock the drive:
        * Steal the recovery password. Difficulty similar to other password theft threat.
        * Extract the protector key from the TPM. Very difficult.

---

You have completed the first assessment task, you can move on to the second phase.

===

## Task 5: Assess the new configuration (2/2)
In this task, you will attempt to alter the platform configuration by disabling SecureBoot. Then, you will use the PCPTool application to decode the TPM measurement log and analyze the changes.

1. []If not done already, start the **Hyper-V Manager** console

1. []Right-click the **WIN-CLI2** VM and select **Settings**.

1. []In the **Settings for WIN-CLI2 on WIN-CLI1** window, select the **Security** tab.

1. []Deselect the **Secure Boot** check-box.

1. []Click **Apply** and **OK** to save the new configuration and exit the settings window.

1. []Start VM **WIN-CLI2**

> You will notice that the VM can start successfully but instead of the Windows loading animation, you are presented with a BitLocker recovery screen.

1. []Enter the recovery key.

1. []When the **WIN-CLI2** machine has finished starting, singin with following credentials:  
	Username: **+++WIN-CLI2\karen+++**  
	Password: Please type the password 

1. []Open the folder `C:\Windows\Logs\MeasuredBoot` in Explorer and locate the name of the most recent file.

1. []Right-click the **Start** menu and select **Command Prompt (Admin)**.

1. []Run the following command to decode the measurement log:

   ```console
   PCPTool decodelog C:\Windows\Logs\MeasuredBoot\<LogFileName>.log > %Userprofile%\Documents\MeasuredBoot_WithoutSB.xml
   ```
   where the variables represent the following values:
   - \<*LogFileName*> = the name of the file to be decoded

    Example:
    ```console
    PCPTool decodelog C:\Windows\Logs\MeasuredBoot\0000000005-0000000000.log > %Userprofile%\Documents\MeasuredBoot_WithoutSB.xml
    ```
    
   To find the PCR information, go to the end of the file.

1. []Shutdown **WIN-CLI2**

1. []Edit the settings of VM **WIN-CLI2**. Ensure SecureBoot is enabled.

1. []Start VM **WIN-CLI2**. This time, you are not prompted for the recovery key.

1. []When the **WIN-CLI2** machine has finished starting, singin with following credentials:  
	Username: **+++WIN-CLI1\karen+++**  
	Password: Please type the password 

1. []Redo the same steps to decode the latest measurement log. Change the output XML file.

   ```console
   PCPTool decodelog C:\Windows\Logs\MeasuredBoot\<LogFileName>.log > %Userprofile%\Documents\MeasuredBoot_WithSB.xml
   ```

1. []Compare the two XML files to see which PCR did change.

---

**Evaluation**

For the PCR questions below, we are only interrested in the PCRs related to the firmware which are from PCR #0 to PCR #7 included.

1. Using the decoded measurement log, what PCR did change?

    PCR #7.

1. What is measured by this PCR (not the details of each component measured)?

    >[!hint] PCR usage is specified in the TCG PC Client Platform Firmware Profile Specification
    At the time of writing the latest version is :  
    https://trustedcomputinggroup.org/wp-content/uploads/TCG_PCClient_PFP_r1p05_v23_pub.pdf

    PCR #7 measures the SecureBoot policy.

1. How is this PCR used by BitLocker?

    PCR #7 is used to control access to the TPM BitLocker protector.

1. Finally, explain why Windows requested the recovery key?

    This is the expected behavior as the SecureBoot state was modified. When you enabled BitLocker, the current status of SecureBoot (which is measured by TPM PCR #7) was taken as part of the authentication policy for the BitLocker TPM protector.

---

You have completed this exercise. Congratulations!

# Exercise 2: Virtualization Based Security

Duration: 30 minutes 

In this exercise, you will enable VBS and the protection of Code Integrity (HVCI) on the **WIN-CLI2** machine.

Synopsis: You will work with the same nested VM machine and enable VBS using the local Group Policy. Then, you will test the different options for enabling VBS and preventing deactivation of the feature.

## Task 1: Enable Virtualization Based Security

In this task, you wil use the local Group Policy to enable VBS. This will add and enable the virtualization-based security features for you if needed.

1. []Ensure you sill have the **Hyper-V Manager** console opened on **WIN-CLI1**.

1. []Start the **WIN-CLI2** VM

1. []Sign in **WIN-CLI2** with following credentials  
	Username: **+++WIN-CLI2\Karen+++**  
	Password: **+++1LoveSecurity!+++**

1. []Right-click the **Start** menu and select **Run**. then, type **gpedit.msc** and hit **Enter**. 

1. []From the Local Group Policy Editor console, go to **Computer Configuration** > **Administrative Templates** > **System** > **Device Guard**.

1. []Select **Turn On Virtualization Based Security**, and then select the **Enabled** option.

1. []In the **Select Platform Security Level** box, choose **Secure Boot**

1. []In the **Virtualization Based Protection of Code Integrity** box, select **Enabled without lock**.

1. []In the **Secure Launch Configuration** box, choose **Not Configured**

1. []Select **OK**, and then close the Group Policy Management Console.

1. []Restart the **WIN-CLI2** VM.

1. []When the **WIN-CLI2** VM is ready, sign in with following credentials  
	Username: **+++WIN-CLI2\Karen+++**  
	Password: **+++1LoveSecurity!+++**

1. []Right-click the **Start** menu and select **Command Prompt (Admin)**.

1. []Type the following command:

    ```console
    msinfo32.exe
    ```

1. []In the left-most panel, select the **System Summary** node. In the right-most panel, scroll down the list of settings until **Virtualization-base Security** is displayed.

You should see **Hypervisor enforced Code Integrity** in the **Virtualization-based security Services Running** list.

---

**Evaluation**

1. Based on results of the `msinfo32.exe` console, how can you ensure that HVCI is effectively running?

    To ensure HVCI is running, we must be able to see **Hypervisor enforced Code Integrity** in the **Virtualization-based security Services Running** list.

1. What is the meaning of the **Virtualization-based security Available Security Properties** value?

    **Virtualization-based security Available Security Properties** represents the list of available hardware and platform security features which are used by VBS services.

1. What can you say about a VBS service if it is present in **Virtualization-based security Services Configured** but absent from **Virtualization-based security Services Running** 

    If a service appears in the **Virtualization-based security Services Configured** list, it means it is required by the VBS policy. But, if it does not appear in the **Virtualization-based security Services Running** list, it means it is required but something is preventing Windows from starting it.

---

You have completed the configuration task. You can now move on to the deactivation test task.

## Task 2: Disable Virtualization Based Security through registry

In this task, you will attempt to disable the VBS services by manually overriding the registry values corresponding to the group policy settings. That scenarios simulates a malicious agent trying to disable the security feature in order to lower the overall platform security level and to ease other forms of attacks.

1. []If not already done, sign in **WIN-CLI2** with following credentials  
	Username: **+++WIN-CLI2\Karen+++**  
	Password: **+++1LoveSecurity!+++**

1. []Launch **gpedit.msc** again and disable the Virtualization Based Security setting.

1. []Right-click the **Start** menu and select **Run**. then, type **regedit** and hit **Enter**. 

1. []Delete all the registry values in

    `HKEY_LOCAL_MACHINE\SOFTWARE\Policies\Microsoft\Windows\DeviceGuard`

1. []Go to `HKEY_LOCAL_MACHINE\SYSTEM\CurrentControlSet\Control\DeviceGuard`

1. []Set the **EnableVirtualizationBasedSecurity** REG_DWORD value to 0

1. []Restart **WIN-CLI2**

1. []When **WIN-CLI2** is ready, open `msinfo32.exe` again. This time, the **Virtualization-based security** setting will be set to **Not enabled**

---

**Evaluation**

1. As you have observed, it was quite easy to disable the VBS security services. Explain the fundamental weakness of controling a security service using an OS-level parameter.

    >[!hint]Think about what it takes to disable the security feature.

    With this specific configuration of Virtualization Based Security, it is still possible for an attacker to disable the whole feature as soon as admin-level control of the system is obtained.

---

You have completed the VBS deactivation test task, you can now move on to the remediation task.

## Task 3: Use UEFI locked variables to store the VBS configuration.
In this task, you will enable VBS with a different configuration. This time, the VBS policy will aslo be stored in a UEFI variable which cannot be modified from the operating system.

1. []If not already done, sign in **WIN-CLI2** with following credentials  
	Username: **+++WIN-CLI2\Karen+++**  
	Password: **+++1LoveSecurity!+++**

1. []Right-click the **Start** menu and select **Run**. then, type **gpedit.msc** and hit **Enter**. 

1. []Using the course material and public documentation, modify the configuration of the Virtualization Based Security policy to prevent disabling the **Virtualization Based Protection of Code Integrity** from the operating system.

1. []Restart the **WIN-CLI2** VM.

1. []Use the same steps as in the previous task to disable HVCI.

1. []Restart the **WIN-CLI2** VM.

1. []This time, you will notice VBS status is always **Running** after you restart **WIN-CLI2**

---

**Evaluation**

1. Explain what setting you changed to prevent disabling the VBS services from the operating system.

    1. From the Local Group Policy Editor console, go to **Computer Configuration** > **Administrative Templates** > **System** > **Device Guard**.
    1. In the **Device Guard** folder, open the **Turn On Virtualization Based Security** setting.
    1. In the **Virtualization Based Protection of Code Integrity** box, select **Enabled with UEFI lock**.
    1. Select **OK**, and then close the Group Policy Management Console.


1. Using the UEFI specification, explain why the VBS setting cannot be modified from the operating system. 

    >[!hint]The UEFI setting is stored by Windows using the SetVariable() UEFI API. The following attributes are set on the variable : EFI_VARIABLE_NON_VOLATILE and EFI_VARIABLE_BOOTSERVICE_ACCESS.  
    At the time of writing, Variable Services documentation can be found in section 8.2 of UEFI specification 2.10 which is available at this location: https://uefi.org/sites/default/files/resources/UEFI_Spec_2_10_Aug29.pdf.  
    If you read a different version of the specification, the section number might be different. In such cases, search for the SetVariable() API.

    The HVCI configuration is stored in a UEFI variable with following attributes:
        - EFI_VARIABLE_NON_VOLATILE : Does not provide security feature. It just means the value will survive a power cycle.
        - EFI_VARIABLE_BOOTSERVICE_ACCESS : Without the EFI_VARIABLE_RUNTIME_ACCESS attribute, the variable is only accessible from the boot services (aka. Windows' boot loader but not after Windows is started ). It means it cannot be read/written after the OS has started.

---

You have completed this exercise. Congratulations!

[!note]Keep the current VBS configuration. You will need it in the next exercise.

# Exercise 3: Deploy a WDAC policy

Duration: 30 minutes

Many of recent attacks are leveraging vulnerable drivers to succeed. Since Windows introduced the requirement for drivers to be signed through the hardware portal, it has become more difficult for an attacker to craft a valid and signed driver. A common practive for attackers is now to use an existing legitimate driver which contains a vulnerability, and exploit it.

Synopsis: In this exercise you will create a WDAC policy that prevent the driver used by the CPU-Z application from running. You will first run the policy in audit mode then, in enforce mode.

## Task 1: Create the policy

In this task, you will create the WDAC policy using the **WDAC Policy Wizard**

1. []Sign in **WIN-CLI2** with following credentials:  
	Username: **+++WIN-CLI2\karen+++**  
	Password: Please type the password 

1. []Launch **WDAC Policy Wizard**

1. []On the **Welcome** page, select **Policy Creator**

1. []On the **Select a Policy Type**, select **Multiple Policy Format** and **Base Policy** and click **Next**

1. []On the **Select a Base template for the policy**, select **Default Windows Mode** then, click **Next**

    > You can set a custom policy name and location or keep the defaults.

1. []On the **Configure Policy Template** page, keep the defaults and enable **Hypervisor-protected Code Integrity** then, click **Next**

1. []On the **Policy Signing Rules** page, select **Add custom rule**. Then, put the following settings

    * On the **Custom Rule Conditions** page, select **Publisher** in the **Rule Type** list.
    * Select **Deny**
    * Click on the **Browse** button and locate the `C:\Lab\Sources\cpuz154_x64.sys` file
    * Keep the defaults and then, click **Create Rule**

1. []Back on the **Policy Signing Rules** page, click **Next**

1. []Wait until the policy is created. This can take few minutes.

1. []Note the location of the `*.cip` file. You'll need it in the next task.

---

**Evaluation**

1. To create the WDAC rule, we used the **Publisher** rule type. Using WDAC documentation, explain what criterias from the file are used to allow or deny the corresponding driver?

    [!hint]WDAC documentation:  
    https://learn.microsoft.com/en-us/windows/security/threat-protection/windows-defender-application-control/windows-defender-application-control-design-guide

    A Publisher rule matches a file based on these criterias:
        * Issuing CA for the code signing certificate
        * Subject of the leaf code signing certificate, or publisher
        * version of the driver
        * driver name

1. What would be the steps to configure a rule based on the hash of the file?

    Change rule type to **File Hash** and select the file or, enter the hash value.

---

You have completed the policy creation task. You can now move on to the deployment task.

## Task 2: Apply the policy

In this task, you will apply the policy to **WIN-CLI2** machine.

1. []Open the **Hyper-V Manager** console opened on **WIN-CLI1**.

1. []Start the **WIN-CLI2** VM

1. []Sign in **WIN-CLI2** with following credentials  
	Username: **+++WIN-CLI2\karen+++**  
	Password: Please type the password 

1. []Copy the `*.cip` file to the `C:\Windows\System32\CodeIntegrity\CiPolicies\Active` folder

1. []On **WIN-CLI2**, right-click the **Start** menu and select **Run**. then, type **gpedit.msc** and hit **Enter**. 

1. []From the Local Group Policy Editor console, go to **Computer Configuration** > **Administrative Templates** > **System** > **Device Guard**.

1. []Right-click **Deploy Windows Defender Application Control** and then click **Edit**.

1. []In the Deploy Windows Defender Application Control dialog box, select the **Enabled** option, and then specify the full path of the `*.cip` file.

1. []Close the **Local Group Policy Editor**, and then restart the **WIN-CLI2** machine. Restarting the computer updates the WDAC policy.

You have completed the deployment task. You can now move on to the testing task.

## Task 3: Try to run CPU-Z

In this task, you will the CPU-Z application which loads the driver. We are still in **Audit** mode.

1. []If not done already, sign in **WIN-CLI2** with following credentials  
	Username: **+++WIN-CLI2\karen+++**  
	Password: Please type the password 

1. []ON **WIN-CLI2**, start `cpuz_x64.exe`

    For the moment, the program is able to run and you will see details about the CPU (which is the purpose of the tool). We can conclude the driver is able to load.

1. []Open **Event viewer** and expand **Event Viewer** > **Applications and Services Logs** > **Microsoft** > **Windows** > **CodeIntegrity** > **Operational**

1. []Locate the event containing the evaluation result for the CPU-Z driver.

---

**Evaluation**

1. Copy-paste or capture a screenshot of the event containing the evaluation result for the CPU-Z driver.

    Event is **3076** events related to the **cpuz154_x64.sys** driver.

---

You have completed the test in audit mode. You can now move on to enforce mode.

## Task 4: Edit the WDAC policy to move from Audit to Enforce mode

In this task, you will set the policy to enforce mode.

1. []Launch **WDAC Policy Wizard**

1. []On the **Welcome** page, select **Policy Editor**

1. []On the **Edit Existing WDAC Policy**, select **Edit Policy XML File**

1. []Click **Browse** and locate your XML Policy file from step 1. Then, click **Next**

1. On the **Configure Policy Template** page, disable the **Audit mode** setting then, click **Next**

1. On the **Policy Signing Rules** page, click **Next**

1. Wait until the policy is created. This can take few minutes.

1. Copy the `*.cip` file to the  `C:\Windows\System32\CodeIntegrity\CiPolicies\Active` folder

1. Restart **WIN-CLI2**

> After the machine has restarted, you will notice you are not able to run the application anymore.