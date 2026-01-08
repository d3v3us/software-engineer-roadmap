# Go Logging Deep Dive - Complete Understanding

## Table of Contents
1. [What is Logging in Go?](#what-is-logging-in-go)
2. [Why Logging Matters](#why-logging-matters)
3. [Standard Library Logging](#standard-library-logging)
4. [Structured Logging](#structured-logging)
5. [Log Levels](#log-levels)
6. [Logging Best Practices](#logging-best-practices)
7. [Best Practices](#best-practices)

---

## What is Logging in Go?

### Definition

**Logging**: Recording application events and information for debugging, monitoring, and auditing.

**Key Characteristics:**
- **Event recording**: Record application events
- **Debugging**: Help with debugging
- **Monitoring**: Support monitoring
- **Auditing**: Enable auditing

### Real-World Analogy

**Logging = Journal:**
- **Events**: Daily events
- **Journal**: Log file
- **History**: Record of what happened
- **Reference**: Reference for future

**Programming:**
- **Events**: Application events
- **Logs**: Log entries
- **Debugging**: Debug information
- **Monitoring**: System monitoring

---

## Why Logging Matters?

### Benefits

**1. Debugging:**
```
Application issues
  ↓
Logs help debug
  ↓
Faster resolution
```

**2. Monitoring:**
```
System health
  ↓
Logs provide insights
  ↓
Better monitoring
```

**3. Auditing:**
```
Security events
  ↓
Logs record events
  ↓
Audit trail
```

---

## Standard Library Logging

### log Package

```go
import "log"

// Basic logging
log.Print("Message")
log.Println("Message with newline")
log.Printf("Formatted: %s", value)

// Fatal (logs and exits)
log.Fatal("Fatal error")

// Panic (logs and panics)
log.Panic("Panic error")
```

### log Package Configuration

```go
import "log"
import "os"

// Set output
log.SetOutput(os.Stdout)

// Set prefix
log.SetPrefix("APP: ")

// Set flags
log.SetFlags(log.Ldate | log.Ltime | log.Lshortfile)
```

---

## Structured Logging

### Using log/slog (Go 1.21+)

```go
import "log/slog"

// Text handler
logger := slog.New(slog.NewTextHandler(os.Stdout, nil))
logger.Info("User logged in", "user_id", 123)

// JSON handler
logger := slog.New(slog.NewJSONHandler(os.Stdout, nil))
logger.Info("User logged in", "user_id", 123)
```

### Using Third-Party Libraries

**Popular libraries:**
- **zerolog**: Fast structured logging
- **zap**: Uber's structured logging
- **logrus**: Structured logger

**Example with zerolog:**
```go
import "github.com/rs/zerolog/log"

log.Info().
    Str("user_id", "123").
    Str("action", "login").
    Msg("User logged in")
```

---

## Log Levels

### Standard Levels

**Common log levels:**
- **DEBUG**: Detailed debugging information
- **INFO**: General informational messages
- **WARN**: Warning messages
- **ERROR**: Error messages
- **FATAL**: Fatal errors (application exits)

### Using Log Levels

```go
import "log/slog"

logger := slog.New(slog.NewJSONHandler(os.Stdout, &slog.HandlerOptions{
    Level: slog.LevelInfo,
}))

logger.Debug("Debug message")  // Not logged (level too low)
logger.Info("Info message")     // Logged
logger.Warn("Warning message")  // Logged
logger.Error("Error message")   // Logged
```

---

## Logging Best Practices

### 1. Use Structured Logging

**Why:**
- **Searchable**: Easy to search
- **Parseable**: Easy to parse
- **Analyzable**: Easy to analyze

**Guidelines:**
- **JSON format**: Use JSON format
- **Key-value pairs**: Use key-value pairs
- **Consistent**: Keep structure consistent

### 2. Include Context

**Why:**
- **Traceability**: Better traceability
- **Debugging**: Easier debugging
- **Understanding**: Better understanding

**Guidelines:**
- **Request ID**: Include request ID
- **User ID**: Include user ID when relevant
- **Timestamps**: Include timestamps
- **Context**: Include relevant context

### 3. Use Appropriate Log Levels

**Why:**
- **Filtering**: Easy filtering
- **Monitoring**: Better monitoring
- **Debugging**: Easier debugging

**Guidelines:**
- **DEBUG**: Detailed debugging
- **INFO**: General information
- **WARN**: Warnings
- **ERROR**: Errors
- **FATAL**: Fatal errors only

### 4. Don't Log Sensitive Information

**Why:**
- **Security**: Security concerns
- **Privacy**: Privacy protection
- **Compliance**: Compliance requirements

**Guidelines:**
- **No passwords**: Never log passwords
- **No tokens**: Never log tokens
- **No PII**: Be careful with PII
- **Sanitize**: Sanitize sensitive data

---

## Best Practices

### 1. Centralize Logging

**Why:**
- **Consistency**: Consistent logging
- **Configuration**: Centralized configuration
- **Maintenance**: Easier maintenance

**Guidelines:**
- **Logger instance**: Use single logger instance
- **Configuration**: Centralize configuration
- **Wrapper**: Use wrapper if needed

### 2. Send Logs to External Services

**Why:**
- **Centralization**: Centralized logs
- **Analysis**: Better analysis
- **Monitoring**: Better monitoring

**Guidelines:**
- **Log aggregation**: Use log aggregation services
- **Cloud services**: Use cloud logging services
- **APIs**: Send logs via APIs

### 3. Monitor Log Volume

**Why:**
- **Performance**: Performance impact
- **Cost**: Storage costs
- **Noise**: Too much noise

**Guidelines:**
- **Volume**: Monitor log volume
- **Rate limiting**: Rate limit if needed
- **Sampling**: Use sampling for high-volume logs

---

## Summary

Logging is essential for debugging, monitoring, and auditing in Go. Understanding standard library logging, structured logging, log levels, and best practices is crucial for effective Go development.

**Key Takeaways:**
- **Logging**: Recording application events (debugging, monitoring, auditing)
- **Standard library logging**: log package (basic logging, configuration, fatal/panic)
- **Structured logging**: log/slog (Go 1.21+), third-party libraries (zerolog, zap, logrus)
- **Log levels**: DEBUG, INFO, WARN, ERROR, FATAL (appropriate usage)
- **Logging best practices**: Use structured logging, include context, use appropriate levels, don't log sensitive information
- **Best practices**: Centralize logging, send logs to external services, monitor log volume

**Logging Benefits:**
- **Debugging**: Faster issue resolution
- **Monitoring**: Better system monitoring
- **Auditing**: Security audit trail

**Best Practices:**
- Use structured logging
- Include context
- Use appropriate log levels
- Don't log sensitive information
- Centralize logging
- Send logs to external services
- Monitor log volume

**Next Steps:**
- Learn structured logging
- Practice log levels
- Set up log aggregation
- Apply best practices

