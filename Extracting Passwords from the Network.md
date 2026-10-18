# Extracting Passwords from the Network

## 1.Credential Hunting in Network Traffic

We'll also take a brief look at Pcredz, a tool that can quickly scan network traffic for such data.

* <https://github.com/lgandx/PCredz>

The table below lists several common protocols alongside their encrypted counterparts. While it is now more common to encounter the secure versions, there was a time when plaintext protocols were widely used.

| Unencrypted Protocol | Encrypted Counterpart      | Description                                                                 |
| -------------------- | -------------------------- | --------------------------------------------------------------------------- |
| `HTTP`               | `HTTPS`                    | Used for transferring web pages and resources over the internet.            |
| `FTP`                | `FTPS/SFTP`                | Used for transferring files between a client and a server.                  |
| `SNMP`               | `SNMPv3 (with encryption)` | Used for monitoring and managing network devices like routers and switches. |
| `POP3`               | `POP3S`                    | Retrieves emails from a mail server to a local client.                      |
| `IMAP`               | `IMAPS`                    | Accesses and manages email messages directly on the mail server.            |
| `SMTP`               | `SMTPS`                    | Sends email messages from client to server or between mail servers.         |
| `LDAP`               | `LDAPS`                    | Queries and modifies directory services like user credentials and roles.    |
| `RDP`                | `RDP (with TLS)`           | Provides remote desktop access to Windows systems.                          |
| `DNS (Traditional)`  | `DNS over HTTPS (DoH)`     | Resolves domain names into IP addresses.                                    |
| `SMB`                | `SMB over TLS (SMB 3.0)`   | Shares files, printers, and other resources over a network.                 |
| `VNC`                | `VNC with TLS/SSL`         | Allows graphical remote control of another computer.                        |

### Wireshark

* <https://www.wireshark.org/docs/man-pages/wireshark-filter.html><br>

| Wireshark filter                                  | Description                                                                                                                                                                          |
| ------------------------------------------------- | ------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------ |
| `ip.addr == 56.48.210.13`                         | Filters packets with a specific IP address                                                                                                                                           |
| `tcp.port == 80`                                  | Filters packets by port (HTTP in this case).                                                                                                                                         |
| `http`                                            | Filters for HTTP traffic.                                                                                                                                                            |
| `dns`                                             | Filters DNS traffic, which is useful to monitor domain name resolution.                                                                                                              |
| `tcp.flags.syn == 1 && tcp.flags.ack == 0`        | Filters SYN packets (used in TCP handshakes), useful for detecting scanning or connection attempts.                                                                                  |
| `icmp`                                            | Filters ICMP packets (used for Ping), which can be useful for reconnaissance or network issues.                                                                                      |
| `http.request.method == "POST"`                   | Filters for HTTP POST requests. In the case that POST requests are sent over unencrypted HTTP, it may be the case that passwords or other sensitive information is contained within. |
| `tcp.stream eq 53`                                | Filters for a specific TCP stream. Helps track a conversation between two hosts.                                                                                                     |
| `eth.addr == 00:11:22:33:44:55`                   | Filters packets from/to a specific MAC address.                                                                                                                                      |
| `ip.src == 192.168.24.3 && ip.dst == 56.48.210.3` | Filters traffic between two specific IP addresses. Helps track communication between specific hosts.                                                                                 |

* &#x20;in the image below we are filtering for unencrypted `HTTP` traffic.

<figure><img src="https://27438919-files.gitbook.io/~/files/v0/b/gitbook-x-prod.appspot.com/o/spaces%2FXIPxco87mgI3Ma3yEBLL%2Fuploads%2FFMJAYpHd00iZtI1Xl4eD%2Fimage.png?alt=media&amp;token=ba045ad8-9f5a-499d-8ac6-bb8fa1b52f8c" alt=""><figcaption></figcaption></figure>

In Wireshark, it's possible to locate packets that contain specific bytes or strings. One way to do this is by using a display filter such as `http contains "passw"`. Alternatively, you can navigate to `Edit > Find Packet` and enter the desired search query manually.

<figure><img src="https://27438919-files.gitbook.io/~/files/v0/b/gitbook-x-prod.appspot.com/o/spaces%2FXIPxco87mgI3Ma3yEBLL%2Fuploads%2FxKMzJe5ypdvfsXsqD869%2Fimage.png?alt=media&amp;token=c29129d7-7a45-4b05-baba-ea74529d1ffa" alt=""><figcaption></figcaption></figure>

