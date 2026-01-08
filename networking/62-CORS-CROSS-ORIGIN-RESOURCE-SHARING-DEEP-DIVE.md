# CORS (Cross-Origin Resource Sharing) Deep Dive - Complete Understanding

## Table of Contents
1. [What is CORS?](#what-is-cors)
2. [Why CORS Matters](#why-cors-matters)
3. [Same-Origin Policy](#same-origin-policy)
4. [CORS Mechanism](#cors-mechanism)
5. [CORS Headers](#cors-headers)
6. [CORS Request Types](#cors-request-types)
7. [CORS Implementation](#cors-implementation)
8. [CORS Security](#cors-security)
9. [Best Practices](#best-practices)

---

## What is CORS?

### Definition

**CORS (Cross-Origin Resource Sharing)**: Mechanism allowing web pages to make requests to different domain.

**Key Characteristics:**
- **Cross-origin**: Cross-origin requests
- **Security**: Security mechanism
- **Headers**: HTTP headers
- **Browser**: Browser-enforced

### Real-World Analogy

**CORS = Passport Control:**
- **Passport**: CORS headers
- **Border**: Origin boundary
- **Control**: Access control
- **Permission**: Permission to cross

**Web Security:**
- **Origin**: Web origin
- **CORS**: Cross-origin sharing
- **Security**: Security control
- **Access**: Controlled access

---

## Why CORS Matters?

### Benefits

**1. Security:**
```
CORS
  ↓
Cross-origin security
  ↓
Prevent attacks
```

**2. Functionality:**
```
CORS
  ↓
Cross-origin requests
  ↓
Web functionality
```

**3. API Access:**
```
CORS
  ↓
API access from browser
  ↓
Web applications
```

---

## Same-Origin Policy

### What is Same-Origin Policy?

**Same-Origin Policy**: Browser security policy restricting access to resources from different origins.

**Origin Components:**
- **Protocol**: http, https
- **Domain**: example.com
- **Port**: 80, 443, 8080

**Same Origin:**
```
https://example.com:443/page1
https://example.com:443/page2
→ Same origin (protocol, domain, port match)
```

**Different Origins:**
```
https://example.com:443/page1
http://example.com:80/page2
→ Different origin (protocol differs)

https://example.com:443/page1
https://api.example.com:443/page2
→ Different origin (domain differs)
```

### Why Same-Origin Policy?

**Security Reasons:**
- **Prevent attacks**: Prevent CSRF attacks
- **Data protection**: Protect user data
- **Session protection**: Protect sessions
- **Privacy**: Protect privacy

---

## CORS Mechanism

### How CORS Works

**CORS Process:**
```
1. Browser sends preflight request (OPTIONS)
2. Server responds with CORS headers
3. Browser checks CORS headers
4. Browser allows or blocks request
```

### CORS Flow

**Simple Request:**
```
Browser → Server (with Origin header)
Server → Browser (with CORS headers)
Browser → Allow/Block
```

**Preflight Request:**
```
Browser → Server (OPTIONS with Origin)
Server → Browser (CORS headers)
Browser → Check CORS headers
Browser → Server (Actual request if allowed)
Server → Browser (Response)
```

---

## CORS Headers

### Request Headers

**Origin Header:**
```
Origin: https://example.com
```
- **Sent by**: Browser automatically
- **Contains**: Request origin
- **Purpose**: Identify origin

### Response Headers

**Access-Control-Allow-Origin:**
```
Access-Control-Allow-Origin: https://example.com
Access-Control-Allow-Origin: *
```
- **Purpose**: Allow specific origin or all
- **Values**: Specific origin or *
- **Security**: * allows all origins

**Access-Control-Allow-Methods:**
```
Access-Control-Allow-Methods: GET, POST, PUT, DELETE
```
- **Purpose**: Allowed HTTP methods
- **Values**: Comma-separated methods
- **Required**: For preflight requests

**Access-Control-Allow-Headers:**
```
Access-Control-Allow-Headers: Content-Type, Authorization
```
- **Purpose**: Allowed request headers
- **Values**: Comma-separated headers
- **Required**: For preflight requests

**Access-Control-Allow-Credentials:**
```
Access-Control-Allow-Credentials: true
```
- **Purpose**: Allow credentials (cookies, auth)
- **Values**: true or false
- **Security**: Requires specific origin (not *)

**Access-Control-Expose-Headers:**
```
Access-Control-Expose-Headers: X-Custom-Header
```
- **Purpose**: Expose custom headers to JavaScript
- **Values**: Comma-separated headers
- **Default**: Only simple headers exposed

**Access-Control-Max-Age:**
```
Access-Control-Max-Age: 86400
```
- **Purpose**: Cache preflight response
- **Values**: Seconds
- **Default**: No caching

---

## CORS Request Types

### Simple Request

**Simple Request Criteria:**
- **Method**: GET, POST, HEAD
- **Headers**: Simple headers only
- **Content-Type**: application/x-www-form-urlencoded, multipart/form-data, text/plain

**Simple Request Flow:**
```
Browser → Server (with Origin)
Server → Browser (with CORS headers)
Browser → Allow/Block
```

**Example:**
```javascript
fetch('https://api.example.com/data', {
    method: 'GET',
    headers: {
        'Content-Type': 'text/plain'
    }
});
```

### Preflight Request

**Preflight Request Triggers:**
- **Method**: PUT, DELETE, PATCH, etc.
- **Headers**: Custom headers
- **Content-Type**: application/json, etc.

**Preflight Request Flow:**
```
Browser → Server (OPTIONS with Origin)
Server → Browser (CORS headers)
Browser → Check CORS headers
Browser → Server (Actual request if allowed)
```

**Example:**
```javascript
fetch('https://api.example.com/data', {
    method: 'PUT',
    headers: {
        'Content-Type': 'application/json',
        'Authorization': 'Bearer token'
    },
    body: JSON.stringify({data: 'value'})
});
```

---

## CORS Implementation

### Server-Side Implementation

**Go Implementation:**
```go
func corsMiddleware(next http.Handler) http.Handler {
    return http.HandlerFunc(func(w http.ResponseWriter, r *http.Request) {
        origin := r.Header.Get("Origin")
        
        // Check if origin is allowed
        if isAllowedOrigin(origin) {
            w.Header().Set("Access-Control-Allow-Origin", origin)
            w.Header().Set("Access-Control-Allow-Methods", "GET, POST, PUT, DELETE, OPTIONS")
            w.Header().Set("Access-Control-Allow-Headers", "Content-Type, Authorization")
            w.Header().Set("Access-Control-Allow-Credentials", "true")
        }
        
        // Handle preflight
        if r.Method == "OPTIONS" {
            w.WriteHeader(http.StatusOK)
            return
        }
        
        next.ServeHTTP(w, r)
    })
}
```

### CORS Configuration

**Allowed Origins:**
```go
var allowedOrigins = []string{
    "https://example.com",
    "https://app.example.com",
}

func isAllowedOrigin(origin string) bool {
    for _, allowed := range allowedOrigins {
        if origin == allowed {
            return true
        }
    }
    return false
}
```

### CORS with Credentials

**Credentials Support:**
```go
w.Header().Set("Access-Control-Allow-Origin", origin)  // Must be specific, not *
w.Header().Set("Access-Control-Allow-Credentials", "true")
w.Header().Set("Access-Control-Allow-Methods", "GET, POST")
w.Header().Set("Access-Control-Allow-Headers", "Content-Type, Authorization")
```

---

## CORS Security

### Security Considerations

**1. Origin Validation:**
- **Validate**: Always validate origin
- **Whitelist**: Use whitelist
- **Never * with credentials**: Never use * with credentials
- **Specific origins**: Use specific origins

**2. Credentials:**
- **Careful**: Be careful with credentials
- **Specific origin**: Require specific origin
- **Secure**: Use secure cookies
- **Validation**: Validate credentials

**3. Methods and Headers:**
- **Minimal**: Allow minimal methods
- **Necessary**: Only necessary headers
- **Validation**: Validate headers
- **Security**: Security considerations

### Common Vulnerabilities

**1. Wildcard Origin:**
```go
// VULNERABLE
w.Header().Set("Access-Control-Allow-Origin", "*")

// SECURE
w.Header().Set("Access-Control-Allow-Origin", "https://example.com")
```

**2. Credentials with Wildcard:**
```go
// VULNERABLE
w.Header().Set("Access-Control-Allow-Origin", "*")
w.Header().Set("Access-Control-Allow-Credentials", "true")  // ERROR

// SECURE
w.Header().Set("Access-Control-Allow-Origin", "https://example.com")
w.Header().Set("Access-Control-Allow-Credentials", "true")
```

**3. Overly Permissive Headers:**
```go
// VULNERABLE
w.Header().Set("Access-Control-Allow-Headers", "*")

// SECURE
w.Header().Set("Access-Control-Allow-Headers", "Content-Type, Authorization")
```

---

## Best Practices

### 1. Validate Origins

**Why:**
- **Security**: Better security
- **Attack prevention**: Prevent attacks
- **Control**: Better control
- **Compliance**: Meet compliance

**Guidelines:**
- **Whitelist**: Use origin whitelist
- **Validate**: Always validate origin
- **Never * with credentials**: Never use * with credentials
- **Specific**: Use specific origins

### 2. Minimal Permissions

**Why:**
- **Security**: Better security
- **Attack surface**: Reduced attack surface
- **Principle**: Least privilege
- **Compliance**: Meet compliance

**Guidelines:**
- **Methods**: Allow only needed methods
- **Headers**: Allow only needed headers
- **Credentials**: Use credentials carefully
- **Expose**: Expose minimal headers

### 3. Use Preflight Caching

**Why:**
- **Performance**: Better performance
- **Efficiency**: More efficient
- **Reduced requests**: Fewer preflight requests
- **User experience**: Better UX

**Guidelines:**
- **Max-Age**: Set Access-Control-Max-Age
- **Reasonable**: Reasonable cache time
- **Balance**: Balance security and performance
- **Monitor**: Monitor cache usage

### 4. Monitor CORS

**Why:**
- **Security**: Detect attacks
- **Usage**: Monitor usage
- **Performance**: Monitor performance
- **Compliance**: Meet compliance

**Guidelines:**
- **Logging**: Log CORS requests
- **Monitoring**: Monitor CORS
- **Alerting**: Alert on issues
- **Analysis**: Analyze patterns

---

## Summary

CORS (Cross-Origin Resource Sharing) enables cross-origin requests while maintaining security. Understanding same-origin policy, CORS mechanism, CORS headers, CORS request types (simple, preflight), CORS implementation, CORS security, and best practices is crucial for building secure web applications.

**Key Takeaways:**
- **CORS**: Mechanism allowing cross-origin requests (cross-origin, security, headers, browser-enforced)
- **Same-origin policy**: Browser security policy (restricts access, origin components: protocol domain port, same origin: all match, different origins: any differs, security reasons: prevent attacks data protection session protection privacy)
- **CORS mechanism**: How CORS works (CORS process: preflight OPTIONS server responds browser checks allow/block), CORS flow (simple request, preflight request)
- **CORS headers**: Request headers (Origin: sent by browser contains origin identifies origin), response headers (Access-Control-Allow-Origin: allow origin or all, Access-Control-Allow-Methods: allowed HTTP methods, Access-Control-Allow-Headers: allowed request headers, Access-Control-Allow-Credentials: allow credentials, Access-Control-Expose-Headers: expose custom headers, Access-Control-Max-Age: cache preflight)
- **CORS request types**: Simple request (GET POST HEAD simple headers, no preflight), preflight request (PUT DELETE PATCH custom headers, requires OPTIONS preflight)
- **CORS implementation**: Server-side implementation (Go middleware, CORS configuration, allowed origins, credentials support)
- **CORS security**: Security considerations (origin validation: validate whitelist never * with credentials specific origins, credentials: careful specific origin secure validation, methods and headers: minimal necessary validation security), common vulnerabilities (wildcard origin, credentials with wildcard, overly permissive headers)
- **Best practices**: Validate origins, minimal permissions, use preflight caching, monitor CORS

**CORS Headers:**
- **Allow-Origin**: Specific origin or *
- **Allow-Methods**: Allowed methods
- **Allow-Headers**: Allowed headers
- **Allow-Credentials**: Credentials support

**Best Practices:**
- Validate origins
- Minimal permissions
- Use preflight caching
- Monitor CORS

**Next Steps:**
- Learn CORS
- Implement CORS
- Secure CORS
- Monitor CORS

