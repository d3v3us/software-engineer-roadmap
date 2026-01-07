# API Design Deep Dive - Complete Understanding

## Table of Contents
1. [What is API Design?](#what-is-api-design)
2. [RESTful API Principles](#restful-api-principles)
3. [API Design Best Practices](#api-design-best-practices)
4. [API Versioning](#api-versioning)
5. [Error Handling](#error-handling)
6. [API Documentation](#api-documentation)
7. [Common API Design Mistakes](#common-api-design-mistakes)

---

## What is API Design?

### Definition

**API Design**: Process of defining how applications communicate with each other through well-defined interfaces.

**Key Aspects:**
- **Structure**: How endpoints are organized
- **Conventions**: Naming, formatting
- **Behavior**: How API responds
- **Documentation**: How to use API

### Why API Design Matters

**Good API Design:**
- **Easy to use**: Intuitive, predictable
- **Developer-friendly**: Clear, well-documented
- **Maintainable**: Easy to evolve
- **Scalable**: Handles growth

**Bad API Design:**
- **Hard to use**: Confusing, inconsistent
- **Frustrating**: Poor developer experience
- **Hard to maintain**: Difficult to change
- **Limited**: Can't scale

---

## RESTful API Principles

### REST Fundamentals

**REST (Representational State Transfer)**: Architectural style for designing web services.

**Key Principles:**

**1. Resource-Based:**
```
Resources: Things (nouns)
  /users
  /orders
  /products
```

**2. HTTP Methods:**
```
GET:    Read resource
POST:   Create resource
PUT:    Update resource (replace)
PATCH:  Update resource (partial)
DELETE: Delete resource
```

**3. Stateless:**
```
Each request contains all information
No server-side session state
```

**4. Uniform Interface:**
```
Consistent patterns
Standard HTTP methods
Standard status codes
```

### RESTful URL Design

**Good Design:**
```
GET    /users              # List users
GET    /users/123          # Get user 123
POST   /users              # Create user
PUT    /users/123          # Update user 123
PATCH  /users/123          # Partial update
DELETE /users/123          # Delete user 123

GET    /users/123/orders   # Get orders for user 123
POST   /users/123/orders   # Create order for user 123
```

**Bad Design:**
```
GET    /getUsers
POST   /createUser
GET    /user?id=123
POST   /updateUser
GET    /deleteUser?id=123
```

### HTTP Status Codes

**2xx Success:**
- **200 OK**: Request succeeded
- **201 Created**: Resource created
- **204 No Content**: Success, no body

**4xx Client Error:**
- **400 Bad Request**: Invalid request
- **401 Unauthorized**: Not authenticated
- **403 Forbidden**: Not authorized
- **404 Not Found**: Resource not found
- **409 Conflict**: Conflict (e.g., duplicate)
- **429 Too Many Requests**: Rate limited

**5xx Server Error:**
- **500 Internal Server Error**: Server error
- **502 Bad Gateway**: Gateway error
- **503 Service Unavailable**: Service down
- **504 Gateway Timeout**: Timeout

---

## API Design Best Practices

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
DELETE /users/123
```

### 2. Use Plural Nouns

**Bad:**
```
GET /user
GET /order
```

**Good:**
```
GET /users
GET /orders
```

### 3. Use Hierarchical Structure

**Good:**
```
GET /users/123/orders
GET /users/123/orders/456
GET /orders/456/items
```

**Shows relationships clearly**

### 4. Use Query Parameters for Filtering

**Good:**
```
GET /users?status=active
GET /users?role=admin&status=active
GET /users?page=1&limit=20
GET /users?sort=name&order=asc
```

### 5. Use Consistent Naming

**Good:**
```
Consistent:
  /users
  /orders
  /products

Not:
  /users
  /order_list
  /productCatalog
```

### 6. Version Your API

**URL Versioning:**
```
/api/v1/users
/api/v2/users
```

**Header Versioning:**
```
Accept: application/vnd.api+json;version=1
```

### 7. Use Pagination

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
    "totalPages": 50,
    "hasNext": true,
    "hasPrev": false
  }
}
```

### 8. Consistent Response Format

**Good:**
```json
{
  "data": {
    "id": 123,
    "name": "Alice",
    "email": "alice@example.com"
  },
  "meta": {
    "timestamp": "2024-01-15T10:30:00Z"
  }
}
```

### 9. Use Proper HTTP Methods

**Good:**
```
GET    /users/123        # Read
POST   /users            # Create
PUT    /users/123        # Full update
PATCH  /users/123        # Partial update
DELETE /users/123        # Delete
```

### 10. Include Relevant Headers

**Good:**
```
Content-Type: application/json
Accept: application/json
Authorization: Bearer token
X-Request-ID: abc123
```

---

## API Versioning

### Why Version APIs?

**Reasons:**
- **Breaking changes**: Can't change existing API
- **Evolution**: Need to add features
- **Backward compatibility**: Old clients still work
- **Gradual migration**: Migrate clients over time

### Versioning Strategies

**1. URL Versioning:**
```
/api/v1/users
/api/v2/users
```

**Pros:**
- Simple
- Clear
- Easy to understand

**Cons:**
- URLs change
- Not RESTful (some argue)

**2. Header Versioning:**
```
Accept: application/vnd.api+json;version=1
```

**Pros:**
- URLs don't change
- More RESTful

**Cons:**
- Less visible
- Harder to test

**3. Query Parameter:**
```
/api/users?version=1
```

**Pros:**
- Simple
- Optional

**Cons:**
- Can be forgotten
- Not standard

### Versioning Best Practices

**1. Version Early:**
- Start with v1
- Don't wait until you need it

**2. Semantic Versioning:**
```
v1.0.0 → Major.Minor.Patch
Major: Breaking changes
Minor: New features (backward compatible)
Patch: Bug fixes
```

**3. Deprecation:**
```
Deprecate old versions
Give notice period
Remove after migration
```

---

## Error Handling

### Error Response Format

**Consistent Format:**
```json
{
  "error": {
    "code": "USER_NOT_FOUND",
    "message": "User with ID 123 does not exist",
    "details": {
      "user_id": 123,
      "timestamp": "2024-01-15T10:30:00Z"
    },
    "request_id": "abc123"
  }
}
```

### Error Codes

**Use Meaningful Codes:**
```
USER_NOT_FOUND
INVALID_EMAIL
DUPLICATE_USER
INSUFFICIENT_PERMISSIONS
RATE_LIMIT_EXCEEDED
```

### Error Handling Best Practices

**1. Use Appropriate Status Codes:**
```
400: Bad Request (client error)
401: Unauthorized (not authenticated)
403: Forbidden (not authorized)
404: Not Found
409: Conflict
429: Too Many Requests
500: Internal Server Error
```

**2. Provide Helpful Messages:**
```
Bad: "Error"
Good: "User with ID 123 does not exist"
```

**3. Include Request ID:**
```
For debugging
Track requests
Correlate logs
```

**4. Don't Expose Internals:**
```
Bad: "Database connection failed: ..."
Good: "Internal server error. Please try again later."
```

---

## API Documentation

### Why Documentation Matters

**Benefits:**
- **Developer experience**: Easy to use
- **Adoption**: More likely to use
- **Support**: Less support needed
- **Onboarding**: Faster onboarding

### Documentation Elements

**1. Overview:**
- What API does
- Authentication
- Base URL
- Rate limits

**2. Endpoints:**
- URL
- Method
- Parameters
- Request body
- Response
- Examples

**3. Authentication:**
- How to authenticate
- Token format
- Token expiration
- Examples

**4. Error Codes:**
- All error codes
- What they mean
- How to handle

**5. Examples:**
- Request examples
- Response examples
- Code samples
- Use cases

### Documentation Tools

**Popular Tools:**
- **OpenAPI/Swagger**: Standard, widely used
- **Postman**: Interactive documentation
- **Redoc**: Beautiful documentation
- **API Blueprint**: Markdown-based

---

## Common API Design Mistakes

### Mistake 1: Inconsistent Naming

**Bad:**
```
GET /getUsers
GET /user_list
GET /fetchProducts
```

**Good:**
```
GET /users
GET /users
GET /products
```

### Mistake 2: Wrong HTTP Methods

**Bad:**
```
GET /users/123/delete
POST /users/123/update
```

**Good:**
```
DELETE /users/123
PUT /users/123
```

### Mistake 3: No Versioning

**Bad:**
```
/api/users (changes break clients)
```

**Good:**
```
/api/v1/users (stable)
/api/v2/users (new version)
```

### Mistake 4: Poor Error Messages

**Bad:**
```
{"error": "Error"}
```

**Good:**
```
{
  "error": {
    "code": "USER_NOT_FOUND",
    "message": "User with ID 123 does not exist"
  }
}
```

### Mistake 5: No Pagination

**Bad:**
```
GET /users → Returns 10,000 users
```

**Good:**
```
GET /users?page=1&limit=20
```

### Mistake 6: Inconsistent Response Format

**Bad:**
```
Sometimes: {"user": {...}}
Sometimes: {...}
Sometimes: [{"user": {...}}]
```

**Good:**
```
Always: {"data": {...}}
```

---

## Summary

Good API design makes APIs easy to use, maintain, and evolve. Following REST principles and best practices creates better developer experiences.

**Key Takeaways:**
- REST: Resource-based, HTTP methods, stateless, uniform interface
- Best practices: Nouns, plural, hierarchical, consistent, versioned
- Error handling: Consistent format, appropriate status codes, helpful messages
- Documentation: Essential for adoption
- Common mistakes: Inconsistent naming, wrong methods, no versioning

**Next Steps:**
- Design APIs following REST principles
- Version from the start
- Document thoroughly
- Get feedback
- Iterate and improve

