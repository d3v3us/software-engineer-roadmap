# Networking - Comprehensive Guide

## Table of Contents
1. [TCP/IP Fundamentals](#tcpip-fundamentals)
2. [HTTP and HTTPS](#http-and-https)
3. [DNS (Domain Name System)](#dns-domain-name-system)
4. [Load Balancing](#load-balancing)
5. [CDN (Content Delivery Network)](#cdn-content-delivery-network)
6. [WebSockets](#websockets)
7. [REST vs GraphQL vs gRPC](#rest-vs-graphql-vs-grpc)
8. [API Design Principles](#api-design-principles)

---

## TCP/IP Fundamentals

### What is TCP?

**Transmission Control Protocol (TCP)** is like a reliable postal service with delivery confirmation. When you send a letter, you get a receipt, and the recipient confirms they received it. If something goes wrong, the postal service tries again.

**Key Characteristics:**
- **Connection-oriented**: Establishes a connection before data transfer (like a phone call)
- **Reliable**: Guarantees data delivery and order
- **Flow control**: Prevents overwhelming the receiver
- **Congestion control**: Adapts to network conditions

### TCP Three-Way Handshake

Imagine you want to have a conversation with someone:

```
Client                          Server
  |                               |
  |---- SYN (seq=x) ------------>|
  |                               |  Server: "I'm ready to talk"
  |                               |
  |<-- SYN-ACK (seq=y, ack=x+1) --|
  |                               |  Server: "Got your message, here's mine"
  |                               |
  |---- ACK (ack=y+1) ----------->|
  |                               |  Client: "Got your message too!"
  |                               |
  |     CONNECTION ESTABLISHED     |
```

**Step-by-step:**
1. **SYN**: Client sends a synchronization packet with a random sequence number (x)
2. **SYN-ACK**: Server acknowledges (x+1) and sends its own sequence number (y)
3. **ACK**: Client acknowledges (y+1), connection is established

**Why three steps?** This ensures both parties know the other is ready and can receive messages. It's like saying "Can you hear me?" → "Yes, can you hear me?" → "Yes!"

### TCP Four-Way Handshake (Connection Termination)

When closing a connection, TCP uses a four-way handshake:

```
Client                          Server
  |                               |
  |---- FIN -------------------->|
  |                               |  Client: "I'm done sending"
  |                               |
  |<-- ACK -----------------------|
  |                               |  Server: "Got it"
  |                               |
  |<-- FIN -----------------------|
  |                               |  Server: "I'm done too"
  |                               |
  |---- ACK -------------------->|
  |                               |  Client: "Got it"
  |                               |
  |     CONNECTION CLOSED          |
```

**Why four steps?** Each side can close independently. One side might finish sending but still need to receive data.

### TCP vs UDP

| Feature | TCP | UDP |
|---------|-----|-----|
| **Reliability** | Guaranteed delivery | Best effort |
| **Ordering** | Maintains order | No ordering |
| **Connection** | Connection-oriented | Connectionless |
| **Speed** | Slower (overhead) | Faster (minimal overhead) |
| **Use Cases** | Web browsing, email, file transfer | Video streaming, gaming, DNS |

**Analogy:**
- **TCP** = Registered mail with tracking (slow but reliable)
- **UDP** = Postcard (fast but might get lost)

### TCP Flow Control

**Problem**: What if the receiver is slower than the sender?

**Solution**: **Sliding Window Protocol**

```
Sender Window: [1][2][3][4][5]
                ↑
              Window size = 5

Receiver Buffer: Can only hold 3 packets
```

The receiver tells the sender: "I can only accept 3 packets at a time" (window size = 3). The sender adjusts its window accordingly.

**Visual Representation:**
```
Time →
Sender:  [1][2][3] | [4][5][6] | [7][8][9]
                    ↑
              Window slides as ACKs arrive
```

### TCP Congestion Control

**Problem**: Network is like a highway. Too many cars = traffic jam.

**Solution**: TCP adapts transmission rate based on network conditions.

**Algorithms:**
1. **Slow Start**: Start small, double window size each RTT (Round Trip Time)
2. **Congestion Avoidance**: Increase window size linearly
3. **Fast Retransmit**: If 3 duplicate ACKs received, retransmit immediately
4. **Fast Recovery**: Reduce window by half, then continue

**Visual Graph:**
```
Window Size
    ↑
    |     /\
    |    /  \___
    |   /       \___
    |  /            \___
    | /                 \___
    |/______________________\___ Time
    Slow Start → Congestion Avoidance
```

---

## HTTP and HTTPS

### HTTP (HyperText Transfer Protocol)

**HTTP** is like a conversation between a client and server using a specific language.

**Request Structure:**
```
GET /api/users HTTP/1.1
Host: example.com
User-Agent: Mozilla/5.0
Accept: application/json

[Body - optional]
```

**Response Structure:**
```
HTTP/1.1 200 OK
Content-Type: application/json
Content-Length: 123

{
  "users": [...]
}
```

### HTTP Methods

| Method | Purpose | Idempotent | Safe |
|--------|---------|------------|------|
| **GET** | Retrieve data | Yes | Yes |
| **POST** | Create resource | No | No |
| **PUT** | Update/replace | Yes | No |
| **PATCH** | Partial update | No | No |
| **DELETE** | Remove resource | Yes | No |

**Idempotent**: Doing it multiple times has the same effect as doing it once.
- GET /users/1 → Always returns same user
- DELETE /users/1 → After first call, user is gone (idempotent)

**Safe**: Doesn't modify server state.
- GET is safe (just reading)
- POST is not safe (creates something)

### HTTP Status Codes

**1xx - Informational**
- 100 Continue: "Keep sending the request"

**2xx - Success**
- 200 OK: "Everything worked"
- 201 Created: "Resource created successfully"
- 204 No Content: "Success, but no body to return"

**3xx - Redirection**
- 301 Moved Permanently: "This URL moved forever"
- 302 Found: "This URL moved temporarily"
- 304 Not Modified: "Use your cached version"

**4xx - Client Error**
- 400 Bad Request: "Your request is malformed"
- 401 Unauthorized: "You need to authenticate"
- 403 Forbidden: "You're authenticated but not allowed"
- 404 Not Found: "Resource doesn't exist"
- 429 Too Many Requests: "Slow down!"

**5xx - Server Error**
- 500 Internal Server Error: "Something broke on our end"
- 502 Bad Gateway: "Upstream server returned invalid response"
- 503 Service Unavailable: "We're temporarily down"
- 504 Gateway Timeout: "Upstream server didn't respond in time"

### HTTPS (HTTP Secure)

**Problem with HTTP**: Data is sent in plain text. Anyone can read it.

**Solution**: HTTPS encrypts the data using TLS/SSL.

**How HTTPS Works:**

```
1. Client: "I want to connect securely"
   ↓
2. Server: "Here's my certificate (public key)"
   ↓
3. Client: "Let me verify this certificate with CA"
   ↓
4. Client: "Certificate is valid! Here's a symmetric key (encrypted with server's public key)"
   ↓
5. Server: "Got it! Now we can communicate securely with this symmetric key"
   ↓
6. Encrypted communication begins
```

**Why both symmetric and asymmetric encryption?**
- **Asymmetric** (RSA): Used for initial key exchange (secure but slow)
- **Symmetric** (AES): Used for actual data transfer (fast)

**Analogy**: 
- Asymmetric = Sending a locked box with a padlock (only server has key)
- Symmetric = Both have the same key, fast to lock/unlock

### HTTP/2 and HTTP/3

**HTTP/1.1 Problems:**
- One request per connection (head-of-line blocking)
- Multiple connections needed for parallel requests

**HTTP/2 Improvements:**
- **Multiplexing**: Multiple requests on one connection
- **Header compression**: Reduces overhead
- **Server push**: Server can send resources before client asks

**HTTP/3 (QUIC):**
- Uses UDP instead of TCP
- Built-in encryption
- Faster connection establishment
- Better handling of network changes (mobile)

---

## DNS (Domain Name System)

### What is DNS?

**DNS** is like a phone book for the internet. You know the name (domain), DNS gives you the number (IP address).

**Example:**
```
You type: google.com
DNS returns: 142.250.191.14
```

### DNS Resolution Process

```
1. Browser checks local cache
   ↓ (not found)
2. Check OS cache
   ↓ (not found)
3. Check router cache
   ↓ (not found)
4. Query Recursive DNS Server (ISP)
   ↓
5. Recursive server queries Root DNS Server
   Root: "I don't know, but ask .com server"
   ↓
6. Query .com TLD Server
   TLD: "I don't know, but ask google.com's nameserver"
   ↓
7. Query Authoritative Nameserver
   Nameserver: "google.com = 142.250.191.14"
   ↓
8. Return IP to browser
```

**Visual Flow:**
```
Browser → Local Cache → OS Cache → Router → ISP DNS → Root → TLD → Authoritative
         (fast)        (fast)      (fast)   (medium)              (slow)
```

### DNS Record Types

| Type | Purpose | Example |
|------|---------|---------|
| **A** | IPv4 address | example.com → 192.0.2.1 |
| **AAAA** | IPv6 address | example.com → 2001:db8::1 |
| **CNAME** | Alias to another domain | www.example.com → example.com |
| **MX** | Mail server | example.com → mail.example.com |
| **TXT** | Text records (SPF, DKIM) | "v=spf1 include:_spf.google.com" |
| **NS** | Nameserver | example.com → ns1.example.com |

### DNS Caching

**TTL (Time To Live)**: How long a DNS record can be cached.

```
example.com A 192.0.2.1 TTL=3600
```

This means: "Cache this for 3600 seconds (1 hour)"

**Why caching?**
- Reduces DNS queries
- Faster responses
- Less load on DNS servers

---

## Load Balancing

### What is Load Balancing?

**Load balancing** is like a restaurant host who directs customers to available tables to ensure no single waiter is overwhelmed.

**Problem**: One server can't handle all traffic.

**Solution**: Distribute requests across multiple servers.

### Load Balancing Algorithms

**1. Round Robin**
```
Request 1 → Server A
Request 2 → Server B
Request 3 → Server C
Request 4 → Server A (cycle repeats)
```

**Pros**: Simple, fair distribution
**Cons**: Doesn't consider server load

**2. Least Connections**
```
Server A: 10 active connections
Server B: 5 active connections
Server C: 15 active connections

New request → Server B (least connections)
```

**Pros**: Considers current load
**Cons**: Doesn't consider server capacity

**3. Weighted Round Robin**
```
Server A: Weight 3 (handles 3 requests)
Server B: Weight 2 (handles 2 requests)
Server C: Weight 1 (handles 1 request)
```

**Pros**: Accounts for server capacity
**Cons**: Static weights don't adapt

**4. IP Hash**
```
Hash(client IP) → Server
Same IP always goes to same server
```

**Pros**: Session persistence
**Cons**: Uneven if IPs are clustered

### Load Balancer Types

**1. Layer 4 (Transport Layer)**
- Works with TCP/UDP
- Routes based on IP and port
- Fast, but less intelligent

**2. Layer 7 (Application Layer)**
- Works with HTTP/HTTPS
- Can route based on URL, headers, cookies
- More intelligent, but slower

**Example:**
```
Layer 4: All traffic to 192.168.1.10:80 → Server A
Layer 7: /api/* → Server A, /static/* → Server B
```

### Health Checks

Load balancers need to know if servers are healthy:

```
Load Balancer → Health Check → Server
                (every 30s)
                
If server doesn't respond:
  - Mark as unhealthy
  - Stop sending traffic
  - Continue checking
  - When healthy again, resume traffic
```

---

## CDN (Content Delivery Network)

### What is a CDN?

**CDN** is like having warehouses in multiple cities. Instead of shipping from one central warehouse, you ship from the nearest one.

**Problem**: 
- User in Tokyo requests content from server in New York
- High latency, slow loading

**Solution**:
- CDN caches content in multiple locations (edge servers)
- User gets content from nearest edge server

### How CDN Works

```
1. User requests example.com/image.jpg
   ↓
2. DNS returns CDN edge server IP (closest to user)
   ↓
3. User requests from edge server
   ↓
4. Edge server checks cache
   - If cached: Return immediately
   - If not cached: Fetch from origin, cache it, return to user
```

**Visual:**
```
Origin Server (New York)
    ↓
    ├── Edge Server (Tokyo) ← User in Tokyo
    ├── Edge Server (London) ← User in London
    └── Edge Server (Sydney) ← User in Sydney
```

### CDN Benefits

1. **Reduced Latency**: Content served from nearby location
2. **Reduced Bandwidth**: Origin server handles less traffic
3. **Better Availability**: If origin is down, cached content still available
4. **DDoS Protection**: CDN can absorb attacks

### Cache Invalidation

**Problem**: Content on origin server updated, but CDN still has old version.

**Solutions:**
1. **TTL-based**: Cache expires after set time
2. **Manual purge**: Invalidate specific files
3. **Versioning**: Use different URLs for new content (`image-v2.jpg`)

---

## WebSockets

### What are WebSockets?

**WebSockets** enable full-duplex communication. Unlike HTTP (request-response), WebSockets allow both client and server to send messages at any time.

**HTTP Analogy**: Like sending letters (one-way, wait for response)
**WebSocket Analogy**: Like a phone call (both can talk anytime)

### WebSocket Handshake

```
Client Request:
GET /chat HTTP/1.1
Host: example.com
Upgrade: websocket
Connection: Upgrade
Sec-WebSocket-Key: dGhlIHNhbXBsZSBub25jZQ==
Sec-WebSocket-Version: 13

Server Response:
HTTP/1.1 101 Switching Protocols
Upgrade: websocket
Connection: Upgrade
Sec-WebSocket-Accept: s3pPLMBiTxaQ9kYGzzhZRbK+xOo=
```

**101 Switching Protocols**: "OK, let's switch to WebSocket protocol"

### WebSocket vs HTTP Polling

**HTTP Polling:**
```
Client: "Any new messages?" → Server: "No"
Client: "Any new messages?" → Server: "No"
Client: "Any new messages?" → Server: "Yes, here's message"
```

**WebSocket:**
```
Connection established
Server: "Here's a message" (whenever available)
Client: "Here's my message" (whenever needed)
```

**WebSocket Advantages:**
- Lower latency (no polling overhead)
- Less bandwidth (no repeated headers)
- Real-time bidirectional communication

**Use Cases:**
- Chat applications
- Real-time gaming
- Live notifications
- Collaborative editing

---

## REST vs GraphQL vs gRPC

### REST (Representational State Transfer)

**REST** uses standard HTTP methods to interact with resources.

**Principles:**
- Stateless: Each request contains all needed information
- Resource-based: URLs represent resources
- Standard methods: GET, POST, PUT, DELETE

**Example:**
```
GET    /api/users          → Get all users
GET    /api/users/1        → Get user 1
POST   /api/users          → Create user
PUT    /api/users/1        → Update user 1
DELETE /api/users/1        → Delete user 1
```

**Pros:**
- Simple, widely understood
- Caching friendly
- Works with HTTP infrastructure

**Cons:**
- Over-fetching (get entire resource when you need one field)
- Under-fetching (need multiple requests for related data)
- Versioning can be messy

### GraphQL

**GraphQL** lets clients request exactly the data they need.

**Example Query:**
```graphql
query {
  user(id: 1) {
    name
    email
    posts {
      title
      comments {
        text
      }
    }
  }
}
```

**Single request gets:**
- User name and email
- User's posts with titles
- Comments on those posts

**Pros:**
- No over-fetching or under-fetching
- Strong typing
- Single endpoint
- Introspection (query schema)

**Cons:**
- More complex than REST
- Caching is harder
- Potential for expensive queries (N+1 problem)

### gRPC

**gRPC** uses Protocol Buffers for efficient binary communication.

**Example:**
```protobuf
service UserService {
  rpc GetUser(UserRequest) returns (User);
  rpc CreateUser(User) returns (UserResponse);
}
```

**Pros:**
- Very fast (binary protocol)
- Strong typing
- Streaming support (client, server, bidirectional)
- Language agnostic

**Cons:**
- Less human-readable
- Browser support limited (needs proxy)
- Steeper learning curve

### Comparison Table

| Feature | REST | GraphQL | gRPC |
|---------|------|---------|------|
| **Protocol** | HTTP/JSON | HTTP/JSON | HTTP/2/Protobuf |
| **Speed** | Medium | Medium | Fast |
| **Flexibility** | Low | High | Medium |
| **Caching** | Easy | Hard | Hard |
| **Browser Support** | Excellent | Excellent | Limited |
| **Use Case** | General APIs | Flexible queries | Microservices |

---

## API Design Principles

### 1. Use Nouns, Not Verbs

**Bad:**
```
GET /getUsers
POST /createUser
DELETE /deleteUser
```

**Good:**
```
GET /users
POST /users
DELETE /users/:id
```

### 2. Use Plural Nouns

**Good:**
```
GET /users
GET /users/1
```

**Avoid:**
```
GET /user
GET /user/1
```

### 3. Use HTTP Status Codes Correctly

- 200: Success
- 201: Created
- 400: Bad Request
- 401: Unauthorized
- 404: Not Found
- 500: Server Error

### 4. Version Your API

**URL Versioning:**
```
/api/v1/users
/api/v2/users
```

**Header Versioning:**
```
Accept: application/vnd.api+json;version=1
```

### 5. Use Pagination

**Bad:**
```
GET /users  → Returns 10,000 users
```

**Good:**
```
GET /users?page=1&limit=20
Response:
{
  "data": [...],
  "pagination": {
    "page": 1,
    "limit": 20,
    "total": 1000,
    "totalPages": 50
  }
}
```

### 6. Filtering, Sorting, Searching

```
GET /users?status=active&sort=name&search=john
```

### 7. Consistent Error Format

```json
{
  "error": {
    "code": "USER_NOT_FOUND",
    "message": "User with ID 123 does not exist",
    "details": {...}
  }
}
```

### 8. Use HTTPS

Always use HTTPS in production for security.

### 9. Rate Limiting

Prevent abuse:
```
X-RateLimit-Limit: 1000
X-RateLimit-Remaining: 999
X-RateLimit-Reset: 1609459200
```

### 10. Documentation

Provide clear, comprehensive API documentation (OpenAPI/Swagger).

---

## Summary

Networking is the foundation of backend systems. Understanding TCP/IP, HTTP/HTTPS, DNS, load balancing, CDNs, WebSockets, and API design principles is essential for building scalable, reliable, and secure backend systems.

**Key Takeaways:**
- TCP provides reliable, ordered delivery
- HTTP is the language of the web
- HTTPS encrypts communication
- DNS translates domains to IPs
- Load balancers distribute traffic
- CDNs reduce latency
- WebSockets enable real-time communication
- Choose the right API style for your use case

