---
{"publish":true,"created":"2025-12-19T17:55:58.000Z","modified":"2026-08-17T03:04:29.326Z"}
---

How does a website work end to end. From entering a url to getting a page load.

## 1. URL Parsing & DNS Lookup

https://user:pass@sub.example.com:8080/path/to/page.html?search=query\&filter=active#section1

```
const url = new URL('https://example.com/path?foo=bar#hash');
console.log(url.protocol); // "https:"
console.log(url.host);     // "example.com"
console.log(url.pathname); // "/path"
console.log(url.search);   // "?foo=bar"
console.log(url.hash);     // "#hash"
```

|                            |                               |                                                                                                     |               |
| -------------------------- | ----------------------------- | --------------------------------------------------------------------------------------------------- | ------------- |
| **Protocol**               | `https://`                    | Tells browser how to connect (http, https, ftp, mailto). HTTPS = encrypted.                         | Yes           |
| **User Info** (deprecated) | `user:pass@`                  | Authentication credentials (avoid in modern URLs).                                                  | No            |
| **Host**                   | `sub.example.com`             | Server address. Includes **subdomain** (`sub.`) + **domain** (`example`) + **TLD** (`.com`).        | Yes           |
| **Port**                   | `:8080`                       | Custom server port (defaults: http=80, https=443).                                                  | No            |
| **Path**                   | `/path/to/page.html`          | File/resource location on server, slash-separated.                                                  | No (root=`/`) |
| **Query String**           | `?search=query&filter=active` | Key-value params (`key=value`) for dynamic data, server-side processing. `?` starts, `&` separates. | No            |
| **Fragment**               | `#section1`                   | Client-side anchor/jump target (e.g., scroll to element ID). Never sent to server.                  | No            |

## 2. TCP Connection & TLS Handshake

Browser establishes TCP connection (3-way handshake). For HTTPS: TLS negotiates encryption (client hello → server cert → key exchange).[](https://dev.to/sayanide/the-what-why-and-how-of-javascript-bundlers-4po9)​

## 3. HTTP Request/Response

Browser sends GET request with headers (User-Agent, cookies, cache info). Server processes (auth, routing), returns HTML + headers (Content-Type, Cache-Control, ETag).[](https://dev.to/sayanide/the-what-why-and-how-of-javascript-bundlers-4po9)​

## 4. Resource Fetching & Processing

|Step|What Happens|Key Tech|
|---|---|---|
|**HTML Parse**|Browser builds DOM tree from HTML|Parser|
|**CSS Fetch/Parse**|Parallel fetches, builds CSSOM|Critical CSS inline|
|**JS Fetch/Parse**|Deferred until DOMContentLoaded|`<script defer/async>`|
|**Critical Render Path**|DOM + CSSOM → Render Tree → Layout → Paint|Composite layers|

## 5. Rendering Pipeline

1. **Layout (Reflow)**: Calculate positions/sizes

2. **Paint**: Rasterize to bitmaps

3. **Composite**: Layer manager assembles final pixels[](https://www.fe.engineer/handbook/bundlers)​

## 6. JavaScript Execution & Hydration

- JS runs, manipulates DOM (React/Vue mount)

- Event listeners attach

- Service workers/cache take over future loads[](https://dev.to/sayanide/the-what-why-and-how-of-javascript-bundlers-4po9)​
