

## 1.Host Discovery

### Scan Network Range

```powershell
sudo nmap 10.129.2.0/24 -sn -oA tnet | grep for | cut -d" " -f5
```

```powershell
# cat hosts.lst
10.129.2.4
10.129.2.10
10.129.2.11
```

we use the same scanning technique on the predefined list

```powershell
sudo nmap -sn -oA tnet -iL hosts.lst | grep for | cut -d" " -f5
```

***

### Scan Multiple IPs

{% code overflow="wrap" %}

```shell-session
sudo nmap -sn -oA tnet 10.129.2.18 10.129.2.19 10.129.2.20| grep for | cut -d" " -f5
```

{% endcode %}

```shell-session
sudo nmap -sn -oA tnet 10.129.2.18-20| grep for | cut -d" " -f5
```

***

### Scan Single IP

**checking alive**

```shell-session
sudo nmap 10.129.2.18 -sn -oA host
```

```shell-session
sudo nmap 10.129.2.18 -sn -oA host -PE --packet-trace 
```

```shell-session
sudo nmap 10.129.2.18 -sn -oA host -PE --reason 
```

```shell-session
sudo nmap 10.129.2.18 -sn -oA host -PE --packet-trace --disable-arp-ping 
```

***

## 2. Host and Port Scanning

{% embed url="<https://nmap.org/book/man-port-scanning-techniques.html>" %}

### states

| **State**          | **Description**                                                                                                                                                                                         |
| ------------------ | ------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| `open`             | This indicates that the connection to the scanned port has been established. These connections can be **TCP connections**, **UDP datagrams** as well as **SCTP associations**.                          |
| `closed`           | When the port is shown as closed, the TCP protocol indicates that the packet we received back contains an `RST` flag. This scanning method can also be used to determine if our target is alive or not. |
| `filtered`         | Nmap cannot correctly identify whether the scanned port is open or closed because either no response is returned from the target for the port or we get an error code from the target.                  |
| `unfiltered`       | This state of a port only occurs during the **TCP-ACK** scan and means that the port is accessible, but it cannot be determined whether it is open or closed.                                           |
| `open\|filtered`   | If we do not get a response for a specific port, `Nmap` will set it to that state. This indicates that a firewall or packet filter may protect the port.                                                |
| `closed\|filtered` | This state only occurs in the **IP ID idle** scans and indicates that it was impossible to determine if the scanned port is closed or filtered by a firewall.                                           |

***

### Scanning Top 10 TCP Ports

We can define the ports one by one (`-p 22,25,80,139,445`)  or  by range (`-p 22-445`) or  top 100 ports (`-F`).

```shell-session
sudo nmap 10.129.2.28 --top-ports=10
```

***

### Trace the Packets

```shell-session
sudo nmap 10.129.2.28 -p 21 --packet-trace -Pn -n --disable-arp-ping
```

### Connect Scan

uses the TCP three-way handshake to determine if a specific port on a target host is open or closed. x highly accurate

```shell-session
sudo nmap 10.129.2.28 -p 443 --packet-trace --disable-arp-ping -Pn -n --reason -sT 
```

### Filtered Ports

```shell-session
sudo nmap 10.129.2.28 -p 139 --packet-trace -n --disable-arp-ping -Pn
```

```shell-session
sudo nmap 10.129.2.28 -p 445 --packet-trace -n --disable-arp-ping -Pn
```

### UDP Port Scan

```shell-session
sudo nmap 10.129.2.28 -F -sU
```

```shell-session
sudo nmap 10.129.2.28 -sU -Pn -n --disable-arp-ping --packet-trace -p 137 --reason 
```

If we get an ICMP response with `error code 3` (port unreachable), we know that the port is indeed `closed`.

```shell-session
sudo nmap 10.129.2.28 -sU -Pn -n --disable-arp-ping --packet-trace -p 100 --reason 
```

For all other ICMP responses, the scanned ports are marked as (`open|filtered`).

```shell-session
sudo nmap 10.129.2.28 -sU -Pn -n --disable-arp-ping --packet-trace -p 138 --reason 
```

***

### Version Scan

```shell-session
sudo nmap 10.129.2.28 -Pn -n --disable-arp-ping --packet-trace -p 445 --reason  -sV
```

***

## 3. Saving Results

* Normal output (`-oN`) with the `.nmap` file extension
* Grepable output (`-oG`) with the `.gnmap` file extension
* XML output (`-oX`) with the `.xml` file extension

```shell-session
sudo nmap 10.129.2.28 -p- -oA target
```

### Normal Output

```shell-session
cat target.nmap
```

### Grepable Output

```shell-session
cat target.gnmap
```

### XML Output

```shell-session
cat target.xml
```

### Style sheets - html report

```shell-session
xsltproc target.xml -o target.html
```

***

## 4. Service Enumeration

### Service Version Detection

```powershell
sudo nmap 10.129.2.28 -p- -sV
sudo nmap 10.129.2.28 -p- -sV --stats-every=5s

#verbosity level
sudo nmap 10.129.2.28 -p- -sV -v # -v -vv
```

### Banner Grabbing

```shell-session
sudo nmap 10.129.2.28 -p- -sV
```

```shell-session
sudo nmap 10.129.2.28 -p- -sV -Pn -n --disable-arp-ping --packet-trace
```

### Tcpdump - Banner Grabbing

```shell-session
sudo tcpdump -i eth0 host 10.10.14.2 and 10.129.2.28
```

### Nc - Banner Grabbing

```shell-session
nc -nv 10.129.2.28 25
```

