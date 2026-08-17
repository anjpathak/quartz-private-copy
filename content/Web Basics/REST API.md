---
{"publish":true,"created":"2026-01-29T20:22:36.000Z","modified":"2026-08-17T03:04:19.871Z"}
---

**CORS** ^55750d

- Browser way of preventing cross origin requests. Automatically send Options call first with origin, Access-Control-Request-Method, Access-Control-Request-Headers
- In Preflight Server Responds with Access-Control-Allow-Origin, Access-Control-Allow-Methods, Access-Control-Allow-Headers etc if it matched browser allows the actual api call otherwise not.
- **Common Errors**

**Error 1:** "No 'Access-Control-Allow-Origin' header is present"

- Cause: Server doesn't have CORS configured​
- Solution: Add CORS headers to server responses\
  **Error 2**: "The 'Access-Control-Allow-Origin' header has a value that is not equal to the supplied origin"
- Cause: Your origin isn't in the allowed list
- Solution: Configure server to allow your specific domain
  **Error 3**: Preflight request fails
- Cause: Server doesn't handle OPTIONS requests or doesn't return proper headers
- Solution: Implement OPTIONS handler on server

**HTTP Methods**

| Method  | Purpose                           | Payload Allowed   | Should Use Payload    | Idempotent | Safe | Response Body | Common Use Cases                                              | Example Scenario                                          |
| ------- | --------------------------------- | ----------------- | --------------------- | ---------- | ---- | ------------- | ------------------------------------------------------------- | --------------------------------------------------------- |
| GET     | Retrieve resource                 | Yes (technically) | No - use query params | Yes        | Yes  | Yes           | Fetching data, listing resources, reading information         | Get user profile, list products, search results           |
| POST    | Create resource or perform action | Yes               | Yes - required        | No         | No   | Yes           | Creating new records, submitting forms, processing operations | Register user, create order, upload file, trigger email   |
| PUT     | Replace entire resource           | Yes               | Yes - required        | Yes        | No   | Optional      | Full resource updates, replacing complete objects             | Update entire user profile, replace product details       |
| PATCH   | Partial resource update           | Yes               | Yes - required        | No         | No   | Optional      | Updating specific fields, efficient bandwidth usage           | Change email only, update order status, mark as read      |
| DELETE  | Remove resource                   | No semantics      | Rarely                | Yes        | No   | Optional      | Deleting records, removing resources                          | Delete user account, remove product, cancel order         |
| HEAD    | Get headers only                  | No semantics      | No                    | Yes        | Yes  | No            | Check existence, get metadata, validate cache                 | Check file exists, get content size, verify last-modified |
| OPTIONS | Discover capabilities             | No semantics      | Rarely                | Yes        | Yes  | Yes           | CORS preflight, API discovery, check allowed methods          | Check what operations allowed, discover API features      |

**Response codes**

| Category              | Code    | Name                  | Description                                                       | Common Use Case          |
| --------------------- | ------- | --------------------- | ----------------------------------------------------------------- | ------------------------ |
| **1xx Informational** | 100     | Continue              | Server received request headers, client should send request body. | Large file uploads.      |
|                       | ==101== | Switching Protocols   | Server is switching protocols as requested.                       | WebSocket upgrades.      |
|                       | 102     | Processing            | Server is processing but no response available yet.               | Long-running operations. |
| **2xx Success**       | 200     | OK                    | Request succeeded.                                                | GET, PUT operations.     |
|                       | 201     | Created               | New resource successfully created.                                | POST operations.         |
|                       | 202     | Accepted              | Request accepted but processing not complete.                     | Async operations.        |
|                       | 204     | No Content            | Success but no content to return.                                 | DELETE operations.       |
|                       | 206     | Partial Content       | Partial resource returned.                                        | Range requests.          |
| **3xx Redirection**   | 301     | Moved Permanently     | Resource permanently moved to new URI.                            | URL changes.             |
|                       | 302     | Found                 | Temporary redirect.                                               | Temporary URL changes.   |
|                       | 307     | Temporary Redirect    | Resubmit to different URI.                                        | Temporary redirects.     |
| **4xx Client Error**  | ==400== | Bad Request           | Invalid request syntax or malformed JSON.                         | Validation errors.       |
|                       | ==401== | Unauthorized          | Authentication required.                                          | Missing credentials.     |
|                       | ==403== | Forbidden             | Authenticated but lacks permissions.                              | Access restrictions.     |
|                       | ==404== | Not Found             | Resource does not exist.                                          | Invalid endpoints.       |
|                       | ==422== | Unprocessable Entity  | Request is well-formed but has semantic errors.                   | Business logic errors.   |
|                       | 429     | Too Many Requests     | Rate limit exceeded.                                              | API throttling.          |
| **5xx Server Error**  | ==500== | Internal Server Error | Unexpected error on the server.                                   | Server bugs.             |
|                       | ==502== | Bad Gateway           | Invalid response from upstream server.                            | Proxy errors.            |
|                       | ==503== | Service Unavailable   | Server temporarily unable to handle the request.                  | Maintenance or overload. |
|                       | ==504== | Gateway Timeout       | No timely response from upstream server.                          | Timeout issues.          |
