# Test Doubles Deep Dive - Complete Understanding

## Table of Contents
1. [What are Test Doubles?](#what-are-test-doubles)
2. [Why Test Doubles Matter](#why-test-doubles-matter)
3. [Types of Test Doubles](#types-of-test-doubles)
4. [When to Use Each Type](#when-to-use-each-type)
5. [Test Doubles Comparison](#test-doubles-comparison)
6. [Best Practices](#best-practices)

---

## What are Test Doubles?

### Definition

**Test Doubles**: Objects that replace real dependencies in tests.

**Key Concepts:**
- **Replacement**: Replace real objects
- **Isolation**: Isolate code under test
- **Control**: Control behavior
- **Verification**: Verify interactions

### Real-World Analogy

**Test Doubles = Stunt Doubles:**
- **Actor**: Real dependency
- **Stunt double**: Test double
- **Scene**: Test scenario
- **Safety**: Safe testing

**Testing:**
- **Real dependency**: Real service
- **Test double**: Replacement object
- **Test**: Unit test
- **Isolation**: Isolated testing

---

## Why Test Doubles Matter?

### Impact of No Test Doubles

**1. Slow Tests:**
```
Real dependencies
  ↓
Slow execution
  ↓
I/O operations
```

**2. Unreliable Tests:**
```
External dependencies
  ↓
Flaky tests
  ↓
Network issues
```

**3. Hard to Test:**
```
Complex setup
  ↓
Hard to control
  ↓
Hard to verify
```

### Benefits of Test Doubles

**1. Fast Tests:**
- **No I/O**: No I/O operations
- **Fast execution**: Fast tests
- **Efficient**: Efficient testing

**2. Reliable Tests:**
- **Controlled**: Controlled behavior
- **Predictable**: Predictable results
- **Isolated**: Isolated tests

**3. Easy Testing:**
- **Easy setup**: Easy setup
- **Easy control**: Easy to control
- **Easy verification**: Easy to verify

---

## Types of Test Doubles

### Type 1: Dummy

**What:**
```
Placeholder object
  ↓
Not actually used
  ↓
Fill parameter
```

**Characteristics:**
- **Unused**: Not used in test
- **Placeholder**: Just fills parameter
- **No behavior**: No behavior needed

**Example:**
```java
// Dummy object
User dummyUser = null; // or new User()

// Used to fill parameter
service.process(dummyUser, otherParam);
```

### Type 2: Fake

**What:**
```
Working implementation
  ↓
Simplified
  ↓
In-memory
```

**Characteristics:**
- **Working**: Actually works
- **Simplified**: Simplified implementation
- **In-memory**: In-memory storage

**Example:**
```java
class FakeUserRepository implements UserRepository {
    private Map<Long, User> users = new HashMap<>();
    
    public User findById(Long id) {
        return users.get(id);
    }
    
    public void save(User user) {
        users.put(user.getId(), user);
    }
}
```

### Type 3: Stub

**What:**
```
Returns predefined data
  ↓
No verification
  ↓
State-based
```

**Characteristics:**
- **Predefined**: Returns predefined data
- **No verification**: No interaction verification
- **State-based**: State-based testing

**Example:**
```java
class StubUserService implements UserService {
    public User getUser(Long id) {
        return new User(id, "John");
    }
}
```

### Type 4: Mock

**What:**
```
Verifies interactions
  ↓
Behavior verification
  ↓
Interaction-based
```

**Characteristics:**
- **Verification**: Verifies method calls
- **Behavior**: Behavior-based
- **Expectations**: Set expectations

**Example:**
```java
UserService mock = mock(UserService.class);
when(mock.getUser(1L)).thenReturn(new User(1L, "John"));

// Use in test
User user = controller.getUser(1L);

// Verify
verify(mock).getUser(1L);
```

### Type 5: Spy

**What:**
```
Real object wrapper
  ↓
Records interactions
  ↓
Verification
```

**Characteristics:**
- **Real object**: Uses real object
- **Recording**: Records interactions
- **Verification**: Can verify calls

**Example:**
```java
UserService spy = spy(new UserService());

// Use real method
User user = spy.getUser(1L);

// Verify call
verify(spy).getUser(1L);
```

---

## When to Use Each Type

### Use Dummy When:
- **Parameter filler**: Need to fill parameter
- **Not used**: Object not actually used
- **Simple**: Simple placeholder needed

### Use Fake When:
- **Working needed**: Need working implementation
- **Simplified**: Simplified is acceptable
- **In-memory**: In-memory is fine

### Use Stub When:
- **Predefined data**: Need predefined responses
- **State testing**: State-based testing
- **No verification**: Don't need verification

### Use Mock When:
- **Interaction verification**: Need to verify calls
- **Behavior testing**: Behavior-based testing
- **Expectations**: Need to set expectations

### Use Spy When:
- **Real object**: Need real object behavior
- **Partial verification**: Partial verification needed
- **Recording**: Need to record interactions

---

## Test Doubles Comparison

### Comparison Table

| Type | Behavior | Verification | Use Case |
|------|----------|--------------|----------|
| Dummy | None | None | Parameter filler |
| Fake | Working | None | Simplified implementation |
| Stub | Predefined | None | Predefined responses |
| Mock | Controlled | Yes | Interaction verification |
| Spy | Real + Recording | Yes | Real object + verification |

### When to Use

**Dummy:**
- Parameter filler
- Not used in test

**Fake:**
- Need working implementation
- Simplified is OK

**Stub:**
- Predefined responses
- State-based testing

**Mock:**
- Verify interactions
- Behavior-based testing

**Spy:**
- Real object + verification
- Partial mocking

---

## Best Practices

### 1. Choose Right Type

**Why:**
- **Appropriate**: Use appropriate type
- **Simplicity**: Keep it simple
- **Effectiveness**: More effective

**Guidelines:**
- **Understand types**: Understand each type
- **Choose wisely**: Choose appropriate type
- **Don't overuse**: Don't overuse mocks

### 2. Keep Simple

**Why:**
- **Readability**: Better readability
- **Maintainability**: Easier maintenance
- **Understanding**: Easier understanding

**Guidelines:**
- **Simple doubles**: Keep doubles simple
- **Clear purpose**: Clear purpose
- **Minimal setup**: Minimal setup

### 3. Verify Appropriately

**Why:**
- **Correctness**: Verify correctness
- **Confidence**: Build confidence
- **Quality**: Test quality

**Guidelines:**
- **Verify interactions**: Verify when needed
- **Don't over-verify**: Don't over-verify
- **Focus on important**: Focus on important calls

### 4. Document Purpose

**Why:**
- **Clarity**: Code clarity
- **Understanding**: Better understanding
- **Maintenance**: Easier maintenance

**Guidelines:**
- **Document why**: Document why using double
- **Clear naming**: Clear naming
- **Comments**: Add comments if needed

---

## Summary

Test doubles are essential for effective unit testing. Understanding types of test doubles, when to use each, comparison, and best practices is crucial for effective testing.

**Key Takeaways:**
- **Test doubles**: Objects that replace real dependencies in tests
- **Types**: Dummy (placeholder), Fake (working simplified), Stub (predefined responses), Mock (verifies interactions), Spy (real + recording)
- **When to use**: Dummy (parameter filler), Fake (working needed), Stub (predefined data), Mock (verify interactions), Spy (real + verification)
- **Comparison**: Different characteristics and use cases
- **Best practices**: Choose right type, keep simple, verify appropriately, document purpose

**Test Doubles Types:**
- **Dummy**: Placeholder
- **Fake**: Working simplified
- **Stub**: Predefined responses
- **Mock**: Verifies interactions
- **Spy**: Real + recording

**Best Practices:**
- Choose right type
- Keep simple
- Verify appropriately
- Document purpose

**Next Steps:**
- Understand test doubles
- Learn when to use each
- Practice using doubles
- Apply best practices

