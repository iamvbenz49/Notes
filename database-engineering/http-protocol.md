# HTTP Protocol & Custom Protocols — Video Notes

---

## What is a Protocol?

- A protocol is a **common language** that two machines agree upon to communicate over a network.
- Without a protocol, machines cannot understand what the other wants — similar to how humans need a shared language like English to converse.
- Example: A simple protocol could define that a command is separated by spaces, followed by arguments, and terminated by a newline (`\n`).
- **Redis Serialization Protocol (RESP)** is a real-world example of a simple, space-separated protocol.

---

## The HTTP Protocol

HTTP is a protocol where the **client initiates a TCP connection**, sends a request, and the server responds.

### Why HTTP is a Protocol
- It defines a **strict, standardized format** for requests that both client and server must follow.
- If rules are violated (e.g., missing a required `Host` header), the server responds with a standardized error like `400 Bad Request`.
- It uses specific separators — **spaces** and `\r\n` (carriage return + line feed) — to delimit lines and sections.

### Request Structure

```
METHOD /url HTTP/version\r\n
Header-Key: Header-Value\r\n
\r\n
[optional body]
```

| Part         | Example             |
|--------------|---------------------|
| Method       | `GET`, `POST`, `PUT`, `DELETE` |
| URL          | `/foo`              |
| HTTP Version | `HTTP/1.1`          |
| Headers      | `Host: localhost`   |
| Body         | POST/PUT requests only |

---

## Practical Demo: Raw TCP Connections

Instead of using `curl`, the speaker interacts with a Go web server directly over raw TCP using **Netcat (`nc`)**.

### Connect to the server
```bash
nc localhost 1729
```

### HTTP GET Request
```
GET /foo HTTP/1.1
```

### GET Request with Host header (fixes 400 error)
```
GET /foo HTTP/1.1
Host: localhost

```
> ⚠️ Two newlines are required after the last header to signal the end of the request.

### HTTP POST Request
```
POST /login HTTP/1.1
Host: localhost
Content-Length: 28

user=arpit&password=pass
```
> The `Content-Length` header tells the server exactly how many bytes to read for the body. The body must match the specified byte count.

### Common Errors
- **`400 Bad Request`** — Caused by not following the protocol strictly, e.g., omitting the required `Host` header.

---

## Creating Custom Protocols

- You can define **your own protocol specification** as long as both the client and server understand it.
- You must write:
  - **Encoder** (client side) — formats the message.
  - **Decoder** (server side) — parses and interprets the message.
- Databases like **MySQL** and **Redis** use custom protocols on top of TCP for more efficient communication, rather than HTTP.
- **HTTP/2** and **HTTP/3** are improved specifications built on the same core request-response principles.

### Why use custom protocols over HTTP?
- HTTP carries overhead (verbose headers, text-based format) that isn't always needed.
- Custom protocols can be **binary, compact, and optimized** for specific use cases (e.g., database queries).
- They allow full control over how data is encoded and decoded for maximum efficiency.

---

## How TCP and HTTP Collaborate

- **TCP** is the transport layer — it provides a reliable, ordered byte stream connection between two machines.
- **HTTP** sits on top of TCP — it defines *what* to send over that TCP connection (the request/response format).
- The browser/client opens a TCP connection, then communicates using the HTTP protocol over it.

---

## What Does a Browser Do During Requests?

1. Resolves the domain via DNS.
2. Opens a TCP connection to the server.
3. Formats and sends an HTTP request (method, URL, headers, optional body).
4. Waits for the HTTP response from the server.
5. Parses the response and renders the content.

---

## How is a Redis Command Structured?

Redis uses the **RESP (Redis Serialization Protocol)** — a simple, space-separated protocol over TCP. Commands are sent as plain text strings separated by spaces and terminated by `\r\n`, e.g.:

```
SET key value\r\n
GET key\r\n
```
