# HTTP Verb Tampering

## 1.Bypassing Basic Authentication

<figure><img src="https://27438919-files.gitbook.io/~/files/v0/b/gitbook-x-prod.appspot.com/o/spaces%2FXIPxco87mgI3Ma3yEBLL%2Fuploads%2FIC0J3Ok7lTeLvJmsFpTG%2Fimage.png?alt=media&amp;token=73cd96e7-a179-4558-ae6b-9a8b5c38de8a" alt=""><figcaption></figcaption></figure>

<figure><img src="https://27438919-files.gitbook.io/~/files/v0/b/gitbook-x-prod.appspot.com/o/spaces%2FXIPxco87mgI3Ma3yEBLL%2Fuploads%2FimGZQ2zeF2yFj4E8mZTu%2Fimage.png?alt=media&amp;token=07b9cd13-b935-4049-9085-abbebc445d4a" alt=""><figcaption></figcaption></figure>

`401 Unauthorized` page:

<figure><img src="https://27438919-files.gitbook.io/~/files/v0/b/gitbook-x-prod.appspot.com/o/spaces%2FXIPxco87mgI3Ma3yEBLL%2Fuploads%2FYp69sZz2hxETwiBvGjoH%2Fimage.png?alt=media&amp;token=af2eb9d9-fa43-4842-a0f8-375045e2dc14" alt=""><figcaption></figcaption></figure>

### Exploit

<figure><img src="https://27438919-files.gitbook.io/~/files/v0/b/gitbook-x-prod.appspot.com/o/spaces%2FXIPxco87mgI3Ma3yEBLL%2Fuploads%2FVKqEMbdq3JeXiR2ioH02%2Fimage.png?alt=media&amp;token=23e5df6d-9f85-4b99-9ed7-d16493100189" alt=""><figcaption></figcaption></figure>

<figure><img src="https://27438919-files.gitbook.io/~/files/v0/b/gitbook-x-prod.appspot.com/o/spaces%2FXIPxco87mgI3Ma3yEBLL%2Fuploads%2FpfVcnQcECf8h33wVh6Dt%2Fimage.png?alt=media&amp;token=b8b5c497-fbfe-4889-bdda-f839514c4557" alt=""><figcaption></figcaption></figure>

<figure><img src="https://27438919-files.gitbook.io/~/files/v0/b/gitbook-x-prod.appspot.com/o/spaces%2FXIPxco87mgI3Ma3yEBLL%2Fuploads%2FaMNgQ22How5JFRZx1R39%2Fimage.png?alt=media&amp;token=352baa3b-ffc3-48df-aab0-decf24f1d349" alt=""><figcaption></figcaption></figure>

```shell-session
curl -i -X OPTIONS http://SERVER_IP:PORT/
```

the response shows `Allow: POST,OPTIONS,HEAD,GET`, which means that the web server indeed accepts `HEAD` requests

<figure><img src="https://27438919-files.gitbook.io/~/files/v0/b/gitbook-x-prod.appspot.com/o/spaces%2FXIPxco87mgI3Ma3yEBLL%2Fuploads%2FFfGsTNsgL3R6mjQi3EcS%2Fimage.png?alt=media&amp;token=86117905-73f1-4d1b-8394-93a47a19b5b1" alt=""><figcaption></figcaption></figure>

<figure><img src="https://27438919-files.gitbook.io/~/files/v0/b/gitbook-x-prod.appspot.com/o/spaces%2FXIPxco87mgI3Ma3yEBLL%2Fuploads%2FZecXINPbAmhBqtwuJE8R%2Fimage.png?alt=media&amp;token=da33131e-7289-4ed7-b553-a25916465bba" alt=""><figcaption></figcaption></figure>

### 2.Bypassing Security Filters

### Identify

<figure><img src="https://27438919-files.gitbook.io/~/files/v0/b/gitbook-x-prod.appspot.com/o/spaces%2FXIPxco87mgI3Ma3yEBLL%2Fuploads%2FRd2DdGjYb1ta619uE4ax%2Fimage.png?alt=media&amp;token=db3510a9-7a78-41af-a119-c86d779b332c" alt=""><figcaption></figcaption></figure>

### Exploit

<figure><img src="https://27438919-files.gitbook.io/~/files/v0/b/gitbook-x-prod.appspot.com/o/spaces%2FXIPxco87mgI3Ma3yEBLL%2Fuploads%2FrOuUdtdK2PBZw92XpHYH%2Fimage.png?alt=media&amp;token=432c061c-5288-44b9-8a53-664d0851ef90" alt=""><figcaption></figcaption></figure>

<figure><img src="https://27438919-files.gitbook.io/~/files/v0/b/gitbook-x-prod.appspot.com/o/spaces%2FXIPxco87mgI3Ma3yEBLL%2Fuploads%2FHG98LqIh23GgW5bnDgg8%2Fimage.png?alt=media&amp;token=9c436c9b-20ba-473e-a3cf-06739fb43e6e" alt=""><figcaption></figcaption></figure>

<figure><img src="https://27438919-files.gitbook.io/~/files/v0/b/gitbook-x-prod.appspot.com/o/spaces%2FXIPxco87mgI3Ma3yEBLL%2Fuploads%2F91RRbVe8B5ewr4X638ZP%2Fimage.png?alt=media&amp;token=79edfab0-d5a4-4c31-9a9a-7278b70973a2" alt=""><figcaption></figcaption></figure>

Once we send our request, we see that this time both `file1` and `file2` were created:

<figure><img src="https://27438919-files.gitbook.io/~/files/v0/b/gitbook-x-prod.appspot.com/o/spaces%2FXIPxco87mgI3Ma3yEBLL%2Fuploads%2FotScB839KDOeOhmQiZbm%2Fimage.png?alt=media&amp;token=38f81901-79a5-4d65-b9fe-c141993fe90f" alt=""><figcaption></figcaption></figure>

### 3.Verb Tampering Prevention

* <https://academy.hackthebox.com/module/134/section/1177> Nothing fancy

To avoid HTTP Verb Tampering vulnerabilities in our code, `we must be consistent with our use of HTTP methods` and ensure that the same method is always used for any specific functionality across the web application. It is always advised to `expand the scope of testing in security filters` by testing all request parameters. This can be done with the following functions and variables:

| Language | Function                        |
| -------- | ------------------------------- |
| PHP      | `$_REQUEST['param']`            |
| Java     | `request.getParameter('param')` |
| C#       | `Request['param']`              |

If our scope in security-related functions covers all methods, we should avoid such vulnerabilities or filter bypasses.
