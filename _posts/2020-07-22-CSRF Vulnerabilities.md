---
title: Common CSRF Vulnerabilities
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
key: csrf00002
---

Many times CSRF Vulnerabilities arise due to mistakes in validation of CSRF Tokens.
In [this](https://github.com/gurnoor6/csec/blob/master/csrf.md#how-does-csrf-work) example, suppose that the application now includes a CSRF token within the request to change the user's password:

```
...
Content-Length: 68
...

csrf=WfF1szMUHhiokx9AHFply5L2xAOfjRkE&email=wiener@normal-user.com
```

At a glance, this violates one of the key conditions for a CSRF Attack, condition #2 (Cookies have to be the sole method of session handling), and the request contains a parameter whose value an attacker cannot determine. However, there are various ways in which the defense can be broken, meaning that the application is still vulnerable to CSRF.

* **Validation of CSRF token depends on request method -** Some applications correctly validate the token when the request uses the POST method but skip the validation when the GET method is used. In this situation, the attacker can switch to the GET method to bypass the validation and deliver a CSRF attack:
    ```
    GET /email/change?email=pwned@evil-user.net HTTP/1.1
    Host: vulnerable-website.com
    Cookie: session=2yQIDcpia41WrATfjPqvm9tOkDvkMvLm
    ```

    You can try this out [here](https://portswigger.net/web-security/csrf/lab-token-validation-depends-on-request-method)

* **Validation of CSRF token depends on token being present -** Some applications correctly validate the token when it is present but skip the validation if the token is omitted. In this situation, the attacker can remove the entire parameter containing the token (not just its value) to bypass the validation and deliver a CSRF attack:

    ```
    POST /email/change HTTP/1.1
    Host: vulnerable-website.com
    Content-Type: application/x-www-form-urlencoded
    Content-Length: 25
    Cookie: session=2yQIDcpia41WrATfjPqvm9tOkDvkMvLm

    email=pwned@evil-user.net
    ```

    You can try this out [here](https://portswigger.net/web-security/csrf/lab-token-validation-depends-on-token-being-present)

* **CSRF token is not tied to the user session -** Some applications do not validate that the token belongs to the same session as the user who is making the request. Instead, the application maintains a global pool of tokens that it has issued and accepts any token that appears in this pool.

    In this situation, the attacker can log in to the application using their own account, obtain a valid token, and then feed that token to the victim user in their CSRF attack.

    You can try this out [here](https://portswigger.net/web-security/csrf/lab-token-not-tied-to-user-session)

* **CSRF token is tied to a non-session cookie -** In a variation on the preceding vulnerability, some applications do tie the CSRF token to a cookie, but not to the same cookie that is used to track sessions. This can easily occur when an application employs two different frameworks, one for session handling and one for CSRF protection, which are not integrated together:

    ```
    POST /email/change HTTP/1.1
    Host: vulnerable-website.com
    Content-Type: application/x-www-form-urlencoded
    Content-Length: 68
    Cookie: session=pSJYSScWKpmC60LpFOAHKixuFuM4uXWF; csrfKey=rZHCnSzEp8dbI6atzagGoSYyqJqTz5dv

    csrf=RhV7yQDO0xcq9gLEah2WVbmuFqyOq7tY&email=wiener@normal-user.com
    ```

    You can try this out [here](https://portswigger.net/web-security/csrf/lab-token-tied-to-non-session-cookie)

* **CSRF token is simply duplicated in a cookie -** In a further variation on the preceding vulnerability, some applications do not maintain any server-side record of tokens that have been issued, but instead duplicate each token within a cookie and a request parameter. When the subsequent request is validated, the application simply verifies that the token submitted in the request parameter matches the value submitted in the cookie. This is sometimes called the "double submit" defense against CSRF, and is advocated because it is simple to implement and avoids the need for any server-side state:

    ```
    POST /email/change HTTP/1.1
    Host: vulnerable-website.com
    Content-Type: application/x-www-form-urlencoded
    Content-Length: 68
    Cookie: session=1DQGdzYbOJQzLP7460tfyiv3do7MjyPw; csrf=R8ov2YBfTYmzFyjit8o2hKBuoIjXXVpa

    csrf=R8ov2YBfTYmzFyjit8o2hKBuoIjXXVpa&email=wiener@normal-user.com
    ```

    You can try this out [here](https://portswigger.net/web-security/csrf/lab-token-duplicated-in-cookie)

* **Referer-based defenses against CSRF -** Aside from defenses that employ CSRF tokens, some applications make use of the HTTP Referer header to attempt to defend against CSRF attacks, normally by verifying that the request originated from the application's own domain. This approach is generally less effective and is often subject to bypasses.

* **Validation of Referer depends on header being present -** Some applications validate the Referer header when it is present in requests but skip the validation if the header is omitted.

    In this situation, an attacker can craft their CSRF exploit in a way that causes the victim user's browser to drop the Referer header in the resulting request. There are various ways to achieve this, but the easiest is using a META tag within the HTML page that hosts the CSRF attack:

    `<meta name="referrer" content="never">`

    You can try this out [here](https://portswigger.net/web-security/csrf/lab-referer-validation-depends-on-header-being-present)

* **Validation of Referer can be circumvented -** Some applications validate the Referer header in a naive way that can be bypassed. For example, if the application simply validates that the Referer contains its own domain name, then the attacker can place the required value elsewhere in the URL:

    `http://attacker-website.com/csrf-attack?vulnerable-website.com`

    If the application validates that the domain in the Referer starts with the expected value, then the attacker can place this as a subdomain of their own domain:

    `http://vulnerable-website.com.attacker-website.com/csrf-attack`

    You can try this out [here](https://portswigger.net/web-security/csrf/lab-referer-validation-broken)

For more details regarding any of the topics related to CSRF, visit [this](https://portswigger.net/web-security/csrf) link.
