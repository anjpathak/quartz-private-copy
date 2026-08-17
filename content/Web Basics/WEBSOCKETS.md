---
{"publish":true,"created":"2026-01-29T18:22:49.000Z","modified":"2026-08-17T03:04:33.499Z"}
---

## Phase 1: Opening Handshake (HTTP Upgrade)

## Step 1: Client Initiates Connection[](https://websocket.org/guides/websocket-protocol/)

The client sends an HTTP GET request with special upgrade headers:

text

```
GET /chat HTTP/1.1
Host: server.example.com
Upgrade: websocket
Connection: Upgrade
Sec-WebSocket-Key: dGhlIHNhbXBsZSBub25jZQ==
Sec-WebSocket-Version: 13
Origin: http://example.com

```

**Key headers**:[](https://app.studyraid.com/en/read/11426/357869/websocket-handshake-process)

- `Upgrade: websocket` - tells server client wants to switch protocols

- `Connection: Upgrade` - instructs intermediaries to upgrade connection

- `Sec-WebSocket-Key` - random base64-encoded 16-byte value for security verification[](https://www.nilebits.com/blog/2023/07/websocket-handshaking-explained-understanding-the-key-to-real-time-communication/)

- `Sec-WebSocket-Version: 13` - current WebSocket protocol version

- `Sec-WebSocket-Protocol` (optional) - application-level subprotocols like "chat" or "json"[](https://app.studyraid.com/en/read/11426/357869/websocket-handshake-process)​

## Step 2: Server Validates and Responds[](https://developer.mozilla.org/en-US/docs/Web/API/WebSockets_API/Writing_WebSocket_servers)

If the server accepts, it responds with **HTTP 101 Switching Protocols**:

text

```
HTTP/1.1 101 Switching Protocols
Upgrade: websocket
Connection: Upgrade
Sec-WebSocket-Accept: s3pPLMBiTxaQ9kYGzzhZRbK+xOo=


```

**Security verification**:[](https://sharooque.hashnode.dev/understanding-websocket-handshake-seamless-two-way-communication-for-web-applications)

- Server concatenates client's `Sec-WebSocket-Key` + predefined GUID (`258EAFA5-E914-47DA-95CA-C5AB0DC85B11`)

- Computes SHA-1 hash of the concatenated string

- Base64 encodes the hash to create `Sec-WebSocket-Accept` value

- Client verifies this to confirm server actually supports WebSocket (not just any HTTP server)[](https://sharooque.hashnode.dev/understanding-websocket-handshake-seamless-two-way-communication-for-web-applications)​

## Step 3: Connection Established[](https://en.wikipedia.org/wiki/WebSocket)

Once both handshakes complete:

- HTTP protocol **stops being used** entirely

- Connection **switches to binary frame-based protocol**[](https://en.wikipedia.org/wiki/WebSocket)​

- The TCP connection remains open for bidirectional communication

## Phase 2: Data Transfer (Bidirectional Messaging)

## Bidirectional Communication[](https://datatracker.ietf.org/doc/html/rfc6455)

Both client and server can now send messages **independently and simultaneously**:

javascript

```
// Client side
const ws = new WebSocket('ws://example.com/chat');

// Send message to server
ws.send('Hello Server!');

// Receive messages from server
ws.onmessage = (event) => {
  console.log('Received:', event.data);
};

```

**Frame-based protocol**:[](https://en.wikipedia.org/wiki/WebSocket)​

- Data is broken into frames (small chunks)

- Supports different message types: text, binary, ping, pong, close

- Each side can send data **at will without waiting** for responses

## Phase 3: Closing Handshake

## Graceful Connection Closure[](https://datatracker.ietf.org/doc/html/rfc6455)​

Either peer can initiate closing:

**Step 1**: One peer sends a Close frame with optional status code\
**Step 2**: Other peer responds with its own Close frame\
**Step 3**: First peer closes the TCP connection[](https://datatracker.ietf.org/doc/html/rfc6455)​

This ensures both sides know the connection is closing and no data is lost.

|Aspect|HTTP|WebSocket|
|---|---|---|
|Connection|New for each request|Persistent after handshake [datatracker.ietf](https://datatracker.ietf.org/doc/html/rfc6455)​|
|Protocol|Request-response only|Full-duplex bidirectional [datatracker.ietf](https://datatracker.ietf.org/doc/html/rfc6455)​|
|Overhead|Headers sent every time|Headers only during handshake [ably](https://ably.com/topic/websockets)​|
|Server push|Not possible|Server can push anytime [datatracker.ietf](https://datatracker.ietf.org/doc/html/rfc6455)​|
|Frame type|Text-based|Binary frames [wikipedia](https://en.wikipedia.org/wiki/WebSocket)​|
