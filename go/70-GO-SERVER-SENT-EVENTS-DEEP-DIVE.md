# Go Server-Sent Events (SSE) Deep Dive - Complete Understanding

## Table of Contents
1. [What are Server-Sent Events?](#what-are-server-sent-events)
2. [Why SSE Matters](#why-sse-matters)
3. [SSE Protocol](#sse-protocol)
4. [SSE Server Implementation](#sse-server-implementation)
5. [SSE Client Implementation](#sse-client-implementation)
6. [Connection Management](#connection-management)
7. [Event Streaming](#event-streaming)
8. [Best Practices](#best-practices)

---

## What are Server-Sent Events?

### Definition

**Server-Sent Events (SSE)**: HTTP-based protocol for server-to-client real-time event streaming.

**Key Characteristics:**
- **One-way**: Server to client only
- **HTTP-based**: Uses HTTP
- **Real-time**: Real-time events
- **Simple**: Simple protocol

### Real-World Analogy

**SSE = Radio Broadcast:**
- **Server**: Radio station
- **Client**: Radio receiver
- **Events**: Broadcast messages
- **One-way**: One-way communication

**Programming:**
- **Server**: Event source
- **Client**: Event receiver
- **Events**: Real-time events
- **HTTP**: HTTP-based

---

## Why SSE Matters?

### Benefits

**1. Simplicity:**
```
Simple protocol
  ↓
SSE
  ↓
Easy to implement
```

**2. Real-Time Updates:**
```
Real-time data
  ↓
SSE
  ↓
Push updates
```

**3. HTTP Compatibility:**
```
HTTP-based
  ↓
SSE
  ↓
Works with HTTP
```

---

## SSE Protocol

### HTTP Headers

**Required headers:**
```
Content-Type: text/event-stream
Cache-Control: no-cache
Connection: keep-alive
```

### Event Format

**Event format:**
```
data: Hello World\n\n
```

**Multiple fields:**
```
event: message
id: 1
data: Hello World\n\n
```

---

## SSE Server Implementation

### Basic Server

**Example:**
```go
func handleSSE(w http.ResponseWriter, r *http.Request) {
    w.Header().Set("Content-Type", "text/event-stream")
    w.Header().Set("Cache-Control", "no-cache")
    w.Header().Set("Connection", "keep-alive")
    
    flusher, ok := w.(http.Flusher)
    if !ok {
        http.Error(w, "Streaming not supported", http.StatusInternalServerError)
        return
    }
    
    for i := 0; i < 10; i++ {
        fmt.Fprintf(w, "data: Message %d\n\n", i)
        flusher.Flush()
        time.Sleep(1 * time.Second)
    }
}

func main() {
    http.HandleFunc("/events", handleSSE)
    http.ListenAndServe(":8080", nil)
}
```

### Event Broadcasting

**Broadcast to multiple clients:**
```go
type SSEServer struct {
    clients map[chan []byte]bool
    mutex   sync.RWMutex
}

func (s *SSEServer) addClient(ch chan []byte) {
    s.mutex.Lock()
    s.clients[ch] = true
    s.mutex.Unlock()
}

func (s *SSEServer) removeClient(ch chan []byte) {
    s.mutex.Lock()
    delete(s.clients, ch)
    s.mutex.Unlock()
    close(ch)
}

func (s *SSEServer) broadcast(message []byte) {
    s.mutex.RLock()
    defer s.mutex.RUnlock()
    
    for ch := range s.clients {
        select {
        case ch <- message:
        default:
            // Client not ready, skip
        }
    }
}
```

---

## SSE Client Implementation

### JavaScript Client

**Example:**
```javascript
const eventSource = new EventSource('/events');

eventSource.onmessage = function(event) {
    console.log('Received:', event.data);
};

eventSource.onerror = function(event) {
    console.error('Error:', event);
};
```

### Go Client

**Example:**
```go
func connectSSE(url string) {
    resp, err := http.Get(url)
    if err != nil {
        log.Fatal(err)
    }
    defer resp.Body.Close()
    
    scanner := bufio.NewScanner(resp.Body)
    for scanner.Scan() {
        line := scanner.Text()
        if strings.HasPrefix(line, "data: ") {
            data := strings.TrimPrefix(line, "data: ")
            fmt.Println("Received:", data)
        }
    }
}
```

---

## Connection Management

### Keep-Alive

**Keep connection alive:**
```go
func handleSSE(w http.ResponseWriter, r *http.Request) {
    // Set headers
    w.Header().Set("Content-Type", "text/event-stream")
    w.Header().Set("Cache-Control", "no-cache")
    w.Header().Set("Connection", "keep-alive")
    
    // Send keep-alive comments
    ticker := time.NewTicker(30 * time.Second)
    defer ticker.Stop()
    
    go func() {
        for range ticker.C {
            fmt.Fprintf(w, ": keep-alive\n\n")
            if flusher, ok := w.(http.Flusher); ok {
                flusher.Flush()
            }
        }
    }()
    
    // Send events
}
```

### Reconnection

**Client reconnection:**
- **Automatic**: Automatic reconnection
- **Last-Event-ID**: Use Last-Event-ID header
- **Resume**: Resume from last event

---

## Event Streaming

### Event Types

**Simple event:**
```
data: Hello\n\n
```

**Event with type:**
```
event: message
data: Hello\n\n
```

**Event with ID:**
```
id: 1
data: Hello\n\n
```

**Multiple data lines:**
```
data: Line 1
data: Line 2\n\n
```

---

## Best Practices

### 1. Set Proper Headers

**Why:**
- **Protocol**: Protocol requirement
- **Compatibility**: Browser compatibility
- **Functionality**: Proper functionality

**Guidelines:**
- **Content-Type**: Set text/event-stream
- **Cache-Control**: Set no-cache
- **Connection**: Set keep-alive

### 2. Handle Client Disconnections

**Why:**
- **Resource management**: Better resource management
- **Performance**: Better performance
- **Reliability**: More reliable

**Guidelines:**
- **Detect**: Detect disconnections
- **Cleanup**: Clean up resources
- **Monitor**: Monitor connections

### 3. Use Keep-Alive

**Why:**
- **Connection**: Keep connection alive
- **Timeout**: Prevent timeouts
- **Reliability**: More reliable

**Guidelines:**
- **Comments**: Send keep-alive comments
- **Interval**: Regular interval
- **Timeout**: Handle timeouts

### 4. Implement Reconnection

**Why:**
- **Reliability**: More reliable
- **Resilience**: More resilient
- **User experience**: Better UX

**Guidelines:**
- **Automatic**: Automatic reconnection
- **Last-Event-ID**: Use Last-Event-ID
- **Resume**: Resume from last event

---

## Summary

Server-Sent Events enable real-time server-to-client event streaming in Go. Understanding SSE protocol, server/client implementation, connection management, event streaming, and best practices is crucial for real-time applications.

**Key Takeaways:**
- **Server-Sent Events**: HTTP-based protocol for server-to-client event streaming (one-way, HTTP-based, real-time, simple)
- **SSE protocol**: HTTP headers (Content-Type, Cache-Control, Connection), event format (data, event, id fields)
- **SSE server implementation**: Basic server (Set headers, Flusher, send events), event broadcasting (multiple clients, broadcast messages)
- **SSE client implementation**: JavaScript client (EventSource API), Go client (HTTP GET, parse events)
- **Connection management**: Keep-alive (send comments, prevent timeout), reconnection (automatic, Last-Event-ID, resume)
- **Event streaming**: Event types (simple, with type, with ID, multiple data lines)
- **Best practices**: Set proper headers, handle client disconnections, use keep-alive, implement reconnection

**SSE Benefits:**
- **Simplicity**: Simple protocol
- **Real-time**: Real-time updates
- **HTTP compatibility**: Works with HTTP

**Best Practices:**
- Set proper headers
- Handle client disconnections
- Use keep-alive
- Implement reconnection

**Next Steps:**
- Learn SSE protocol
- Practice server/client implementation
- Manage connections
- Apply best practices