### Pcredz

[Pcredz](https://github.com/lgandx/PCredz) is a tool that can be used to extract credentials from live traffic or network packet captures. Specifically, it supports extracting the following information:

* Credit card numbers
* POP credentials
* SMTP credentials
* IMAP credentials
* SNMP community strings
* FTP credentials
* Credentials from HTTP NTLM/Basic headers, as well as HTTP Forms
* NTLMv1/v2 hashes from various traffic including DCE-RPC, SMBv1/2, LDAP, MSSQL, and HTTP
* Kerberos (AS-REQ Pre-Auth etype 23) hashes

{% embed url="<https://github.com/lgandx/PCredz?tab=readme-ov-file#install>" %}

```shell-session
./Pcredz -f demo.pcapng -t -v
```

## 2.Credential Hunting in Network Shares

### **Common credential patterns**

Before diving into specialized tools, it's important to understand the types of patterns and file formats that often reveal sensitive information. This was covered in earlier sections, so we won't repeat it in detail here. But as a quick reminder, here are some general tips:

* Look for keywords within files such as `passw`, `user`, `token`, `key`, and `secret`.
* Search for files with extensions commonly associated with stored credentials, such as `.ini`, `.cfg`, `.env`, `.xlsx`, `.ps1`, and `.bat`.
* Watch for files with "interesting" names that include terms like `config`, `user`, `passw`, `cred`, or `initial`.
* If you're trying to locate credentials within the `INLANEFREIGHT.LOCAL` domain, it may be helpful to search for files containing the string `INLANEFREIGHT\`.
* Keywords should be localized based on the target; if you are attacking a German company, it's more likely they will reference a `"Benutzer"` than a `"User"`.
* Pay attention to the shares you are looking at, and be strategic. If you scan ten shares with thousands of files each, it's going to take a signifcant amount of time. Shares used by `IT employees` might be a more valuable target than those used for company photos.

With all of this in mind, you may want to begin with basic command-line searches (e.g., `Get-ChildItem -Recurse -Include *.ext \\Server\Share | Select-String -Pattern ...`) before scaling up to more advanced tools. Let's take a look at how we can use `MANSPIDER`, `Snaffler`, `SnafflePy`, and `NetExec` to automate and enhance this credential hunting process.

***

## Hunting from Windows

### Snaffer

* This is a C# program that, when run on a `domain-joined` machine, automatically identifies accessible network shares and searches for interesting files.&#x20;
* &#x20;The `README` file in the Github repository describes the numerous configuration options in great detail

{% embed url="<https://github.com/SnaffCon/Snaffler>" %}

```cmd-session
c:\Users\Public>Snaffler.exe -s
```

Two useful parameters that can help refine Snaffler's search process are:

* `-u` retrieves a list of users from Active Directory and searches for references to them in files
* `-i` and `-n` allow you to specify which shares should be included in the search

***

### PowerHuntShares

* <https://github.com/NetSPI/PowerHuntShares>

<figure><img src="https://27438919-files.gitbook.io/~/files/v0/b/gitbook-x-prod.appspot.com/o/spaces%2FXIPxco87mgI3Ma3yEBLL%2Fuploads%2F73CfLnuv1DxJXrE1NVZ6%2Fimage.png?alt=media&amp;token=acf7802b-6991-4ffd-a4fa-37bf388f85ab" alt=""><figcaption></figcaption></figure>

We can run a basic scan using `PowerHuntShares`

```powershell-session
PS C:\Users\Public\PowerHuntShares> Invoke-HuntSMBShares -Threads 100 -OutputDirectory c:\Users\Public
```

## Hunting from Linux

### Manspider

* <https://github.com/blacklanternsecurity/MANSPIDER>

```shell-session
docker run --rm -v ./manspider:/root/.manspider blacklanternsecurity/manspider 10.129.234.121 -c 'passw' -u 'mendres' -p 'Inlanefreight2025!'
```

### Netexec

```shell-session
nxc smb 10.129.234.121 -u mendres -p 'Inlanefreight2025!' --spider IT --content --pattern "passw"
```
