# Go HTTP Client and Server Deep Dive - Complete Understanding

## Table of Contents
1. [What is HTTP in Go?](#what-is-http-in-go)
2. [HTTP Server](#http-server)
3. [HTTP Client](#http-client)
4. [HTTP Patterns](#http-patterns)
5. [Best Practices](#best-practices)

---

## What is HTTP in Go?

### Definition

**HTTP**: Go's standard library provides comprehensive HTTP client and server functionality.

**Key Features:**
- **Server**: HTTP server implementation
- **Client**: HTTP client implementation
- **Standard library**: Built-in support
- **Production-ready**: Production-ready

---

## HTTP Server

### Basic Server

```go
import "net/http"

func handler(w http.ResponseWriter, r *http.Request) {
    fmt.Fprintf(w, "Hello, World!")
}

func main() {
    http.HandleFunc("/", handler)
    http.ListenAndServe(":8080", nil)
}
```

### Server with Mux

```go
mux := http.NewServeMux()
mux.HandleFunc("/", handler)
mux.HandleFunc("/api/users", usersHandler)

http.ListenAndServe(":8080", mux)
```

### Server with Middleware

```go
func loggingMiddleware(next http.Handler) http.Handler {
    return http.HandlerFunc(func(w http.ResponseWriter, r *http.Request) {
        log.Printf("%s %s", r.Method, r.URL.Path)
        next.ServeHTTP(w, r)
    })
}

mux := http.NewServeMux()
mux.HandleFunc("/", handler)
http.ListenAndServe(":8080", loggingMiddleware(mux))
```

---

## HTTP Client

### Basic Client

```go
import "net/http"

resp, err := http.Get("https://api.example.com/data")
if err != nil {
    // Handle error
}
defer resp.Body.Close()

body, err := io.ReadAll(resp.Body)
```

### Client with Request

```go
req, err := http.NewRequest("GET", "https://api.example.com/data", nil)
if err != nil {
    // Handle error
}

req.Header.Set("Authorization", "Bearer token")

client := &http.Client{}
resp, err := client.Do(req)
```

### Client with Timeout

```go
client := &http.Client{
    Timeout: 5 * time.Second,
}

resp, err := client.Get("https://api.example.com/data")
```

---

## HTTP Patterns

### Pattern 1: REST API Handler

```go
func usersHandler(w http.ResponseWriter, r *http.Request) {
    switch r.Method {
    case "GET":
        getUsers(w, r)
    case "POST":
        createUser(w, r)
    default:
        http.Error(w, "Method not allowed", http.StatusMethodNotAllowed)
    }
}
```

### Pattern 2: JSON Response

```go
func jsonResponse(w http.ResponseWriter, data interface{}) {
    w.Header().Set("Content-Type", "application/json")
    json.NewEncoder(w).Encode(data)
}
```

### Pattern 3: Context in Handlers

```go
func handler(w http.ResponseWriter, r *http.Request) {
    ctx := r.Context()
    
    // Use context in operations
    result, err := operation(ctx)
    if err != nil {
        http.Error(w, err.Error(), http.StatusInternalServerError)
        return
    }
    
    jsonResponse(w, result)
}
```

---

## Best Practices

### 1. Use Context for Cancellation

**Why:**
- **Cancellation**: Request cancellation
- **Timeouts**: Request timeouts
- **Resource cleanup**: Resource cleanup

**Guidelines:**
- **Context**: Use request context
- **Propagation**: Propagate context
- **Cancellation**: Handle cancellation

### 2. Handle Errors Properly

**Why:**
- **Reliability**: Reliable HTTP handling
- **User experience**: Better user experience
- **Debugging**: Easier debugging

**Guidelines:**
- **Check errors**: Always check errors
- **Appropriate status**: Use appropriate status codes
- **Error messages**: Clear error messages

### 3. Use Middleware

**Why:**
- **Cross-cutting**: Cross-cutting concerns
- **Reusability**: Reusable functionality
- **Separation**: Separation of concerns

**Guidelines:**
- **Middleware**: Use middleware for common functionality
- **Logging**: Logging middleware
- **Authentication**: Authentication middleware

---

## Summary

HTTP client and server are essential for web development in Go. Understanding HTTP server, client, patterns, and best practices is crucial for effective Go web development.

**Key Takeaways:**
- **HTTP in Go**: Standard library HTTP support (server, client, production-ready)
- **HTTP server**: Basic server, server with mux, server with middleware
- **HTTP client**: Basic client, client with request, client with timeout
- **HTTP patterns**: REST API handler, JSON response, context in handlers
- **Best practices**: Use context for cancellation, handle errors properly, use middleware

**HTTP Features:**
- **Server**: Full HTTP server support
- **Client**: Full HTTP client support
- **Standard library**: Built-in support

**Best Practices:**
- Use context for cancellation
- Handle errors properly
- Use middleware

**Next Steps:**
- Practice HTTP server
- Learn HTTP client
- Master HTTP patterns
- Apply best practices

