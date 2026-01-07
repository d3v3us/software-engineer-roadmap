# API Authentication and Authorization Deep Dive - Complete Understanding

## Table of Contents
1. [Authentication vs Authorization](#authentication-vs-authorization)
2. [Authentication Methods](#authentication-methods)
3. [API Keys](#api-keys)
4. [Basic Authentication](#basic-authentication)
5. [Bearer Tokens (JWT)](#bearer-tokens-jwt)
6. [OAuth 2.0](#oauth-20)
7. [OAuth 2.0 Flows](#oauth-20-flows)
8. [OpenID Connect (OIDC)](#openid-connect-oidc)
9. [Session-Based Authentication](#session-based-authentication)
10. [Token-Based Authentication](#token-based-authentication)
11. [Authorization Models](#authorization-models)
12. [Role-Based Access Control (RBAC)](#role-based-access-control-rbac)
13. [Attribute-Based Access Control (ABAC)](#attribute-based-access-control-abac)
14. [Best Practices](#best-practices)
15. [Common Security Issues](#common-security-issues)

---

## Authentication vs Authorization

### Definitions

**Authentication**: Process of verifying who a user is.

**Authorization**: Process of verifying what a user is allowed to do.

### Real-World Analogy

**Authentication = ID Check:**
- **Who are you?**: Verify identity
- **ID card**: Credentials (username/password)
- **Security guard**: System verifies

**Authorization = Permission Check:**
- **What can you do?**: Verify permissions
- **Access card**: Permissions (roles)
- **Door lock**: System checks permissions

### Example

**Authentication:**
```
User: "I'm Alice"
System: "Prove it" (enter password)
User: "password123"
System: "Verified, you are Alice"
```

**Authorization:**
```
User: "I want to delete this file"
System: "Are you authorized?" (check permissions)
System: "You are admin, allowed"
```

---

## Authentication Methods

### Common Methods

**1. API Keys:**
- **Simple**: Simple to implement
- **Static**: Static key
- **Use**: Service-to-service

**2. Basic Authentication:**
- **Username/password**: Username and password
- **Base64 encoded**: Base64 encoded
- **Use**: Simple APIs

**3. Bearer Tokens (JWT):**
- **Token-based**: Token-based
- **Stateless**: Stateless
- **Use**: Modern APIs

**4. OAuth 2.0:**
- **Delegated access**: Delegated access
- **Standard**: Industry standard
- **Use**: Third-party access

**5. Session-Based:**
- **Server-side**: Server-side sessions
- **Stateful**: Stateful
- **Use**: Web applications

---

## API Keys

### What are API Keys?

**API Key**: Unique identifier used to authenticate API requests.

**Characteristics:**
- **Static**: Usually static (doesn't change)
- **Simple**: Simple to use
- **Service-level**: Identifies service, not user

### How API Keys Work

**1. Generate Key:**
```
User registers for API
  ↓
System generates API key
  ↓
User receives key: "sk_live_abc123xyz"
```

**2. Use Key:**
```
Request → API
Headers:
  X-API-Key: sk_live_abc123xyz
  ↓
API verifies key
  ↓
If valid → Process request
```

**3. Verify Key:**
```python
def verify_api_key(api_key):
    # Check if key exists and is valid
    key_record = db.query("SELECT * FROM api_keys WHERE key = ?", api_key)
    if key_record and key_record.active:
        return True
    return False
```

### API Key Best Practices

**1. Store Securely:**
- **Environment variables**: Store in environment variables
- **Not in code**: Never in code
- **Encrypted**: Encrypt at rest

**2. Rotate Regularly:**
- **Periodic rotation**: Rotate periodically
- **Revoke compromised**: Revoke if compromised
- **Version keys**: Version keys

**3. Scope Keys:**
- **Limit permissions**: Limit what key can do
- **Read-only keys**: Separate read-only keys
- **Write keys**: Separate write keys

---

## Basic Authentication

### How Basic Auth Works

**Process:**
```
1. Client sends username:password
2. Encode in Base64
3. Send in Authorization header
4. Server decodes and verifies
```

**Example:**
```python
import base64

# Client
username = "alice"
password = "secret123"
credentials = f"{username}:{password}"
encoded = base64.b64encode(credentials.encode()).decode()

# Request
headers = {
    "Authorization": f"Basic {encoded}"
}

# Server
def verify_basic_auth(auth_header):
    # Extract credentials
    encoded = auth_header.replace("Basic ", "")
    credentials = base64.b64decode(encoded).decode()
    username, password = credentials.split(":")
    
    # Verify
    return verify_user(username, password)
```

### Basic Auth Limitations

**1. Not Secure Over HTTP:**
- **Base64 is not encryption**: Base64 is encoding, not encryption
- **Easily decoded**: Can be easily decoded
- **Must use HTTPS**: Must use HTTPS

**2. No Expiration:**
- **Static**: Credentials don't expire
- **If compromised**: If compromised, always compromised
- **No revocation**: Hard to revoke

**3. Not Suitable for APIs:**
- **User credentials**: Uses user credentials
- **Not ideal**: Not ideal for API access
- **Better alternatives**: Better alternatives exist

---

## Bearer Tokens (JWT)

### What is JWT?

**JWT (JSON Web Token)**: Compact, URL-safe token that contains claims about a user.

**Structure:**
```
Header.Payload.Signature
```

**Example:**
```
eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9.eyJzdWIiOiIxMjM0NTY3ODkwIiwibmFtZSI6IkpvaG4gRG9lIiwiaWF0IjoxNTE2MjM5MDIyfQ.SflKxwRJSMeKKF2QT4fwpMeJf36POk6yJV_adQssw5c
```

### JWT Structure

**1. Header:**
```json
{
  "alg": "HS256",
  "typ": "JWT"
}
```

**2. Payload:**
```json
{
  "sub": "1234567890",
  "name": "John Doe",
  "iat": 1516239022,
  "exp": 1516242622
}
```

**3. Signature:**
```
HMACSHA256(
  base64UrlEncode(header) + "." +
  base64UrlEncode(payload),
  secret
)
```

### JWT Flow

**1. Login:**
```
User → API: POST /login {username, password}
  ↓
API verifies credentials
  ↓
API generates JWT
  ↓
API → User: {token: "eyJ..."}
```

**2. Authenticated Request:**
```
User → API: GET /users/me
Headers: Authorization: Bearer eyJ...
  ↓
API verifies JWT signature
  ↓
API extracts claims
  ↓
API processes request
```

**3. JWT Verification:**
```python
import jwt

def verify_jwt(token, secret):
    try:
        payload = jwt.decode(token, secret, algorithms=["HS256"])
        return payload
    except jwt.ExpiredSignatureError:
        return None  # Token expired
    except jwt.InvalidTokenError:
        return None  # Invalid token
```

### JWT Advantages

**1. Stateless:**
- **No server storage**: No server-side storage
- **Scalable**: Easy to scale
- **Distributed**: Works in distributed systems

**2. Self-Contained:**
- **Contains claims**: Contains user claims
- **No database lookup**: No database lookup needed
- **Fast**: Fast verification

**3. Standard:**
- **Industry standard**: Industry standard
- **Widely supported**: Widely supported
- **Interoperable**: Interoperable

### JWT Disadvantages

**1. Cannot Revoke:**
- **No revocation**: Cannot revoke before expiration
- **Must wait**: Must wait for expiration
- **Blacklist needed**: Need blacklist for revocation

**2. Size:**
- **Larger than session ID**: Larger than session ID
- **Header overhead**: Header overhead
- **Bandwidth**: More bandwidth

**3. Security:**
- **Secret management**: Must protect secret
- **Algorithm choice**: Must choose secure algorithm
- **Token storage**: Must store securely on client

---

## OAuth 2.0

### What is OAuth 2.0?

**OAuth 2.0**: Authorization framework that allows third-party applications to obtain limited access to a service.

**Key Concepts:**
- **Resource Owner**: User (owns the data)
- **Client**: Application requesting access
- **Authorization Server**: Issues tokens
- **Resource Server**: API that has the data

### OAuth 2.0 Flow

**Basic Flow:**
```
1. Client requests authorization
2. User authorizes
3. Client receives authorization code
4. Client exchanges code for access token
5. Client uses access token to access API
```

### OAuth 2.0 Roles

**1. Resource Owner:**
- **User**: The user
- **Owns data**: Owns the data
- **Grants access**: Grants access

**2. Client:**
- **Application**: Third-party application
- **Requests access**: Requests access
- **Uses token**: Uses access token

**3. Authorization Server:**
- **Issues tokens**: Issues access tokens
- **Verifies credentials**: Verifies credentials
- **Manages tokens**: Manages token lifecycle

**4. Resource Server:**
- **API**: The API
- **Validates tokens**: Validates access tokens
- **Serves data**: Serves protected resources

---

## OAuth 2.0 Flows

### Flow 1: Authorization Code Flow

**Most Secure Flow:**
```
1. Client redirects user to authorization server
2. User authorizes
3. Authorization server redirects back with code
4. Client exchanges code for token (server-to-server)
5. Client uses token
```

**Use Case:**
- **Web applications**: Web applications
- **Most secure**: Most secure flow
- **Server-side**: Server-side applications

### Flow 2: Client Credentials Flow

**Service-to-Service:**
```
1. Client authenticates with client_id and client_secret
2. Authorization server issues token
3. Client uses token
```

**Use Case:**
- **Service-to-service**: Service-to-service
- **No user**: No user involved
- **Machine-to-machine**: Machine-to-machine

### Flow 3: Implicit Flow

**Less Secure (Deprecated):**
```
1. Client redirects user
2. User authorizes
3. Authorization server redirects back with token
4. Client uses token
```

**Use Case:**
- **Single-page apps**: Single-page applications
- **Less secure**: Less secure (deprecated)
- **Use PKCE**: Use Authorization Code with PKCE instead

### Flow 4: Resource Owner Password Credentials

**Direct Credentials:**
```
1. Client sends username/password
2. Authorization server issues token
3. Client uses token
```

**Use Case:**
- **Trusted clients**: Trusted clients only
- **Not recommended**: Not recommended for third-party
- **Legacy**: Legacy applications

---

## OpenID Connect (OIDC)

### What is OIDC?

**OpenID Connect**: Authentication layer on top of OAuth 2.0.

**Difference:**
- **OAuth 2.0**: Authorization (what can you do?)
- **OIDC**: Authentication (who are you?)

### OIDC Flow

**Similar to OAuth 2.0, but:**
```
1. Same OAuth 2.0 flow
2. Also receive ID token (JWT)
3. ID token contains user identity
4. Use ID token for authentication
```

**ID Token:**
```json
{
  "sub": "user123",
  "email": "user@example.com",
  "name": "John Doe",
  "iat": 1516239022,
  "exp": 1516242622
}
```

---

## Session-Based Authentication

### How Sessions Work

**1. Login:**
```
User → Server: POST /login {username, password}
  ↓
Server verifies credentials
  ↓
Server creates session
  ↓
Server → User: Set-Cookie: session_id=abc123
```

**2. Authenticated Request:**
```
User → Server: GET /users/me
Cookie: session_id=abc123
  ↓
Server looks up session
  ↓
Server verifies session
  ↓
Server processes request
```

**3. Session Storage:**
```python
# Server stores session
sessions = {
    "abc123": {
        "user_id": 123,
        "created_at": "2024-01-15T10:00:00Z",
        "expires_at": "2024-01-15T11:00:00Z"
    }
}
```

### Session Advantages

**1. Revocable:**
- **Can revoke**: Can revoke sessions
- **Server control**: Server has control
- **Immediate**: Immediate revocation

**2. Secure:**
- **Server-side**: Server-side storage
- **Not exposed**: Session ID not meaningful
- **HTTPS**: Use HTTPS for cookies

### Session Disadvantages

**1. Stateful:**
- **Server storage**: Requires server storage
- **Scaling**: Harder to scale
- **Sticky sessions**: Need sticky sessions

**2. CSRF:**
- **CSRF attacks**: Vulnerable to CSRF
- **CSRF tokens**: Need CSRF tokens
- **Same-site cookies**: Use SameSite cookies

---

## Token-Based Authentication

### How Token Auth Works

**1. Login:**
```
User → API: POST /login {username, password}
  ↓
API verifies credentials
  ↓
API generates token
  ↓
API → User: {token: "..."}
```

**2. Authenticated Request:**
```
User → API: GET /users/me
Headers: Authorization: Bearer ...
  ↓
API verifies token
  ↓
API processes request
```

**3. Token Storage:**
```python
# Tokens stored in database or cache
tokens = {
    "token_abc123": {
        "user_id": 123,
        "created_at": "2024-01-15T10:00:00Z",
        "expires_at": "2024-01-15T11:00:00Z"
    }
}
```

### Token vs Session

| Aspect | Session | Token |
|--------|---------|-------|
| **Storage** | Server-side | Client-side |
| **Stateful** | Yes | No (if JWT) |
| **Revocable** | Yes | Hard (if JWT) |
| **Scalability** | Harder | Easier |
| **CSRF** | Vulnerable | Not vulnerable |

---

## Authorization Models

### What is Authorization?

**Authorization**: Determining what a user is allowed to do.

**After Authentication:**
```
User authenticated (who they are)
  ↓
Check permissions (what they can do)
  ↓
Allow or deny
```

### Authorization Levels

**1. Resource-Level:**
- **Specific resource**: Specific resource
- **Example**: "Can user delete this file?"

**2. Action-Level:**
- **Specific action**: Specific action
- **Example**: "Can user delete files?"

**3. Role-Level:**
- **User role**: User role
- **Example**: "Is user admin?"

---

## Role-Based Access Control (RBAC)

### What is RBAC?

**RBAC**: Authorization model based on roles.

**Concepts:**
- **Roles**: Admin, User, Guest
- **Permissions**: Read, Write, Delete
- **Role-Permission mapping**: Roles have permissions

### RBAC Example

**Roles:**
```
Admin: [read, write, delete, manage_users]
User: [read, write]
Guest: [read]
```

**Implementation:**
```python
def check_permission(user, resource, action):
    role = get_user_role(user)
    permissions = get_role_permissions(role)
    return action in permissions

# Usage
if check_permission(user, "file", "delete"):
    delete_file()
else:
    return {"error": "Forbidden"}, 403
```

### RBAC Advantages

**1. Simple:**
- **Easy to understand**: Easy to understand
- **Common pattern**: Common pattern
- **Well-known**: Well-known

**2. Manageable:**
- **Role-based**: Manage by role
- **Not user-based**: Not per-user
- **Scalable**: Scalable

### RBAC Limitations

**1. Rigid:**
- **Fixed roles**: Fixed roles
- **Not flexible**: Not flexible
- **Hard to customize**: Hard to customize per user

**2. Role Explosion:**
- **Many roles**: Many roles needed
- **Complex**: Complex role hierarchy
- **Hard to manage**: Hard to manage

---

## Attribute-Based Access Control (ABAC)

### What is ABAC?

**ABAC**: Authorization model based on attributes.

**Concepts:**
- **Attributes**: User attributes, resource attributes, environment
- **Policies**: Rules based on attributes
- **Dynamic**: Dynamic authorization

### ABAC Example

**Policy:**
```
IF user.department == "Finance" 
   AND resource.type == "financial_data"
   AND time.hour >= 9 AND time.hour <= 17
THEN allow read
```

**Implementation:**
```python
def check_abac_policy(user, resource, action, environment):
    # Evaluate policy
    if (user.department == "Finance" and
        resource.type == "financial_data" and
        9 <= environment.time.hour <= 17):
        return True
    return False
```

### ABAC Advantages

**1. Flexible:**
- **Dynamic**: Dynamic policies
- **Fine-grained**: Fine-grained control
- **Context-aware**: Context-aware

**2. Scalable:**
- **Attribute-based**: Based on attributes
- **Not role-based**: Not limited to roles
- **Flexible**: Very flexible

### ABAC Disadvantages

**1. Complex:**
- **More complex**: More complex than RBAC
- **Policy management**: Policy management needed
- **Performance**: May be slower

**2. Hard to Understand:**
- **Complex policies**: Complex policies
- **Hard to debug**: Hard to debug
- **Requires expertise**: Requires expertise

---

## Best Practices

### 1. Always Use HTTPS

**Why:**
- **Encrypt credentials**: Encrypt credentials in transit
- **Prevent interception**: Prevent interception
- **Security**: Essential for security

### 2. Use Strong Secrets

**Why:**
- **JWT secrets**: Strong JWT secrets
- **API keys**: Strong API keys
- **Prevent brute force**: Prevent brute force

### 3. Implement Token Expiration

**Why:**
- **Limit exposure**: Limit exposure time
- **Security**: Better security
- **Revocation**: Natural revocation

### 4. Use Refresh Tokens

**Why:**
- **Long-lived sessions**: Long-lived sessions
- **Short access tokens**: Short access tokens
- **Balance**: Balance security and UX

### 5. Implement Rate Limiting

**Why:**
- **Prevent brute force**: Prevent brute force attacks
- **Protect resources**: Protect resources
- **Security**: Better security

### 6. Log Authentication Events

**Why:**
- **Audit trail**: Audit trail
- **Security monitoring**: Security monitoring
- **Debugging**: Easier debugging

---

## Common Security Issues

### Issue 1: Token in URL

**Problem:**
```
GET /api/data?token=abc123
  ↓
Token in URL
  ↓
Logged in server logs
  ↓
Security risk
```

**Solution:**
```
Use Authorization header
Authorization: Bearer abc123
```

### Issue 2: Weak Secrets

**Problem:**
```
JWT secret: "secret"
  ↓
Too weak
  ↓
Easily cracked
```

**Solution:**
```
Use strong, random secrets
At least 256 bits
```

### Issue 3: No Token Expiration

**Problem:**
```
Token never expires
  ↓
If compromised, always compromised
```

**Solution:**
```
Set expiration
Use refresh tokens
```

### Issue 4: Storing Tokens Insecurely

**Problem:**
```
Store token in localStorage
  ↓
Vulnerable to XSS
```

**Solution:**
```
Use httpOnly cookies
Or secure storage
```

---

## Summary

Authentication and authorization are fundamental for API security. Understanding different methods, OAuth 2.0, and authorization models is essential for building secure APIs.

**Key Takeaways:**
- **Authentication**: Verify who user is
- **Authorization**: Verify what user can do
- **Methods**: API keys, Basic, JWT, OAuth 2.0, Sessions
- **OAuth 2.0**: Industry standard for authorization
- **JWT**: Stateless tokens
- **RBAC**: Role-based authorization
- **ABAC**: Attribute-based authorization
- **Best practices**: HTTPS, strong secrets, expiration, rate limiting

**Authentication Methods:**
- **API Keys**: Simple, service-to-service
- **Basic Auth**: Simple, not secure over HTTP
- **JWT**: Stateless, self-contained
- **OAuth 2.0**: Delegated access
- **Sessions**: Stateful, revocable

**Authorization Models:**
- **RBAC**: Role-based, simple
- **ABAC**: Attribute-based, flexible

**Best Practices:**
- Always use HTTPS
- Use strong secrets
- Implement token expiration
- Use refresh tokens
- Implement rate limiting
- Log authentication events

**Common Issues:**
- Token in URL
- Weak secrets
- No expiration
- Insecure storage

**Next Steps:**
- Choose authentication method
- Implement authorization
- Add security measures
- Test thoroughly
- Monitor authentication

