# Database Access Control Deep Dive

## Table of Contents
1. [What is Database Access Control?](#what-is-database-access-control)
2. [Why Database Access Control Matters](#why-database-access-control-matters)
3. [Access Control Models](#access-control-models)
4. [Implementation](#implementation)
5. [Best Practices](#best-practices)

---

## What is Database Access Control?

### Definition

**Database Access Control**: Controlling who can access what data.

**Key Concepts:**
- **Authentication**: Verify user identity
- **Authorization**: Control access permissions
- **Roles**: Role-based access
- **Permissions**: Access permissions

---

## Why Database Access Control Matters?

### Benefits

1. **Security**: Protect data from unauthorized access
2. **Compliance**: Meet compliance requirements
3. **Data protection**: Protect sensitive data
4. **Accountability**: User accountability

---

## Access Control Models

### Model 1: Role-Based Access Control (RBAC)

**What:**
```
Roles define permissions
  ↓
Users assigned roles
  ↓
Role-based access
```

### Model 2: Discretionary Access Control (DAC)

**What:**
```
Owner controls access
  ↓
Owner grants permissions
  ↓
User-based access
```

### Model 3: Mandatory Access Control (MAC)

**What:**
```
System-enforced access
  ↓
Security labels
  ↓
Policy-based access
```

---

## Implementation

### PostgreSQL RBAC

```sql
-- Create role
CREATE ROLE app_user;

-- Grant permissions
GRANT SELECT, INSERT, UPDATE ON users TO app_user;

-- Assign role
GRANT app_user TO user1;
```

---

## Best Practices

1. **Least privilege**: Grant minimum necessary permissions
2. **Role-based**: Use role-based access control
3. **Regular review**: Regularly review access permissions
4. **Audit**: Audit access control

---

## Summary

Database access control is essential for security. Implement RBAC, DAC, or MAC based on requirements.

