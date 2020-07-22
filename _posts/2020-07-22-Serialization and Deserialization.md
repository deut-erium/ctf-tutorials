---
title: Insecure Deserialization
author: gurnoor6
tags: web
aside:
  toc: true
sidebar:
  nav: layouts
mathjax: false
mathjax_autoNumber: false
mermaid: false
chart: false
excerpt_separator: <!--more-->
key: srlz00001
---
### What is serialization and deserialization?
"Serialization is the process of converting complex data structures, such as objects and their fields, into a 'flatter' format that can be sent and received as a sequential stream of bytes." Let's explain this with an example.
Suppose this is our user object,<br>
`$user->name = "carlos";` <br>
`$user->isLoggedIn = true;`

when this data is serialized, it looks like this `O:4:"User":2:{s:4:"name":s:6:"carlos"; s:10:"isLoggedIn":b:1;}`
This can be interpreted as follows:

* O:4:"User" - An object with the 4-character class name "User"
* 2 - the object has 2 attributes
* s:4:"name" - The key of the first attribute is the 4-character string "name"
* s:6:"carlos" - The value of the first attribute is the 6-character string "carlos"
* s:10:"isLoggedIn" - The key of the second attribute is the 10-character string "isLoggedIn"
* b:1 - The value of the second attribute is the boolean value true

Deserialization refers to reversing the serialization on the server side to get back the object that was initially created on the client side.

### Why can it be harmful?

#### Modifying object attributes
Suppose the serialized object is as follows `O:4:"User":2:{s:8:"username":s:6:"carlos"; s:7:"isAdmin":b:0;}`.
If the attacker modifies the `isAdmin` field in the serialized data and change it to `1`, he can gain access to confidential information and priviledges.
<br>

To get hands on experience with this task, you can try [this](https://portswigger.net/web-security/deserialization/exploiting/lab-deserialization-modifying-serialized-objects) lab on portswigger.

#### Modifying object types
PHP based applications are particularly vulnerable to these kind of vulnerabilities, due to presence of a loose comparison operator `==`.
<br>
What this means is that in PHP if we evaluate the statement `5=="5"`, it evaluates to true and so does `5 == "5 of something"`. However, if there is no number in the string's start, it is evaluated as `0`. So `0=="any string"` would evaluate `true` in PHP.

So what this means is that if the attacker modifies the serialized object to contain password as an integer and set its value to `0`, he can potentially login to any acount in the system!

To get a hands on experience with this task, you can try [this](https://portswigger.net/web-security/deserialization/exploiting/lab-deserialization-modifying-serialized-data-types) lab.

So this was a basic overview of attacks related to insecure deserialization. If you wish to read more, you might like to [this](https://portswigger.net/web-security/deserialization) check out.

