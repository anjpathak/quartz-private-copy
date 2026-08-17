---
{"publish":true,"created":"2026-01-30T08:19:01.000Z","modified":"2026-08-17T03:04:09.204Z"}
---

**Authorisation** - is which role is allowed to access what resources
**Authentication** - the user is what he says he is. Confiming user identity

# Read this for system design[[Auth Concerns.pdf]]

# Session Based

#### Step-by-Step Flow with Code Examples

**A. Authentication (The Handshake)**

The client sends credentials. If valid, the server creates a session and instructs the browser to save a unique ID.

```
@PostMapping("/login")
fun login(request: HttpServletRequest): ResponseEntity<String> {
    // 1. Verify credentials (DB check)
    // 2. request.getSession(true) creates a new session in server memory
    val session = request.getSession(true)
    
    // 3. Store only the "Key" to the user (Primary Key)
    session.setAttribute("user_id", 101) 
    
    // Spring automatically adds 'Set-Cookie: JSESSIONID=XYZ123' to response
    return ResponseEntity.ok("Session Created")
}
```

The browser handles the session cookie **automatically**. No manual token management is required in JS.

```
// Browser receives 'Set-Cookie' and stores it.
// All subsequent requests automatically include 'Cookie: JSESSIONID=XYZ123'.
fetch('/api/profile')
  .then(res => res.json())
  .then(data => console.log(data));
```

**B. Authorization (Verification)**\
The server identifies the user by looking up the incoming Cookie ID in its internal memory.

```
@GetMapping("/profile")
fun getProfile(session: HttpSession): Any {
    // Spring extracts JSESSIONID from header and finds matching session object
    val userId = session.getAttribute("user_id")
    return userId ?: throw UnauthorizedException()
}
```

**C. Logout (Termination)**\
The server must explicitly destroy the record to invalidate the ID.

```
@PostMapping("/logout")
fun logout(session: HttpSession) {
    session.invalidate() // Deletes server-side record
}
```

#### Data Management: What gets stored and where?

**A. Storage Locations**

- **Server-Side:** The actual "Session Object" (Map of data) lives in **Server RAM (Memory)** or a **Distributed Cache (Redis/SQL)**. It never leaves the server environment.
- **Client-Side:** Only the **Session ID** (the "Opaque" key) is stored in the browser's cookie jar. The client remains "blind" to the actual data.

**B. Data Content Rules**

- **What to Store:**
  - **Identifiers:** Database Primary Key (`user_id`).
  - **Permissions:** Short strings for roles (`ADMIN`, `USER`) to avoid repetitive DB lookups.
  - **Metadata:** Timestamp of last activity (for timeout calculation).
- **What NOT to Store:**
  - **Passwords/Hashes:** Never store credentials in the session; once logged in, the ID is the proof.
  - **Large Objects:** Avoid storing full User profiles or lists. Sessions consume **Server RAM**; large objects lead to memory exhaustion as concurrent users increase.
  - **Sensitive PII:** Avoid storing Credit Card numbers or SSNs.

**C. Naming & Security**

- **Renaming:** Override the default `JSESSIONID` in `application.properties` (e.g., `server.servlet.session.cookie.name=MY_SESSION_KEY`) to obscure your tech stack from attackers.
- **Attributes:** Ensure the cookie is sent with `HttpOnly` (stops JS theft), `Secure` (HTTPS only), and `SameSite=Strict` (stops CSRF). Example in application.yml.

```
server.servlet.session.cookie.http-only=true
server.servlet.session.cookie.secure=true
```

#### Tradeoffs

- Server needs to maintain user session data (in memory, in db or in cache like Redis).  ==Creates storage overhead==
- Different microservices will need to call Redis seperately creating a ==network overhead== if millions of user.

# JWT

A JWT consists of three **Base64URL-encoded** parts separated by dots
Encoded String Example: **eyJhbGci... . eyJzdWIi... . SflKxwRJ...**

