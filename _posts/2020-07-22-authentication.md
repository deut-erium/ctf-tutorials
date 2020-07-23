---
title: Vulnerabilities in Authentication Mechanisms
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
key: authentication00001
---

# Authentication
In this section, we'll look at some of the most common authentication mechanisms used by websites and discuss potential vulnerabilities in them.

## What is authentication?
Authentication is the process of verifying the identity of a given user or client.

There are three authentication factors into which different types of authentication can be categorized: 
* Something you know, such as a password.
* Something you have, that is, a physical object like a mobile phone or security token.
* Something you are or do, for example, your biometrics or patterns of behavior.

## What is the difference between authentication and authorization?
Authentication is the process of verifying that a user really is who they claim to be, whereas authorization involves verifying whether a user is allowed to do something. 

## How do authentication vulnerabilities arise?
* The authentication mechanisms are weak because they fail to adequately protect against brute-force attacks.
* Logic flaws or poor coding in the implementation allow the authentication mechanisms to be bypassed entirely by an attacker. This is sometimes referred to as "broken authentication". 

## What is the impact of vulnerable authentication?
The impact of authentication vulnerabilities can be very severe. Once an attacker has either bypassed authentication or has brute-forced their way into another user's account, they have access to all the data and functionality that the compromised account has. If they are able to compromise a high-privileged account, such as a system administrator, they could take full control over the entire application and potentially gain access to internal infrastructure. 

Authentication based vulnerabilities can be broadly classified into three categories-
* Password based vulnerabilities
* Multi factor authentication based vulnerabilities
* Vulnerabilities in other authentication mechanisms

# Vulnerabilities in Password Based Login
In this scenario, the mere fact that the user knows the secret password is taken as sufficient proof of the user's identity. Consequently, the security of the website would be compromised if an attacker is able to either obtain or guess the login credentials of another user. 

## Brute-force attacks
A brute-force attack is when an attacker uses a system of trial and error in an attempt to guess valid user credentials. These attacks are typically automated using wordlists of usernames and passwords, and plenty of tools are available for this purpose (Burp Suite is one such tool). 

### Brute-forcing usernames
Often times the attacker knows the username of the victim. But there are cases where he might not know that. So he needs to find the username by brute forcing.  Usernames are especially easy to guess if they conform to a recognizable pattern, such as an email address. For example, it is very common to see business logins in the format `firstname.lastname@somecompany.com` . However, even if there is no obvious pattern, sometimes even high-privileged accounts are created using predictable usernames, such as admin or administrator. 

### Brute-forcing passwords
Passwords can similarly be brute-forced, with the difficulty varying based on the strength of the password. Many websites adopt some form of password policy, which forces users to create high-entropy passwords that are, theoretically at least, harder to crack using brute-force alone. This typically involves enforcing passwords with: 

* A minimum number of characters
* A mixture of lower and uppercase letters
* At least one special character

However, while high-entropy passwords are difficult for computers alone to crack, we can use a basic knowledge of human behavior to exploit the vulnerabilities that users unwittingly introduce to this system. Rather than creating a strong password with a random combination of characters, users often take a password that they can remember and try to crowbar it into fitting the password policy. For example, if `mypassword` is not allowed, users may try something like `Mypassword1!` or `Myp4$$w0rd` instead. 

### Username enumeration
Username enumeration is when an attacker is able to observe changes in the website's behavior in order to
identify whether a given username is valid. 

While attempting to brute-force a login page to find out the username, you should pay particular attention to any differences in:

* **Status codes**: During a brute-force attack, the returned HTTP status code is likely to be the same for the vast majority of guesses because most of them will be wrong. If a guess returns a different status code, this is a strong indication that the username was correct. It is best practice for websites to always return the same status code regardless of the outcome, but this practice is not always followed.

* **Error messages**: Sometimes the returned error message is different depending on whether both the username AND password are incorrect or only the password was incorrect. It is best practice for websites to use identical, generic messages in both cases, but small typing errors sometimes creep in.

* **Response times**: If most of the requests were handled with a similar response time, any that deviate from this suggest that something different was happening behind the scenes. This is another indication that the guessed username might be correct. For example, a website might only check whether the password is correct if the username is valid. This extra step might cause a slight increase in the response time. This may be subtle, but an attacker can make this delay more obvious by entering an excessively long password that the website takes noticeably longer to handle.

## Flawed brute-force protection
Many websites adopt measures to prevent brute force attacks. The two most common ways of preventing brute-force attacks are: 

* Locking the account that the remote user is trying to access if they make too many failed login attempts.
* Blocking the remote user's IP address if they make too many login attempts in quick succession.

Both approaches offer varying degrees of protection, but neither is invulnerable, especially if implemented using flawed logic.

For example, you might sometimes find that your IP is blocked if you fail to log in too many times. In some implementations, the counter for the number of failed attempts resets if the IP owner logs in successfully. This means an attacker would simply have to log in to their own account every few attempts to prevent this limit from ever being reached.
 
### Account locking
One way in which websites try to prevent brute-forcing is to lock the account if certain suspicious criteria are met, usually a set number of failed login attempts. Just as with normal login errors, responses from the server indicating that an account is locked can also help an attacker to enumerate usernames. 

The following method can be used to work around this kind of protection:

* Establish a list of candidate usernames that are likely to be valid. This could be through username enumeration or simply based on a list of common usernames. 
* Decide on a very small shortlist of passwords that you think at least one user is likely to have. Crucially, the number of passwords you select must not exceed the number of login attempts allowed. For example, if you have worked out that limit is 3 attempts, you need to pick a maximum of 3 password guesses. 
* Using a tool such as Burp Intruder, try each of the selected passwords with each of the candidate usernames. This way, you can attempt to brute-force every account without triggering the account lock. You only need a single user to use one of the three passwords in order to compromise an account.

