# Network Protocols

## What are Network Protocols?

Network protocols are sets of rules and conventions that govern how devices communicate over a network. They define the format, timing, sequencing, and error checking of data transmission, ensuring reliable and efficient communication between different systems.

## Key Protocol Layers (OSI Model)

### 1. Application Layer (Layer 7)

- HTTP/HTTPS
- FTP
- SMTP
- DNS
- WebSocket

### 2. Presentation Layer (Layer 6)

- Data formatting
- Encryption/Decryption
- Character encoding
- Data compression

### 3. Session Layer (Layer 5)

- Session management
- Authentication
- Authorization
- Session recovery

### 4. Transport Layer (Layer 4)

- TCP (Transmission Control Protocol)
- UDP (User Datagram Protocol)
- Port management
- Flow control

### 5. Network Layer (Layer 3)

- IP (Internet Protocol)
- ICMP
- Routing
- Packet forwarding

### 6. Data Link Layer (Layer 2)

- MAC addressing
- Error detection
- Frame synchronization
- Flow control

### 7. Physical Layer (Layer 1)

- Physical transmission
- Bit-level communication
- Hardware specifications
- Signal encoding

## Common Protocols

### 1. HTTP/HTTPS

- Stateless protocol
- Request-Response model
- Methods: GET, POST, PUT, DELETE
- Status codes
- Headers and cookies

#### Example HTTP Request

```
GET /api/users HTTP/1.1
Host: api.example.com
Accept: application/json
Authorization: Bearer token123
```

#### Example HTTP Response

```
HTTP/1.1 200 OK
Content-Type: application/json
Cache-Control: max-age=3600

{
    "users": [...]
}
```

### 2. TCP/IP

- Connection-oriented
- Reliable delivery
- Flow control
- Congestion control

#### TCP Three-Way Handshake

```
[Client] ----SYN----> [Server]
[Client] <--SYN-ACK-- [Server]
[Client] ----ACK----> [Server]
```

### 3. WebSocket

- Full-duplex communication
- Persistent connection
- Real-time data transfer
- Lower latency than HTTP

#### WebSocket Connection

```
[Client] ----HTTP Upgrade Request----> [Server]
[Client] <--HTTP 101 Switching Protocols-- [Server]
[Client] <===== WebSocket Connection =====> [Server]
```

## Protocol Characteristics

### 1. Connection-Oriented vs Connectionless

- TCP: Connection-oriented, reliable
- UDP: Connectionless, best-effort

### 2. Stateful vs Stateless

- Stateful: Maintains session information
- Stateless: Each request is independent

### 3. Synchronous vs Asynchronous

- Synchronous: Wait for response
- Asynchronous: Non-blocking communication

## Security Considerations

### 1. Encryption

- TLS/SSL
- End-to-end encryption
- Certificate management

### 2. Authentication

- Basic Auth
- OAuth
- JWT
- API keys

### 3. Authorization

- Role-based access control
- Permission management
- Token validation

## Performance Optimization

### 1. Connection Pooling

- Reuse connections
- Reduce overhead
- Manage connection limits

### 2. Compression

- Gzip
- Brotli
- Deflate

### 3. Caching

- HTTP caching
- CDN caching
- Browser caching

## Common Issues and Solutions

### 1. Network Latency

- Use CDNs
- Implement caching
- Optimize payload size

### 2. Connection Issues

- Implement retry mechanisms
- Use circuit breakers
- Handle timeouts properly

### 3. Security Vulnerabilities

- Regular security audits
- Keep protocols updated
- Implement proper authentication

## Interview Questions

1. What are the differences between TCP and UDP?
2. How does HTTPS ensure secure communication?
3. What is the purpose of the three-way handshake in TCP?
4. How do you handle protocol versioning in APIs?
5. What are the trade-offs between different transport protocols?

## Further Reading

- [HTTP/1.1 Specification](https://tools.ietf.org/html/rfc7231)
- [TCP/IP Protocol Suite](https://tools.ietf.org/html/rfc793)
- [WebSocket Protocol](https://tools.ietf.org/html/rfc6455)
- [TLS 1.3 Specification](https://tools.ietf.org/html/rfc8446)
