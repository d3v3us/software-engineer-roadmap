# Go Authentication Patterns Deep Dive - Complete Understanding

## Table of Contents
1. [What are Authentication Patterns in Go?](#what-are-authentication-patterns-in-go)
2. [Why Authentication Patterns Matter](#why-authentication-patterns-matter)
3. [JWT Implementation](#jwt-implementation)
4. [OAuth2](#oauth2)
5. [Session Management](#session-management)
6. [Token Refresh](#token-refresh)
7. [Best Practices](#best-practices)

---

## What are Authentication Patterns in Go?

### Definition

**Authentication Patterns**: Patterns for verifying user identity in Go applications.

**Key Characteristics:**
- **Identity verification**: Verify user identity
- **Security**: Secure authentication
- **Token-based**: Token-based auth
- **Session-based**: Session-based auth

### Real-World Analogy

**Authentication = ID Check:**
- **User**: Person
- **Authentication**: ID check
- **Credentials**: ID card
- **Verification**: Verify identity

**Programming:**
- **User**: Application user
- **Authentication**: Verify identity
- **Credentials**: Username/password, tokens
- **Verification**: Authentication process

---

## Why Authentication Patterns Matter?

### Benefits

**1. Security:**
```
User identity
  ↓
Authentication
  ↓
Secure access
```

**2. Access Control:**
```
Access control
  ↓
Authentication
  ↓
Controlled access
```

**3. User Experience:**
```
User experience
  ↓
Authentication
  ↓
Smooth UX
```

---

## JWT Implementation

### JWT Library

**Installation:**
```bash
go get github.com/golang-jwt/jwt/v5
```

### JWT Generation

**Example:**
```go
import "github.com/golang-jwt/jwt/v5"

type Claims struct {
    UserID string `json:"user_id"`
    Email  string `json:"email"`
    jwt.RegisteredClaims
}

func GenerateToken(userID, email string, secret []byte) (string, error) {
    claims := Claims{
        UserID: userID,
        Email:  email,
        RegisteredClaims: jwt.RegisteredClaims{
            ExpiresAt: jwt.NewNumericDate(time.Now().Add(24 * time.Hour)),
            IssuedAt:  jwt.NewNumericDate(time.Now()),
            Issuer:    "my-app",
        },
    }
    
    token := jwt.NewWithClaims(jwt.SigningMethodHS256, claims)
    return token.SignedString(secret)
}
```

### JWT Validation

**Example:**
```go
func ValidateToken(tokenString string, secret []byte) (*Claims, error) {
    token, err := jwt.ParseWithClaims(tokenString, &Claims{}, func(token *jwt.Token) (interface{}, error) {
        if _, ok := token.Method.(*jwt.SigningMethodHMAC); !ok {
            return nil, fmt.Errorf("unexpected signing method: %v", token.Header["alg"])
        }
        return secret, nil
    })
    
    if err != nil {
        return nil, err
    }
    
    if claims, ok := token.Claims.(*Claims); ok && token.Valid {
        return claims, nil
    }
    
    return nil, fmt.Errorf("invalid token")
}
```

---

## OAuth2

### OAuth2 Client

**Installation:**
```bash
go get golang.org/x/oauth2
```

### OAuth2 Implementation

**Example:**
```go
import "golang.org/x/oauth2"

var oauth2Config = oauth2.Config{
    ClientID:     "client-id",
    ClientSecret: "client-secret",
    RedirectURL:  "http://localhost:8080/callback",
    Scopes:       []string{"read", "write"},
    Endpoint: oauth2.Endpoint{
        AuthURL:  "https://provider.com/oauth/authorize",
        TokenURL: "https://provider.com/oauth/token",
    },
}

func handleLogin(w http.ResponseWriter, r *http.Request) {
    url := oauth2Config.AuthCodeURL("state", oauth2.AccessTypeOffline)
    http.Redirect(w, r, url, http.StatusFound)
}

func handleCallback(w http.ResponseWriter, r *http.Request) {
    code := r.URL.Query().Get("code")
    token, err := oauth2Config.Exchange(r.Context(), code)
    if err != nil {
        http.Error(w, err.Error(), http.StatusInternalServerError)
        return
    }
    
    // Use token
}
```

---

## Session Management

### Session Store

**Example:**
```go
import (
    "github.com/gorilla/sessions"
    "net/http"
)

var store = sessions.NewCookieStore([]byte("secret-key"))

func handleLogin(w http.ResponseWriter, r *http.Request) {
    session, _ := store.Get(r, "session-name")
    session.Values["user_id"] = userID
    session.Values["email"] = email
    session.Save(r, w)
}

func handleProtected(w http.ResponseWriter, r *http.Request) {
    session, _ := store.Get(r, "session-name")
    userID := session.Values["user_id"]
    if userID == nil {
        http.Redirect(w, r, "/login", http.StatusFound)
        return
    }
    // Handle protected resource
}
```

### Redis Session Store

**Example:**
```go
import "github.com/boj/redistore"

store, err := redistore.NewRediStore(10, "tcp", ":6379", "", []byte("secret-key"))
if err != nil {
    log.Fatal(err)
}
defer store.Close()
```

---

## Token Refresh

### Refresh Token Implementation

**Example:**
```go
type TokenPair struct {
    AccessToken  string
    RefreshToken string
    ExpiresIn    int
}

func RefreshAccessToken(refreshToken string, secret []byte) (*TokenPair, error) {
    // Validate refresh token
    claims, err := ValidateToken(refreshToken, secret)
    if err != nil {
        return nil, err
    }
    
    // Generate new access token
    accessToken, err := GenerateToken(claims.UserID, claims.Email, secret)
    if err != nil {
        return nil, err
    }
    
    return &TokenPair{
        AccessToken:  accessToken,
        RefreshToken: refreshToken,
        ExpiresIn:    3600,
    }, nil
}
```

### Token Refresh Middleware

**Example:**
```go
func TokenRefreshMiddleware(next http.Handler) http.Handler {
    return http.HandlerFunc(func(w http.ResponseWriter, r *http.Request) {
        token := extractToken(r)
        if token == "" {
            next.ServeHTTP(w, r)
            return
        }
        
        claims, err := ValidateToken(token, secret)
        if err != nil && err.Error() == "token is expired" {
            refreshToken := extractRefreshToken(r)
            if refreshToken != "" {
                newToken, err := RefreshAccessToken(refreshToken, secret)
                if err == nil {
                    w.Header().Set("Authorization", "Bearer "+newToken.AccessToken)
                }
            }
        }
        
        next.ServeHTTP(w, r)
    })
}
```

---

## Best Practices

### 1. Use Secure Tokens

**Why:**
- **Security**: Better security
- **Protection**: Token protection
- **Safety**: Safer authentication

**Guidelines:**
- **Strong secrets**: Use strong secrets
- **Expiration**: Set expiration
- **HTTPS**: Use HTTPS

### 2. Store Tokens Securely

**Why:**
- **Security**: Token security
- **Protection**: Protect tokens
- **Safety**: Safer storage

**Guidelines:**
- **HttpOnly**: Use HttpOnly cookies
- **Secure**: Use Secure flag
- **Storage**: Secure storage

### 3. Implement Token Refresh

**Why:**
- **Security**: Better security
- **User experience**: Better UX
- **Tokens**: Shorter-lived tokens

**Guidelines:**
- **Refresh tokens**: Use refresh tokens
- **Automatic**: Automatic refresh
- **Expiration**: Short access token expiration

### 4. Monitor Authentication

**Why:**
- **Security**: Monitor security
- **Issues**: Identify issues
- **Performance**: Monitor performance

**Guidelines:**
- **Metrics**: Track auth metrics
- **Alerts**: Set up alerts
- **Logging**: Log auth events

---

## Summary

Authentication patterns enable secure user authentication in Go. Understanding JWT implementation, OAuth2, session management, token refresh, and best practices is crucial for secure applications.

**Key Takeaways:**
- **Authentication patterns in Go**: Patterns for verifying identity (identity verification, security, token-based, session-based)
- **JWT implementation**: JWT library (golang-jwt/jwt), JWT generation (GenerateToken, Claims, SignedString), JWT validation (ValidateToken, ParseWithClaims)
- **OAuth2**: OAuth2 client (golang.org/x/oauth2), OAuth2 implementation (Config, AuthCodeURL, Exchange)
- **Session management**: Session store (gorilla/sessions, CookieStore), Redis session store (redistore)
- **Token refresh**: Refresh token implementation (RefreshAccessToken, validate refresh token, generate new access token), token refresh middleware (TokenRefreshMiddleware, automatic refresh)
- **Best practices**: Use secure tokens, store tokens securely, implement token refresh, monitor authentication

**Authentication Benefits:**
- **Security**: Secure access
- **Access control**: Controlled access
- **User experience**: Smooth UX

**Best Practices:**
- Use secure tokens
- Store tokens securely
- Implement token refresh
- Monitor authentication

**Next Steps:**
- Learn JWT
- Practice OAuth2
- Implement sessions
- Apply best practices

