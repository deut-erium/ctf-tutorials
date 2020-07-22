---
title: Vulnerabilities in Web Applications
author: gurnoor6
tags: web introduction
aside:
  toc: true
sidebar:
  nav: layouts
mathjax: false
mathjax_autoNumber: false
mermaid: false
chart: false
excerpt_separator: <!--more-->
key: web00001
---

## What does a vulnerability mean?
"A website vulnerability is a weakness or misconfiguration in a website or web application code that allows an attacker to gain some level of control of the site, and possibly the hosting server."

### What kind of harm does it pose?
There are different types of vulnerabilities, targetting different functions of a web application like gaining access to the database of a website revealing personal information of users, running a script in the background to access data from your computer, or a script that transfers money from your bank account!

### Types of Vulnerabilities
Different kinds of attacks have different purposes and can be categorized as follows-

* **Insecure Deserialization -** Many websites have some hidden content in them, which is only visible once you verify that you have appropriate priviledges. For example, you visit to an email website, you cannot see your emails unless you login. When you log in, the system generates an *object* that contains information on what content should be shown to you (like if you are logging in with your credentials,you can only view you emails and  not your friends'). Serialization vulnerabilities tamper with the generated *object*, to allow you to see content you are not authorised to. <br>
Another example where this applies is admin portal on websites. So there are many websites that have an admin portal to manage the data associated with the site. If you tamper with the *object* as described above, you can even gain admin access to a website!

* **SQL Injection-** A lot of websites have database in the backend. What happens is that when you go to a website that has an input field, the data you enter is sent to the server, an *SQL query* is run, and the result is returned back. But what if the input you enter is an *SQL command*? Chances are, the backend program runs your input as a query on the database and the result you receive is all of the data which is stored in the database. To get more idea about SQL Injection, see https://www.youtube.com/watch?v=_jKylhJtPmI . [This](https://www.youtube.com/watch?v=J6v_W-LFK1c) video shows how SQL Injection works. 

* **Cross Site Scripting (XSS)-** Cross site scripting runs unintended JavaScript code on the website. This can be used to steal valuable user data or show spam content to the user. To get more idea about XSS, see https://www.youtube.com/watch?v=L5l9lSnNMxg . [This](https://www.youtube.com/watch?v=9kaihe5m3Lk) video demonstrates XSS quite nicely.

* **Cross Site Request Forgery (CSRF)-** In this kind of attack, the attacker causes the victim user to carry out an action unintentionally.  For example, this might be to change the email address on their account, to change their password, or to make a funds transfer. Depending on the nature of the action, the attacker might be able to gain full control over the user's account. To get more idea about CSRF, do watch https://www.youtube.com/watch?v=vRBihr41JTo or https://www.youtube.com/watch?v=eWEgUcHPle0 .