***

## 5. NSE

{% embed url="<https://nmap.org/nsedoc/index.html>" %}

### Catagory

| **Category** | **Description**                                                                                                                         |
| ------------ | --------------------------------------------------------------------------------------------------------------------------------------- |
| `auth`       | Determination of authentication credentials.                                                                                            |
| `broadcast`  | Scripts, which are used for host discovery by broadcasting and the discovered hosts, can be automatically added to the remaining scans. |
| `brute`      | Executes scripts that try to log in to the respective service by brute-forcing with credentials.                                        |
| `default`    | Default scripts executed by using the `-sC` option.                                                                                     |
| `discovery`  | Evaluation of accessible services.                                                                                                      |
| `dos`        | These scripts are used to check services for denial of service vulnerabilities and are used less as it harms the services.              |
| `exploit`    | This category of scripts tries to exploit known vulnerabilities for the scanned port.                                                   |
| `external`   | Scripts that use external services for further processing.                                                                              |
| `fuzzer`     | This uses scripts to identify vulnerabilities and unexpected packet handling by sending different fields, which can take much time.     |
| `intrusive`  | Intrusive scripts that could negatively affect the target system.                                                                       |
| `malware`    | Checks if some malware infects the target system.                                                                                       |
| `safe`       | Defensive scripts that do not perform intrusive and destructive access.                                                                 |
| `version`    | Extension for service detection.                                                                                                        |
| `vuln`       | Identification of specific vulnerabilities.                                                                                             |

***

### Default Scripts

```shell-session
sudo nmap <target> -sC
```

### Specific Scripts Category

```shell-session
 sudo nmap <target> --script <category>
```

### Defined Scripts

```shell-session
 sudo nmap <target> --script <script-name>,<script-name>,...
```

### Specifying Scripts

```shell-session
sudo nmap 10.129.2.28 -p 25 --script banner,smtp-commands
```

### Aggressive Scan

```shell-session
sudo nmap 10.129.2.28 -p 80 -A
```

### Vulnerability Assessment

```shell-session
sudo nmap 10.129.2.28 -p 80 -sV --script vuln 
```

***

## 6. Performance Tuning

{% embed url="<https://nmap.org/book/man-performance.html>" %}

value is `10`, but we can reduce it to `0`

### Timeouts - default

```shell-session
sudo nmap 10.129.2.0/24 -F
```

### Optimized RTT

```shell-session
sudo nmap 10.129.2.0/24 -F --initial-rtt-timeout 50ms --max-rtt-timeout 100ms
```

***

### Max Retries - default

```shell-session
sudo nmap 10.129.2.0/24 -F | grep "/tcp" | wc -l
```

### **Reduced Retries**

```shell-session
sudo nmap 10.129.2.0/24 -F --max-retries 0 | grep "/tcp" | wc -l
```

***

### Rates

```powershell
sudo nmap 10.129.2.0/24 -F -oN tnet.default

# Optimized Scan
sudo nmap 10.129.2.0/24 -F -oN tnet.minrate300 --min-rate 300

# Default Scan - Found Open Ports
cat tnet.default | grep "/tcp" | wc -l

# Optimized Scan - Found Open Ports
cat tnet.minrate300 | grep "/tcp" | wc -l
```

### Timing  - T 0 - 5

* `-T 0` / `-T paranoid`
* `-T 1` / `-T sneaky`
* `-T 2` / `-T polite`
* `-T 3` / `-T normal`
* `-T 4` / `-T aggressive`
* `-T 5` / `-T insane`

### **Default Scan**

```shell-session
sudo nmap 10.129.2.0/24 -F -oN tnet.default 
```

### **Insane Scan**

```shell-session
sudo nmap 10.129.2.0/24 -F -oN tnet.T5 -T 5
```

### **Default Scan - Found Open Ports**

```shell-session
cat tnet.default | grep "/tcp" | wc -l
```

### **Insane Scan - Found Open Ports**

```shell-session
cat tnet.T5 | grep "/tcp" | wc -l
```

***

## 7. Firewall / IDS IPS Evasion

Such errors include:

* Net Unreachable
* Net Prohibited
* Host Unreachable
* Host Prohibited
* Port Unreachable
* Proto Unreachable

```powershell
# SYN-Scan
sudo nmap 10.129.2.28 -p 21,22,25 -sS -Pn -n --disable-arp-ping --packet-trace

# ACK-Scan
sudo nmap 10.129.2.28 -p 21,22,25 -sA -Pn -n --disable-arp-ping --packet-trace
```

### Decoys

```shell-session
 sudo nmap 10.129.2.28 -p 80 -sS -Pn -n --disable-arp-ping --packet-trace -D RND:5
```

### Testing Firewall Rule

```shell-session
 sudo nmap 10.129.2.28 -n -Pn -p445 -O
```

### Scan by Using Different Source IP

```shell-session
sudo nmap 10.129.2.28 -n -Pn -p 445 -O -S 10.129.2.200 -e tun0
```

***

### DNS Proxying: SYN-Scan of a Filtered Port

```shell-session
sudo nmap 10.129.2.28 -p50000 -sS -Pn -n --disable-arp-ping --packet-trace
```

### SYN-Scan From DNS Port

{% code overflow="wrap" %}

```shell-session
sudo nmap 10.129.2.28 -p50000 -sS -Pn -n --disable-arp-ping --packet-trace --source-port 53
```

{% endcode %}

### Connect To The Filtered Port

```shell-session
ncat -nv --source-port 53 10.129.2.28 50000
```
