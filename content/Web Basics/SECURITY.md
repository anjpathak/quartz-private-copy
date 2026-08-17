---
{"publish":true,"created":"2026-01-29T20:22:47.000Z","modified":"2026-08-17T03:04:25.627Z"}
---

# Security

**CORS** [[REST API#^55750d | READ THIS]]

| Aspect                | XSS                                                                                                                                                                                                   | CSRF                                                                                                                                                                                                   |
| --------------------- | ----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- | ------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------ |
| **What is exploited** | Exploits the user's trust in a website [escape](https://escape.tech/blog/csrf-vs-xss/)​                                                                                                               | Exploits the website's trust in the user's browser [openappsec](https://www.openappsec.io/post/csrf-vs-xss)​                                                                                           |
| **Attack mechanism**  | Injects malicious client-side scripts into a website [geeksforgeeks](https://www.geeksforgeeks.org/computer-networks/difference-between-xss-and-csrf/)​                                               | Tricks users into sending malicious requests to a target site [geeksforgeeks](https://www.geeksforgeeks.org/computer-networks/difference-between-xss-and-csrf/)​                                       |
| **Target**            | Affects all users visiting the compromised site (one-to-many) YouTube​                                                                                                                                | Targets specific authenticated users (one-to-one) YouTube​                                                                                                                                             |
| **Goal**              | Steal user data, session tokens, or manipulate the site's content [locknetmanagedit](https://www.locknetmanagedit.com/blog/cybersecurity/cross-site-request-forgery-vs-cross-site-scripting)​YouTube​ | Perform unauthorized actions like transferring money or changing settings [locknetmanagedit+1](https://www.locknetmanagedit.com/blog/cybersecurity/cross-site-request-forgery-vs-cross-site-scripting) |
| **Requirement**       | Website must be vulnerable to script injection YouTube​                                                                                                                                               | User must be authenticated to the target site YouTube​[openappsec](https://www.openappsec.io/post/csrf-vs-xss)​                                                                                        |

**CSRF**

- `Same-origin / Same - origin` : Two URLs have the same origin if they share the same protocol, domain, and port.
  - https://example.com/page1 and https://example.com/page2 are same-origin
  - https://example.com and http://example.com are different origins (different protocol)
  - https://example.com and https://api.example.com are different origins (different subdomain)
- `Cross-site/Cross-origin` :  A request from one origin to a different origin. For example, if you're on https://attacker.com and JavaScript or HTML on that page makes a request to , that's a cross-site request
- [[Storage#^1fd65d| ALSO CHECK]] **Cookies are domain-scoped, not page-scoped**. When your browser stores a cookie, it associates that cookie with a specific domain (like yourbank.com). The key browser behavior: Whenever your browser makes any HTTP request to a domain, it automatically includes all cookies that belong to that domain—regardless of which website initiated the request. ^76c7e6
- TLDR, cookies are sent based on target domain of the request and not based on where the request originates.
- To prevent CSRF we use cookie like this :
  ```
  Set-Cookie: sessionid=abc123; SameSite=Strict
  ```
- SameSite = Strict : Don't send cookies with corss site req
- SameSite = Lax (Default): Only send cookie with corss-site req if top level navigation.
- SameSite = none : Allows CSRF.

```## ✅ Cookies ARE Sent
**1. Same-site requests** (any method)

- You're on `yourbank.com/home` and click a link to `yourbank.com/transfer`
    
- Cookies are always sent for requests within the same site
    

**2. Top-level navigation with safe HTTP methods (GET)**

- You're on `google.com` and click a link: `<a href="https://yourbank.com">Visit Bank</a>`
    
- Your browser navigates to `yourbank.com` and includes the session cookie
    
- This allows you to stay logged in when arriving from external links[](https://owasp.org/www-community/SameSite)​
    

## ❌ Cookies Are NOT Sent

**1. Embedded resources from cross-site**

xml

`<!-- You're on evil.com --> <img src="https://yourbank.com/transfer?amount=5000"> <script src="https://yourbank.com/api/data.js"></script> <iframe src="https://yourbank.com/account"></iframe>`

None of these requests include your `yourbank.com` cookies.[](https://cookie-script.com/documentation/samesite-cookie-attribute-explained)​
```

​
