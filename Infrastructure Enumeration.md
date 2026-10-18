
## 1. Domain Information Gathering

SSL certificate

<figure><img src="https://27438919-files.gitbook.io/~/files/v0/b/gitbook-x-prod.appspot.com/o/spaces%2FXIPxco87mgI3Ma3yEBLL%2Fuploads%2FmEBNHU87WgKzmRyQwIRq%2Fimage.png?alt=media&amp;token=2084f1c2-ad20-4c71-a7e1-8898ace079ba" alt=""><figcaption></figcaption></figure>

### Find more subdomains

{% embed url="<https://crt.sh/>" %}

### Certificate Transparency

{% code overflow="wrap" %}

```powershell
curl -s https://crt.sh/\?q\=inlanefreight.com\&output\=json | jq .

# Powershell
curl -s https://crt.sh/\?q\=inlanefreight.com\&output\=json | jq . | grep name | cut -d":" -f2 | grep -v "CN=" | cut -d'"' -f2 | awk '{gsub(/\\n/,"\n");}1;' | sort -u
```

{% endcode %}

Next, we can identify the hosts directly accessible from the Internet and not hosted by third-party providers.

### Company Hosted Servers

```shell-session
for i in $(cat subdomainlist);do host $i | grep "has address" | grep inlanefreight.com | cut -d" " -f1,4;done
```

### Shodan

{% code overflow="wrap" %}

```powershell
for i in $(cat subdomainlist);do host $i | grep "has address" | grep inlanefreight.com | cut -d" " -f4 >> ip-addresses.txt;done
 
for i in $(cat ip-addresses.txt);do shodan host $i;done
```

{% endcode %}

### DNS Records

```shell-session
dig any inlanefreight.com
```

***

## 2. Cloud Resources

### Company Hosted Servers

{% code overflow="wrap" %}

```shell-session
for i in $(cat subdomainlist);do host $i | grep "has address" | grep inlanefreight.com | cut -d" " -f1,4;done
```

{% endcode %}

### **Google Search for AWS**

<figure><img src="https://27438919-files.gitbook.io/~/files/v0/b/gitbook-x-prod.appspot.com/o/spaces%2FXIPxco87mgI3Ma3yEBLL%2Fuploads%2FGtPktWFz9davFcu0nBcm%2Fimage.png?alt=media&amp;token=35f2c29b-e03f-4daf-ad9e-01fd952a34ff" alt=""><figcaption></figcaption></figure>

### **Google Search for Azure**

<figure><img src="https://27438919-files.gitbook.io/~/files/v0/b/gitbook-x-prod.appspot.com/o/spaces%2FXIPxco87mgI3Ma3yEBLL%2Fuploads%2F5c2JCghZXdbTEQBsEkaZ%2Fimage.png?alt=media&amp;token=fdea341d-a596-4b40-8e82-8f0fa1b4727d" alt=""><figcaption></figcaption></figure>

### **Target Website - Source Code**

<figure><img src="https://27438919-files.gitbook.io/~/files/v0/b/gitbook-x-prod.appspot.com/o/spaces%2FXIPxco87mgI3Ma3yEBLL%2Fuploads%2FxyhcDsRnYDZjrvKcKXDt%2Fimage.png?alt=media&amp;token=dd0a253b-4abe-4ed6-a40d-5045412202bb" alt=""><figcaption></figcaption></figure>

### Domain.Glass

{% embed url="<https://domain.glass/>" %}

<figure><img src="https://27438919-files.gitbook.io/~/files/v0/b/gitbook-x-prod.appspot.com/o/spaces%2FXIPxco87mgI3Ma3yEBLL%2Fuploads%2F4r2Rf3sJGKO492vnpvrd%2Fimage.png?alt=media&amp;token=9e52e98c-52e1-444e-bb84-106f79b94a78" alt=""><figcaption></figcaption></figure>

### GrayHatWarfare - Leak SSH keys

{% embed url="<https://buckets.grayhatwarfare.com/>" %}

**Private and Public SSH Keys Leaked**

