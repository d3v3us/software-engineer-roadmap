# Go Authorization Patterns Deep Dive - Complete Understanding

## Table of Contents
1. [What are Authorization Patterns in Go?](#what-are-authorization-patterns-in-go)
2. [Why Authorization Patterns Matter](#why-authorization-patterns-matter)
3. [RBAC Implementation](#rbac-implementation)
4. [ABAC Patterns](#abac-patterns)
5. [Permission Checking](#permission-checking)
6. [Implementation Patterns](#implementation-patterns)
7. [Best Practices](#best-practices)

---

## What are Authorization Patterns in Go?

### Definition

**Authorization Patterns**: Patterns for controlling access to resources based on user permissions.

**Key Characteristics:**
- **Access control**: Control access
- **Permissions**: Permission-based
- **Roles**: Role-based
- **Attributes**: Attribute-based

### Real-World Analogy

**Authorization = Access Card:**
- **User**: Person
- **Authorization**: Access card
- **Permissions**: Card permissions
- **Access**: Controlled access

**Programming:**
- **User**: Application user
- **Authorization**: Access control
- **Permissions**: User permissions
- **Resources**: Protected resources

---

## Why Authorization Patterns Matter?

### Benefits

**1. Security:**
```
Access control
  ↓
Authorization
  ↓
Secure access
```

**2. Fine-Grained Control:**
```
Permissions
  ↓
Authorization
  ↓
Fine-grained control
```

**3. Compliance:**
```
Compliance
  ↓
Authorization
  ↓
Meet requirements
```

---

## RBAC Implementation

### Role-Based Access Control

**RBAC structure:**
```go
type Role string

const (
    RoleAdmin    Role = "admin"
    RoleUser     Role = "user"
    RoleGuest    Role = "guest"
)

type Permission string

const (
    PermissionRead   Permission = "read"
    PermissionWrite  Permission = "write"
    PermissionDelete Permission = "delete"
)

type RolePermissions map[Role][]Permission

var rolePermissions = RolePermissions{
    RoleAdmin: {PermissionRead, PermissionWrite, PermissionDelete},
    RoleUser:  {PermissionRead, PermissionWrite},
    RoleGuest: {PermissionRead},
}

type User struct {
    ID    string
    Email string
    Role  Role
}

func (u *User) HasPermission(permission Permission) bool {
    permissions := rolePermissions[u.Role]
    for _, p := range permissions {
        if p == permission {
            return true
        }
    }
    return false
}
```

### RBAC Middleware

**Middleware:**
```go
func RBACMiddleware(requiredPermission Permission) func(http.Handler) http.Handler {
    return func(next http.Handler) http.Handler {
        return http.HandlerFunc(func(w http.ResponseWriter, r *http.Request) {
            user := getUserFromContext(r.Context())
            if user == nil {
                http.Error(w, "Unauthorized", http.StatusUnauthorized)
                return
            }
            
            if !user.HasPermission(requiredPermission) {
                http.Error(w, "Forbidden", http.StatusForbidden)
                return
            }
            
            next.ServeHTTP(w, r)
        })
    }
}
```

---

## ABAC Patterns

### Attribute-Based Access Control

**ABAC:**
```go
type Attribute struct {
    Key   string
    Value interface{}
}

type Policy struct {
    Subject    []Attribute
    Resource   []Attribute
    Action     string
    Condition  func(context.Context) bool
    Effect     string // "allow" or "deny"
}

type ABACEngine struct {
    policies []Policy
}

func (e *ABACEngine) CheckAccess(ctx context.Context, subject []Attribute, resource []Attribute, action string) bool {
    for _, policy := range e.policies {
        if e.matchesPolicy(ctx, policy, subject, resource, action) {
            return policy.Effect == "allow"
        }
    }
    return false
}

func (e *ABACEngine) matchesPolicy(ctx context.Context, policy Policy, subject, resource []Attribute, action string) bool {
    if !e.matchAttributes(policy.Subject, subject) {
        return false
    }
    if !e.matchAttributes(policy.Resource, resource) {
        return false
    }
    if policy.Action != action {
        return false
    }
    if policy.Condition != nil && !policy.Condition(ctx) {
        return false
    }
    return true
}
```

---

## Permission Checking

### Permission Checker

**Example:**
```go
type PermissionChecker interface {
    HasPermission(userID string, resource string, action string) (bool, error)
}

type SimplePermissionChecker struct {
    userPermissions map[string][]Permission
}

func (c *SimplePermissionChecker) HasPermission(userID, resource, action string) (bool, error) {
    permissions := c.userPermissions[userID]
    required := Permission{Resource: resource, Action: action}
    
    for _, p := range permissions {
        if p.Resource == required.Resource && p.Action == required.Action {
            return true, nil
        }
    }
    
    return false, nil
}
```

### Resource-Based Permissions

**Example:**
```go
type ResourcePermission struct {
    Resource string
    Action   string
    UserID   string
}

func CheckResourcePermission(userID, resource, action string) (bool, error) {
    // Check database
    var count int
    err := db.QueryRow(
        "SELECT COUNT(*) FROM permissions WHERE user_id = $1 AND resource = $2 AND action = $3",
        userID, resource, action,
    ).Scan(&count)
    
    if err != nil {
        return false, err
    }
    
    return count > 0, nil
}
```

---

## Implementation Patterns

### Pattern 1: Middleware Pattern

**Middleware:**
```go
func AuthorizationMiddleware(checker PermissionChecker) func(http.Handler) http.Handler {
    return func(next http.Handler) http.Handler {
        return http.HandlerFunc(func(w http.ResponseWriter, r *http.Request) {
            userID := getUserIDFromContext(r.Context())
            resource := extractResource(r)
            action := extractAction(r)
            
            allowed, err := checker.HasPermission(userID, resource, action)
            if err != nil || !allowed {
                http.Error(w, "Forbidden", http.StatusForbidden)
                return
            }
            
            next.ServeHTTP(w, r)
        })
    }
}
```

### Pattern 2: Decorator Pattern

**Decorator:**
```go
type AuthorizedService struct {
    service   Service
    checker   PermissionChecker
}

func (s *AuthorizedService) GetResource(ctx context.Context, resourceID string) (*Resource, error) {
    userID := getUserIDFromContext(ctx)
    
    allowed, err := s.checker.HasPermission(userID, "resource", "read")
    if err != nil || !allowed {
        return nil, ErrForbidden
    }
    
    return s.service.GetResource(ctx, resourceID)
}
```

---

## Best Practices

### 1. Principle of Least Privilege

**Why:**
- **Security**: Better security
- **Risk**: Lower risk
- **Compliance**: Compliance

**Guidelines:**
- **Minimal**: Grant minimal permissions
- **Review**: Review permissions regularly
- **Revoke**: Revoke unused permissions

### 2. Centralize Authorization Logic

**Why:**
- **Consistency**: Consistent authorization
- **Maintenance**: Easier maintenance
- **Testing**: Easier testing

**Guidelines:**
- **Centralize**: Centralize logic
- **Reuse**: Reuse authorization code
- **Test**: Test authorization

### 3. Cache Permission Checks

**Why:**
- **Performance**: Better performance
- **Efficiency**: More efficient
- **Scalability**: Better scalability

**Guidelines:**
- **Cache**: Cache permission checks
- **Invalidate**: Invalidate on changes
- **TTL**: Set appropriate TTL

### 4. Audit Authorization

**Why:**
- **Security**: Monitor security
- **Compliance**: Compliance
- **Debugging**: Easier debugging

**Guidelines:**
- **Log**: Log authorization decisions
- **Audit**: Audit access
- **Monitor**: Monitor patterns

---

## Summary

Authorization patterns enable access control in Go. Understanding RBAC implementation, ABAC patterns, permission checking, implementation patterns, and best practices is crucial for secure applications.

**Key Takeaways:**
- **Authorization patterns in Go**: Patterns for access control (access control, permissions, roles, attributes)
- **RBAC implementation**: Role-based access control (Role, Permission, RolePermissions, HasPermission), RBAC middleware (RBACMiddleware, check permission)
- **ABAC patterns**: Attribute-based access control (Attribute, Policy, ABACEngine, CheckAccess, matchesPolicy)
- **Permission checking**: Permission checker (PermissionChecker interface, HasPermission), resource-based permissions (CheckResourcePermission, database check)
- **Implementation patterns**: Middleware pattern (AuthorizationMiddleware, check permission), decorator pattern (AuthorizedService, check before service call)
- **Best practices**: Principle of least privilege, centralize authorization logic, cache permission checks, audit authorization

**Authorization Benefits:**
- **Security**: Secure access
- **Fine-grained control**: Fine-grained control
- **Compliance**: Meet requirements

**Best Practices:**
- Principle of least privilege
- Centralize authorization logic
- Cache permission checks
- Audit authorization

**Next Steps:**
- Learn RBAC
- Practice ABAC
- Implement permissions
- Apply best practices

