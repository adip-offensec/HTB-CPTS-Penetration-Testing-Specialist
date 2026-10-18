# Parameter Fuzzing

## 1.Parameter Fuzzing - GET

## GET Request Fuzzing

> [http://admin.academy.htb:PORT/admin/admin.php?param1=key\\](https://adipsharif.gitbook.io/playbook/cpts-exploitation-and-lateral-movement/cpts-web-exploitation/http:/admin.academy.htb:PORT/admin/admin.php?param1=key\\)
> \
> So, all we have to do is replace param1 in the example above with FUZZ and rerun our scan. Before we can start, however, we must pick an appropriate wordlist. Once again, SecLists has just that in /opt/useful/seclists/Discovery/Web-Content/burp-parameter-names.txt. With that, we can run our scan.

```shell-session
ffuf -w /opt/useful/seclists/Discovery/Web-Content/burp-parameter-names.txt:FUZZ -u http://admin.academy.htb:PORT/admin/admin.php?FUZZ=key -fs xxx
```

## 2.Parameter Fuzzing - POST

> To fuzz the `data` field with `ffuf`, we can use the `-d` flag, as we saw previously in the output of `ffuf -h`. We also have to add `-X POST` to send `POST` requests.

> Tip: In PHP, "POST" data "content-type" can only accept "application/x-www-form-urlencoded". So, we can set that in "ffuf" with "-H 'Content-Type: application/x-www-form-urlencoded'".

```shell-session
ffuf -w /opt/useful/seclists/Discovery/Web-Content/burp-parameter-names.txt:FUZZ -u http://admin.academy.htb:PORT/admin/admin.php -X POST -d 'FUZZ=key' -H 'Content-Type: application/x-www-form-urlencoded' -fs xxx
```

```shell-session
curl http://admin.academy.htb:PORT/admin/admin.php -X POST -d 'id=key' -H 'Content-Type: application/x-www-form-urlencoded'
```

## 3.Value Fuzzing

### Custom Wordlist

```shell-session
for i in $(seq 1 1000); do echo $i >> ids.txt; done

cat ids.txt
```

```shell-session
ffuf -w ids.txt:FUZZ -u http://admin.academy.htb:PORT/admin/admin.php -X POST -d 'id=FUZZ' -H 'Content-Type: application/x-www-form-urlencoded' -fs xxx
```
