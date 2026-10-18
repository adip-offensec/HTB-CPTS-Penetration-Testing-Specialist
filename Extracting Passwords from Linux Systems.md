# Extracting Passwords from Linux Systems

## 1.Linux Authentication Process

### Passwd file&#x20;

* The primary files it reads from and writes to are `/etc/passwd` and `/etc/shadow`.

```shell-session
htb-student:x:1000:1000:,,,:/home/htb-student:/bin/bash
```

```shell-session
head -n 1 /etc/passwd
su
```

### Shadow file

```shell-session
htb-student:$y$j9T$3QSBB6CbHEu...SNIP...f8Ms:18955:0:99999:7:::
```

As we can see here, the hashed passwords are divided into three parts. The `ID` value specifies which cryptographic hash algorithm was used, typically one of the following:

| ID     | Cryptographic Hash Algorithm                                          |
| ------ | --------------------------------------------------------------------- |
| `1`    | [MD5](https://en.wikipedia.org/wiki/MD5)                              |
| `2a`   | [Blowfish](https://en.wikipedia.org/wiki/Blowfish_\(cipher\))         |
| `5`    | [SHA-256](https://en.wikipedia.org/wiki/SHA-2)                        |
| `6`    | [SHA-512](https://en.wikipedia.org/wiki/SHA-2)                        |
| `sha1` | [SHA1crypt](https://en.wikipedia.org/wiki/SHA-1)                      |
| `y`    | [Yescrypt](https://github.com/openwall/yescrypt)                      |
| `gy`   | [Gost-yescrypt](https://www.openwall.com/lists/yescrypt/2019/06/30/1) |
| `7`    | [Scrypt](https://en.wikipedia.org/wiki/Scrypt)                        |

### Opasswd

```shell-session
sudo cat /etc/security/opasswd
```

### Cracking Linux Credentials

* <https://github.com/pmittaldev/john-the-ripper/blob/master/src/unshadow.c>

```shell-session
sudo cp /etc/passwd /tmp/passwd.bak 
sudo cp /etc/shadow /tmp/shadow.bak 
unshadow /tmp/passwd.bak /tmp/shadow.bak > /tmp/unshadowed.hashes
```

This "unshadowed" file can now be attacked with either JtR or hashcat.

```shell-session
hashcat -m 1800 -a 0 /tmp/unshadowed.hashes rockyou.txt -o /tmp/unshadowed.cracked
```

## 2.Credential Hunting in Linux

There are several sources that can provide us with credentials that we put in four categories. These include, but are not limited to:

* `Files` including configs, databases, notes, scripts, source code, cronjobs, and SSH keys
* `History` including logs, and command-line history
* `Memory` including cache, and in-memory processing
* `Key-rings` such as browser stored credentials

### Files

These categories are the following:

* Configuration files
* Databases
* Notes
* Scripts
* Cronjobs
* SSH keys

Usually, the configuration files are marked with the following three file extensions (`.config`, `.conf`, `.cnf`).

### **Searching for configuration files**

```shell-session
for l in $(echo ".conf .config .cnf");do echo -e "\nFile extension: " $l; find / -name *$l 2>/dev/null | grep -v "lib\|fonts\|share\|core" ;done
```

Optionally, we can save the result in a text file and use it to examine the individual files one after the other. Another option is to run the scan directly for each file found with the specified file extension and output the contents. In this example, we search for three words (`user`, `password`, `pass`) in each file with the file extension `.cnf`.

```shell-session
for i in $(find / -name *.cnf 2>/dev/null | grep -v "doc\|lib");do echo -e "\nFile: " $i; grep "user\|password\|pass" $i 2>/dev/null | grep -v "\#";done
```

### Searching for databases

```shell-session
for l in $(echo ".sql .db .*db .db*");do echo -e "\nDB File extension: " $l; find / -name *$l 2>/dev/null | grep -v "doc\|lib\|headers\|share\|man";done
```

### Searching for notes

```shell-session
find /home/* -type f -name "*.txt" -o ! -name "*.*"
```

### Searching for scripts

```shell-session
for l in $(echo ".py .pyc .pl .go .jar .c .sh");do echo -e "\nFile extension: " $l; find / -name *$l 2>/dev/null | grep -v "doc\|lib\|headers\|share";done
```

### Enumerating cronjobs

* Cronjobs are independent execution of commands, programs, scripts. These are divided into the system-wide area (`/etc/crontab`) and user-dependent executions.

there are the areas that are divided into different time ranges (`/etc/cron.daily`, `/etc/cron.hourly`, `/etc/cron.monthly`, `/etc/cron.weekly`). The scripts and files used by `cron` can also be found in `/etc/cron.d/` for Debian-based distributions.

```shell-session
cat /etc/crontab 
ls -la /etc/cron.*/
```

### **Enumerating history files**

`.bash_history`

`.bashrc` or `.bash_profile`

```
tail -n5 /home/*/.bash*
```

### **Enumerating log files**

The entirety of log files can be divided into four categories:

* Application logs
* Event logs
* Service logs
* System logs

Many different logs exist on the system. These can vary depending on the applications installed, but here are some of the most important ones:

| <p><br><strong>Description</strong></p> |                                                    |
| --------------------------------------- | -------------------------------------------------- |
| `/var/log/messages`                     | Generic system activity logs.                      |
| `/var/log/syslog`                       | Generic system activity logs.                      |
| `/var/log/auth.log`                     | (Debian) All authentication related logs.          |
| `/var/log/secure`                       | (RedHat/CentOS) All authentication related logs.   |
| `/var/log/boot.log`                     | Booting information.                               |
| `/var/log/dmesg`                        | Hardware and drivers related information and logs. |
| `/var/log/kern.log`                     | Kernel related warnings, errors and logs.          |
| `/var/log/faillog`                      | Failed login attempts.                             |
| `/var/log/cron`                         | Information related to cron jobs.                  |
| `/var/log/mail.log`                     | All mail server related logs.                      |
| `/var/log/httpd`                        | All Apache related logs.                           |
| `/var/log/mysqld.log`                   | All MySQL server related logs.                     |

* here are some strings we can use to find interesting content in the logs:

```shell-session
for i in $(ls /var/log/* 2>/dev/null);do GREP=$(grep "accepted\|session opened\|session closed\|failure\|failed\|ssh\|password changed\|new user\|delete user\|sudo\|COMMAND\=\|logs" $i 2>/dev/null); if [[ $GREP ]];then echo -e "\n#### Log file: " $i; grep "accepted\|session opened\|session closed\|failure\|failed\|ssh\|password changed\|new user\|delete user\|sudo\|COMMAND\=\|logs" $i 2>/dev/null;fi;done
```

## Memory and cache

### Mimipenguin

* <https://github.com/huntergregal/mimipenguin>

```shell-session
sudo python3 mimipenguin.py
```

### LaZagne

The passwords and hashes we can obtain come from the following sources but are not limited to:

* Wifi
* Wpa\_supplicant
* Libsecret
* Kwallet
* Chromium-based
* CLI
* Mozilla
* Thunderbird
* Git
* ENV variables
* Grub
* Fstab
* AWS
* Filezilla
* Gftp
* SSH
* Apache
* Shadow
* Docker
* Keepass
* Mimipy
* Sessions
* Keyrings

```shell-session
sudo python2.7 laZagne.py all
```

## Browser credentials&#x20;

### Firefox\_decrypt.py

* &#x20;Firefox browser, they are encrypted and stored in `logins.json` on the system.

```shell-session
ls -l .mozilla/firefox/ | grep default 
cat .mozilla/firefox/1bplpd86.default-release/logins.json | jq .
```

* <https://github.com/unode/firefox_decrypt>

```shell-session
python3.9 firefox_decrypt.py
```

Alternatively, `LaZagne` can also return results if the user has used the supported browser.

```shell-session
python3 laZagne.py browsers
```
