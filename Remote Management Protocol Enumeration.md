
## 1. SSH - 22

OpenSSH has six different authentication methods:

1. Password authentication
2. Public-key authentication
3. Host-based authentication
4. Keyboard authentication
5. Challenge-response authentication
6. GSSAPI authentication

### Default Configuration

```shell-session
cat /etc/ssh/sshd_config  | grep -v "#" | sed -r '/^\s*$/d'
```

### Dangerous Settings

| `PasswordAuthentication yes` | Allows password-based authentication.       |
| ---------------------------- | ------------------------------------------- |
| `PermitEmptyPasswords yes`   | Allows the use of empty passwords.          |
| `PermitRootLogin yes`        | Allows to log in as the root user.          |
| `Protocol 1`                 | Uses an outdated version of encryption.     |
| `X11Forwarding yes`          | Allows X11 forwarding for GUI applications. |
| `AllowTcpForwarding yes`     | Allows forwarding of TCP ports.             |
| `PermitTunnel`               | Allows tunneling.                           |
| `DebianBanner yes`           | Displays a specific banner when logging in. |

## Footprinting the Service

{% embed url="<https://github.com/jtesta/ssh-audit>" %}

```shell-session
S4D3B@htb[/htb]$ git clone https://github.com/jtesta/ssh-audit.git && cd ssh-audit
S4D3B@htb[/htb]$ ./ssh-audit.py 10.129.14.132
```

### Change Authentication Method

```shell-session
ssh -v cry0l1t3@10.129.14.132
```

### brute-force attacks

```shell-session
ssh -v cry0l1t3@10.129.14.132 -o PreferredAuthentications=password
```

## Rsync

### Scan

```shell-session
sudo nmap -sV -p 873 127.0.0.1
```

### Probing for Accessible Shares

```shell-session
nc -nv 127.0.0.1 873
#list
```

### Enumerating an Open Share

dev is the share

```shell-session
rsync -av --list-only rsync://127.0.0.1/dev
```

## R-Services

### Commands

* rcp (`remote copy`)
* rexec (`remote execution`)
* rlogin (`remote login`)
* rsh (`remote shell`)
* rstat
* ruptime
* rwho (`remote who`)

#### info's

| **Command** | **Service Daemon** | **Port** | **Transport Protocol** | **Description**                                                                                                                                                                                                                                                            |
| ----------- | ------------------ | -------- | ---------------------- | -------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| `rcp`       | `rshd`             | 514      | TCP                    | Copy a file or directory bidirectionally from the local system to the remote system (or vice versa) or from one remote system to another. It works like the `cp` command on Linux but provides `no warning to the user for overwriting existing files on a system`.        |
| `rsh`       | `rshd`             | 514      | TCP                    | Opens a shell on a remote machine without a login procedure. Relies upon the trusted entries in the `/etc/hosts.equiv` and `.rhosts` files for validation.                                                                                                                 |
| `rexec`     | `rexecd`           | 512      | TCP                    | Enables a user to run shell commands on a remote machine. Requires authentication through the use of a `username` and `password` through an unencrypted network socket. Authentication is overridden by the trusted entries in the `/etc/hosts.equiv` and `.rhosts` files. |
| `rlogin`    | `rlogind`          | 513      | TCP                    | Enables a user to log in to a remote host over the network. It works similarly to `telnet` but can only connect to Unix-like hosts. Authentication is overridden by the trusted entries in the `/etc/hosts.equiv` and `.rhosts` files.                                     |

```shell-session
cat /etc/hosts.equiv
```

### Scan for **R-Services**

```shell-session
sudo nmap -sV -p 512,513,514 10.0.17.2
```

### **Logging in Using Rlogin**

```shell-session
 rlogin 10.0.17.2 -l htb-student
```

### Listing Authenticated Users Using Rwho

```shell-session
rwho
```

### Listing Authenticated Users Using Rusers

```shell-session
rusers -al 10.0.17.5
```

***

## 2. RDP 3389

### Footprinting the Service

```powershell
nmap -sV -sC 10.129.201.248 -p3389 --script rdp*
nmap -sV -sC 10.129.201.248 -p3389 --packet-trace --disable-arp-ping -n # manual
```

{% embed url="<https://github.com/CiscoCXSecurity/rdp-sec-check>" %}

## RDP Security Check

```shell-session
sudo cpan

S4D3B@htb[/htb]$ git clone https://github.com/CiscoCXSecurity/rdp-sec-check.git && cd rdp-sec-check
S4D3B@htb[/htb]$ ./rdp-sec-check.pl 10.129.201.248

```

## Initiate an RDP Session

```shell-session
xfreerdp /u:cry0l1t3 /p:"P455w0rd!" /v:10.129.201.248
```

## 3. WinRM `5985` (`HTTP`) and `5986` (`HTTPS`)&#x20;

### Footprinting the Service

```shell-session
nmap -sV -sC 10.129.201.248 -p5985,5986 --disable-arp-ping -n
```

### Evil-winrm

```shell-session
evil-winrm -i 10.129.201.248 -u Cry0l1t3 -p P455w0rD!
```

## 4. WMI 135

### Footprinting the Service

{% embed url="<https://github.com/SecureAuthCorp/impacket/blob/master/examples/wmiexec.py>" %}

```shell-session
/usr/share/doc/python3-impacket/examples/wmiexec.py Cry0l1t3:"P455w0rD!"@10.129.201.248 "hostname"
```
