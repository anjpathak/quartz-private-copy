---
{"publish":true,"created":"2026-01-29T11:02:13.000Z","modified":"2026-08-17T03:04:27.652Z"}
---

| Type           | Persistence             | Capacity          | Use Case                   | Key Traits                                        |
| -------------- | ----------------------- | ----------------- | -------------------------- | ------------------------------------------------- |
| localStorage   | Forever (until cleared) | 5-10MB per origin | User settings, app state   | Synchronous, simple key-value ​                   |
| sessionStorage | Current tab/session     | 5-10MB per origin | Temporary form data        | Clears on tab close, synchronous ​                |
| Cookies        | Configurable expiry     | ~4KB per cookie   | Auth tokens, tracking      | Sent with every HTTP request, server-accessible ​ |
| IndexedDB      | Forever (until cleared) | Hundreds of MB+   | Complex data, offline apps | Asynchronous NoSQL database with queries ​        |
**Local Storage**

- Data is stored as strings. So complex objects require JSON serialization before saving (`JSON.stringify()`) and deserialization after retrieval (`JSON.parse()`).
- Access it via `window.localStorage` with methods like `setItem(key, value)`, `getItem(key)`, `removeItem(key)`, and `clear()` for the entire storage
- It's origin-specific (protocol + domain + port), blocking cross-site access for security.
- Exceeding the localStorage quota triggers a `QuotaExceededError` (or `DOMException`) on `setItem()`, preventing the new data from being saved without affecting existing data.
- Old data is not overridden or deleted by the browser in this case; the operation simply fails synchronously.
- Wrap `localStorage.setItem()` in a try-catch block to detect the error. This allows manual cleanup of `least-used keys` before retrying.

**Cookies** [[SECURITY#^76c7e6| See Also]] ^1fd65d

- set cookie:

```
document.cookie = "theme=dark"; 
OR
document.cookie = "sessionId=abc123; max-age=3600; secure; httpOnly; path=/";`
	
Only sets one cookie at a time. max-age,secure etc are set as attributes and consumed by browser directly. JS in future can't access these attributes in future directly it can only set them

document.cookie = "username=JohnDoe"; 
document.cookie = "username=Ramesh";
Overwrites cookie.
```

- get cookie

```
cookies are stored as semi-colon seprated properties as `username=JohnDoe; theme=dark; lang=en`

console.log(document.cookie);
outputs : "username=JohnDoe ; theme=dark ; lang=en"


- Can't be accessd one by one. 
- To get one cookie value need to have a seperate extractCookie or get cookie function.

```
