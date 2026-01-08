# Go Production Best Practices Deep Dive - Complete Understanding

## Table of Contents
1. [What are Production Best Practices?](#what-are-production-best-practices)
2. [Why Production Best Practices Matter](#why-production-best-practices-matter)
3. [Application Configuration](#application-configuration)
4. [Logging and Observability](#logging-and-observability)
5. [Error Handling](#error-handling)
6. [Graceful Shutdown](#graceful-shutdown)
7. [Health Checks](#health-checks)
8. [Resource Management](#resource-management)
9. [Security](#security)
10. [Performance](#performance)
11. [Deployment](#deployment)
12. [Monitoring and Alerting](#monitoring-and-alerting)
13. [Best Practices](#best-practices)

---

## What are Production Best Practices?

### Definition

**Production Best Practices**: Practices and patterns for running Go applications in production.

**Key Characteristics:**
- **Reliability**: High reliability
- **Observability**: Full observability
- **Performance**: Optimal performance
- **Security**: Secure by default

### Real-World Analogy

**Production Best Practices = Building Code:**
- **Development**: Building construction
- **Production**: Building maintenance
- **Best practices**: Safety standards
- **Reliability**: Building reliability

**Programming:**
- **Application**: Go application
- **Production**: Production environment
- **Best practices**: Production practices
- **Reliability**: Application reliability

---

## Why Production Best Practices Matter?

### Benefits

**1. Reliability:**
```
Best practices
  ↓
Reliable application
  ↓
Fewer incidents
```

**2. Observability:**
```
Best practices
  ↓
Full observability
  ↓
Better debugging
```

**3. Performance:**
```
Best practices
  ↓
Optimal performance
  ↓
Better user experience
```

---

## Application Configuration

### Environment Variables

**Best Practice:**
```go
package main

import (
    "os"
    "strconv"
)

type Config struct {
    Port        int
    DatabaseURL string
    LogLevel    string
}

func LoadConfig() (*Config, error) {
    port, err := strconv.Atoi(os.Getenv("PORT"))
    if err != nil {
        port = 8080 // Default
    }
    
    return &Config{
        Port:        port,
        DatabaseURL: os.Getenv("DATABASE_URL"),
        LogLevel:    getEnvOrDefault("LOG_LEVEL", "info"),
    }, nil
}

func getEnvOrDefault(key, defaultValue string) string {
    if value := os.Getenv(key); value != "" {
        return value
    }
    return defaultValue
}
```

### Configuration Validation

**Best Practice:**
```go
func (c *Config) Validate() error {
    if c.Port < 1 || c.Port > 65535 {
        return errors.New("invalid port")
    }
    if c.DatabaseURL == "" {
        return errors.New("database URL required")
    }
    return nil
}
```

### Configuration Files

**Best Practice:**
```go
import "gopkg.in/yaml.v3"

type Config struct {
    Server   ServerConfig   `yaml:"server"`
    Database DatabaseConfig `yaml:"database"`
}

func LoadConfigFromFile(path string) (*Config, error) {
    data, err := os.ReadFile(path)
    if err != nil {
        return nil, err
    }
    
    var config Config
    if err := yaml.Unmarshal(data, &config); err != nil {
        return nil, err
    }
    
    return &config, config.Validate()
}
```

---

## Logging and Observability

### Structured Logging

**Best Practice:**
```go
import "github.com/sirupsen/logrus"

func setupLogger() *logrus.Logger {
    logger := logrus.New()
    logger.SetFormatter(&logrus.JSONFormatter{})
    logger.SetLevel(logrus.InfoLevel)
    return logger
}

func logRequest(logger *logrus.Logger, method, path string, status int, duration time.Duration) {
    logger.WithFields(logrus.Fields{
        "method":   method,
        "path":     path,
        "status":   status,
        "duration": duration,
    }).Info("request completed")
}
```

### Log Levels

**Best Practice:**
```go
func logWithLevel(logger *logrus.Logger, level string, message string, fields logrus.Fields) {
    entry := logger.WithFields(fields)
    switch level {
    case "debug":
        entry.Debug(message)
    case "info":
        entry.Info(message)
    case "warn":
        entry.Warn(message)
    case "error":
        entry.Error(message)
    }
}
```

### Context Logging

**Best Practice:**
```go
func logWithContext(ctx context.Context, logger *logrus.Logger, message string) {
    requestID := ctx.Value("request_id")
    logger.WithField("request_id", requestID).Info(message)
}
```

---

## Error Handling

### Error Wrapping

**Best Practice:**
```go
func processData(data []byte) error {
    if err := validateData(data); err != nil {
        return fmt.Errorf("validation failed: %w", err)
    }
    
    if err := saveData(data); err != nil {
        return fmt.Errorf("failed to save data: %w", err)
    }
    
    return nil
}
```

### Error Types

**Best Practice:**
```go
type ValidationError struct {
    Field   string
    Message string
}

func (e *ValidationError) Error() string {
    return fmt.Sprintf("validation error on %s: %s", e.Field, e.Message)
}

func IsValidationError(err error) bool {
    _, ok := err.(*ValidationError)
    return ok
}
```

### Error Logging

**Best Practice:**
```go
func handleError(logger *logrus.Logger, err error, context map[string]interface{}) {
    logger.WithFields(logrus.Fields{
        "error": err.Error(),
    }).WithFields(context).Error("operation failed")
}
```

---

## Graceful Shutdown

### Signal Handling

**Best Practice:**
```go
func main() {
    ctx, cancel := context.WithCancel(context.Background())
    defer cancel()
    
    sigChan := make(chan os.Signal, 1)
    signal.Notify(sigChan, os.Interrupt, syscall.SIGTERM)
    
    go func() {
        <-sigChan
        cancel()
    }()
    
    if err := run(ctx); err != nil {
        log.Fatal(err)
    }
}
```

### Graceful Shutdown Implementation

**Best Practice:**
```go
func run(ctx context.Context) error {
    server := &http.Server{
        Addr:    ":8080",
        Handler: setupRouter(),
    }
    
    go func() {
        if err := server.ListenAndServe(); err != nil && err != http.ErrServerClosed {
            log.Fatal(err)
        }
    }()
    
    <-ctx.Done()
    
    shutdownCtx, cancel := context.WithTimeout(context.Background(), 30*time.Second)
    defer cancel()
    
    return server.Shutdown(shutdownCtx)
}
```

### Resource Cleanup

**Best Practice:**
```go
func cleanupResources(logger *logrus.Logger) {
    logger.Info("cleaning up resources")
    
    // Close database connections
    if db != nil {
        db.Close()
    }
    
    // Close cache connections
    if cache != nil {
        cache.Close()
    }
    
    logger.Info("resources cleaned up")
}
```

---

## Health Checks

### Health Check Endpoint

**Best Practice:**
```go
func healthCheckHandler(db *sql.DB, cache *redis.Client) http.HandlerFunc {
    return func(w http.ResponseWriter, r *http.Request) {
        status := map[string]string{
            "status": "ok",
        }
        
        // Check database
        if err := db.Ping(); err != nil {
            status["database"] = "unhealthy"
            w.WriteHeader(http.StatusServiceUnavailable)
        } else {
            status["database"] = "healthy"
        }
        
        // Check cache
        if err := cache.Ping(context.Background()).Err(); err != nil {
            status["cache"] = "unhealthy"
            w.WriteHeader(http.StatusServiceUnavailable)
        } else {
            status["cache"] = "healthy"
        }
        
        json.NewEncoder(w).Encode(status)
    }
}
```

### Readiness Check

**Best Practice:**
```go
func readinessCheckHandler(ready *atomic.Bool) http.HandlerFunc {
    return func(w http.ResponseWriter, r *http.Request) {
        if ready.Load() {
            w.WriteHeader(http.StatusOK)
            w.Write([]byte("ready"))
        } else {
            w.WriteHeader(http.StatusServiceUnavailable)
            w.Write([]byte("not ready"))
        }
    }
}
```

---

## Resource Management

### Connection Pooling

**Best Practice:**
```go
func setupDatabase(url string) (*sql.DB, error) {
    db, err := sql.Open("postgres", url)
    if err != nil {
        return nil, err
    }
    
    db.SetMaxOpenConns(25)
    db.SetMaxIdleConns(5)
    db.SetConnMaxLifetime(5 * time.Minute)
    
    if err := db.Ping(); err != nil {
        return nil, err
    }
    
    return db, nil
}
```

### Context Timeouts

**Best Practice:**
```go
func processWithTimeout(ctx context.Context, timeout time.Duration) error {
    ctx, cancel := context.WithTimeout(ctx, timeout)
    defer cancel()
    
    return doWork(ctx)
}
```

### Resource Limits

**Best Practice:**
```go
func setupResourceLimits() {
    // Limit goroutines
    sem := make(chan struct{}, 100)
    
    for i := 0; i < 1000; i++ {
        sem <- struct{}{}
        go func() {
            defer func() { <-sem }()
            // Work
        }()
    }
}
```

---

## Security

### Input Validation

**Best Practice:**
```go
import "github.com/go-playground/validator/v10"

type User struct {
    Email    string `validate:"required,email"`
    Password string `validate:"required,min=8"`
}

func validateUser(u *User) error {
    validate := validator.New()
    return validate.Struct(u)
}
```

### Secure Headers

**Best Practice:**
```go
func securityHeaders(next http.Handler) http.Handler {
    return http.HandlerFunc(func(w http.ResponseWriter, r *http.Request) {
        w.Header().Set("X-Content-Type-Options", "nosniff")
        w.Header().Set("X-Frame-Options", "DENY")
        w.Header().Set("X-XSS-Protection", "1; mode=block")
        w.Header().Set("Strict-Transport-Security", "max-age=31536000")
        next.ServeHTTP(w, r)
    })
}
```

### Secrets Management

**Best Practice:**
```go
func loadSecrets() (map[string]string, error) {
    secrets := make(map[string]string)
    
    // Load from environment (preferred)
    if apiKey := os.Getenv("API_KEY"); apiKey != "" {
        secrets["api_key"] = apiKey
    }
    
    // Or load from secret manager
    // secrets["api_key"] = getSecretFromManager("api_key")
    
    return secrets, nil
}
```

---

## Performance

### Profiling

**Best Practice:**
```go
import _ "net/http/pprof"

func main() {
    go func() {
        log.Println(http.ListenAndServe("localhost:6060", nil))
    }()
    // Application code
}
```

### Caching

**Best Practice:**
```go
type Cache struct {
    data map[string]interface{}
    mu   sync.RWMutex
    ttl  time.Duration
}

func (c *Cache) Get(key string) (interface{}, bool) {
    c.mu.RLock()
    defer c.mu.RUnlock()
    return c.data[key], true
}

func (c *Cache) Set(key string, value interface{}) {
    c.mu.Lock()
    defer c.mu.Unlock()
    c.data[key] = value
}
```

### Connection Reuse

**Best Practice:**
```go
func setupHTTPClient() *http.Client {
    return &http.Client{
        Timeout: 30 * time.Second,
        Transport: &http.Transport{
            MaxIdleConns:        100,
            MaxIdleConnsPerHost: 10,
            IdleConnTimeout:     90 * time.Second,
        },
    }
}
```

---

## Deployment

### Build Optimization

**Best Practice:**
```bash
# Build flags
go build -ldflags="-s -w" -o app

# Cross-compilation
GOOS=linux GOARCH=amd64 go build -o app-linux
```

### Docker Best Practices

**Best Practice:**
```dockerfile
# Multi-stage build
FROM golang:1.21-alpine AS builder
WORKDIR /app
COPY . .
RUN go build -ldflags="-s -w" -o app

FROM alpine:latest
RUN apk --no-cache add ca-certificates
WORKDIR /root/
COPY --from=builder /app/app .
CMD ["./app"]
```

### Version Information

**Best Practice:**
```go
var (
    Version   = "unknown"
    BuildTime = "unknown"
    GitCommit = "unknown"
)

func versionHandler(w http.ResponseWriter, r *http.Request) {
    json.NewEncoder(w).Encode(map[string]string{
        "version":    Version,
        "build_time": BuildTime,
        "git_commit": GitCommit,
    })
}
```

---

## Monitoring and Alerting

### Metrics Collection

**Best Practice:**
```go
import "github.com/prometheus/client_golang/prometheus"

var (
    requestDuration = prometheus.NewHistogramVec(
        prometheus.HistogramOpts{
            Name: "http_request_duration_seconds",
            Help: "HTTP request duration",
        },
        []string{"method", "endpoint"},
    )
)

func init() {
    prometheus.MustRegister(requestDuration)
}
```

### Distributed Tracing

**Best Practice:**
```go
import "go.opentelemetry.io/otel/trace"

func tracingMiddleware(tracer trace.Tracer) func(http.Handler) http.Handler {
    return func(next http.Handler) http.Handler {
        return http.HandlerFunc(func(w http.ResponseWriter, r *http.Request) {
            ctx, span := tracer.Start(r.Context(), "http.request")
            defer span.End()
            next.ServeHTTP(w, r.WithContext(ctx))
        })
    }
}
```

---

## Best Practices

### 1. Configuration Management

**Why:**
- **Flexibility**: Easy configuration
- **Security**: Secure secrets
- **Environment**: Environment-specific

**Guidelines:**
- **Environment variables**: Use environment variables
- **Validation**: Validate configuration
- **Defaults**: Provide sensible defaults

### 2. Observability

**Why:**
- **Debugging**: Easy debugging
- **Monitoring**: Full monitoring
- **Alerting**: Proactive alerting

**Guidelines:**
- **Structured logging**: Use structured logging
- **Metrics**: Collect metrics
- **Tracing**: Use distributed tracing

### 3. Error Handling

**Why:**
- **Reliability**: More reliable
- **Debugging**: Easier debugging
- **User experience**: Better UX

**Guidelines:**
- **Wrap errors**: Wrap errors with context
- **Log errors**: Log errors appropriately
- **Handle gracefully**: Handle errors gracefully

### 4. Graceful Shutdown

**Why:**
- **Data integrity**: Preserve data
- **User experience**: Better UX
- **Resource cleanup**: Clean resources

**Guidelines:**
- **Signal handling**: Handle signals
- **Timeout**: Use shutdown timeout
- **Cleanup**: Clean up resources

### 5. Security

**Why:**
- **Protection**: Protect application
- **Compliance**: Meet compliance
- **Trust**: Build trust

**Guidelines:**
- **Input validation**: Validate all input
- **Secure headers**: Use security headers
- **Secrets management**: Manage secrets securely

---

## Summary

Production best practices ensure Go applications run reliably in production. Understanding application configuration, logging and observability, error handling, graceful shutdown, health checks, resource management, security, performance, deployment, monitoring and alerting, and best practices is crucial for production-ready applications.

**Key Takeaways:**
- **Production best practices**: Practices for production (reliability, observability, performance, security)
- **Application configuration**: Environment variables, configuration validation, configuration files
- **Logging and observability**: Structured logging, log levels, context logging
- **Error handling**: Error wrapping, error types, error logging
- **Graceful shutdown**: Signal handling, graceful shutdown implementation, resource cleanup
- **Health checks**: Health check endpoint, readiness check
- **Resource management**: Connection pooling, context timeouts, resource limits
- **Security**: Input validation, secure headers, secrets management
- **Performance**: Profiling, caching, connection reuse
- **Deployment**: Build optimization, Docker best practices, version information
- **Monitoring and alerting**: Metrics collection, distributed tracing
- **Best practices**: Configuration management, observability, error handling, graceful shutdown, security

**Production Requirements:**
- **Reliability**: High reliability
- **Observability**: Full observability
- **Performance**: Optimal performance
- **Security**: Secure by default

**Best Practices:**
- Configuration management
- Observability
- Error handling
- Graceful shutdown
- Security

**Next Steps:**
- Learn best practices
- Apply to projects
- Monitor and improve
- Share knowledge

