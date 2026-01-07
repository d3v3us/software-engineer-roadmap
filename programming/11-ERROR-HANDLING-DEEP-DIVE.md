# Error Handling Deep Dive - Complete Understanding

## Table of Contents
1. [What is Error Handling?](#what-is-error-handling)
2. [Why Error Handling Matters](#why-error-handling-matters)
3. [Types of Errors](#types-of-errors)
4. [Error Handling Strategies](#error-handling-strategies)
5. [Exception Handling](#exception-handling)
6. [Error Codes vs Exceptions](#error-codes-vs-exceptions)
7. [Error Propagation](#error-propagation)
8. [Error Recovery](#error-recovery)
9. [Error Logging](#error-logging)
10. [Error Handling Patterns](#error-handling-patterns)
11. [Best Practices](#best-practices)
12. [Common Mistakes](#common-mistakes)

---

## What is Error Handling?

### Definition

**Error Handling**: Process of responding to and recovering from error conditions in a program.

**Key Concept:**
- **Detect errors**: Detect when errors occur
- **Handle errors**: Handle errors appropriately
- **Recover**: Recover from errors when possible
- **Inform**: Inform user/system about errors

### Real-World Analogy

**Error Handling = Safety Systems:**
- **Error**: Problem occurs (fire)
- **Detection**: Detect problem (smoke detector)
- **Handling**: Handle problem (sprinkler system)
- **Recovery**: Recover (evacuate, fire department)
- **Communication**: Communicate (alarm)

**Software:**
- **Error**: Exception occurs
- **Detection**: Try-catch detects
- **Handling**: Handle exception
- **Recovery**: Retry or fallback
- **Logging**: Log error

---

## Why Error Handling Matters?

### Problems Without Error Handling

**1. Crashes:**
```
Error occurs
  ↓
No error handling
  ↓
Program crashes
  ↓
User loses work
```

**2. Data Loss:**
```
Error during save
  ↓
No error handling
  ↓
Data not saved
  ↓
Data lost
```

**3. Poor User Experience:**
```
Error occurs
  ↓
No user-friendly message
  ↓
User confused
  ↓
Poor experience
```

### Benefits of Error Handling

**1. Reliability:**
- **Handle errors**: Handle errors gracefully
- **No crashes**: Prevent crashes
- **Stable**: More stable application

**2. User Experience:**
- **Clear messages**: Clear error messages
- **Helpful**: Helpful information
- **Better UX**: Better user experience

**3. Debugging:**
- **Error logs**: Error logs help debugging
- **Context**: Error context
- **Faster fixes**: Faster bug fixes

---

## Types of Errors

### Error Categories

**1. Syntax Errors:**
```
Compile-time errors
  ↓
Code doesn't compile
  ↓
Fixed before runtime
```

**2. Runtime Errors:**
```
Runtime exceptions
  ↓
Occur during execution
  ↓
Must handle at runtime
```

**3. Logic Errors:**
```
Program runs but wrong result
  ↓
Hard to detect
  ↓
Testing needed
```

### Common Runtime Errors

**1. Null Pointer:**
```python
user = None
name = user.name  # NullPointerException
```

**2. Division by Zero:**
```python
result = 10 / 0  # DivisionByZeroError
```

**3. Index Out of Bounds:**
```python
arr = [1, 2, 3]
value = arr[10]  # IndexError
```

**4. Type Errors:**
```python
result = "hello" + 5  # TypeError
```

**5. File Not Found:**
```python
file = open("missing.txt")  # FileNotFoundError
```

---

## Error Handling Strategies

### Strategy 1: Fail Fast

**Approach:**
```
Error detected
  ↓
Fail immediately
  ↓
Don't continue with invalid state
```

**Example:**
```python
def process_payment(amount):
    if amount <= 0:
        raise ValueError("Amount must be positive")
    if amount > MAX_AMOUNT:
        raise ValueError("Amount exceeds maximum")
    # Process payment
```

### Strategy 2: Fail Safe

**Approach:**
```
Error detected
  ↓
Use safe default
  ↓
Continue with safe state
```

**Example:**
```python
def get_config_value(key, default=None):
    try:
        return config[key]
    except KeyError:
        return default  # Safe default
```

### Strategy 3: Retry

**Approach:**
```
Error detected
  ↓
Retry operation
  ↓
May succeed on retry
```

**Example:**
```python
def fetch_data(url, max_retries=3):
    for attempt in range(max_retries):
        try:
            return requests.get(url)
        except requests.RequestException:
            if attempt == max_retries - 1:
                raise
            time.sleep(2 ** attempt)  # Exponential backoff
```

---

## Exception Handling

### Try-Catch-Finally

**Basic Structure:**
```python
try:
    # Code that might fail
    risky_operation()
except SpecificError:
    # Handle specific error
    handle_error()
except AnotherError:
    # Handle another error
    handle_another()
finally:
    # Always executed
    cleanup()
```

### Exception Hierarchy

**Python:**
```
BaseException
  ├── Exception
  │   ├── ArithmeticError
  │   ├── LookupError
  │   ├── ValueError
  │   └── ...
  └── SystemExit
  └── KeyboardInterrupt
```

**Java:**
```
Throwable
  ├── Error
  └── Exception
      ├── RuntimeException
      └── CheckedException
```

### Catching Exceptions

**Specific Exceptions:**
```python
try:
    result = 10 / 0
except ZeroDivisionError:
    print("Cannot divide by zero")
```

**Multiple Exceptions:**
```python
try:
    process_data()
except ValueError as e:
    print(f"Value error: {e}")
except TypeError as e:
    print(f"Type error: {e}")
except Exception as e:
    print(f"Unexpected error: {e}")
```

**Catch All (Not Recommended):**
```python
try:
    risky_operation()
except:  # Catches all - not recommended
    print("Something went wrong")
```

---

## Error Codes vs Exceptions

### Error Codes

**Approach:**
```
Function returns error code
  ↓
Caller checks code
  ↓
Handle based on code
```

**Example:**
```c
int divide(int a, int b, int* result) {
    if (b == 0) {
        return ERROR_DIVISION_BY_ZERO;
    }
    *result = a / b;
    return SUCCESS;
}

// Usage
int result;
int error = divide(10, 0, &result);
if (error != SUCCESS) {
    handle_error(error);
}
```

**Pros:**
- **Explicit**: Explicit error handling
- **No exceptions**: No exception overhead
- **Predictable**: Predictable performance

**Cons:**
- **Easy to ignore**: Easy to ignore errors
- **Verbose**: More verbose code
- **Error-prone**: Error-prone (forget to check)

### Exceptions

**Approach:**
```
Function throws exception
  ↓
Exception propagates
  ↓
Caught by handler
```

**Example:**
```python
def divide(a, b):
    if b == 0:
        raise ZeroDivisionError("Cannot divide by zero")
    return a / b

# Usage
try:
    result = divide(10, 0)
except ZeroDivisionError as e:
    handle_error(e)
```

**Pros:**
- **Cannot ignore**: Cannot ignore (must handle)
- **Clean code**: Cleaner code
- **Automatic propagation**: Automatic propagation

**Cons:**
- **Overhead**: Exception overhead
- **Less explicit**: Less explicit
- **Performance**: Performance impact

---

## Error Propagation

### How Errors Propagate

**Call Stack:**
```
Function A
  ↓ calls
Function B
  ↓ calls
Function C
  ↓ throws exception
Exception propagates up
  ↓
Function B (doesn't catch)
  ↓
Function A (catches)
```

**Example:**
```python
def function_a():
    try:
        function_b()
    except ValueError as e:
        print(f"Caught in A: {e}")

def function_b():
    function_c()  # Doesn't catch

def function_c():
    raise ValueError("Error in C")

# Exception propagates: C → B → A
```

### Propagation Strategies

**1. Let Propagate:**
```python
def process():
    # Don't catch, let propagate
    risky_operation()
```

**2. Catch and Re-throw:**
```python
def process():
    try:
        risky_operation()
    except SpecificError as e:
        log_error(e)
        raise  # Re-throw
```

**3. Transform Exception:**
```python
def process():
    try:
        risky_operation()
    except SpecificError as e:
        raise CustomError("Custom message") from e
```

---

## Error Recovery

### Recovery Strategies

**1. Retry:**
```
Error occurs
  ↓
Retry operation
  ↓
May succeed
```

**2. Fallback:**
```
Error occurs
  ↓
Use fallback mechanism
  ↓
Continue with fallback
```

**3. Graceful Degradation:**
```
Error occurs
  ↓
Reduce functionality
  ↓
Continue with reduced features
```

### Recovery Examples

**Retry:**
```python
def fetch_with_retry(url, max_retries=3):
    for attempt in range(max_retries):
        try:
            return requests.get(url)
        except requests.RequestException:
            if attempt < max_retries - 1:
                time.sleep(2 ** attempt)
                continue
            raise
```

**Fallback:**
```python
def get_data():
    try:
        return fetch_from_primary()
    except ConnectionError:
        return fetch_from_secondary()  # Fallback
```

**Graceful Degradation:**
```python
def get_user_preferences():
    try:
        return fetch_from_database()
    except DatabaseError:
        return get_default_preferences()  # Degraded mode
```

---

## Error Logging

### Why Log Errors?

**Reasons:**
- **Debugging**: Help debug issues
- **Monitoring**: Monitor error rates
- **Audit**: Audit trail
- **Analysis**: Analyze error patterns

### What to Log

**1. Error Message:**
```
Error: Division by zero
```

**2. Stack Trace:**
```
Traceback (most recent call last):
  File "script.py", line 10, in function
    result = 10 / 0
ZeroDivisionError: division by zero
```

**3. Context:**
```
User ID: 123
Request ID: abc-123
Timestamp: 2024-01-15T10:30:00Z
```

**4. State:**
```
Variables: a=10, b=0
Environment: production
Version: 1.2.3
```

### Logging Best Practices

**1. Appropriate Level:**
```python
logger.debug("Detailed debug info")
logger.info("General information")
logger.warning("Warning message")
logger.error("Error occurred")
logger.critical("Critical error")
```

**2. Structured Logging:**
```python
logger.error("Payment failed", extra={
    "user_id": user_id,
    "amount": amount,
    "error_code": error_code
})
```

**3. Don't Log Sensitive Data:**
```python
# Bad
logger.error(f"Login failed: username={username}, password={password}")

# Good
logger.error(f"Login failed: username={username}")
```

---

## Error Handling Patterns

### Pattern 1: Result Type

**Result Type:**
```python
from typing import Union

Result = Union[Success[T], Error]

def divide(a: int, b: int) -> Result[int]:
    if b == 0:
        return Error("Division by zero")
    return Success(a / b)

# Usage
result = divide(10, 0)
if result.is_error():
    handle_error(result.error)
else:
    use_value(result.value)
```

### Pattern 2: Optional/Maybe

**Optional:**
```python
from typing import Optional

def find_user(user_id: int) -> Optional[User]:
    try:
        return database.get_user(user_id)
    except UserNotFound:
        return None

# Usage
user = find_user(123)
if user is None:
    handle_not_found()
else:
    use_user(user)
```

### Pattern 3: Either/Result

**Either:**
```python
from typing import Union

Either = Union[Left[Error], Right[Value]]

def process(value: int) -> Either[str, int]:
    if value < 0:
        return Left("Value must be positive")
    return Right(value * 2)
```

---

## Best Practices

### 1. Be Specific

**Why:**
- **Better handling**: Better error handling
- **Clear errors**: Clear error messages
- **Easier debugging**: Easier debugging

**Implementation:**
```python
# Bad
except Exception:
    pass

# Good
except ValueError as e:
    handle_value_error(e)
except FileNotFoundError as e:
    handle_file_not_found(e)
```

### 2. Don't Swallow Exceptions

**Why:**
- **Silent failures**: Silent failures are bad
- **Hard to debug**: Hard to debug
- **Data corruption**: May cause data corruption

**Implementation:**
```python
# Bad
try:
    risky_operation()
except:
    pass  # Swallowing exception!

# Good
try:
    risky_operation()
except SpecificError as e:
    log_error(e)
    handle_error(e)
```

### 3. Provide Context

**Why:**
- **Better debugging**: Better debugging
- **Clear errors**: Clear error messages
- **User-friendly**: User-friendly messages

**Implementation:**
```python
try:
    process_payment(user_id, amount)
except PaymentError as e:
    raise PaymentError(
        f"Payment failed for user {user_id}, amount {amount}: {e}"
    ) from e
```

### 4. Log Errors

**Why:**
- **Debugging**: Help debugging
- **Monitoring**: Monitor errors
- **Analysis**: Analyze patterns

**Implementation:**
```python
try:
    risky_operation()
except Exception as e:
    logger.error("Operation failed", exc_info=True, extra={
        "context": context
    })
    raise
```

### 5. Fail Fast

**Why:**
- **Early detection**: Detect errors early
- **No invalid state**: Don't continue with invalid state
- **Easier debugging**: Easier to debug

**Implementation:**
```python
def process(data):
    if not data:
        raise ValueError("Data cannot be empty")
    if not is_valid(data):
        raise ValueError("Data is invalid")
    # Process valid data
```

---

## Common Mistakes

### Mistake 1: Catching Too Broad

**Bad:**
```python
try:
    risky_operation()
except Exception:  # Too broad!
    handle_error()
```

**Good:**
```python
try:
    risky_operation()
except SpecificError:  # Specific
    handle_error()
```

### Mistake 2: Swallowing Exceptions

**Bad:**
```python
try:
    risky_operation()
except:
    pass  # Silent failure!
```

**Good:**
```python
try:
    risky_operation()
except SpecificError as e:
    log_error(e)
    handle_error(e)
```

### Mistake 3: Empty Catch Blocks

**Bad:**
```python
try:
    risky_operation()
except:
    pass  # Do nothing!
```

**Good:**
```python
try:
    risky_operation()
except ExpectedError:
    # Expected, handle appropriately
    handle_expected_error()
```

### Mistake 4: Not Providing Context

**Bad:**
```python
raise ValueError("Error")
```

**Good:**
```python
raise ValueError(f"Invalid value: {value}, expected positive number")
```

---

## Summary

Error handling is essential for building reliable software. Understanding strategies, patterns, and best practices is crucial for production systems.

**Key Takeaways:**
- **Error handling**: Respond to and recover from errors
- **Strategies**: Fail fast, fail safe, retry
- **Exception handling**: Try-catch-finally
- **Error codes vs exceptions**: Different approaches
- **Error propagation**: How errors propagate
- **Error recovery**: Retry, fallback, graceful degradation
- **Error logging**: Log errors with context
- **Best practices**: Be specific, don't swallow, provide context, log, fail fast

**Error Handling Strategies:**
- **Fail fast**: Fail immediately
- **Fail safe**: Use safe defaults
- **Retry**: Retry on failure

**Best Practices:**
- Be specific
- Don't swallow exceptions
- Provide context
- Log errors
- Fail fast

**Common Mistakes:**
- Catching too broad
- Swallowing exceptions
- Empty catch blocks
- Not providing context

**Next Steps:**
- Implement error handling
- Add error logging
- Test error scenarios
- Monitor errors
- Apply best practices

