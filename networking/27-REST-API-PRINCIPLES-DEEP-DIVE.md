# REST API Principles Deep Dive - Complete Understanding

## Table of Contents
1. [What is REST?](#what-is-rest)
2. [REST Principles](#rest-principles)
3. [RESTful Resources](#restful-resources)
4. [HTTP Methods](#http-methods)
5. [Status Codes](#status-codes)
6. [REST Constraints](#rest-constraints)
7. [REST vs RPC](#rest-vs-rpc)
8. [REST Best Practices](#rest-best-practices)
9. [Common Mistakes](#common-mistakes)

---

## What is REST?

### Definition

**REST (Representational State Transfer)**: Architectural style for designing networked applications.

**Key Concept:**
- **Stateless**: Stateless communication
- **Resource-based**: Resource-based
- **HTTP**: Uses HTTP methods
- **Standard**: Standard approach

### Real-World Analogy

**REST = Library System:**
- **Books**: Resources
- **Actions**: HTTP methods (GET, POST, etc.)
- **Status**: Status codes (available, checked out)
- **Stateless**: Each request independent

**API:**
- **Resources**: API resources (users, orders)
- **HTTP methods**: GET, POST, PUT, DELETE
- **Status codes**: 200, 404, 500
- **Stateless**: No session state

---

## REST Principles

### Principle 1: Stateless

**What:**
```
Each request contains all information
  ↓
No server-side session
  ↓
Server doesn't remember previous requests
```

**Benefits:**
- **Scalability**: Easy to scale
- **Reliability**: More reliable
- **Simplicity**: Simpler design

### Principle 2: Client-Server

**What:**
```
Client and server separated
  ↓
Independent evolution
  ↓
Clear separation of concerns
```

**Benefits:**
- **Flexibility**: Flexible architecture
- **Evolution**: Independent evolution
- **Reusability**: Reusable components

### Principle 3: Uniform Interface

**What:**
```
Standard interface
  ↓
HTTP methods
  ↓
Consistent patterns
```

**Benefits:**
- **Simplicity**: Simple to understand
- **Standard**: Standard approach
- **Interoperability**: Interoperable

### Principle 4: Resource-Based

**What:**
```
Everything is a resource
  ↓
Resources identified by URI
  ↓
Resources manipulated via representations
```

**Benefits:**
- **Clarity**: Clear resource model
- **Intuitive**: Intuitive design
- **RESTful**: RESTful design

---

## RESTful Resources

### Resource Identification

**URIs:**
```
/users          # Collection of users
/users/123      # Specific user
/users/123/orders  # User's orders
/orders/456     # Specific order
```

### Resource Naming

**Best Practices:**
- **Nouns**: Use nouns, not verbs
- **Plural**: Use plural for collections
- **Hierarchical**: Hierarchical structure
- **Consistent**: Consistent naming

**Good:**
```
GET /users
GET /users/123
GET /users/123/orders
```

**Bad:**
```
GET /getUsers
GET /user/123
GET /getUserOrders/123
```

---

## HTTP Methods

### GET

**Purpose:**
```
Retrieve resource
  ↓
Read-only
  ↓
Idempotent
```

**Example:**
```
GET /users/123
  ↓
Returns user 123
```

### POST

**Purpose:**
```
Create resource
  ↓
Not idempotent
  ↓
Side effects
```

**Example:**
```
POST /users
Body: {name: "John", email: "john@example.com"}
  ↓
Creates new user
```

### PUT

**Purpose:**
```
Update/replace resource
  ↓
Idempotent
  ↓
Full update
```

**Example:**
```
PUT /users/123
Body: {name: "John Doe", email: "john@example.com"}
  ↓
Replaces user 123
```

### PATCH

**Purpose:**
```
Partial update
  ↓
Idempotent
  ↓
Partial update
```

**Example:**
```
PATCH /users/123
Body: {name: "John Doe"}
  ↓
Updates only name
```

### DELETE

**Purpose:**
```
Delete resource
  ↓
Idempotent
  ↓
Removes resource
```

**Example:**
```
DELETE /users/123
  ↓
Deletes user 123
```

### Method Summary

| Method | Purpose | Idempotent | Safe |
|--------|---------|------------|------|
| **GET** | Read | Yes | Yes |
| **POST** | Create | No | No |
| **PUT** | Update/Replace | Yes | No |
| **PATCH** | Partial Update | Yes | No |
| **DELETE** | Delete | Yes | No |

---

## Status Codes

### 2xx Success

**200 OK:**
```
Request successful
  ↓
GET, PUT, PATCH
```

**201 Created:**
```
Resource created
  ↓
POST
```

**204 No Content:**
```
Success, no body
  ↓
DELETE
```

### 4xx Client Error

**400 Bad Request:**
```
Invalid request
  ↓
Malformed request
```

**401 Unauthorized:**
```
Authentication required
  ↓
Not authenticated
```

**403 Forbidden:**
```
Not authorized
  ↓
Authenticated but not authorized
```

**404 Not Found:**
```
Resource not found
  ↓
Resource doesn't exist
```

**409 Conflict:**
```
Conflict
  ↓
Resource conflict
```

### 5xx Server Error

**500 Internal Server Error:**
```
Server error
  ↓
Unexpected error
```

**503 Service Unavailable:**
```
Service unavailable
  ↓
Temporary unavailability
```

---

## REST Constraints

### Constraint 1: Stateless

**Requirement:**
```
No server-side session
  ↓
Each request independent
  ↓
All state in request
```

### Constraint 2: Cacheable

**Requirement:**
```
Responses cacheable
  ↓
Explicit caching
  ↓
Cache-Control headers
```

### Constraint 3: Layered System

**Requirement:**
```
Layered architecture
  ↓
Load balancers
  ↓
Proxies
  ↓
Transparent to client
```

### Constraint 4: Code on Demand (Optional)

**Requirement:**
```
Server can send code
  ↓
Client executes
  ↓
Optional constraint
```

---

## REST vs RPC

### REST

**Approach:**
```
Resource-based
  ↓
HTTP methods
  ↓
Stateless
```

**Example:**
```
GET /users/123
POST /users
PUT /users/123
DELETE /users/123
```

### RPC

**Approach:**
```
Action-based
  ↓
Function calls
  ↓
Stateful
```

**Example:**
```
POST /getUser
POST /createUser
POST /updateUser
POST /deleteUser
```

### When to Use

**Use REST When:**
- **CRUD operations**: CRUD operations
- **Resource-based**: Resource-based design
- **Standard**: Standard approach

**Use RPC When:**
- **Complex operations**: Complex operations
- **Action-based**: Action-based design
- **Performance**: Performance critical

---

## REST Best Practices

### 1. Use Proper HTTP Methods

**Why:**
- **Semantics**: Clear semantics
- **Standard**: Standard approach
- **Caching**: Better caching

**Guidelines:**
- **GET**: For reading
- **POST**: For creating
- **PUT**: For full update
- **PATCH**: For partial update
- **DELETE**: For deleting

### 2. Use Proper Status Codes

**Why:**
- **Clarity**: Clear response meaning
- **Error handling**: Better error handling
- **Standard**: Standard approach

**Guidelines:**
- **200**: Success
- **201**: Created
- **400**: Bad request
- **404**: Not found
- **500**: Server error

### 3. Use Plural Nouns

**Why:**
- **Consistency**: Consistent naming
- **Standard**: Standard practice
- **Clarity**: Clear resource model

**Example:**
```
/users (not /user)
/orders (not /order)
/products (not /product)
```

### 4. Use Hierarchical Resources

**Why:**
- **Relationships**: Show relationships
- **Intuitive**: Intuitive design
- **RESTful**: RESTful design

**Example:**
```
/users/123/orders
/users/123/orders/456
```

### 5. Version APIs

**Why:**
- **Evolution**: API evolution
- **Backward compatibility**: Backward compatibility
- **Migration**: Gradual migration

**Example:**
```
/api/v1/users
/api/v2/users
```

---

## Common Mistakes

### Mistake 1: Using Verbs in URLs

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

### Mistake 3: Ignoring Status Codes

**Bad:**
```
Always return 200 OK
  ↓
Error in response body
```

**Good:**
```
Return appropriate status code
  ↓
200 for success
404 for not found
500 for server error
```

### Mistake 4: Not Being Stateless

**Bad:**
```
Store state on server
  ↓
Session-based
  ↓
Not RESTful
```

**Good:**
```
Stateless
  ↓
All state in request
  ↓
RESTful
```

---

## Summary

REST is a standard architectural style for APIs. Understanding REST principles, HTTP methods, status codes, and best practices is essential for building RESTful APIs.

**Key Takeaways:**
- **REST**: Architectural style for networked applications
- **Principles**: Stateless, client-server, uniform interface, resource-based
- **Resources**: Everything is a resource, identified by URI
- **HTTP methods**: GET, POST, PUT, PATCH, DELETE
- **Status codes**: 2xx success, 4xx client error, 5xx server error
- **Constraints**: Stateless, cacheable, layered system
- **REST vs RPC**: Resource-based vs action-based
- **Best practices**: Proper methods, status codes, plural nouns, hierarchical, versioning
- **Common mistakes**: Verbs in URLs, wrong methods, ignoring status codes, not stateless

**REST Principles:**
- **Stateless**: No server-side session
- **Client-server**: Separation of concerns
- **Uniform interface**: Standard interface
- **Resource-based**: Resource-based design

**HTTP Methods:**
- **GET**: Read (idempotent, safe)
- **POST**: Create (not idempotent)
- **PUT**: Update/Replace (idempotent)
- **PATCH**: Partial update (idempotent)
- **DELETE**: Delete (idempotent)

**Best Practices:**
- Use proper HTTP methods
- Use proper status codes
- Use plural nouns
- Use hierarchical resources
- Version APIs

**Common Mistakes:**
- Using verbs in URLs
- Wrong HTTP methods
- Ignoring status codes
- Not being stateless

**Next Steps:**
- Design RESTful APIs
- Use proper HTTP methods
- Return appropriate status codes
- Follow REST principles
- Test and validate

