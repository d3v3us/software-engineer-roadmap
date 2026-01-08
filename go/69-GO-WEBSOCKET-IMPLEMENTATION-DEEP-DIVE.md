# Go WebSocket Implementation Deep Dive - Complete Understanding

## Table of Contents
1. [What is WebSocket in Go?](#what-is-websocket-in-go)
2. [Why WebSocket Matters](#why-websocket-matters)
3. [WebSocket Protocol](#websocket-protocol)
4. [gorilla/websocket Package](#gorillawebsocket-package)
5. [WebSocket Server](#websocket-server)
6. [WebSocket Client](#websocket-client)
7. [Connection Management](#connection-management)
8. [Message Handling](#message-handling)
9. [Best Practices](#best-practices)

---

## What is WebSocket in Go?

### Definition

**WebSocket**: Full-duplex communication protocol over TCP, enabling real-time bidirectional communication.

**Key Characteristics:**
- **Full-duplex**: Bidirectional communication
- **Real-time**: Real-time data
- **Low overhead**: Low protocol overhead
- **Persistent**: Persistent connection

### Real-World Analogy

**WebSocket = Phone Call:**
- **HTTP**: Letters (request-response)
- **WebSocket**: Phone call (bidirectional)
- **Real-time**: Real-time communication
- **Persistent**: Persistent connection

**Programming:**
- **HTTP**: Request-response
- **WebSocket**: Bidirectional
- **Real-time**: Real-time updates
- **Efficient**: Efficient communication

---

## Why WebSocket Matters?

### Benefits

**1. Real-Time Communication:**
```
Real-time updates
  ↓
WebSocket
  ↓
Bidirectional communication
```

**2. Low Overhead:**
```
HTTP overhead
  ↓
WebSocket
  ↓
Lower overhead
```

**3. Efficiency:**
```
Multiple HTTP requests
  ↓
WebSocket
  ↓
Single connection
```

---

## WebSocket Protocol

### Handshake

**HTTP Upgrade:**
```
Client → Server: HTTP Upgrade request
Server → Client: HTTP 101 Switching Protocols
Connection upgraded to WebSocket
```

### Frame Format

**WebSocket frames:**
- **FIN**: Final frame flag
- **RSV**: Reserved bits
- **Opcode**: Frame type
- **Mask**: Masking flag
- **Payload**: Frame data

---

## gorilla/websocket Package

### Installation

```bash
go get github.com/gorilla/websocket
```

### Upgrader

**Upgrade HTTP to WebSocket:**
```go
import "github.com/gorilla/websocket"

var upgrader = websocket.Upgrader{
    CheckOrigin: func(r *http.Request) bool {
        return true  // Allow all origins (configure properly)
    },
}
```

---

## WebSocket Server

### Basic Server

**Example:**
```go
func handleWebSocket(w http.ResponseWriter, r *http.Request) {
    conn, err := upgrader.Upgrade(w, r, nil)
    if err != nil {
        log.Println(err)
        return
    }
    defer conn.Close()
    
    for {
        messageType, message, err := conn.ReadMessage()
        if err != nil {
            log.Println(err)
            break
        }
        
        log.Printf("Received: %s", message)
        
        err = conn.WriteMessage(messageType, message)
        if err != nil {
            log.Println(err)
            break
        }
    }
}

func main() {
    http.HandleFunc("/ws", handleWebSocket)
    http.ListenAndServe(":8080", nil)
}
```

### Connection Handling

**Handle multiple connections:**
```go
type Hub struct {
    clients    map[*websocket.Conn]bool
    broadcast  chan []byte
    register   chan *websocket.Conn
    unregister chan *websocket.Conn
    mutex      sync.RWMutex
}

func (h *Hub) run() {
    for {
        select {
        case conn := <-h.register:
            h.mutex.Lock()
            h.clients[conn] = true
            h.mutex.Unlock()
            
        case conn := <-h.unregister:
            h.mutex.Lock()
            delete(h.clients, conn)
            h.mutex.Unlock()
            conn.Close()
            
        case message := <-h.broadcast:
            h.mutex.RLock()
            for conn := range h.clients {
                err := conn.WriteMessage(websocket.TextMessage, message)
                if err != nil {
                    delete(h.clients, conn)
                    conn.Close()
                }
            }
            h.mutex.RUnlock()
        }
    }
}
```

---

## WebSocket Client

### Client Connection

**Example:**
```go
import "github.com/gorilla/websocket"

func main() {
    u := url.URL{Scheme: "ws", Host: "localhost:8080", Path: "/ws"}
    
    conn, _, err := websocket.DefaultDialer.Dial(u.String(), nil)
    if err != nil {
        log.Fatal(err)
    }
    defer conn.Close()
    
    // Send message
    err = conn.WriteMessage(websocket.TextMessage, []byte("Hello"))
    if err != nil {
        log.Println(err)
        return
    }
    
    // Read message
    _, message, err := conn.ReadMessage()
    if err != nil {
        log.Println(err)
        return
    }
    
    log.Printf("Received: %s", message)
}
```

---

## Connection Management

### Ping/Pong

**Keep connection alive:**
```go
conn.SetReadDeadline(time.Now().Add(60 * time.Second))
conn.SetPongHandler(func(string) error {
    conn.SetReadDeadline(time.Now().Add(60 * time.Second))
    return nil
})

ticker := time.NewTicker(54 * time.Second)
defer ticker.Stop()

for {
    select {
    case <-ticker.C:
        if err := conn.WriteMessage(websocket.PingMessage, nil); err != nil {
            return
        }
    }
}
```

### Error Handling

**Handle errors:**
```go
for {
    messageType, message, err := conn.ReadMessage()
    if err != nil {
        if websocket.IsUnexpectedCloseError(err, websocket.CloseGoingAway, websocket.CloseAbnormalClosure) {
            log.Printf("error: %v", err)
        }
        break
    }
    // Process message
}
```

---

## Message Handling

### Message Types

**Text messages:**
```go
conn.WriteMessage(websocket.TextMessage, []byte("Hello"))
```

**Binary messages:**
```go
conn.WriteMessage(websocket.BinaryMessage, data)
```

**JSON messages:**
```go
var msg Message
conn.ReadJSON(&msg)
conn.WriteJSON(msg)
```

---

## Best Practices

### 1. Handle Origins Properly

**Why:**
- **Security**: Security concern
- **CSRF**: Prevent CSRF
- **Access control**: Control access

**Guidelines:**
- **CheckOrigin**: Implement CheckOrigin
- **Whitelist**: Whitelist origins
- **Validate**: Validate origins

### 2. Manage Connections

**Why:**
- **Resource management**: Better resource management
- **Performance**: Better performance
- **Reliability**: More reliable

**Guidelines:**
- **Close**: Always close connections
- **Monitor**: Monitor connections
- **Limit**: Limit connections

### 3. Use Ping/Pong

**Why:**
- **Keep-alive**: Keep connection alive
- **Detection**: Detect dead connections
- **Reliability**: More reliable

**Guidelines:**
- **Ping**: Send ping regularly
- **Pong**: Handle pong
- **Timeout**: Set timeout

### 4. Handle Errors Gracefully

**Why:**
- **Robustness**: More robust
- **Reliability**: More reliable
- **User experience**: Better UX

**Guidelines:**
- **Check errors**: Always check errors
- **Handle**: Handle all errors
- **Recover**: Recover from errors

---

## Summary

WebSocket implementation enables real-time bidirectional communication in Go. Understanding WebSocket protocol, gorilla/websocket package, server/client implementation, connection management, message handling, and best practices is crucial for real-time applications.

**Key Takeaways:**
- **WebSocket in Go**: Full-duplex communication protocol (full-duplex, real-time, low overhead, persistent)
- **WebSocket protocol**: Handshake (HTTP Upgrade, 101 Switching Protocols), frame format (FIN, Opcode, Mask, Payload)
- **gorilla/websocket package**: Installation, upgrader (Upgrade HTTP to WebSocket)
- **WebSocket server**: Basic server (Upgrade, ReadMessage, WriteMessage), connection handling (Hub pattern, multiple connections)
- **WebSocket client**: Client connection (Dial, WriteMessage, ReadMessage)
- **Connection management**: Ping/Pong (keep-alive, SetPongHandler), error handling (IsUnexpectedCloseError)
- **Message handling**: Message types (TextMessage, BinaryMessage, JSON messages)
- **Best practices**: Handle origins properly, manage connections, use ping/pong, handle errors gracefully

**WebSocket Benefits:**
- **Real-time**: Real-time communication
- **Efficiency**: Efficient communication
- **Low overhead**: Low protocol overhead

**Best Practices:**
- Handle origins properly
- Manage connections
- Use ping/pong
- Handle errors gracefully

**Next Steps:**
- Learn WebSocket protocol
- Practice server/client implementation
- Manage connections
- Apply best practices

