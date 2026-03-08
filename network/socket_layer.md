# Networking Notes: L3/L4/L7, Socket, HTTP, WebSocket

## 1) Quick Layer Mapping
- `L7 (Application)`: HTTP, WebSocket, gRPC, DNS.
- `L4 (Transport)`: TCP/UDP, ports.
- `L3 (Network)`: IP routing.

Simple memory line:
- App request at `L7` -> transport connection at `L4` -> packet routing at `L3`.

## 2) What Happens When Browser Opens a URL
Example: `https://example.com/users`

1. Browser prepares HTTP request (`L7`).
2. DNS resolves `example.com` to an IP (`L7` protocol).
3. TCP connection is created to server `IP:443` (`L4`).
4. TLS handshake happens for HTTPS (presentation/security layer concept).
5. HTTP request/response flows on that TCP connection (`L7 over L4`).

## 3) Socket Meaning
A socket is an OS endpoint for network communication.

- Client opens socket to `serverIP:port`.
- For TCP:
  - `socket()`
  - `connect(serverIP, port)`
  - `send/recv`
  - `close`

So yes, client `L4` (TCP) connects to server `L4` endpoint (`IP + port`).

## 4) L4 vs L7 Calls
- `L4-style`: raw TCP socket to `host:port` (e.g., Redis `:6379`, Postgres `:5432`).
- `L7-style`: protocol-aware request (HTTP URL/path, headers, method, body).

## 5) WebSocket Clarification
- WebSocket is an `L7` protocol.
- It runs over TCP (`L4`).
- Flow:
  1. TCP connection established.
  2. HTTP Upgrade request sent.
  3. Connection upgraded to WebSocket frames on same TCP connection.

`ws://` = WebSocket over TCP  
`wss://` = WebSocket over TLS over TCP

## 6) Interview One-Liners
- "`L7` defines request semantics (URL, method, headers), `L4` provides reliable transport via TCP ports, and `L3` routes packets using IP."
- "WebSocket is an application protocol established over TCP, typically via HTTP Upgrade."
