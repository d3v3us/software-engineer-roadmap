# Go Distributed Tracing Deep Dive - Complete Understanding

## Table of Contents
1. [What is Distributed Tracing in Go?](#what-is-distributed-tracing-in-go)
2. [Why Distributed Tracing Matters](#why-distributed-tracing-matters)
3. [OpenTelemetry](#opentelemetry)
4. [Trace Context Propagation](#trace-context-propagation)
5. [Sampling Strategies](#sampling-strategies)
6. [Implementation Patterns](#implementation-patterns)
7. [Best Practices](#best-practices)

---

## What is Distributed Tracing in Go?

### Definition

**Distributed Tracing**: Technique for tracking requests across multiple services in distributed systems.

**Key Characteristics:**
- **Request tracking**: Tracks requests
- **Service correlation**: Correlates services
- **Performance analysis**: Performance analysis
- **Debugging**: Easier debugging

### Real-World Analogy

**Distributed Tracing = Package Tracking:**
- **Package**: Request
- **Tracking**: Trace
- **Stops**: Services
- **Complete path**: Complete request path

**Programming:**
- **Request**: User request
- **Trace**: Distributed trace
- **Services**: Multiple services
- **Path**: Complete request path

---

## Why Distributed Tracing Matters?

### Benefits

**1. Request Visibility:**
```
Request path
  ↓
Distributed tracing
  ↓
Complete visibility
```

**2. Performance Analysis:**
```
Service performance
  ↓
Distributed tracing
  ↓
Performance analysis
```

**3. Debugging:**
```
Request issues
  ↓
Distributed tracing
  ↓
Easier debugging
```

---

## OpenTelemetry

### What is OpenTelemetry?

**OpenTelemetry**: Open standard for observability.

**Components:**
- **Tracing**: Distributed tracing
- **Metrics**: Metrics collection
- **Logs**: Logging

### Installation

**Installation:**
```bash
go get go.opentelemetry.io/otel
go get go.opentelemetry.io/otel/trace
go get go.opentelemetry.io/otel/exporters/jaeger
```

### Basic Setup

**Example:**
```go
import (
    "go.opentelemetry.io/otel"
    "go.opentelemetry.io/otel/exporters/jaeger"
    "go.opentelemetry.io/otel/sdk/trace"
)

func setupTracing() (*trace.TracerProvider, error) {
    exporter, err := jaeger.New(jaeger.WithCollectorEndpoint(
        jaeger.WithEndpoint("http://localhost:14268/api/traces"),
    ))
    if err != nil {
        return nil, err
    }
    
    tp := trace.NewTracerProvider(
        trace.WithBatcher(exporter),
        trace.WithResource(resource.NewWithAttributes(
            semconv.SchemaURL,
            semconv.ServiceNameKey.String("my-service"),
        )),
    )
    
    otel.SetTracerProvider(tp)
    return tp, nil
}
```

---

## Trace Context Propagation

### Context Propagation

**Propagation:**
```go
import "go.opentelemetry.io/otel/propagation"

func handleRequest(w http.ResponseWriter, r *http.Request) {
    ctx := r.Context()
    
    // Extract trace context
    propagator := propagation.TraceContext{}
    ctx = propagator.Extract(ctx, propagation.HeaderCarrier(r.Header))
    
    // Create span
    tracer := otel.Tracer("my-service")
    ctx, span := tracer.Start(ctx, "handleRequest")
    defer span.End()
    
    // Call downstream service
    callDownstream(ctx)
}

func callDownstream(ctx context.Context) {
    tracer := otel.Tracer("my-service")
    ctx, span := tracer.Start(ctx, "callDownstream")
    defer span.End()
    
    // Inject trace context
    propagator := propagation.TraceContext{}
    req, _ := http.NewRequestWithContext(ctx, "GET", "http://downstream", nil)
    propagator.Inject(ctx, propagation.HeaderCarrier(req.Header))
    
    // Make request
    http.DefaultClient.Do(req)
}
```

---

## Sampling Strategies

### Sampling Types

**1. Always Sample:**
- Sample all traces
- High overhead
- Complete data

**2. Never Sample:**
- Sample no traces
- No overhead
- No data

**3. Probabilistic:**
- Sample percentage
- Balanced
- Configurable

**4. Rate Limiting:**
- Sample by rate
- Controlled
- Efficient

### Sampling Implementation

**Example:**
```go
import "go.opentelemetry.io/otel/sdk/trace"

sampler := trace.ParentBased(trace.TraceIDRatioBased(0.1)) // 10% sampling

tp := trace.NewTracerProvider(
    trace.WithSampler(sampler),
    trace.WithBatcher(exporter),
)
```

---

## Implementation Patterns

### Pattern 1: HTTP Middleware

**Middleware:**
```go
func TracingMiddleware(next http.Handler) http.Handler {
    return http.HandlerFunc(func(w http.ResponseWriter, r *http.Request) {
        ctx := r.Context()
        propagator := propagation.TraceContext{}
        ctx = propagator.Extract(ctx, propagation.HeaderCarrier(r.Header))
        
        tracer := otel.Tracer("http")
        ctx, span := tracer.Start(ctx, r.Method+" "+r.URL.Path)
        defer span.End()
        
        // Add attributes
        span.SetAttributes(
            attribute.String("http.method", r.Method),
            attribute.String("http.url", r.URL.String()),
        )
        
        next.ServeHTTP(w, r.WithContext(ctx))
    })
}
```

### Pattern 2: gRPC Interceptor

**Interceptor:**
```go
func TracingInterceptor(ctx context.Context, req interface{}, info *grpc.UnaryServerInfo, handler grpc.UnaryHandler) (interface{}, error) {
    tracer := otel.Tracer("grpc")
    ctx, span := tracer.Start(ctx, info.FullMethod)
    defer span.End()
    
    span.SetAttributes(
        attribute.String("grpc.method", info.FullMethod),
    )
    
    return handler(ctx, req)
}
```

---

## Best Practices

### 1. Use OpenTelemetry

**Why:**
- **Standard**: Open standard
- **Vendor-neutral**: Vendor-neutral
- **Ecosystem**: Rich ecosystem

**Guidelines:**
- **OpenTelemetry**: Use OpenTelemetry
- **Standard**: Follow standard
- **Compatibility**: Ensure compatibility

### 2. Propagate Context

**Why:**
- **Correlation**: Request correlation
- **Complete trace**: Complete trace
- **Debugging**: Easier debugging

**Guidelines:**
- **Propagate**: Always propagate context
- **Headers**: Use standard headers
- **Verify**: Verify propagation

### 3. Use Sampling

**Why:**
- **Overhead**: Reduce overhead
- **Cost**: Reduce cost
- **Balance**: Balance data and cost

**Guidelines:**
- **Sampling**: Use sampling
- **Strategy**: Choose appropriate strategy
- **Monitor**: Monitor sampling

### 4. Add Meaningful Attributes

**Why:**
- **Debugging**: Easier debugging
- **Analysis**: Better analysis
- **Filtering**: Better filtering

**Guidelines:**
- **Attributes**: Add meaningful attributes
- **Standard**: Use standard attributes
- **Relevant**: Add relevant information

---

## Summary

Distributed tracing enables tracking requests across services in Go. Understanding OpenTelemetry, trace context propagation, sampling strategies, implementation patterns, and best practices is crucial for observability.

**Key Takeaways:**
- **Distributed tracing in Go**: Technique for tracking requests (request tracking, service correlation, performance analysis, debugging)
- **OpenTelemetry**: Open standard for observability (tracing, metrics, logs), installation, basic setup (exporter, TracerProvider)
- **Trace context propagation**: Context propagation (Extract, Inject, propagation.TraceContext), HTTP propagation, gRPC propagation
- **Sampling strategies**: Sampling types (always sample, never sample, probabilistic, rate limiting), sampling implementation (TraceIDRatioBased, ParentBased)
- **Implementation patterns**: HTTP middleware (TracingMiddleware, extract context, create span), gRPC interceptor (TracingInterceptor, span creation)
- **Best practices**: Use OpenTelemetry, propagate context, use sampling, add meaningful attributes

**Distributed Tracing Benefits:**
- **Request visibility**: Complete visibility
- **Performance analysis**: Performance analysis
- **Debugging**: Easier debugging

**Best Practices:**
- Use OpenTelemetry
- Propagate context
- Use sampling
- Add meaningful attributes

**Next Steps:**
- Learn OpenTelemetry
- Practice context propagation
- Implement tracing
- Apply best practices

