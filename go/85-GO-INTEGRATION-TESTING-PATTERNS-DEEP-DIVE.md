# Go Integration Testing Patterns Deep Dive - Complete Understanding

## Table of Contents
1. [What are Integration Testing Patterns?](#what-are-integration-testing-patterns)
2. [Why Integration Testing Patterns Matter](#why-integration-testing-patterns-matter)
3. [Integration Test Structure](#integration-test-structure)
4. [Test Containers](#test-containers)
5. [Mock Services](#mock-services)
6. [Test Databases](#test-databases)
7. [Common Patterns](#common-patterns)
8. [Best Practices](#best-practices)

---

## What are Integration Testing Patterns?

### Definition

**Integration Testing Patterns**: Patterns for testing how multiple components work together.

**Key Characteristics:**
- **Component interaction**: Tests component interaction
- **Real dependencies**: Uses real dependencies
- **End-to-end**: End-to-end scenarios
- **Realistic**: More realistic tests

### Real-World Analogy

**Integration Testing = System Test:**
- **Unit tests**: Component test
- **Integration tests**: System test
- **Interaction**: Tests interaction
- **Realistic**: Realistic scenarios

**Programming:**
- **Unit tests**: Single component
- **Integration tests**: Multiple components
- **Dependencies**: Real dependencies
- **Scenarios**: Real scenarios

---

## Why Integration Testing Patterns Matter?

### Benefits

**1. Real Scenarios:**
```
Real dependencies
  ↓
Integration tests
  ↓
Real scenarios
```

**2. Component Interaction:**
```
Multiple components
  ↓
Integration tests
  ↓
Test interaction
```

**3. Confidence:**
```
System works
  ↓
Integration tests
  ↓
More confidence
```

---

## Integration Test Structure

### Basic Structure

**Example:**
```go
func TestIntegration(t *testing.T) {
    // Setup
    db := setupTestDB(t)
    defer db.Close()
    
    service := NewService(db)
    
    // Test
    result, err := service.Process()
    if err != nil {
        t.Fatal(err)
    }
    
    // Verify
    if result != expected {
        t.Errorf("got %v, want %v", result, expected)
    }
}
```

### Test Helpers

**Helper functions:**
```go
func setupTestDB(t *testing.T) *sql.DB {
    db, err := sql.Open("postgres", testDBURL)
    if err != nil {
        t.Fatal(err)
    }
    
    // Run migrations
    runMigrations(t, db)
    
    return db
}

func teardownTestDB(t *testing.T, db *sql.DB) {
    // Cleanup
    db.Close()
}
```

---

## Test Containers

### Using testcontainers-go

**Installation:**
```bash
go get github.com/testcontainers/testcontainers-go
```

**Example:**
```go
import (
    "github.com/testcontainers/testcontainers-go"
    "github.com/testcontainers/testcontainers-go/wait"
)

func TestWithPostgres(t *testing.T) {
    ctx := context.Background()
    
    req := testcontainers.ContainerRequest{
        Image:        "postgres:15",
        ExposedPorts: []string{"5432/tcp"},
        Env: map[string]string{
            "POSTGRES_PASSWORD": "test",
            "POSTGRES_USER":     "test",
            "POSTGRES_DB":       "test",
        },
        WaitingFor: wait.ForLog("database system is ready"),
    }
    
    postgresC, err := testcontainers.GenericContainer(ctx, testcontainers.GenericContainerRequest{
        ContainerRequest: req,
        Started:          true,
    })
    if err != nil {
        t.Fatal(err)
    }
    defer postgresC.Terminate(ctx)
    
    // Use postgres container
}
```

---

## Mock Services

### HTTP Mock Server

**Example:**
```go
func TestWithMockService(t *testing.T) {
    // Start mock server
    server := httptest.NewServer(http.HandlerFunc(func(w http.ResponseWriter, r *http.Request) {
        w.WriteHeader(http.StatusOK)
        w.Write([]byte(`{"status": "ok"}`))
    }))
    defer server.Close()
    
    // Use mock server
    client := &http.Client{}
    resp, err := client.Get(server.URL)
    if err != nil {
        t.Fatal(err)
    }
    defer resp.Body.Close()
}
```

### gRPC Mock Server

**Example:**
```go
func TestWithMockGRPC(t *testing.T) {
    lis, err := net.Listen("tcp", ":0")
    if err != nil {
        t.Fatal(err)
    }
    
    s := grpc.NewServer()
    pb.RegisterServiceServer(s, &mockServer{})
    
    go s.Serve(lis)
    defer s.Stop()
    
    // Use mock gRPC server
}
```

---

## Test Databases

### In-Memory Database

**Example:**
```go
func TestWithInMemoryDB(t *testing.T) {
    db, err := sql.Open("sqlite3", ":memory:")
    if err != nil {
        t.Fatal(err)
    }
    defer db.Close()
    
    // Run migrations
    runMigrations(t, db)
    
    // Run tests
}
```

### Test Database Setup

**Setup:**
```go
func setupTestDatabase(t *testing.T) *sql.DB {
    db, err := sql.Open("postgres", testDBURL)
    if err != nil {
        t.Fatal(err)
    }
    
    // Create schema
    _, err = db.Exec(createSchemaSQL)
    if err != nil {
        t.Fatal(err)
    }
    
    return db
}

func cleanupTestDatabase(t *testing.T, db *sql.DB) {
    _, err := db.Exec("DROP SCHEMA public CASCADE; CREATE SCHEMA public;")
    if err != nil {
        t.Fatal(err)
    }
}
```

---

## Common Patterns

### Pattern 1: Test Fixtures

**Fixtures:**
```go
func loadTestFixtures(t *testing.T, db *sql.DB) {
    fixtures := []string{
        "INSERT INTO users (id, name) VALUES (1, 'Alice')",
        "INSERT INTO users (id, name) VALUES (2, 'Bob')",
    }
    
    for _, fixture := range fixtures {
        _, err := db.Exec(fixture)
        if err != nil {
            t.Fatal(err)
        }
    }
}
```

### Pattern 2: Test Isolation

**Isolation:**
```go
func TestIsolated(t *testing.T) {
    db := setupTestDB(t)
    defer cleanupTestDB(t, db)
    
    // Each test gets clean database
}
```

### Pattern 3: Parallel Testing

**Parallel:**
```go
func TestParallel(t *testing.T) {
    t.Parallel()
    
    // Test runs in parallel
}
```

---

## Best Practices

### 1. Isolate Tests

**Why:**
- **Independence**: Independent tests
- **Reliability**: More reliable
- **Debugging**: Easier debugging

**Guidelines:**
- **Isolation**: Isolate each test
- **Cleanup**: Clean up after tests
- **Fixtures**: Use test fixtures

### 2. Use Real Dependencies

**Why:**
- **Realistic**: More realistic
- **Confidence**: More confidence
- **Integration**: Real integration

**Guidelines:**
- **Real**: Use real dependencies
- **Containers**: Use test containers
- **Mocks**: Use mocks when needed

### 3. Fast Execution

**Why:**
- **Feedback**: Faster feedback
- **CI/CD**: CI/CD compatibility
- **Efficiency**: More efficient

**Guidelines:**
- **Fast**: Keep tests fast
- **Optimize**: Optimize setup
- **Parallel**: Run in parallel

### 4. Document Setup

**Why:**
- **Clarity**: Clear setup
- **Maintenance**: Easier maintenance
- **Understanding**: Better understanding

**Guidelines:**
- **Document**: Document setup
- **Comments**: Add comments
- **Examples**: Provide examples

---

## Summary

Integration testing patterns enable testing of component interactions. Understanding integration test structure, test containers, mock services, test databases, common patterns, and best practices is crucial for effective integration testing.

**Key Takeaways:**
- **Integration testing patterns**: Patterns for testing component interaction (component interaction, real dependencies, end-to-end, realistic)
- **Integration test structure**: Basic structure (setup, test, verify), test helpers (setupTestDB, teardownTestDB)
- **Test containers**: Using testcontainers-go (postgres container, setup, cleanup)
- **Mock services**: HTTP mock server (httptest.NewServer), gRPC mock server (mock server implementation)
- **Test databases**: In-memory database (sqlite3 :memory:), test database setup (setupTestDatabase, cleanupTestDatabase)
- **Common patterns**: Test fixtures (loadTestFixtures), test isolation (isolate each test), parallel testing (t.Parallel)
- **Best practices**: Isolate tests, use real dependencies, fast execution, document setup

**Integration Testing Benefits:**
- **Real scenarios**: Realistic scenarios
- **Component interaction**: Test interaction
- **Confidence**: More confidence

**Best Practices:**
- Isolate tests
- Use real dependencies
- Fast execution
- Document setup

**Next Steps:**
- Learn integration patterns
- Practice test setup
- Use test containers
- Apply best practices