### User rate limiting
Another way websites try to prevent brute-force attacks is through user rate limiting. In this case, making too many login requests within a short period of time causes your IP address to be blocked. Typically, the IP can only be unblocked in one of the following ways:

* Automatically after a certain period of time has elapsed
* Manually by an administrator
* Manually by the user after successfully completing a CAPTCHA

User rate limiting is sometimes preferred to account locking due to being less prone to username enumeration and denial of service attacks. However, it is still not completely secure. There are several ways an attacker can manipulate their apparent IP in order to bypass the block. 

# Vulnerabilities in multi-factor authentication
In this section, we'll look at some of the vulnerabilities that can occur in multi-factor authentication mechanisms. The details are a bit more involved, so we won't be touching them. If you wish to learn more about it, please refer to [here](https://portswigger.net/web-security/authentication/multi-factor).

Many websites rely exclusively on single-factor authentication using a password to authenticate users. However, some require users to prove their identity using multiple authentication factors- which is usually *something you know* and *something you have*. This usually requires users to enter both a traditional password and a temporary verification code from an out-of-band physical device in their possession. 

The major vulnerabilities in Two-factor authentication are:
## Transmitting tokens through SMS
Many websites send a token to the users through text messages, which is again vulnerable to being intercepted. There is also a risk of SIM swapping, whereby an attacker fraudulently obtains a SIM card with the victim's phone number. The attacker would then receive all SMS messages sent to the victim, including the one containing their verification code.
## Bypassing two-factor authentication
At times, the implementation of two-factor authentication is flawed to the point where it can be bypassed entirely. If the user is first prompted to enter a password, and then prompted to enter a verification code on a separate page, the user is effectively in a "logged in" state before they have entered the verification code. In this case, it is worth testing to see if you can directly skip to "logged-in only" pages after completing the first authentication step. 
## Flawed two-factor verification logic
Sometimes flawed logic in two-factor authentication means that after a user has completed the initial login step, the website doesn't adequately verify that the same user is completing the second step. So what actually happens is, once you log in, you are assigned a cookie that contains the username of the account you are trying to access. If the attacker manipulates the username in the cookie and brute force the verification token, he can effectively log in to any random user's account by just knowing the username!! (Refer [here](https://portswigger.net/web-security/authentication/multi-factor) to know the specifics)
## Brute-forcing 2FA verification codes
As with passwords, websites need to take steps to prevent brute-forcing of the 2FA verification code. This is especially important because the code is often a simple 4 or 6-digit number. Without adequate brute-force protection, cracking such a code is trivial. 

# Vulnerabilities in other authentication mechanisms
In addition to the basic login functionality, most websites provide supplementary functionality to allow users to manage their account. For example, users can typically change their password or reset their password when they forget it. These mechanisms can also introduce vulnerabilities that can be exploited by an attacker. 

## Keeping users logged in
A common feature is the option to stay logged in even after closing a browser session. This is usually a simple checkbox labeled something like "Remember me" or "Keep me logged in".

This functionality is often implemented by generating a "remember me" token of some kind, which is then stored in a persistent cookie. As possessing this cookie effectively allows you to bypass the entire login process, it is best practice for this cookie to be impractical to guess. However, some websites generate this cookie based on a predictable concatenation of static values, such as the username and a timestamp. Some even use the password as part of the cookie. This approach is particularly dangerous if an attacker is able to create their own account because they can study their own cookie and potentially deduce how it is generated. Once they work out the formula, they can try to brute-force other users' cookies to gain access to their accounts. 

Even if the attacker is not able to create their own account, they may still be able to exploit this vulnerability. Using the usual techniques, such as XSS, an attacker could steal another user's "remember me" cookie and deduce how the cookie is constructed from that. If the website was built using an open-source framework, the key details of the cookie construction may even be publicly documented.

## Resetting user passwords
In practice, it is a given that some users will forget their password, so it is common to have a way for them to reset it. As now the users cannot be authenticated using their passwords, this feature needs to be implemented very securely.

### Sending passwords by email
Generally, when you reset a password, you receive an email with a new temporary password which is valid for a short period of time. This approach is susceptible to man in the middle attacks, so is vulnerable. Also, amny users sync their emails across multiple devices, transmitting sensitive data over insecure channels.

### Resetting passwords using a URL
A more robust method of resetting passwords is to send a unique URL to users that takes them to a password reset page. Less secure implementations of this method use a URL with an easily guessable parameter to identify which account is being reset, for example: ` http://vulnerable-website.com/reset-password?user=victim-user`.
In this example, an attacker could change the user parameter to refer to any username they have identified. They would then be taken straight to a page where they can potentially set a new password for this arbitrary user. 

### Password reset poisoning
This method relies on manipulating the HTTP request headers so that the link for password reset that the victim user receives, is what the attacker controls. So when the victim clicks on that link, the attacker gets access to the original URL that can be used for the password reset of the victim. To get more idea about how it actually works, solve the lab for it [here](https://portswigger.net/web-security/authentication/other-mechanisms/lab-password-reset-poisoning) .

### Changing user passwords
Typically what happens is that when you click on the link for resetting the password, you are redirected to a page that asks for a new password twice. Once you enter them and submit the form, the password is reset. However, if there is a hidden `username` field, that is used to display the form, the attacker might manipulate it to reset the password of any user, without even having them to click the link!!



The material has been taken from [here](https://portswigger.net/web-security/authentication). So if you wish to learn more about any of the above sections or want to gain hands on experience with such attacks, do check it out!!
{:.info}


