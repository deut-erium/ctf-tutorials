---
title: SQL Injection
author: 0sidharth
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
key: sqli00001
---
# SQL Injection
## What are SQL Queries?
SQL is a standardized language used to access and manipulate databases to build customizable data views for each user. SQL queries are used to execute commands, such as data retrieval, updates, and record removal. Different SQL elements implement these tasks, e.g., queries using the SELECT statement to retrieve data, based on user-provided parameters.

A typical SQL database query may look like the following:  
`SELECT name, age FROM users WHERE name = UserName`  
As you can gather from the syntax, this query returns the name and age of the user having username `UserName`  

## What is an SQL Injection (SQLi)?
"SQLi is a web security vulnerability that allows an attacker to interfere with the queries that an application makes to its database. It generally allows an attacker to view data that they are not normally able to retrieve." An SQLi involves giving input to the SQL query that is not simply data, but an SQL query in itself. For example, consider the SQL query -  
`SELECT * FROM users WHERE name = '" + userName + "'`  
This code is designed to pull up the records of only the specified username. This query leaves room for exploitation. The idea is to construct the parameter `userName` in such a way that it reveals information that should otherwise be hidden, like the records of ALL users.
This can be done by simply crafting `userName` to be `' OR '1'='1`  
The full query now becomes:  
`SELECT * FROM users WHERE name = '' OR '1'='1';`  
As you can see, it will now return the records of all users in the database. So this is the basic idea behind an SQL Injection.

## Why is this harmful?
A successful SQL injection attack can result in unauthorized access to sensitive data, such as passwords, credit card details, or personal user information. Many high-profile data breaches in recent years have been the result of SQL injection attacks, leading to reputational damage and regulatory fines. In some cases, an attacker can obtain a persistent backdoor into an organization's systems, leading to a long-term compromise that can go unnoticed for an extended period.

## Examples of SQL Injections
### Retrieving hidden data
Consider a shopping application that displays products in different categories. When the user clicks on the Gifts category, their browser requests the URL:  
`https://insecure-website.com/products?category=Gifts`  
and this causes the application to make an SQL query to retrieve details of the relevant products from the database:  
`SELECT * FROM products WHERE category = 'Gifts' AND released = 1`  
This SQL query asks the database to return all records from the table `products` where the category is `Gifts` and released is `1`. We can guess that unreleased products probably have `released` set to 0. This gives us room to make an SQLi that reveals the unreleased products to us in such a way:  
`SELECT * FROM products WHERE category = 'Gifts'--' AND released = 1`  
The `--` indicates a comment, so the query won't consider the condition of released being set to 1, and will thus reveal data for both released and unreleased products. The URL corresponding to this attack would be  
`https://insecure-website.com/products?category=Gifts'--`  

You can try this out [here](https://portswigger.net/web-security/sql-injection/lab-retrieve-hidden-data)

### Subverting Application Logic
Consider a web application having a user log in system. It requests for the username and password from the user, and returns all details corresponding to that username if the SQL query returns an entry. The corresponding SQL Query could look like:  
`SELECT * FROM users WHERE username = 'wiener' AND password = 'bluecheese'`  
Here, an attacker can give `username` to be `admin'--` and any random value to password (as it is being commented). This will allow the user to log into the admin account and view all coressponding details. The corresponding query would look like:  
`SELECT * FROM users WHERE username = 'admin'--' AND password = ''`  

You can try this out [here](https://portswigger.net/web-security/sql-injection/lab-login-bypass)

### Retrieving data from other tables
In cases where the results of an SQL query are returned within the application's responses, an attacker can leverage an SQL injection vulnerability to retrieve data from other tables within the database. This is done using the UNION keyword, which lets you execute an additional SELECT query and append the results to the original query.

Lets consider:  
`SELECT name, description FROM products WHERE category = 'Gifts'`  
What if we modify the input to `' UNION SELECT username, password FROM users--`?  
This query would give us additional information about the usernames and passwords stored in the database.

You can read more about this [here](https://portswigger.net/web-security/sql-injection/union-attacks)

### Examining the database
In relational databases, the information schema (information_schema) is an ANSI-standard set of read-only views that provide information about all of the tables, views, columns, and procedures in a database.
You can construct your query to something like:  
`SELECT * FROM information_schema.tables`  
This would reveal to you all the tables that are present in the database.  

You can read more about this [here](https://portswigger.net/web-security/sql-injection/examining-the-database)

### Blind SQL Injections
These are used when the application does not return the results of the SQL query or the details of any database errors within its responses. Blind vulnerabilities can still be exploited to access unauthorized data, but the techniques involved are generally more complicated and difficult to perform.  
Consider a web application that returns the message "Welcome Back!" if the username you entered exists (query returns true), or "Incorrect username! Check Again!" if the username you entered does not exist (query returns false). In this case, a blind SQLi attack can be used to obtain sensitive information like password.  
Consider the input as:  
`xyz' UNION SELECT 'a' FROM Users WHERE Username = 'Administrator' and SUBSTRING(Password, 1, 1) > 'm'--`  
If this gives us the welcome back message, then we can deduce that the first character of the password has an ASCII value greater than 'm', and vice versa. Now coupling this with the concept of binary search, we can bruteforce our way to the password.

You can read in detail about various blind sql injections [here](https://portswigger.net/web-security/sql-injection/blind)