<figure><img src="https://27438919-files.gitbook.io/~/files/v0/b/gitbook-x-prod.appspot.com/o/spaces%2FXIPxco87mgI3Ma3yEBLL%2Fuploads%2FPEwZNk1EIIeeaXDDhzyj%2Fimage.png?alt=media&amp;token=a5a3e02d-8817-4afc-8c67-ed7c35cff7c2" alt=""><figcaption></figcaption></figure>

<figure><img src="https://27438919-files.gitbook.io/~/files/v0/b/gitbook-x-prod.appspot.com/o/spaces%2FXIPxco87mgI3Ma3yEBLL%2Fuploads%2FzK1xKIF3PuArV8XuPi1n%2Fimage.png?alt=media&amp;token=fbad401c-9acb-4b00-9d72-93ae1a1ff0c4" alt=""><figcaption></figcaption></figure>

***

## 3. Staff

**LinkedIn - Job Post**

```txt
Required Skills/Knowledge/Experience:

* 3-10+ years of experience on professional software development projects.

« An active US Government TS/SCI Security Clearance (current SSBI) or eligibility to obtain TS/SCI within nine months.
« Bachelor's degree in computer science/computer engineering with an engineering/math focus or another equivalent field of discipline.
« Experience with one or more object-oriented languages (e.g., Java, C#, C++).
« Experience with one or more scripting languages (e.g., Python, Ruby, PHP, Perl).
« Experience using SQL databases (e.g., PostgreSQL, MySQL, SQL Server, Oracle).
« Experience using ORM frameworks (e.g., SQLAIchemy, Hibernate, Entity Framework).
« Experience using Web frameworks (e.g., Flask, Django, Spring, ASP.NET MVC).
« Proficient with unit testing and test frameworks (e.g., pytest, JUnit, NUnit, xUnit).
« Service-Oriented Architecture (SOA)/microservices & RESTful API design/implementation.
« Familiar and comfortable with Agile Development Processes.
« Familiar and comfortable with Continuous Integration environments.
« Experience with version control systems (e.g., Git, SVN, Mercurial, Perforce).

Desired Skills/Knowledge/ Experience:

« CompTIA Security+ certification (or equivalent).
« Experience with Atlassian suite (Confluence, Jira, Bitbucket).
« Algorithm Development (e.g., Image Processing algorithms).
« Software security.
« Containerization and container orchestration (Docker, Kubernetes, etc.)
« Redis.
« NumPy.
```

From a job post like this, we can see, for example, which programming languages are preferred: `Java, C#, C++, Python, Ruby, PHP, Perl`. It also required that the applicant be familiar with different databases, such as: `PostgreSQL, Mysql, and Oracle`. In addition, we know that different frameworks are used for web application development, such as: `Flask, Django, ASP.NET, Spring`.

Furthermore, we use `REST APIs, Github, SVN, and Perforce`. The job offer also results that the company works with Atlassian Suite, and therefore there may be resources that we could potentially access.

***

**LinkedIn - Employee #1 About**

webpage maybe build on Django

<figure><img src="https://27438919-files.gitbook.io/~/files/v0/b/gitbook-x-prod.appspot.com/o/spaces%2FXIPxco87mgI3Ma3yEBLL%2Fuploads%2FxXGxXcU0bHTui0Z7eVoT%2Fimage.png?alt=media&amp;token=8e1882d9-0306-4f6d-8906-22d12820512d" alt=""><figcaption></figcaption></figure>

{% embed url="<https://github.com/boomcamp/django-security>" %}

{% embed url="<https://www.jwt.io/>" %}

#### LinkedIn - Employee #2 Career

<figure><img src="https://27438919-files.gitbook.io/~/files/v0/b/gitbook-x-prod.appspot.com/o/spaces%2FXIPxco87mgI3Ma3yEBLL%2Fuploads%2FhMDOPu7vqXL10FnurGeR%2Fimage.png?alt=media&amp;token=6165e4d3-9aaf-4ec5-b532-28740fac22f6" alt=""><figcaption></figcaption></figure>
