# Hunting For Users

## 1.Enumerating & Retrieving Password Policies

### Enumerating the Password Policy - from Linux - Credentialed

With valid domain credentials, the password policy can also be obtained remotely using tools such as [CrackMapExec](https://github.com/byt3bl33d3r/CrackMapExec) or `rpcclient`.

```shell-session
crackmapexec smb 172.16.5.5 -u avazquez -p Password123 --pass-pol
```

## Enumerating the Password Policy - from Linux - SMB NULL Sessions

For enumeration, we can use tools such as `enum4linux`, `CrackMapExec`, `rpcclient`, etc.

We can use [rpcclient](https://www.samba.org/samba/docs/current/man-html/rpcclient.1.html) to check a Domain Controller for SMB NULL session access.

### Using rpcclient

```shell-session
rpcclient -U "" -N 172.16.5.5
rpcclient $> querydominfo
```

### Obtaining the Password Policy using rpcclient

```shell-session
rpcclient $> querydominfo
rpcclient $> getdompwinfo
```

## Enum4linux

> &#x20;[enum4linux](https://labs.portcullis.co.uk/tools/enum4linux). `enum4linux` is a tool built around the [Samba suite of tools](https://www.samba.org/samba/docs/current/man-html/samba.7.html) `nmblookup`, `net`, `rpcclient` and `smbclient` to use for enumeration of windows hosts and domains. It can be found pre-installed on many different penetration testing distros, including Parrot Security Linux. Below we have an example output displaying information that can be provided by `enum4linux`. Here are some common enumeration tools and the ports they use:

| Tool      | Ports                                             |
| --------- | ------------------------------------------------- |
| nmblookup | 137/UDP                                           |
| nbtstat   | 137/UDP                                           |
| net       | 139/TCP, 135/TCP, TCP and UDP 135 and 49152-65535 |
| rpcclient | 135/TCP                                           |
| smbclient | 445/TCP                                           |

```shell-session
enum4linux -P 172.16.5.5
```

***

## Enum4linux-ng

```shell-session
enum4linux-ng -P 172.16.5.5 -oA ilfreight
```

```shell-session
cat ilfreight.json 
```

***

## Enumerating Null Session - from Windows

### Establish a null session from windows

```cmd-session
C:\htb> net use \\DC01\ipc$ "" /u:""
```

We can also use a username/password combination to attempt to connect. Let's see some common errors when trying to authenticate:

### **Error: Account is Disabled**

```cmd-session
C:\htb> net use \\DC01\ipc$ "" /u:guest
```

### Error: Password is Incorrect

```cmd-session
C:\htb> net use \\DC01\ipc$ "password" /u:guest
```

### Error: Account is locked out (Password Policy)

```cmd-session
C:\htb> net use \\DC01\ipc$ "password" /u:guest
```

***

## Enumerating the Password Policy - from Linux - LDAP Anonymous Bind

> With an LDAP anonymous bind, we can use LDAP-specific enumeration tools such as `windapsearch.py`, `ldapsearch`, `ad-ldapdomaindump.py`, etc., to pull the password policy. With [ldapsearch](https://linux.die.net/man/1/ldapsearch), it can be a bit cumbersome but doable. One example command to get the password policy is as follows:

### ldapsearch

```
ldapsearch -h 172.16.5.5 -x -b "DC=INLANEFREIGHT,DC=LOCAL" -s sub "*" | grep -m 1 -B 10 pwdHistoryLength
```

> Note: In newer versions of `ldapsearch`, the `-h` parameter was deprecated in favor for `-H`<br>
>
> Here we can see the minimum password length of 8, lockout threshold of 5, and password complexity is set (`pwdProperties` set to `1`).

## Enumerating the Password Policy - from Windows

### Using net.exe

```cmd-session
C:\htb> net accounts
```

### **Using PowerView**

```powershell-session
PS C:\htb> import-module .\PowerView.ps1
PS C:\htb> Get-DomainPolicy
```

***

## 2.Password Spraying - Making a Target User List

### Detailed User Enumeration

To mount a successful password spraying attack, we first need a list of valid domain users to attempt to authenticate with. There are several ways that we can gather a target list of valid users:

* By leveraging an SMB NULL session to retrieve a complete list of domain users from the domain controller
* Utilizing an LDAP anonymous bind to query LDAP anonymously and pull down the domain user list
* Using a tool such as `Kerbrute` to validate users utilizing a word list from a source such as the [statistically-likely-usernames](https://github.com/insidetrust/statistically-likely-usernames) GitHub repo, or gathered by using a tool such as [linkedin2username](https://github.com/initstring/linkedin2username) to create a list of potentially valid users
* Using a set of credentials from a Linux or Windows attack system either provided by our client or obtained through another means such as LLMNR/NBT-NS response poisoning using `Responder` or even a successful password spray using a smaller wordlist

## SMB NULL Session to Pull User List

### Using enum4linux

```shell-session
enum4linux -U 172.16.5.5  | grep "user:" | cut -f2 -d"[" | cut -f1 -d"]"
```

* We can use the `enumdomusers` command after connecting anonymously using `rpcclient`.

### Using rpcclient

```shell-session
rpcclient -U "" -N 172.16.5.5

rpcclient $> enumdomusers 
```

### **Using CrackMapExec --users Flag**

```shell-session
crackmapexec smb 172.16.5.5 --users
```

## Gathering Users with LDAP Anonymous

### **Using ldapsearch**

```shell-session
ldapsearch -h 172.16.5.5 -x -b "DC=INLANEFREIGHT,DC=LOCAL" -s sub "(&(objectclass=user))"  | grep sAMAccountName: | cut -f2 -d" "
```

> Tools such as `windapsearch` make this easier (though we should still understand how to create our own LDAP search filters). Here we can specify anonymous access by providing a blank username with the `-u` flag and the `-U` flag to tell the tool to retrieve just users.

### Using windapsearch

```shell-session
./windapsearch.py --dc-ip 172.16.5.5 -u "" -U
```

## Enumerating Users with Kerbrute

### **Kerbrute User Enumeration**

> Let's try out this method using the [jsmith.txt](https://github.com/insidetrust/statistically-likely-usernames/blob/master/jsmith.txt) wordlist of 48,705 possible common usernames in the format `flast`. The [statistically-likely-usernames](https://github.com/insidetrust/statistically-likely-usernames) GitHub repo is an excellent resource for this type of attack and contains a variety of different username lists that we can use to enumerate valid usernames using `Kerbrute`.

```shell-session
kerbrute userenum -d inlanefreight.local --dc 172.16.5.5 /opt/jsmith.txt 
```

## Credentialed Enumeration to Build our User List

With valid credentials, we can use any of the tools stated previously to build a user list. A quick and easy way is using CrackMapExec.

### **Using CrackMapExec with Valid Credentials**

```shell-session
sudo crackmapexec smb 172.16.5.5 -u htb-student -p Academy_student_AD! --users
```
