# Health Checks Deep Dive

## Table of Contents
1. [What are Health Checks?](#what-are-health-checks)
2. [Why Health Checks Matter](#why-health-checks-matter)
3. [Health Check Types](#health-check-types)
4. [Health Check Implementation](#health-check-implementation)
5. [Health Check Best Practices](#health-check-best-practices)

---

## What are Health Checks?

### Definition

**Health Checks**: Mechanisms to verify service health and availability.

**Key Concepts:**
- **Health status**: Service health status
- **Availability**: Service availability
- **Readiness**: Service readiness
- **Liveness**: Service liveness

---

## Why Health Checks Matter?

### Benefits

1. **Service Discovery**: Remove unhealthy services
2. **Load Balancing**: Route traffic to healthy services
3. **Auto-scaling**: Scale based on health
4. **Monitoring**: Monitor service health

---

## Health Check Types

### Type 1: Liveness Check

**What:**
```
Is service alive?
  ↓
Process running
  ↓
Container running
```

**Use for:**
- **Restart**: Restart unhealthy services
- **Recovery**: Automatic recovery

### Type 2: Readiness Check

**What:**
```
Is service ready?
  ↓
Can accept traffic
  ↓
Dependencies ready
```

**Use for:**
- **Traffic routing**: Route traffic only to ready services
- **Startup**: Wait for service to be ready

### Type 3: Startup Check

**What:**
```
Is service started?
  ↓
Initialization complete
  ↓
Ready to start
```

**Use for:**
- **Startup**: Verify startup completion
- **Initialization**: Check initialization

---

## Health Check Implementation

### HTTP Health Check

```java
@RestController
public class HealthController {
    @GetMapping("/health")
    public ResponseEntity<Map<String, String>> health() {
        Map<String, String> status = new HashMap<>();
        status.put("status", "UP");
        return ResponseEntity.ok(status);
    }
    
    @GetMapping("/health/liveness")
    public ResponseEntity<Map<String, String>> liveness() {
        // Check if service is alive
        return ResponseEntity.ok(Map.of("status", "UP"));
    }
    
    @GetMapping("/health/readiness")
    public ResponseEntity<Map<String, String>> readiness() {
        // Check if service is ready
        if (isReady()) {
            return ResponseEntity.ok(Map.of("status", "UP"));
        }
        return ResponseEntity.status(503).body(Map.of("status", "DOWN"));
    }
}
```

### Kubernetes Health Checks

```yaml
apiVersion: v1
kind: Pod
spec:
  containers:
  - name: app
    livenessProbe:
      httpGet:
        path: /health/liveness
        port: 8080
      initialDelaySeconds: 30
      periodSeconds: 10
    readinessProbe:
      httpGet:
        path: /health/readiness
        port: 8080
      initialDelaySeconds: 5
      periodSeconds: 5
```

---

## Health Check Best Practices

1. **Separate endpoints**: Separate liveness and readiness
2. **Fast checks**: Keep health checks fast
3. **Dependency checks**: Check critical dependencies in readiness
4. **Timeout**: Set appropriate timeouts

---

## Summary

Health checks are essential for service reliability. Implement liveness, readiness, and startup checks appropriately.

