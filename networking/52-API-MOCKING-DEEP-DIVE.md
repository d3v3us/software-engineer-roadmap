# API Mocking Deep Dive - Complete Understanding

## Table of Contents
1. [What is API Mocking?](#what-is-api-mocking)
2. [Why API Mocking Matters](#why-api-mocking-matters)
3. [Mocking Approaches](#mocking-approaches)
4. [Mocking Tools](#mocking-tools)
5. [Mocking Implementation](#mocking-implementation)
6. [Best Practices](#best-practices)

---

## What is API Mocking?

### Definition

**API Mocking**: Creating fake API responses for testing and development.

**Key Concepts:**
- **Fake responses**: Simulated API responses
- **Testing**: Testing without real API
- **Development**: Development without dependencies
- **Isolation**: Isolated testing

### Real-World Analogy

**API Mocking = Movie Props:**
- **Real object**: Real API
- **Prop**: Mock API
- **Scene**: Test/development
- **Realism**: Realistic responses

**Development:**
- **Real API**: Production API
- **Mock API**: Mock service
- **Testing**: Test environment
- **Development**: Development environment

---

## Why API Mocking Matters?

### Impact of No Mocking

**1. Slow Development:**
```
Real API dependency
  ↓
Slow development
  ↓
Network delays
```

**2. Unreliable Tests:**
```
External dependency
  ↓
Flaky tests
  ↓
Network issues
```

**3. Cost:**
```
API costs
  ↓
Rate limits
  ↓
Resource usage
```

### Benefits of API Mocking

**1. Fast Development:**
- **No network**: No network calls
- **Fast execution**: Fast execution
- **Efficient**: Efficient development

**2. Reliable Testing:**
- **Controlled**: Controlled responses
- **Predictable**: Predictable behavior
- **Isolated**: Isolated tests

**3. Cost Savings:**
- **No API costs**: No API costs
- **No rate limits**: No rate limits
- **Resource efficient**: Resource efficient

---

## Mocking Approaches

### Approach 1: In-Code Mocking

**What:**
```
Mock in test code
  ↓
Test framework
  ↓
Code-based
```

**Use when:**
- **Unit tests**: Unit testing
- **Simple**: Simple mocking
- **Quick**: Quick setup

### Approach 2: Mock Server

**What:**
```
Standalone mock server
  ↓
HTTP server
  ↓
API simulation
```

**Use when:**
- **Integration tests**: Integration testing
- **Multiple consumers**: Multiple consumers
- **Persistent**: Persistent mocking

### Approach 3: Proxy Mocking

**What:**
```
Proxy server
  ↓
Intercept requests
  ↓
Return mocks
```

**Use when:**
- **Transparent**: Transparent mocking
- **No code changes**: No code changes
- **Easy**: Easy setup

---

## Mocking Tools

### Tool 1: WireMock

**What:**
```
Mock HTTP server
  ↓
Java-based
  ↓
Standalone or embedded
```

**Features:**
- **HTTP mocking**: HTTP request/response mocking
- **Standalone**: Standalone server
- **Embedded**: Embedded in tests
- **Record/playback**: Record and playback

### Tool 2: MockServer

**What:**
```
Mock server
  ↓
Java-based
  ↓
HTTP/HTTPS
```

**Features:**
- **HTTP mocking**: HTTP mocking
- **Expectations**: Set expectations
- **Verification**: Verify requests
- **Proxy**: Proxy mode

### Tool 3: JSON Server

**What:**
```
REST API mock
  ↓
JSON-based
  ↓
Quick setup
```

**Features:**
- **REST API**: REST API mocking
- **JSON**: JSON-based
- **Quick**: Quick setup
- **Simple**: Simple configuration

### Tool 4: Postman Mock Server

**What:**
```
Postman mock server
  ↓
Cloud-based
  ↓
Easy setup
```

**Features:**
- **Cloud**: Cloud-based
- **Easy**: Easy setup
- **Postman integration**: Postman integration
- **Collections**: Use Postman collections

---

## Mocking Implementation

### Implementation Example

**WireMock Example:**
```java
import com.github.tomakehurst.wiremock.WireMockServer;
import static com.github.tomakehurst.wiremock.client.WireMock.*;

public class ApiMockingExample {
    private WireMockServer wireMockServer;
    
    @Before
    public void setup() {
        wireMockServer = new WireMockServer(8080);
        wireMockServer.start();
        
        // Configure mock
        stubFor(get(urlEqualTo("/api/users/1"))
            .willReturn(aResponse()
                .withStatus(200)
                .withHeader("Content-Type", "application/json")
                .withBody("{\"id\":1,\"name\":\"John\",\"email\":\"john@example.com\"}")));
    }
    
    @After
    public void teardown() {
        wireMockServer.stop();
    }
    
    @Test
    public void testUserService() {
        // Test code uses mock API
        UserService service = new UserService("http://localhost:8080");
        User user = service.getUser(1L);
        
        assertEquals("John", user.getName());
        
        // Verify request
        verify(getRequestedFor(urlEqualTo("/api/users/1")));
    }
}
```

**JSON Server Example:**
```json
// db.json
{
  "users": [
    { "id": 1, "name": "John", "email": "john@example.com" },
    { "id": 2, "name": "Jane", "email": "jane@example.com" }
  ]
}
```

```bash
# Start mock server
json-server --watch db.json --port 3000
```

---

## Best Practices

### 1. Match Real API

**Why:**
- **Realism**: Realistic mocks
- **Compatibility**: API compatibility
- **Testing**: Better testing

**Guidelines:**
- **Same structure**: Same response structure
- **Same status codes**: Same status codes
- **Same headers**: Same headers

### 2. Handle Edge Cases

**Why:**
- **Comprehensive**: Comprehensive testing
- **Error handling**: Test error handling
- **Robustness**: Robust code

**Guidelines:**
- **Error responses**: Mock error responses
- **Edge cases**: Handle edge cases
- **Various scenarios**: Various scenarios

### 3. Version Mocks

**Why:**
- **API evolution**: API evolution
- **Compatibility**: Maintain compatibility
- **Testing**: Test different versions

**Guidelines:**
- **Version mocks**: Version mock responses
- **API versions**: Match API versions
- **Update mocks**: Update mocks with API

### 4. Document Mocks

**Why:**
- **Understanding**: Better understanding
- **Maintenance**: Easier maintenance
- **Onboarding**: Easier onboarding

**Guidelines:**
- **Document purpose**: Document mock purpose
- **Document structure**: Document response structure
- **Document scenarios**: Document test scenarios

---

## Summary

API mocking is essential for fast and reliable development and testing. Understanding mocking approaches, tools, implementation, and best practices is crucial for effective API mocking.

**Key Takeaways:**
- **API mocking**: Creating fake API responses for testing and development
- **Mocking approaches**: In-code mocking (test framework), mock server (standalone HTTP server), proxy mocking (intercept requests)
- **Mocking tools**: WireMock (HTTP mocking Java), MockServer (HTTP mocking Java), JSON Server (REST API JSON), Postman Mock Server (cloud-based)
- **Mocking implementation**: Configure mocks, set expectations, verify requests
- **Best practices**: Match real API, handle edge cases, version mocks, document mocks

**Mocking Approaches:**
- **In-Code**: Test framework
- **Mock Server**: Standalone server
- **Proxy**: Intercept requests

**Best Practices:**
- Match real API
- Handle edge cases
- Version mocks
- Document mocks

**Next Steps:**
- Understand mocking approaches
- Choose appropriate tool
- Implement mocking
- Apply best practices

