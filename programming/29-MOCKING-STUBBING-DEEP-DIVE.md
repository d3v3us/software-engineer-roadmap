# Mocking and Stubbing Deep Dive - Complete Understanding

## Table of Contents
1. [What are Mocks and Stubs?](#what-are-mocks-and-stubs)
2. [Why Mocking and Stubbing Matter](#why-mocking-and-stubbing-matter)
3. [Test Doubles](#test-doubles)
4. [Mocking](#mocking)
5. [Stubbing](#stubbing)
6. [Spies and Fakes](#spies-and-fakes)
7. [Best Practices](#best-practices)

---

## What are Mocks and Stubs?

### Definition

**Mocks and Stubs**: Test doubles that replace real dependencies in tests.

**Key Concepts:**
- **Test doubles**: Replace real objects
- **Isolation**: Isolate code under test
- **Control**: Control dependencies
- **Verification**: Verify interactions

### Real-World Analogy

**Mocks and Stubs = Movie Stunt Doubles:**
- **Actor**: Real dependency
- **Stunt double**: Mock/stub
- **Scene**: Test
- **Safety**: Safe testing

**Testing:**
- **Real dependency**: Real service
- **Mock/stub**: Test double
- **Test**: Unit test
- **Isolation**: Isolated testing

---

## Why Mocking and Stubbing Matter?

### Impact of No Mocking

**1. Slow Tests:**
```
Real dependencies
  ↓
Slow execution
  ↓
Network calls
  ↓
Database calls
```

**2. Flaky Tests:**
```
External dependencies
  ↓
Unreliable tests
  ↓
Network issues
  ↓
Service failures
```

**3. Hard to Test:**
```
Complex dependencies
  ↓
Hard to set up
  ↓
Hard to control
  ↓
Hard to verify
```

### Benefits of Mocking

**1. Fast Tests:**
- **No I/O**: No I/O operations
- **Fast execution**: Fast test execution
- **Efficient**: Efficient testing

**2. Reliable Tests:**
- **Controlled**: Controlled behavior
- **Predictable**: Predictable results
- **Isolated**: Isolated tests

**3. Easy Testing:**
- **Easy setup**: Easy test setup
- **Easy control**: Easy to control
- **Easy verification**: Easy to verify

---

## Test Doubles

### Types of Test Doubles

**1. Dummy:**
```
Placeholder object
  ↓
Not used
  ↓
Fill parameter
```

**2. Fake:**
```
Working implementation
  ↓
Simplified
  ↓
In-memory
```

**3. Stub:**
```
Returns predefined data
  ↓
No verification
  ↓
State-based
```

**4. Mock:**
```
Verifies interactions
  ↓
Behavior verification
  ↓
Interaction-based
```

**5. Spy:**
```
Real object wrapper
  ↓
Records interactions
  ↓
Verification
```

---

## Mocking

### What is Mocking?

**Mocking**: Creating objects that verify interactions.

**Characteristics:**
- **Interaction verification**: Verifies method calls
- **Behavior-based**: Behavior-based testing
- **Expectations**: Set expectations
- **Verification**: Verify interactions

### Mocking Example

**Java (Mockito):**
```java
// Create mock
UserService userService = mock(UserService.class);

// Set expectations
when(userService.getUser(1L)).thenReturn(new User(1L, "John"));

// Use in test
UserController controller = new UserController(userService);
User user = controller.getUser(1L);

// Verify interaction
verify(userService).getUser(1L);
```

**Python (unittest.mock):**
```python
from unittest.mock import Mock, patch

# Create mock
user_service = Mock()
user_service.get_user.return_value = User(1, "John")

# Use in test
controller = UserController(user_service)
user = controller.get_user(1)

# Verify interaction
user_service.get_user.assert_called_once_with(1)
```

---

## Stubbing

### What is Stubbing?

**Stubbing**: Creating objects that return predefined data.

**Characteristics:**
- **Predefined responses**: Return predefined data
- **State-based**: State-based testing
- **No verification**: No interaction verification
- **Simple**: Simple implementation

### Stubbing Example

**Java:**
```java
// Create stub
UserService userService = new UserService() {
    @Override
    public User getUser(Long id) {
        return new User(id, "John");
    }
};

// Use in test
UserController controller = new UserController(userService);
User user = controller.getUser(1L);
```

**Python:**
```python
class StubUserService:
    def get_user(self, user_id):
        return User(user_id, "John")

# Use in test
user_service = StubUserService()
controller = UserController(user_service)
user = controller.get_user(1)
```

---

## Spies and Fakes

### Spies

**What:**
```
Wrap real object
  ↓
Record interactions
  ↓
Verify later
```

**Use when:**
- **Real object**: Need real object
- **Verification**: Need to verify calls
- **Partial**: Partial mocking

**Example:**
```java
// Create spy
UserService userService = spy(new UserService());

// Use real method
User user = userService.getUser(1L);

// Verify call
verify(userService).getUser(1L);
```

### Fakes

**What:**
```
Working implementation
  ↓
Simplified
  ↓
In-memory
```

**Use when:**
- **Simple implementation**: Simple implementation needed
- **In-memory**: In-memory storage
- **Fast**: Fast execution

**Example:**
```java
// Fake repository
class FakeUserRepository implements UserRepository {
    private Map<Long, User> users = new HashMap<>();
    
    @Override
    public User findById(Long id) {
        return users.get(id);
    }
    
    @Override
    public void save(User user) {
        users.put(user.getId(), user);
    }
}
```

---

## Best Practices

### 1. Mock External Dependencies

**Why:**
- **Isolation**: Isolate code under test
- **Speed**: Fast tests
- **Reliability**: Reliable tests

**Guidelines:**
- **External services**: Mock external services
- **Database**: Mock database
- **Network**: Mock network calls
- **File system**: Mock file system

### 2. Don't Mock Everything

**Why:**
- **Over-mocking**: Over-mocking is bad
- **Value objects**: Don't mock value objects
- **Simple objects**: Don't mock simple objects

**Guidelines:**
- **Mock external**: Mock external dependencies
- **Don't mock internals**: Don't mock internal classes
- **Use real objects**: Use real objects when possible

### 3. Verify Interactions

**Why:**
- **Correctness**: Verify correctness
- **Behavior**: Verify behavior
- **Confidence**: Build confidence

**Guidelines:**
- **Verify calls**: Verify method calls
- **Verify parameters**: Verify parameters
- **Verify order**: Verify call order if needed

### 4. Keep Tests Simple

**Why:**
- **Readability**: Better readability
- **Maintainability**: Easier maintenance
- **Understanding**: Easier understanding

**Guidelines:**
- **Simple mocks**: Keep mocks simple
- **Clear expectations**: Clear expectations
- **Minimal setup**: Minimal setup

---

## Summary

Mocking and stubbing are essential for unit testing. Understanding test doubles, mocking, stubbing, spies, fakes, and best practices is crucial for effective testing.

**Key Takeaways:**
- **Mocks and stubs**: Test doubles that replace real dependencies
- **Test doubles**: Dummy, fake, stub, mock, spy
- **Mocking**: Verify interactions (behavior-based)
- **Stubbing**: Return predefined data (state-based)
- **Spies**: Wrap real objects, record interactions
- **Fakes**: Working simplified implementations
- **Best practices**: Mock external dependencies, don't mock everything, verify interactions, keep tests simple

**Test Doubles:**
- **Dummy**: Placeholder
- **Fake**: Working implementation
- **Stub**: Predefined responses
- **Mock**: Verify interactions
- **Spy**: Record interactions

**Best Practices:**
- Mock external dependencies
- Don't mock everything
- Verify interactions
- Keep tests simple

**Next Steps:**
- Understand test doubles
- Learn mocking frameworks
- Practice mocking
- Apply best practices

