# HTTP/HTTPS Deep Dive - Complete Understanding

## Table of Contents
1. [HTTP Fundamentals - How Web Communication Works](#http-fundamentals---how-web-communication-works)
2. [HTTP Request-Response Cycle - Deep Dive](#http-request-response-cycle---deep-dive)
3. [HTTP Methods - When and Why to Use Each](#http-methods---when-and-why-to-use-each)
4. [HTTP Status Codes - Complete Reference](#http-status-codes---complete-reference)
5. [HTTP Headers - The Metadata of Web Communication](#http-headers---the-metadata-of-web-communication)
6. [HTTPS - Securing HTTP with TLS](#https---securing-http-with-tls)
7. [HTTP/2 and HTTP/3 - Evolution of the Protocol](#http2-and-http3---evolution-of-the-protocol)
8. [Caching - Making the Web Fast](#caching---making-the-web-fast)
9. [Cookies and Sessions - State Management](#cookies-and-sessions---state-management)
10. [RESTful API Design - Best Practices](#restful-api-design---best-practices)

---

## HTTP Fundamentals - How Web Communication Works

### What is HTTP?

**HTTP (HyperText Transfer Protocol)** is the language of the web. It's how your browser talks to web servers.

**Simple Analogy:**
- **HTTP** = The language you use to order food at a restaurant
- **Request** = "I'd like a burger and fries"
- **Response** = "Here's your burger and fries" or "Sorry, we're out of burgers"

**Key Characteristics:**
- **Stateless**: Each request is independent (server doesn't remember previous requests)
- **Request-Response**: Client asks, server answers
- **Text-based**: Human-readable format
- **Application Layer**: Runs on top of TCP

### The HTTP Model

**Client-Server Architecture:**
```
Client (Browser)          Server (Web Server)
    |                           |
    |---- HTTP Request -------->|
    |                           |
    |<--- HTTP Response --------|
    |                           |
```

**How it Works:**
1. Client opens TCP connection to server
2. Client sends HTTP request
3. Server processes request
4. Server sends HTTP response
5. Connection closes (HTTP/1.0) or stays open (HTTP/1.1+)

### HTTP vs Other Protocols

**HTTP vs TCP:**
- **TCP**: Reliable transport (ensures data arrives)
- **HTTP**: Application protocol (defines what data means)

**Analogy:**
- **TCP** = Postal service (delivers mail reliably)
- **HTTP** = Letter format (defines how to write the letter)

**HTTP vs FTP:**
- **HTTP**: Web pages, APIs, stateless
- **FTP**: File transfer, stateful, different protocol

**HTTP vs WebSocket:**
- **HTTP**: Request-response, one-way
- **WebSocket**: Full-duplex, bidirectional, persistent connection

---

## HTTP Request-Response Cycle - Deep Dive

### Complete Request Structure

**HTTP Request Format:**
```
Request Line
Headers (optional)
Blank Line
Body (optional)
```

**Example Request:**
```
GET /api/users HTTP/1.1
Host: example.com
User-Agent: Mozilla/5.0
Accept: application/json
Authorization: Bearer token123

[Body - empty for GET]
```

**Breaking it Down:**

**1. Request Line:**
```
GET /api/users HTTP/1.1
│   │           │
│   │           └─ Protocol version
│   └───────────── Path (resource)
└───────────────── Method (what to do)
```

**2. Headers:**
```
Host: example.com
│    │
│    └─ Value
└────── Key
```

**3. Body:**
- Optional
- Used for POST, PUT, PATCH
- Contains data to send

### Complete Response Structure

**HTTP Response Format:**
```
Status Line
Headers
Blank Line
Body
```

**Example Response:**
```
HTTP/1.1 200 OK
Content-Type: application/json
Content-Length: 123
Date: Mon, 01 Jan 2024 12:00:00 GMT

{
  "users": [...]
}
```

**Breaking it Down:**

**1. Status Line:**
```
HTTP/1.1 200 OK
│        │   │
│        │   └─ Status text
│        └───── Status code
└────────────── Protocol version
```

**2. Headers:**
- Metadata about response
- Content type, length, caching, etc.

**3. Body:**
- Actual data
- HTML, JSON, images, etc.

### Request-Response Flow

**Step-by-Step:**

**1. DNS Lookup:**
```
Browser: "What's the IP of example.com?"
DNS: "142.250.191.14"
```

**2. TCP Connection:**
```
Browser → Server: TCP handshake
Connection established
```

**3. HTTP Request:**
```
Browser → Server: GET /api/users HTTP/1.1
                    Host: example.com
                    ...
```

**4. Server Processing:**
```
Server receives request
Processes (queries database, etc.)
Prepares response
```

**5. HTTP Response:**
```
Server → Browser: HTTP/1.1 200 OK
                    Content-Type: application/json
                    ...
                    {data}
```

**6. Browser Processing:**
```
Browser receives response
Parses headers
Renders body (if HTML)
Executes JavaScript (if any)
```

**7. Connection:**
```
HTTP/1.0: Connection closes
HTTP/1.1: Connection may stay open (keep-alive)
```

### Visual Timeline

```
Time →
Browser                    Server
  |                         |
  |-- DNS Lookup ---------->|
  |<-- IP Address ----------|
  |                         |
  |-- TCP Connect --------->|
  |<-- Connected -----------|
  |                         |
  |-- HTTP Request -------->|
  |                         | Process
  |                         | Query DB
  |                         | Generate response
  |<-- HTTP Response -------|
  |                         |
  | Parse & Render          |
  |                         |
  |-- Close (or keep) ------>|
```

---

## HTTP Methods - When and Why to Use Each

### GET - Retrieve Data

**Purpose:** Fetch data from server

**Characteristics:**
- **Idempotent**: Same request = same result
- **Safe**: Doesn't modify server state
- **Cacheable**: Can be cached
- **No body**: Shouldn't have request body

**Example:**
```
GET /api/users/123 HTTP/1.1
Host: example.com

Response: User data
```

**When to Use:**
- Fetching resources
- Reading data
- Search queries (in URL)

**When NOT to Use:**
- Sending sensitive data (use POST)
- Modifying data (use PUT/PATCH/DELETE)

### POST - Create Resource

**Purpose:** Create new resource or submit data

**Characteristics:**
- **Not idempotent**: Same request may create multiple resources
- **Not safe**: Modifies server state
- **Not cacheable**: Usually not cached
- **Has body**: Data in request body

**Example:**
```
POST /api/users HTTP/1.1
Host: example.com
Content-Type: application/json

{
  "name": "Alice",
  "email": "alice@example.com"
}

Response: 201 Created
Location: /api/users/456
```

**When to Use:**
- Creating resources
- Submitting forms
- Non-idempotent operations

**Idempotency Example:**
```
POST /api/orders
First time: Creates order #1
Second time: Creates order #2 (different!)
→ Not idempotent
```

### PUT - Replace Resource

**Purpose:** Replace entire resource

**Characteristics:**
- **Idempotent**: Same request = same result
- **Not safe**: Modifies server state
- **Has body**: Complete resource in body

**Example:**
```
PUT /api/users/123 HTTP/1.1
Host: example.com
Content-Type: application/json

{
  "name": "Alice Updated",
  "email": "alice.new@example.com",
  "age": 30
}

Response: 200 OK (or 204 No Content)
```

**Important:**
- **Replaces entire resource**
- Must send all fields
- Missing fields = removed

**PUT vs PATCH:**
```
PUT: Replace entire resource
PATCH: Update part of resource
```

### PATCH - Partial Update

**Purpose:** Update part of resource

**Characteristics:**
- **Idempotent**: Same request = same result
- **Not safe**: Modifies server state
- **Has body**: Only changed fields

**Example:**
```
PATCH /api/users/123 HTTP/1.1
Host: example.com
Content-Type: application/json

{
  "email": "alice.new@example.com"
}

Response: 200 OK
```

**Important:**
- **Only updates specified fields**
- Other fields unchanged
- More flexible than PUT

### DELETE - Remove Resource

**Purpose:** Delete resource

**Characteristics:**
- **Idempotent**: Deleting twice = same result (already deleted)
- **Not safe**: Modifies server state
- **No body**: Usually no body

**Example:**
```
DELETE /api/users/123 HTTP/1.1
Host: example.com

Response: 204 No Content (or 200 OK)
```

**Idempotency:**
```
First DELETE: Removes resource → 204
Second DELETE: Already deleted → 204 (same result)
→ Idempotent
```

### Method Comparison Table

| Method | Idempotent | Safe | Body | Use Case |
|--------|------------|------|------|----------|
| **GET** | Yes | Yes | No | Retrieve data |
| **POST** | No | No | Yes | Create resource |
| **PUT** | Yes | No | Yes | Replace resource |
| **PATCH** | Yes | No | Yes | Update resource |
| **DELETE** | Yes | No | No | Delete resource |

### Safe vs Idempotent

**Safe:**
- Doesn't modify server state
- Can be called multiple times without side effects
- GET, HEAD, OPTIONS

**Idempotent:**
- Same request multiple times = same result
- May modify state, but result is same
- GET, PUT, PATCH, DELETE

**Example:**
```
GET /users/1 (Safe & Idempotent)
  Call 1: Returns user
  Call 2: Returns user (same)
  Call 3: Returns user (same)
  No side effects

DELETE /users/1 (Not Safe, but Idempotent)
  Call 1: Deletes user → 204
  Call 2: Already deleted → 204 (same result)
  Call 3: Already deleted → 204 (same result)
  Has side effects, but result is same
```

---

## HTTP Status Codes - Complete Reference

### Status Code Categories

**1xx - Informational:**
- Request received, continuing process
- Rarely used

**2xx - Success:**
- Request successful
- Most common: 200, 201, 204

**3xx - Redirection:**
- Further action needed
- Most common: 301, 302, 304

**4xx - Client Error:**
- Client made mistake
- Most common: 400, 401, 403, 404

**5xx - Server Error:**
- Server error
- Most common: 500, 502, 503, 504

### Detailed Status Codes

#### 1xx - Informational

**100 Continue:**
```
Client: "I'm about to send large body, can you handle it?"
Server: "100 Continue - Yes, send it"
Client: Sends body
```

**Use Case:**
- Large file uploads
- Client checks before sending

**101 Switching Protocols:**
```
Client: "Can we switch to WebSocket?"
Server: "101 Switching Protocols - OK"
→ Protocol changes
```

**Use Case:**
- Protocol upgrade (HTTP → WebSocket)

#### 2xx - Success

**200 OK:**
```
Request successful
Most common success response
```

**Example:**
```
GET /api/users/1
Response: 200 OK
Body: {user data}
```

**201 Created:**
```
Resource created successfully
Should include Location header
```

**Example:**
```
POST /api/users
Response: 201 Created
Location: /api/users/123
Body: {created user}
```

**204 No Content:**
```
Success, but no body to return
Common for DELETE, PUT
```

**Example:**
```
DELETE /api/users/123
Response: 204 No Content
(No body)
```

#### 3xx - Redirection

**301 Moved Permanently:**
```
Resource moved permanently
Browser caches redirect
```

**Example:**
```
GET /old-page
Response: 301 Moved Permanently
Location: /new-page

Browser: Automatically requests /new-page
Browser: Caches redirect (always goes to /new-page)
```

**302 Found (Temporary Redirect):**
```
Resource moved temporarily
Browser doesn't cache
```

**Example:**
```
GET /page
Response: 302 Found
Location: /other-page

Browser: Requests /other-page
Browser: Next time, still tries /page first
```

**304 Not Modified:**
```
Resource not modified since last request
Use cached version
```

**Example:**
```
GET /page
If-Modified-Since: Mon, 01 Jan 2024 12:00:00 GMT

Response: 304 Not Modified
(No body - use cached version)
```

#### 4xx - Client Error

**400 Bad Request:**
```
Request is malformed
Client's fault
```

**Example:**
```
POST /api/users
Body: {invalid json}

Response: 400 Bad Request
Body: {"error": "Invalid JSON"}
```

**401 Unauthorized:**
```
Authentication required
Not authenticated
```

**Example:**
```
GET /api/protected
(No authentication)

Response: 401 Unauthorized
WWW-Authenticate: Bearer
```

**403 Forbidden:**
```
Authenticated but not authorized
Don't have permission
```

**Example:**
```
GET /api/admin/users
(Authenticated as regular user)

Response: 403 Forbidden
Body: {"error": "Admin access required"}
```

**404 Not Found:**
```
Resource doesn't exist
Most common error
```

**Example:**
```
GET /api/users/999
(User doesn't exist)

Response: 404 Not Found
```

**409 Conflict:**
```
Request conflicts with current state
```

**Example:**
```
PUT /api/users/1
Body: {email: "existing@example.com"}
(Email already taken)

Response: 409 Conflict
Body: {"error": "Email already in use"}
```

**429 Too Many Requests:**
```
Rate limit exceeded
Too many requests
```

**Example:**
```
GET /api/data
(100 requests in last minute, limit is 60)

Response: 429 Too Many Requests
Retry-After: 60
```

#### 5xx - Server Error

**500 Internal Server Error:**
```
Generic server error
Something went wrong
```

**Example:**
```
GET /api/users
(Server code crashed)

Response: 500 Internal Server Error
Body: {"error": "Internal server error"}
```

**502 Bad Gateway:**
```
Gateway/proxy received invalid response
Upstream server error
```

**Example:**
```
Client → Load Balancer → Backend Server
Backend returns invalid response
Load Balancer → Client: 502 Bad Gateway
```

**503 Service Unavailable:**
```
Service temporarily unavailable
Overloaded or down for maintenance
```

**Example:**
```
GET /api/users
(Server overloaded)

Response: 503 Service Unavailable
Retry-After: 60
```

**504 Gateway Timeout:**
```
Gateway didn't receive response in time
Upstream server too slow
```

**Example:**
```
Client → Load Balancer → Backend Server
Backend takes 60 seconds (timeout is 30)
Load Balancer → Client: 504 Gateway Timeout
```

### Status Code Decision Tree

```
Request received
    │
    ├─ Success?
    │   ├─ Yes → 200 OK (or 201, 204)
    │   └─ No
    │       │
    │       ├─ Client error?
    │       │   ├─ Not authenticated → 401
    │       │   ├─ Not authorized → 403
    │       │   ├─ Not found → 404
    │       │   ├─ Bad request → 400
    │       │   └─ Other → 4xx
    │       │
    │       └─ Server error?
    │           ├─ Internal error → 500
    │           ├─ Service down → 503
    │           ├─ Gateway error → 502/504
    │           └─ Other → 5xx
```

---

## HTTP Headers - The Metadata of Web Communication

### What are Headers?

**Headers** are metadata about the request or response. They provide additional information beyond the basic request/response.

**Analogy:**
- **Request/Response Body** = The letter content
- **Headers** = The envelope (sender, recipient, postage, etc.)

### Request Headers

#### Common Request Headers

**Host:**
```
Host: example.com
```
- **Required in HTTP/1.1**
- Specifies server domain
- Allows multiple domains on one IP

**User-Agent:**
```
User-Agent: Mozilla/5.0 (Windows NT 10.0; Win64; x64)
```
- Identifies client (browser, app)
- Server can customize response
- Can be spoofed (not reliable for security)

**Accept:**
```
Accept: application/json, text/html
```
- What content types client accepts
- Server chooses appropriate type
- Quality values: `application/json;q=0.9, text/html;q=0.8`

**Accept-Language:**
```
Accept-Language: en-US, en;q=0.9, fr;q=0.8
```
- Preferred languages
- Server can return localized content

**Accept-Encoding:**
```
Accept-Encoding: gzip, deflate, br
```
- Compression methods client supports
- Server can compress response

**Authorization:**
```
Authorization: Bearer eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9...
```
- Authentication credentials
- Common: Bearer token, Basic auth

**Content-Type:**
```
Content-Type: application/json
```
- Type of data in body
- Required for POST/PUT/PATCH with body

**Content-Length:**
```
Content-Length: 1234
```
- Size of body in bytes
- Helps server allocate buffer

**Cookie:**
```
Cookie: session=abc123; theme=dark
```
- Cookies to send
- Stored by browser, sent automatically

**If-Modified-Since:**
```
If-Modified-Since: Mon, 01 Jan 2024 12:00:00 GMT
```
- Conditional request
- "Only return if modified since this date"
- Server responds 304 if not modified

**If-None-Match:**
```
If-None-Match: "abc123"
```
- Conditional request with ETag
- "Only return if ETag doesn't match"
- Server responds 304 if matches

### Response Headers

#### Common Response Headers

**Content-Type:**
```
Content-Type: application/json; charset=utf-8
```
- Type of data in body
- Browser uses to parse response

**Content-Length:**
```
Content-Length: 1234
```
- Size of body in bytes
- Client knows when response is complete

**Cache-Control:**
```
Cache-Control: max-age=3600, public
```
- Caching instructions
- `max-age`: How long to cache
- `public`: Can be cached by CDN
- `private`: Only browser can cache
- `no-cache`: Must revalidate
- `no-store`: Don't cache

**ETag:**
```
ETag: "abc123"
```
- Entity tag (version identifier)
- Used for conditional requests
- Client sends If-None-Match with this value

**Last-Modified:**
```
Last-Modified: Mon, 01 Jan 2024 12:00:00 GMT
```
- When resource was last modified
- Used for conditional requests

**Location:**
```
Location: /api/users/123
```
- Where to find resource
- Used for redirects (3xx) and created resources (201)

**Set-Cookie:**
```
Set-Cookie: session=abc123; Path=/; HttpOnly; Secure
```
- Sets cookie in browser
- `HttpOnly`: Not accessible via JavaScript
- `Secure`: Only sent over HTTPS
- `SameSite`: CSRF protection

**WWW-Authenticate:**
```
WWW-Authenticate: Bearer realm="api"
```
- Authentication challenge
- Sent with 401 Unauthorized
- Tells client how to authenticate

### Custom Headers

**You can create custom headers:**
```
X-Request-ID: 12345
X-API-Version: v2
X-Custom-Header: value
```

**Convention:**
- Custom headers often start with `X-`
- But not required
- Use descriptive names

---

## HTTPS - Securing HTTP with TLS

### Why HTTPS?

**Problem with HTTP:**
- Data sent in **plain text**
- Anyone can read it
- No authentication
- No integrity check

**Solution: HTTPS**
- **Encrypts** data
- **Authenticates** server
- **Ensures** data integrity

### How HTTPS Works

**HTTPS = HTTP + TLS (Transport Layer Security)**

**Process:**

**1. TCP Connection:**
```
Client → Server: TCP handshake
```

**2. TLS Handshake:**
```
Client → Server: "I want secure connection"
Server → Client: "Here's my certificate"
Client: Verifies certificate
Client → Server: "OK, here's encrypted key"
Server: "Got it, let's encrypt"
```

**3. Encrypted HTTP:**
```
All HTTP data encrypted
Looks like gibberish to eavesdroppers
```

### TLS Handshake Deep Dive

**Step 1: Client Hello**
```
Client sends:
  - Supported TLS versions
  - Supported cipher suites
  - Random number (client random)
```

**Step 2: Server Hello**
```
Server sends:
  - Selected TLS version
  - Selected cipher suite
  - Server certificate
  - Random number (server random)
```

**Step 3: Certificate Verification**
```
Client:
  1. Checks certificate validity
  2. Verifies certificate chain
  3. Checks expiration
  4. Verifies domain matches
```

**Step 4: Key Exchange**
```
Client:
  1. Generates pre-master secret
  2. Encrypts with server's public key
  3. Sends to server
```

**Step 5: Session Keys**
```
Both derive session keys from:
  - Pre-master secret
  - Client random
  - Server random
```

**Step 6: Encrypted Communication**
```
Now using symmetric encryption (AES)
Fast and secure
```

### Why Both Symmetric and Asymmetric?

**Asymmetric (RSA):**
- **Slow**: Encrypting large data is expensive
- **Secure**: Public key can't decrypt
- **Use**: Initial key exchange

**Symmetric (AES):**
- **Fast**: Encrypting large data is efficient
- **Secure**: With good key
- **Use**: Actual data encryption

**Hybrid Approach:**
```
1. Use RSA to exchange AES key (once)
2. Use AES to encrypt all data (many times)
Best of both worlds!
```

### Certificate Chain

**How certificates are verified:**

```
Server Certificate
    ↓ (signed by)
Intermediate CA Certificate
    ↓ (signed by)
Root CA Certificate
    ↓ (trusted by)
Operating System / Browser
```

**Process:**
1. Server sends its certificate
2. Browser checks: "Who signed this?"
3. Finds intermediate CA certificate
4. Checks: "Who signed intermediate?"
5. Finds root CA certificate
6. Checks: "Is root CA in my trust store?"
7. If yes → Trusted ✓

### Mixed Content

**Problem:**
```
HTTPS page loads HTTP resources
→ Security warning
→ Some browsers block
```

**Solution:**
- Use HTTPS for all resources
- Or use protocol-relative URLs: `//example.com/resource`

---

## HTTP/2 and HTTP/3 - Evolution of the Protocol

### HTTP/1.1 Problems

**1. Head-of-Line Blocking:**
```
Request 1: Large, slow
Request 2: Small, fast
Request 3: Small, fast

Result: Request 2 and 3 wait for Request 1
```

**2. Multiple Connections:**
```
Browser opens 6 connections per domain
Wastes resources
```

**3. Redundant Headers:**
```
Every request sends same headers:
  Host: example.com
  User-Agent: ...
  Accept: ...
Wastes bandwidth
```

### HTTP/2 Improvements

**1. Multiplexing:**
```
Multiple requests on one connection
No head-of-line blocking
```

**Visual:**
```
HTTP/1.1:
Request 1 ──────────────┐
Request 2 ──────────────┤ (waiting)
Request 3 ──────────────┘ (waiting)

HTTP/2:
Request 1 ────┐
Request 2 ────┼─── (all in parallel)
Request 3 ────┘
```

**2. Header Compression:**
```
Headers compressed (HPACK)
Reduces overhead
```

**3. Server Push:**
```
Server can send resources before client asks
Example: Send CSS when sending HTML
```

**4. Binary Protocol:**
```
HTTP/1.1: Text-based
HTTP/2: Binary
More efficient parsing
```

### HTTP/3 (QUIC)

**Built on UDP instead of TCP:**

**Why UDP?**
- TCP has head-of-line blocking
- UDP allows independent streams

**Features:**
- **Built-in encryption**: TLS 1.3
- **Faster connection**: 0-RTT for some cases
- **Better mobility**: Handles network changes
- **Multiplexing**: Like HTTP/2, but better

**QUIC Benefits:**
```
Network change (WiFi → Mobile):
  TCP: Connection breaks, must reconnect
  QUIC: Connection continues seamlessly
```

---

## Caching - Making the Web Fast

### Why Caching?

**Problem:**
- Every request goes to server
- Server processes request
- Sends response
- **Slow and expensive**

**Solution: Caching**
- Store responses
- Serve from cache when possible
- **Fast and cheap**

### Cache Locations

**1. Browser Cache:**
```
User's browser stores responses
Fastest (no network)
```

**2. CDN Cache:**
```
Content Delivery Network caches
Close to user
```

**3. Proxy Cache:**
```
Corporate proxy caches
Shared among users
```

**4. Server Cache:**
```
Application cache
Redis, Memcached
```

### Cache Headers

**Cache-Control:**
```
Cache-Control: max-age=3600, public
```
- `max-age=3600`: Cache for 3600 seconds
- `public`: Can be cached by CDN
- `private`: Only browser can cache
- `no-cache`: Must revalidate
- `no-store`: Don't cache

**ETag:**
```
ETag: "abc123"
If-None-Match: "abc123"
```
- Version identifier
- Conditional request: "Only send if changed"

**Last-Modified:**
```
Last-Modified: Mon, 01 Jan 2024 12:00:00 GMT
If-Modified-Since: Mon, 01 Jan 2024 12:00:00 GMT
```
- When resource was modified
- Conditional request: "Only send if modified since"

### Cache Strategies

**1. Cache-First (Stale-While-Revalidate):**
```
1. Serve from cache (even if stale)
2. Revalidate in background
3. Update cache if changed
```

**2. Network-First:**
```
1. Try network first
2. If fails, serve from cache
```

**3. Cache-Only:**
```
1. Serve from cache
2. If not in cache, error
```

---

## Cookies and Sessions - State Management

### The Stateless Problem

**HTTP is stateless:**
- Each request is independent
- Server doesn't remember previous requests
- **Problem**: How to maintain user state?

**Solutions:**
- **Cookies**: Store data in browser
- **Sessions**: Store data on server, identify with cookie

### Cookies

**What are Cookies?**
- Small pieces of data stored by browser
- Sent with every request to same domain
- Set by server via `Set-Cookie` header

**Example:**
```
Server: Set-Cookie: session=abc123; Path=/; HttpOnly
Browser: Stores cookie
Next request: Cookie: session=abc123
```

**Cookie Attributes:**
- `Path=/`: Sent for all paths
- `Domain=example.com`: Sent for this domain
- `HttpOnly`: Not accessible via JavaScript (security)
- `Secure`: Only sent over HTTPS
- `SameSite=Strict`: CSRF protection
- `Expires`: When cookie expires
- `Max-Age`: How long cookie lives

### Sessions

**How Sessions Work:**
```
1. User logs in
2. Server creates session (stores in memory/DB)
3. Server sends session ID in cookie
4. Browser stores session ID
5. Next request: Browser sends session ID
6. Server looks up session, knows user
```

**Session Storage:**
- **Memory**: Fast, but lost on restart
- **Database**: Persistent, but slower
- **Redis**: Fast and persistent

**Session vs Cookie:**
- **Cookie**: Data stored in browser (can be read/modified)
- **Session**: Data stored on server (secure)

---

## RESTful API Design - Best Practices

### REST Principles

**1. Resource-Based URLs:**
```
Good:  /api/users/123
Bad:   /api/getUser?id=123
```

**2. Use HTTP Methods:**
```
GET    /api/users      → List users
GET    /api/users/123  → Get user
POST   /api/users      → Create user
PUT    /api/users/123  → Replace user
PATCH  /api/users/123  → Update user
DELETE /api/users/123  → Delete user
```

**3. Use Status Codes:**
```
200 OK
201 Created
204 No Content
400 Bad Request
401 Unauthorized
403 Forbidden
404 Not Found
500 Server Error
```

**4. JSON Response:**
```
{
  "id": 123,
  "name": "Alice",
  "email": "alice@example.com"
}
```

### API Versioning

**URL Versioning:**
```
/api/v1/users
/api/v2/users
```

**Header Versioning:**
```
Accept: application/vnd.api+json;version=1
```

### Pagination

**Offset-Based:**
```
GET /api/users?page=1&limit=20
```

**Cursor-Based:**
```
GET /api/users?cursor=abc123&limit=20
```

### Filtering, Sorting, Searching

```
GET /api/users?status=active&sort=name&search=alice
```

### Error Format

```json
{
  "error": {
    "code": "USER_NOT_FOUND",
    "message": "User with ID 123 does not exist",
    "details": {
      "user_id": 123
    }
  }
}
```

---

## Summary

HTTP/HTTPS is the foundation of web communication. Understanding requests, responses, methods, status codes, headers, and security is essential for backend engineers.

**Key Takeaways:**
- HTTP is stateless, request-response protocol
- Methods have specific purposes and characteristics
- Status codes communicate request outcome
- Headers provide metadata
- HTTPS secures HTTP with TLS
- HTTP/2 and HTTP/3 improve performance
- Caching makes web fast
- Cookies and sessions manage state
- REST principles guide API design

**Next Steps:**
- Practice with HTTP tools (curl, Postman)
- Monitor HTTP traffic (browser dev tools)
- Understand your application's HTTP usage
- Implement proper error handling
- Use appropriate status codes
- Design RESTful APIs

