---
{"publish":true,"created":"2026-01-29T18:21:12.000Z","modified":"2026-08-17T03:04:31.852Z"}
---

###### **Application layer protocol** -

**HTTP / HTTPS**

- Stateless, req response protocol, only client can intitiate
- works over tcp
- HTTP 1.0 Each req opens new TCP connection. Inefficient.
- HTTP 1.1 Allowed multiple req in one connection, still processed the req sequentially. (send multiple requests together  but response will come one by one)
- HTTP2 true multiplexing. Multiple parallel requests in one TCP connection. Breaks data into frames. BUT if one pakcet is lost all streams wait.
- HTTP3 ==Works on UDP not TCP== truly independent parallel req. One packet lost only that stream will wait.

**Websocket** [[WEBSOCKETS | CHECK FULL FLOW]]

- Works on TCP
- Persistent full-duplex bidirectional connection.
- Initial HTTP handshake
- Consists of opening handshake, bidirectional data transfer, and closing handshake

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

**gRPC**

- Remote Procedure Call framework built on HTTP/2
- Used in - Microservices communication, Internal service-to-service APIs

**SSE**

- one way communication from server to client.
- Client must initiate the connection first. Here's the detailed process :
  **1. Client initiates connection**:
  `const eventSource = new EventSource('/events');`
  - Client sends standard HTTP GET request to SSE endpoint
  - This is a regular HTTP request, nothing special yet
    **2. Server responds and keeps connection open**:
  - Server responds with `200 OK` status
  - Sets ==Content-Type: text/event-stream== header
  - **Crucially: Server does NOT close the connection after sending response**
  - Connection remains open indefinitely
    **3. Server pushes events over open connection**:​
    `// Server code res.write('data: Hello from server\n\n');`
  - Server writes data to the open HTTP connection whenever it wants
  - Each event follows format: `data: message\n\n`
  - Client receives events as they arrive
    **4. Automatic reconnection**:​
  - If connection drops, browser automatically reconnects
  - Client handles reconnection without additional code

###### **Transport Layer Protocols**

**TCP**

- Connection-oriented, reliable, ordered delivery with error checking
- Ensures data arrives correctly but adds latency due to acknowledgments

**UDP**

- Connectionless, no reliability guarantees, faster
- Used for video streaming, VoIP, gaming where speed matters more than perfect delivery

###### **Handshakes**

**TCP Three-Way Handshake**
**Step 1 - SYN (Synchronize)**:
\- Client sends a SYN packet with an initial sequence number to the server
\- "I want to connect and will start counting from number X"
**Step 2 - SYN-ACK**:
\- Server responds with SYN-ACK packet containing its own sequence number
\- "Got it! I acknowledge your number X, and I'll start counting from number Y"
**Step 3 - ACK (Acknowledge)**:
\- Client sends ACK packet to confirm
\- "Perfect, I acknowledge your number Y. Connection established!"
After these three steps, both sides can begin transferring data.

**TLS/SSL Handshake**

- Happens after TCP handshake for HTTPS connections
- Negotiates encryption algorithms and exchanges cryptographic keys
- Establishes secure encrypted communication channel
  **WebSocket Handshake**