- User first needs to verify either using password or through Oauth providers.
- Stateless, Server doesn't need to maintain user session data(as opposed to session based authentication).
- In distributed architecture each microservice can verify token independenly w/o any network calls or storage overhead.
- Consists of 3 parts :
  - Header : algorithm and token type `{"alg": "HS256", "typ": "JWT"}`
  - payload : user ==claims== like user id, expiration time `{"sub": "123", "role": "ADMIN", "exp": 1711100000}`
  - signature: Cryptographic proof of integrity. ==A hash of (Header + Payload + Secret Key)==
  - ex: Encoded String Example: **eyJhbGci... . eyJzdWIi... . SflKxwRJ...**
- Usually servers return both :
  - Access token - short lived - sent with api requests
  - [[Authentication#^37051b | Refresh token]] - long lived - used to obtain new access token.
- Passed in Authorisation headers like this :

```
axios.get(URL, {
    headers: {
        'Authorization': 'Bearer ' + token,
    },
})
```

- Storage can be in :
  - localStorage : Persists browser close, but vulnerable to XSS since JS can access it
  - sessionStorage : Lost on reloads. Offers better security than localStorage, but requires manual state management.
  - Cookies : most secure. Configure with attributes :
    - **HTTPOnly** : prevents JS access to cookie via `document.cookie` API. Only web server can access. Prevents XSS.
    - **Secure :** send cookie only over HTTPS and not HTTP. Prevents man in the middle.
    - **SameSite** :  \`
      - `Strict`: Never sent with cross-site requests
      - `Lax`: Sent with top-level navigation only
      - `None`: Sent with all requests (requires `Secure` flag)
- **JWT is Sent as Authorisation Header or in cookie**
  - Backend architecture dictates how we want to send tokens. If backend sends the token in set-cookie header in a response, than on subsequent req we dont need to add it manually in headers, as cookies will always be sent automatically by browser. Server will check like this ==let token = cookie.auth&#x20;==
  - if not using cookies for token storage, than token needs to be sent in Authorisation Header.
  - `Authorization: <scheme> <credentials> `
  - **scheme can be Bearer | Basic | OAuth**
  - Scheme can be omitted theoretically but would most probably fail authentication as most backend frameworks use scheme to process the type of token. ==Basic is for user id pwd==
  - ==\*\*Cookies\*\* are preferred when you control both frontend and backend, especially for server-side rendered apps, because they're automatic and more secure with HttpOnly flag. \*\*localStorage\*\* is common in SPAs and when using third-party APIs that can't set cookies for your domain or specifically require the Authorization header format==

```
	// After login
localStorage.setItem('token', responseToken);

// For every API request
const token = localStorage.getItem('token');
axios.get(URL, {
    headers: {
        'Authorization': 'Bearer ' + token
    }
});

```

###### Refresh Token

- Refresh tokens are only used to fetch new access tokens if they are expiring,
- If Using cookies for storage :
  - Server will initially send access and refresh tokoen in set-cookie.
  - in subsequent requests server can always check if the access token is about to expire and can send a new access token in set-cookie on its own. No UI logic needed.
  - UI can also try to refesh token based on a regular interval timer, or if the access token is going to expire soon checking before every api call or if 401 recieved.
  - Best approach is to use interceptors to intercept 401 responses and check if original request has not being retried than refresh token and retry once.^37051b
  - ==Cookie Path Optimization== Backend can set path for the refresh token so that from frontend browser includes this refresh token cookie only when sending refrersh req to the url and not every url:

```
// Refresh token only sent to /api/refresh
res.cookie('refreshToken', refreshToken, {
  httpOnly: true,
  path: '/api/refresh', // ⭐ Key optimization
  maxAge: 7 * 24 * 60 * 60 * 1000
});

```

---

==Symmetric Signature (HS256) —== 
Your logic is spot on. It is a "Shared Secret" approach.

- **Creation:** `signature = HMAC_SHA256(base64(Header) + "." + base64(Payload), SharedSecret)`
- **Verification:** The server repeats the exact same math using its copy of the `SharedSecret`. If the results match, the token is valid.

---

==Asymmetric Signature (RS256) —==
In Asymmetric signing, there is **no "Secret Key" (string)** inside the hash. Instead, the **Private Key** itself performs the mathematical transformation.

**How it actually works:**

- **Creation (Auth Server):**

  1. `hash = SHA256(base64(Header) + "." + base64(Payload))` (Just a plain fingerprint).
  2. `signature = PrivateKey_Sign(hash)` (The Private Key "encrypts" or signs that fingerprint).

  - **JWT** = `base64(Header) . base64(Payload) . signature`
- **Verification (Resource Server):**
  1. The server receives the JWT.
  2. `decryptedHash = PublicKey_Verify(signature)` (The Public Key "unlocks" the signature to reveal the original fingerprint).
  3. `calculatedHash = SHA256(base64(Header) . base64(Payload))` (The server calculates its own fingerprint from the data it received).
  4. **The Match:** If `decryptedHash == calculatedHash`, the token is authentic.

---

# OAuth 2.0: Complete Summary

## What is OAuth?

**OAuth** is an **authorization protocol** (not authentication) that solves the problem of **delegated access**—allowing third-party apps to access your resources without sharing your password.[](https://supertokens.com/blog/oauth-vs-jwt)

**Key Problem Solved**: How can a photo printing app access your Google Photos without you giving it your Google password?[](https://permify.co/post/oauth-jwt-comparison/)

**Analogy**: OAuth is like giving a **guest pass** (limited, time-bound, revocable) instead of your **house keys** (full access, permanent, irrevocable).[](https://www.linkedin.com/posts/inc-abhishek_oauth-backendengineering-systemdesign-activity-7417510906462416896-uili)​

## OAuth vs JWT vs SSO vs SAML

|Technology|Type|Purpose|Use Case|
|---|---|---|---|
|**JWT**|Token format [](https://supertokens.com/blog/oauth-vs-jwt)​|Stateless data transmission|Any app needing tokens [](https://www.descope.com/blog/post/jwt-vs-oauth)​|
|**OAuth**|Authorization protocol [](https://supertokens.com/blog/oauth-vs-jwt)​|Third-party access without passwords|"Login with Google", API integrations [](https://permify.co/post/oauth-jwt-comparison/)​|
|**SSO**|Authentication pattern [](https://workos.com/blog/sso-vs-oauth)​|One login for multiple apps|Company apps (Gmail, Drive, Calendar) [](https://systemdesignschool.io/blog/sso-vs-oauth)​|
|**SAML**|Protocol (XML-based) [](https://ones.com/blog/demystifying-saml-authentication-flow-diagrams/)​|Enterprise SSO|Corporate identity systems [](https://ones.com/blog/saml-authentication-flow-diagram-explained/)​|

**They often work together**: OAuth can use JWT as its token format; SSO can use OAuth/OpenID Connect or SAML as the underlying protocol.[](https://www.fortinet.com/resources/cyberglossary/saml-vs-oauth)

# OAuth 2.0 Authorization Code Flow (Most Common)

## Participants

- **User**: Person who owns the data

- **Client**: Your app (frontend + backend)

- **Authorization Server**: OAuth provider (Google, GitHub)

- **Resource Server**: API with user's data[](https://dev.to/devcorner/how-oauth-20-authorization-flow-works-explained-for-developers-2kj3)​

## The Flow

**1. User initiates login** → **Authorization Server**\
Client sends: `client_id`, `redirect_uri`, `scope`, `response_type=code`[](https://www.oauth.com/oauth2-servers/server-side-apps/example-flow/)

**2. User authenticates** at **Authorization Server**\
User enters: **username + password** (directly to Google, never to your app)[](https://supertokens.com/blog/how-does-oauth-work)​

**3. Authorization Server** → **Client**\
Returns: **Authorization Code** (temporary, single-use, ~10 min expiry)[](https://auth0.com/docs/get-started/authentication-and-authorization-flow/authorization-code-flow)

**4. Client Backend** → **Authorization Server**\
Sends: Authorization Code, `client_id`, **client\_secret**, `redirect_uri`[](https://dev.to/devcorner/how-oauth-20-authorization-flow-works-explained-for-developers-2kj3)

**5. Authorization Server** → **Client Backend**\
Returns: **Access Token** (1 hour), **Refresh Token** (long-lived), **ID Token** (if using OpenID Connect)[](https://www.altexsoft.com/blog/oauth/)

**6. Client** → **Resource Server**\
Sends: `Authorization: Bearer {access_token}`[](https://dev.to/devcorner/how-oauth-20-authorization-flow-works-explained-for-developers-2kj3)​

**7. When token expires** → **Authorization Server**\
Sends: Refresh Token\
Gets: New Access Token[](https://www.altexsoft.com/blog/oauth/)​

## Key Tokens in OAuth

- **Authorization Code**: Temporary code exchanged for tokens (prevents token exposure in URLs)[](https://supertokens.com/blog/how-does-oauth-work)​

- **Access Token**: Proves authorization to access resources[](https://dev.to/devcorner/how-oauth-20-authorization-flow-works-explained-for-developers-2kj3)​

- **Refresh Token**: Gets new access tokens without re-login[](https://www.altexsoft.com/blog/oauth/)​

- **ID Token** (OpenID Connect): Contains user identity as JWT[](https://developer.okta.com/docs/concepts/oauth-openid/)​

- **Client Secret**: Your app's password, authenticates your backend[](https://workos.com/blog/oauth-authorization-code-grant)​

## Why Backend is Required in OAuth

**The security problem**: Exchanging authorization code for tokens requires sending `client_secret`—your app's password.[](https://workos.com/blog/oauth-authorization-code-grant)​

**Frontend can't do this because**:

- Client secret would be visible in JavaScript source code

- Anyone could extract it via browser DevTools

- Attackers could impersonate your application[](https://www.authgear.com/post/common-oauth-2-0-grant-types)

**Backend keeps the secret safe**: Stored in environment variables on secure servers, never exposed to users.[](https://workos.com/blog/oauth-authorization-code-grant)​

## Alternative: PKCE for Frontend-Only Apps

**PKCE** (Proof Key for Code Exchange) allows frontends to exchange authorization codes **without needing client\_secret**:[](https://workos.com/blog/oauth-authorization-code-grant)​

1. Frontend generates random `code_verifier`

2. Creates hash: `code_challenge = SHA256(code_verifier)`

3. Sends `code_challenge` with authorization request

4. OAuth provider stores it and returns authorization code

5. Frontend exchanges code + original `code_verifier` for token

6. OAuth provider verifies hash matches stored `code_challenge`[](https://workos.com/blog/oauth-authorization-code-grant)​

**Result**: Proves same app completed the flow, without needing a secret. **Mandatory in OAuth 2.1** for all clients.[](https://workos.com/blog/oauth-authorization-code-grant)​

## Token Storage Options

|Storage|Pros|Cons|Best For|
|---|---|---|---|
|**localStorage**|Persists across sessions, easy access [](https://secture.com/en/how-to-correctly-store-jwt-tokens-in-the-front-end/)​|Vulnerable to XSS attacks [](https://secture.com/en/how-to-correctly-store-jwt-tokens-in-the-front-end/)|Short-lived access tokens in SPAs|
|**sessionStorage**|Cleared on tab close [](https://secture.com/en/how-to-correctly-store-jwt-tokens-in-the-front-end/)​|Lost on reload, still XSS vulnerable [](https://secture.com/en/how-to-correctly-store-jwt-tokens-in-the-front-end/)​|Temporary data|
|**HttpOnly Cookies**|JavaScript can't access (XSS-proof) [](https://inventivehq.com/blog/what-do-secure-httponly-samesite-cookie-attributes-do)​|Need CSRF protection [](https://inventivehq.com/blog/what-do-secure-httponly-samesite-cookie-attributes-do)​|Most secure, recommended [](https://secture.com/en/how-to-correctly-store-jwt-tokens-in-the-front-end/)​|

## Cookie Security Attributes

- **HttpOnly**: Prevents JavaScript access, blocks XSS token theft[](https://inventivehq.com/blog/what-do-secure-httponly-samesite-cookie-attributes-do)

- **Secure**: Only sent over HTTPS, prevents interception[](https://inventivehq.com/blog/what-do-secure-httponly-samesite-cookie-attributes-do)​

- **SameSite**: Controls cross-site requests[](https://www.shdev.blog/en/post/cookies-httponly-secure-samesite)

  - `Strict`: Never sent with cross-site requests

  - `Lax`: Sent with top-level navigation only

  - `None`: Sent with all requests (requires `Secure` flag)

## Cookie vs localStorage for Tokens

**Cookies** (automatic):

- Backend sends: `Set-Cookie: token=...; HttpOnly; Secure; SameSite=Strict`

- Browser **automatically** includes cookie with every request to same domain[](https://dev.to/cotter/localstorage-vs-cookies-all-you-need-to-know-about-storing-jwt-tokens-securely-in-the-front-end-15id)

- **No manual code needed**[](https://dev.to/cotter/localstorage-vs-cookies-all-you-need-to-know-about-storing-jwt-tokens-securely-in-the-front-end-15id)​

**localStorage** (manual):

- Backend sends token in response body

- Frontend must **manually** retrieve and attach to each request:

  javascript

  `const token = localStorage.getItem('token'); fetch(url, { headers: { 'Authorization': 'Bearer ' + token } });`

- Requires interceptor to avoid repeating code[](https://www.pivotpointsecurity.com/local-storage-versus-cookies-which-to-use-to-securely-store-session-tokens/)

## Client Credentials Flow (Machine-to-Machine)

For **backend-to-backend** communication where no user is involved:[](https://www.infisign.ai/blog/oauth-client-credentials-flow)

javascript

`POST /oauth/token {   "grant_type": "client_credentials",  "client_id": "appUsername",      // app's identifier  "client_secret": "appPassword",  // app's password  "scope": "read write" }`

**Use cases**: Microservices, cron jobs, server daemons[](https://www.infisign.ai/blog/oauth-client-credentials-flow)​\
**Never for frontends**: Client secret must never be in public clients[](https://blog.sentry.security/oauth-2-0-client-credentials-misuse-in-public-apps/)​

## Why Client Secret Can't Be in Frontend

**The catastrophe**:

- JavaScript source code is **public**—anyone can extract secrets via DevTools[](https://blog.sentry.security/oauth-2-0-client-credentials-misuse-in-public-apps/)​

- Mobile apps can be **decompiled** to extract embedded secrets[](https://blog.sentry.security/oauth-2-0-client-credentials-misuse-in-public-apps/)​

- Attackers use stolen secrets to **impersonate your app**[](https://blog.sentry.security/oauth-2-0-client-credentials-misuse-in-public-apps/)​

- **No way to revoke** without breaking all existing deployments[](https://blog.sentry.security/oauth-2-0-client-credentials-misuse-in-public-apps/)​

**Real attack**: Desktop app embedded client\_id/secret in config files → attackers extracted credentials → gained admin access to enumerate users and modify tenant configuration[](https://blog.sentry.security/oauth-2-0-client-credentials-misuse-in-public-apps/)​

## Backend Security: Defense in Depth

**Backend isn't unhackable, but offers layered defenses**:[](https://www.scottbrady.io/oauth/client-authentication)​

- **Secret rotation**: Regular credential changes limit breach window[](https://www.scottbrady.io/oauth/client-authentication)​

- **Monitoring**: Detect unusual API patterns

- **Rate limiting**: Prevent abuse at scale

- **Better auth methods**: Private Key JWT, mTLS instead of shared secrets[](https://www.scottbrady.io/oauth/client-authentication)​

- **Infrastructure security**: Firewalls, encryption, least privilege access

- **Rapid response**: Revoke secrets, regenerate credentials if compromised

**Trade-off**: Backend = **possible** compromise with many defenses vs. Frontend = **guaranteed** exposure with zero defenses[](https://www.scottbrady.io/oauth/client-authentication)

## OAuth Without Backend (Naive Approach)

**What would happen**: User enters Google password directly into third-party app[](https://www.linkedin.com/posts/inc-abhishek_oauth-backendengineering-systemdesign-activity-7417510906462416896-uili)​

**Why it's catastrophic**:

- Third-party has **full account access** (not just photos, but emails, Drive, everything)[](https://www.linkedin.com/posts/inc-abhishek_oauth-backendengineering-systemdesign-activity-7417510906462416896-uili)​

- **No revocation** without changing password everywhere[](https://www.linkedin.com/posts/inc-abhishek_oauth-backendengineering-systemdesign-activity-7417510906462416896-uili)​

- If third-party is hacked, your credentials are stolen[](https://stackoverflow.com/questions/43474267/how-to-store-third-party-credentials-no-api-no-oauth-for-automatic-reuse)​

- **No auditability** of what apps are doing[](https://www.linkedin.com/posts/inc-abhishek_oauth-backendengineering-systemdesign-activity-7417510906462416896-uili)​

- 2FA breaks the flow[](https://stackoverflow.com/questions/43474267/how-to-store-third-party-credentials-no-api-no-oauth-for-automatic-reuse)​

**This is exactly what OAuth prevents**.[](https://www.linkedin.com/posts/inc-abhishek_oauth-backendengineering-systemdesign-activity-7417510906462416896-uili)​

## Implementing OAuth in React

## Option 1: Use a Library (Recommended)

typescript

`import { GoogleOAuthProvider, GoogleLogin } from '@react-oauth/google'; <GoogleOAuthProvider clientId="YOUR_CLIENT_ID">   <GoogleLogin    onSuccess={(response) => {      // Send to backend    }}  /> </GoogleOAuthProvider>`

## Option 2: Manual Implementation

1. Register app at Google Cloud Console → get `client_id`

2. Redirect to Google: `https://accounts.google.com/o/oauth2/auth?client_id=...&redirect_uri=...&response_type=code`

3. Extract authorization code from callback URL

4. Send code to your backend

5. Backend exchanges code for tokens using client\_secret[](https://www.descope.com/blog/post/oauth2-react-authentication-authorization)

## When to Use What

**Use JWT alone**: First-party apps where you control both frontend and backend[](https://stackoverflow.com/questions/39909419/what-are-the-main-differences-between-jwt-and-oauth-authentication)

**Use OAuth**: Third-party apps need access to user data without passwords[](https://www.linkedin.com/pulse/oauth-vs-sso-which-one-should-i-use-abdul-waheed)

**Use SSO**: Internal apps where one login accesses multiple services[](https://systemdesignschool.io/blog/sso-vs-oauth)

**Use SAML**: Enterprise environments with corporate identity providers[](https://ones.com/blog/demystifying-saml-authentication-flow-diagrams/)

---

## The Core Principle

OAuth separates **who you are** (authentication) from **what you're allowed to do** (authorization). It enables secure, granular, revocable access for third parties without the catastrophic security risks of password sharing.

# Session-Based vs. JWT Authentication: Technical Comparison

1. Storage & State

- **Session-Based (Stateful):**
  - **Server:** Stores a "Memory" (Map/Database) of every logged-in user. It must track who is active.
  - **Client:** Stores only a random **ID** (Reference) in a Cookie.
- **JWT (Stateless):**
  - **Server:** Stores **nothing**. It forgets the user immediately after sending the token.
  - **Client:** Stores the **entire User Data** (Encoded) in the token itself (LocalStorage or Cookie).

2. Verification Process

- **Session:** When a request arrives, the server performs a **Lookup**. It searches its RAM/Database for the ID to find the User Data.
- **JWT:** When a request arrives, the server performs a **Cryptographic Check**. It uses a "Secret Key" to verify the token's digital signature. If the signature is valid, the server trusts the data inside the token without a database lookup.

3. Revocation & Control

- **Session:** **Easy Revocation.** If you want to force-logout a user, you simply delete their Session ID from the server's memory. The next time they present that ID, it will be rejected.
- **JWT:** **Difficult Revocation.** Once a JWT is issued, it is valid until it expires. You cannot "un-log" a user easily unless you build a complex "Blacklist" of blocked tokens, which essentially makes it stateful again.

4. Scalability (Horizontal)

- **Session:** **Hard to Scale.** If you have 5 servers, Server A doesn't know about a session created on Server B. You must use a central "Session Store" (like **Redis**) so all servers share the same memory.
- **JWT:** **Easy to Scale.** Any server can verify the token as long as it has the "Secret Key." No shared database or communication between servers is required.

5. Data Visibility

- **Session:** **Opaque.** The client sees a random string (`ABC123XYZ`). They cannot see their User ID or Roles.
- **JWT:** **Readable.** The token is only _encoded_ (Base64), not _encrypted_. Anyone can paste a JWT into `jwt.io` and read the user's ID, Name, and Roles. **Never put sensitive data like passwords inside a JWT.**
