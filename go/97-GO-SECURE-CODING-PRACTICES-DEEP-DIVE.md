# Go Secure Coding Practices Deep Dive - Complete Understanding

## Table of Contents
1. [What are Secure Coding Practices?](#what-are-secure-coding-practices)
2. [Why Secure Coding Matters](#why-secure-coding-matters)
3. [Input Validation](#input-validation)
4. [SQL Injection Prevention](#sql-injection-prevention)
5. [XSS Prevention](#xss-prevention)
6. [CSRF Protection](#csrf-protection)
7. [Secure Defaults](#secure-defaults)
8. [Best Practices](#best-practices)

---

## What are Secure Coding Practices?

### Definition

**Secure Coding Practices**: Practices for writing secure code that prevents vulnerabilities.

**Key Characteristics:**
- **Security-first**: Security-first approach
- **Vulnerability prevention**: Prevents vulnerabilities
- **Best practices**: Follows best practices
- **Defense in depth**: Multiple layers

### Real-World Analogy

**Secure Coding = Security System:**
- **Building**: Application
- **Security system**: Secure coding
- **Layers**: Multiple security layers
- **Protection**: Comprehensive protection

**Programming:**
- **Application**: Go application
- **Secure coding**: Security practices
- **Vulnerabilities**: Prevent vulnerabilities
- **Protection**: Protect application

---

## Why Secure Coding Matters?

### Benefits

**1. Security:**
```
Vulnerabilities
  ↓
Secure coding
  ↓
Prevent vulnerabilities
```

**2. Trust:**
```
User trust
  ↓
Secure coding
  ↓
Build trust
```

**3. Compliance:**
```
Compliance
  ↓
Secure coding
  ↓
Meet requirements
```

---

## Input Validation

### Validation Library

**Installation:**
```bash
go get github.com/go-playground/validator/v10
```

### Input Validation

**Example:**
```go
import "github.com/go-playground/validator/v10"

type UserInput struct {
    Email    string `validate:"required,email"`
    Username string `validate:"required,min=3,max=20"`
    Age      int    `validate:"required,min=18,max=120"`
}

func validateInput(input UserInput) error {
    validate := validator.New()
    return validate.Struct(input)
}
```

### Sanitization

**Sanitization:**
```go
import "html"

func sanitizeInput(input string) string {
    // Remove HTML tags
    sanitized := html.EscapeString(input)
    return sanitized
}

func sanitizeSQL(input string) string {
    // Use parameterized queries instead
    // This is just for illustration
    return strings.ReplaceAll(input, "'", "''")
}
```

---

## SQL Injection Prevention

### Parameterized Queries

**Prepared statements:**
```go
func GetUser(db *sql.DB, userID string) (*User, error) {
    // Good: Parameterized query
    query := "SELECT id, email, name FROM users WHERE id = $1"
    row := db.QueryRow(query, userID)
    
    var user User
    err := row.Scan(&user.ID, &user.Email, &user.Name)
    if err != nil {
        return nil, err
    }
    
    return &user, nil
}
```

### Bad Example

**Bad:**
```go
// BAD: SQL injection vulnerability
func GetUserBad(db *sql.DB, userID string) (*User, error) {
    query := fmt.Sprintf("SELECT * FROM users WHERE id = '%s'", userID)
    // Vulnerable to SQL injection
    row := db.QueryRow(query)
    // ...
}
```

---

## XSS Prevention

### HTML Escaping

**Escaping:**
```go
import (
    "html"
    "html/template"
)

func renderTemplate(w http.ResponseWriter, data interface{}) error {
    tmpl := template.Must(template.New("page").Parse(`
        <div>{{.Content}}</div>
    `))
    
    // Template automatically escapes
    return tmpl.Execute(w, data)
}

func escapeHTML(input string) string {
    return html.EscapeString(input)
}
```

### Content Security Policy

**CSP:**
```go
func CSPMiddleware(next http.Handler) http.Handler {
    return http.HandlerFunc(func(w http.ResponseWriter, r *http.Request) {
        w.Header().Set("Content-Security-Policy",
            "default-src 'self'; script-src 'self' 'unsafe-inline';")
        next.ServeHTTP(w, r)
    })
}
```

---

## CSRF Protection

### CSRF Token

**CSRF protection:**
```go
import "github.com/gorilla/csrf"

func setupCSRF() func(http.Handler) http.Handler {
    return csrf.Protect(
        []byte("32-byte-long-auth-key"),
        csrf.Secure(false), // Set to true in production with HTTPS
    )
}

func handler(w http.ResponseWriter, r *http.Request) {
    token := csrf.Token(r)
    // Include token in form
    fmt.Fprintf(w, `<form method="POST">
        <input type="hidden" name="csrf_token" value="%s">
        <!-- form fields -->
    </form>`, token)
}
```

---

## Secure Defaults

### Secure Configuration

**Secure defaults:**
```go
type Config struct {
    HTTPSOnly    bool
    SecureCookie bool
    MaxAge       int
    SameSite     http.SameSite
}

func getSecureConfig() Config {
    return Config{
        HTTPSOnly:    true,  // Secure default
        SecureCookie: true,  // Secure default
        MaxAge:       3600,  // 1 hour
        SameSite:     http.SameSiteStrictMode,
    }
}
```

### Security Headers

**Headers:**
```go
func SecurityHeadersMiddleware(next http.Handler) http.Handler {
    return http.HandlerFunc(func(w http.ResponseWriter, r *http.Request) {
        w.Header().Set("X-Content-Type-Options", "nosniff")
        w.Header().Set("X-Frame-Options", "DENY")
        w.Header().Set("X-XSS-Protection", "1; mode=block")
        w.Header().Set("Strict-Transport-Security", "max-age=31536000")
        next.ServeHTTP(w, r)
    })
}
```

---

## Best Practices

### 1. Validate All Inputs

**Why:**
- **Security**: Prevent attacks
- **Correctness**: Ensure correctness
- **Reliability**: More reliable

**Guidelines:**
- **Validate**: Validate all inputs
- **Sanitize**: Sanitize when needed
- **Whitelist**: Use whitelist validation

### 2. Use Parameterized Queries

**Why:**
- **SQL injection**: Prevent SQL injection
- **Security**: Better security
- **Correctness**: Correct queries

**Guidelines:**
- **Prepared statements**: Always use prepared statements
- **Parameters**: Use parameters
- **Never**: Never concatenate SQL

### 3. Escape Output

**Why:**
- **XSS**: Prevent XSS
- **Security**: Better security
- **Protection**: Protect users

**Guidelines:**
- **Escape**: Escape all output
- **Templates**: Use safe templates
- **Context**: Escape for context

### 4. Use HTTPS

**Why:**
- **Encryption**: Encrypt data
- **Security**: Better security
- **Privacy**: Protect privacy

**Guidelines:**
- **HTTPS**: Always use HTTPS
- **TLS**: Use TLS 1.2+
- **Certificates**: Valid certificates

---

## Summary

Secure coding practices enable writing secure Go applications. Understanding input validation, SQL injection prevention, XSS prevention, CSRF protection, secure defaults, and best practices is crucial for application security.

**Key Takeaways:**
- **Secure coding practices**: Practices for secure code (security-first, vulnerability prevention, best practices, defense in depth)
- **Input validation**: Validation library (go-playground/validator), input validation (validate tags, Struct validation), sanitization (html.EscapeString, sanitizeInput)
- **SQL injection prevention**: Parameterized queries (prepared statements, QueryRow with parameters), bad example (SQL injection vulnerability, string concatenation)
- **XSS prevention**: HTML escaping (html.EscapeString, template escaping), Content Security Policy (CSP headers, CSPMiddleware)
- **CSRF protection**: CSRF token (gorilla/csrf, csrf.Protect, csrf.Token)
- **Secure defaults**: Secure configuration (HTTPSOnly, SecureCookie, SameSite), security headers (X-Content-Type-Options, X-Frame-Options, HSTS)
- **Best practices**: Validate all inputs, use parameterized queries, escape output, use HTTPS

**Secure Coding Benefits:**
- **Security**: Prevent vulnerabilities
- **Trust**: Build trust
- **Compliance**: Meet requirements

**Best Practices:**
- Validate all inputs
- Use parameterized queries
- Escape output
- Use HTTPS

**Next Steps:**
- Learn secure practices
- Practice validation
- Implement security
- Apply best practices

