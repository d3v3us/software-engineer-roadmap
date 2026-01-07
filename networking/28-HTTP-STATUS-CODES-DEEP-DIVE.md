# HTTP Status Codes Deep Dive - Complete Understanding

## Table of Contents
1. [What are HTTP Status Codes?](#what-are-http-status-codes)
2. [Why Status Codes Matter](#why-status-codes-matter)
3. [Status Code Categories](#status-code-categories)
4. [1xx Informational](#1xx-informational)
5. [2xx Success](#2xx-success)
6. [3xx Redirection](#3xx-redirection)
7. [4xx Client Error](#4xx-client-error)
8. [5xx Server Error](#5xx-server-error)
9. [Status Code Best Practices](#status-code-best-practices)
10. [Common Mistakes](#common-mistakes)

---

## What are HTTP Status Codes?

### Definition

**HTTP Status Code**: Three-digit number returned by server to indicate result of HTTP request.

**Key Concept:**
- **Response code**: Response status code
- **Indicates result**: Indicates request result
- **Standard**: Standard codes
- **Client handling**: Client can handle based on code

### Real-World Analogy

**Status Code = Traffic Light:**
- **Green (200)**: Go (success)
- **Yellow (3xx)**: Redirect
- **Red (4xx/5xx)**: Stop (error)

**HTTP:**
- **200 OK**: Request successful
- **301 Moved**: Resource moved
- **404 Not Found**: Resource not found
- **500 Error**: Server error

---

## Why Status Codes Matter?

### Benefits

**1. Clear Communication:**
```
Status code
  ↓
Clear result
  ↓
Client knows what happened
```

**2. Error Handling:**
```
Different codes
  ↓
Different handling
  ↓
Better error handling
```

**3. Standard:**
```
Standard codes
  ↓
Consistent
  ↓
Interoperable
```

---

## Status Code Categories

### Categories

**1xx Informational:**
```
Informational response
  ↓
Request received
  ↓
Processing
```

**2xx Success:**
```
Request successful
  ↓
Operation completed
  ↓
Success
```

**3xx Redirection:**
```
Further action needed
  ↓
Redirect client
  ↓
Location change
```

**4xx Client Error:**
```
Client error
  ↓
Invalid request
  ↓
Client must fix
```

**5xx Server Error:**
```
Server error
  ↓
Server problem
  ↓
Server must fix
```

---

## 1xx Informational

### 100 Continue

**Meaning:**
```
Client should continue
  ↓
Request body to follow
  ↓
Provisional response
```

**Use Case:**
- **Large requests**: Large request bodies
- **Expect header**: Expect: 100-continue

### 101 Switching Protocols

**Meaning:**
```
Switching protocols
  ↓
Upgrade request
  ↓
Protocol upgrade
```

**Use Case:**
- **WebSocket**: HTTP to WebSocket
- **HTTP/2**: HTTP/1.1 to HTTP/2

---

## 2xx Success

### 200 OK

**Meaning:**
```
Request successful
  ↓
Standard success
  ↓
Most common
```

**Use Case:**
- **GET**: Successful GET
- **POST**: Successful POST
- **PUT**: Successful PUT
- **PATCH**: Successful PATCH

### 201 Created

**Meaning:**
```
Resource created
  ↓
New resource
  ↓
Location header
```

**Use Case:**
- **POST**: Create new resource
- **Location**: Location header with new resource URL

### 204 No Content

**Meaning:**
```
Success, no content
  ↓
No response body
  ↓
Operation successful
```

**Use Case:**
- **DELETE**: Successful deletion
- **PUT**: Update with no content
- **No body needed**: No response body needed

### 202 Accepted

**Meaning:**
```
Request accepted
  ↓
Processing asynchronously
  ↓
Not yet completed
```

**Use Case:**
- **Async processing**: Asynchronous processing
- **Background jobs**: Background job submission

---

## 3xx Redirection

### 301 Moved Permanently

**Meaning:**
```
Resource moved permanently
  ↓
New location
  ↓
Update bookmarks
```

**Use Case:**
- **Permanent redirect**: Permanent URL change
- **SEO**: Update search engines

### 302 Found (Temporary Redirect)

**Meaning:**
```
Resource found
  ↓
Temporary location
  ↓
Don't update bookmarks
```

**Use Case:**
- **Temporary redirect**: Temporary URL change
- **Load balancing**: Load balancing

### 304 Not Modified

**Meaning:**
```
Resource not modified
  ↓
Use cached version
  ↓
Conditional request
```

**Use Case:**
- **Caching**: Cache validation
- **ETag**: ETag validation
- **If-None-Match**: If-None-Match header

---

## 4xx Client Error

### 400 Bad Request

**Meaning:**
```
Invalid request
  ↓
Malformed request
  ↓
Client error
```

**Use Case:**
- **Invalid syntax**: Invalid request syntax
- **Missing required**: Missing required fields
- **Invalid format**: Invalid data format

### 401 Unauthorized

**Meaning:**
```
Authentication required
  ↓
Not authenticated
  ↓
Need credentials
```

**Use Case:**
- **No auth**: No authentication
- **Invalid auth**: Invalid credentials
- **Expired**: Expired token

### 403 Forbidden

**Meaning:**
```
Not authorized
  ↓
Authenticated but not authorized
  ↓
Access denied
```

**Use Case:**
- **No permission**: No permission
- **Access denied**: Access denied
- **Authorization**: Authorization failure

### 404 Not Found

**Meaning:**
```
Resource not found
  ↓
Resource doesn't exist
  ↓
Common error
```

**Use Case:**
- **Missing resource**: Resource doesn't exist
- **Wrong URL**: Wrong URL
- **Deleted**: Resource deleted

### 409 Conflict

**Meaning:**
```
Conflict
  ↓
Resource conflict
  ↓
Concurrent modification
```

**Use Case:**
- **Concurrent update**: Concurrent update conflict
- **Duplicate**: Duplicate resource
- **State conflict**: State conflict

### 429 Too Many Requests

**Meaning:**
```
Rate limit exceeded
  ↓
Too many requests
  ↓
Rate limiting
```

**Use Case:**
- **Rate limiting**: Rate limit exceeded
- **Throttling**: Request throttling
- **Retry-After**: Retry-After header

---

## 5xx Server Error

### 500 Internal Server Error

**Meaning:**
```
Server error
  ↓
Unexpected error
  ↓
Server problem
```

**Use Case:**
- **Unexpected error**: Unexpected server error
- **Exception**: Unhandled exception
- **Server failure**: Server failure

### 502 Bad Gateway

**Meaning:**
```
Bad gateway
  ↓
Invalid response from upstream
  ↓
Gateway issue
```

**Use Case:**
- **Proxy error**: Proxy/gateway error
- **Upstream error**: Upstream server error
- **Network issue**: Network issue

### 503 Service Unavailable

**Meaning:**
```
Service unavailable
  ↓
Temporary unavailability
  ↓
Maintenance or overload
```

**Use Case:**
- **Maintenance**: Service maintenance
- **Overload**: Server overload
- **Temporary**: Temporary unavailability

### 504 Gateway Timeout

**Meaning:**
```
Gateway timeout
  ↓
Upstream timeout
  ↓
Gateway timeout
```

**Use Case:**
- **Timeout**: Upstream timeout
- **Slow response**: Slow upstream response
- **Network**: Network timeout

---

## Status Code Best Practices

### 1. Use Appropriate Codes

**Why:**
- **Clarity**: Clear communication
- **Error handling**: Better error handling
- **Standard**: Follow standards

**Guidelines:**
- **200**: Success
- **201**: Created
- **400**: Bad request
- **404**: Not found
- **500**: Server error

### 2. Be Consistent

**Why:**
- **Predictability**: Predictable behavior
- **Error handling**: Easier error handling
- **User experience**: Better UX

**Guidelines:**
- **Same code**: Same situation, same code
- **Document**: Document status codes
- **Consistent**: Consistent usage

### 3. Include Error Details

**Why:**
- **Debugging**: Easier debugging
- **User experience**: Better UX
- **Support**: Easier support

**Guidelines:**
- **Error message**: Include error message
- **Error code**: Include error code
- **Details**: Include relevant details

### 4. Handle All Codes

**Why:**
- **Robustness**: Robust client
- **Error handling**: Proper error handling
- **User experience**: Better UX

**Guidelines:**
- **Handle all**: Handle all status codes
- **Default handling**: Default handling
- **User-friendly**: User-friendly messages

---

## Common Mistakes

### Mistake 1: Always Return 200

**Problem:**
```
Always return 200 OK
  ↓
Error in response body
  ↓
Client doesn't know it's error
```

**Solution:**
```
Return appropriate status code
  ↓
200 for success
  ↓
4xx/5xx for errors
```

### Mistake 2: Wrong Status Code

**Problem:**
```
Use wrong status code
  ↓
404 for authentication error
  ↓
Confusing
```

**Solution:**
```
Use correct status code
  ↓
401 for authentication
  ↓
404 for not found
```

### Mistake 3: Ignoring Status Codes

**Problem:**
```
Client ignores status codes
  ↓
Always treats as success
  ↓
Wrong behavior
```

**Solution:**
```
Check status codes
  ↓
Handle appropriately
  ↓
Error handling
```

---

## Summary

HTTP status codes communicate request results. Understanding status codes, their meanings, and best practices is essential for building robust APIs.

**Key Takeaways:**
- **HTTP status codes**: Three-digit response codes
- **Categories**: 1xx informational, 2xx success, 3xx redirection, 4xx client error, 5xx server error
- **1xx**: Informational (100 Continue, 101 Switching Protocols)
- **2xx**: Success (200 OK, 201 Created, 204 No Content, 202 Accepted)
- **3xx**: Redirection (301 Moved, 302 Found, 304 Not Modified)
- **4xx**: Client error (400 Bad Request, 401 Unauthorized, 403 Forbidden, 404 Not Found, 409 Conflict, 429 Too Many Requests)
- **5xx**: Server error (500 Internal Server Error, 502 Bad Gateway, 503 Service Unavailable, 504 Gateway Timeout)
- **Best practices**: Use appropriate codes, be consistent, include error details, handle all codes
- **Common mistakes**: Always 200, wrong code, ignoring codes

**Status Code Categories:**
- **1xx**: Informational
- **2xx**: Success
- **3xx**: Redirection
- **4xx**: Client error
- **5xx**: Server error

**Best Practices:**
- Use appropriate codes
- Be consistent
- Include error details
- Handle all codes

**Common Mistakes:**
- Always return 200
- Wrong status code
- Ignoring status codes

**Next Steps:**
- Understand status codes
- Use appropriate codes
- Handle all codes
- Include error details
- Test error handling

