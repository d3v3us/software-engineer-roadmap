# Memory Leaks Deep Dive - Complete Understanding

## Table of Contents
1. [What is a Memory Leak?](#what-is-a-memory-leak)
2. [How Memory Leaks Occur](#how-memory-leaks-occur)
3. [Common Causes](#common-causes)
4. [Detecting Memory Leaks](#detecting-memory-leaks)
5. [Preventing Memory Leaks](#preventing-memory-leaks)
6. [Fixing Memory Leaks](#fixing-memory-leaks)

---

## What is a Memory Leak?

### Definition

**Memory Leak**: Situation where memory that is no longer needed is not released, causing memory usage to grow over time.

**Key Concept:**
- **Memory allocated**: Memory is allocated
- **Not released**: Memory is never freed
- **Accumulates**: Memory usage grows continuously
- **Eventually**: System runs out of memory

### Real-World Analogy

**Memory Leak = Leaky Faucet:**
- **Normal**: Turn on faucet, use water, turn off → Water stops
- **Leak**: Turn on faucet, use water, forget to turn off → Water keeps flowing
- **Result**: Eventually, water overflows (out of memory)

### Impact

**Symptoms:**
- Application slows down
- System becomes unresponsive
- Out of memory errors
- Application crashes

**Severity:**
- **Small leak**: May take days/weeks to notice
- **Large leak**: Crashes within hours/minutes
- **Critical**: Immediate failure

---

## How Memory Leaks Occur

### Memory Lifecycle

**Normal Flow:**
```
1. Allocate memory
2. Use memory
3. Release memory
4. Memory available again
```

**Leak Flow:**
```
1. Allocate memory
2. Use memory
3. Forget to release
4. Memory lost forever (until program ends)
```

### Visual Example

**Normal:**
```
Memory Pool: [████████████]
Allocate:    [██░░░░░░░░░░] (2 used, 10 free)
Use:         [██░░░░░░░░░░]
Release:     [████████████] (back to pool)
```

**Leak:**
```
Memory Pool: [████████████]
Allocate:    [██░░░░░░░░░░] (2 used)
Allocate:    [████░░░░░░░░] (4 used)
Allocate:    [██████░░░░░░] (6 used)
... (never release)
Eventually:  [████████████] (all used, out of memory!)
```

---

## Common Causes

### 1. Forgotten References

**Problem:**
- Keep reference to object
- Object never garbage collected
- Memory never freed

**Example (Java):**
```java
public class Leak {
    private static List<Object> cache = new ArrayList<>();
    
    public void addToCache(Object obj) {
        cache.add(obj);  // Add to cache
        // Never remove!
        // Objects accumulate forever
    }
}
```

**Solution:**
```java
public class Fixed {
    private static List<Object> cache = new ArrayList<>();
    private static final int MAX_SIZE = 100;
    
    public void addToCache(Object obj) {
        cache.add(obj);
        if (cache.size() > MAX_SIZE) {
            cache.remove(0);  // Remove oldest
        }
    }
}
```

### 2. Event Listeners

**Problem:**
- Register event listener
- Never unregister
- Listener holds reference
- Object never collected

**Example (JavaScript):**
```javascript
function setupListeners() {
    const button = document.getElementById('button');
    button.addEventListener('click', function() {
        // Do something
        // Listener never removed!
    });
    // button element holds reference to listener
    // Listener holds reference to closure
    // Memory leak!
}
```

**Solution:**
```javascript
function setupListeners() {
    const button = document.getElementById('button');
    const handler = function() {
        // Do something
    };
    button.addEventListener('click', handler);
    
    // Later, remove listener
    button.removeEventListener('click', handler);
}
```

### 3. Circular References

**Problem:**
- Objects reference each other
- No external reference
- But can't be collected (in some languages)

**Example:**
```python
class Node:
    def __init__(self):
        self.parent = None
        self.children = []

# Create circular reference
parent = Node()
child = Node()
parent.children.append(child)
child.parent = parent

# Delete references
del parent
del child
# But objects still reference each other!
# In Python: GC handles this, but not in all languages
```

**Solution:**
```python
# Break circular reference before deleting
child.parent = None
parent.children = []
del parent
del child
```

### 4. Caches Without Limits

**Problem:**
- Cache grows indefinitely
- Never evicts entries
- Memory grows forever

**Example:**
```python
cache = {}  # No limit!

def get_data(key):
    if key not in cache:
        cache[key] = fetch_from_database(key)
    return cache[key]
    # Cache grows forever!
```

**Solution:**
```python
from collections import OrderedDict

class LRUCache:
    def __init__(self, max_size=100):
        self.cache = OrderedDict()
        self.max_size = max_size
    
    def get(self, key):
        if key in self.cache:
            # Move to end (most recently used)
            self.cache.move_to_end(key)
            return self.cache[key]
        return None
    
    def set(self, key, value):
        if key in self.cache:
            self.cache.move_to_end(key)
        self.cache[key] = value
        if len(self.cache) > self.max_size:
            # Remove oldest (least recently used)
            self.cache.popitem(last=False)
```

### 5. Thread Local Storage

**Problem:**
- Thread-local variables
- Thread dies but storage remains
- Memory not freed

**Example (Java):**
```java
public class ThreadLocalLeak {
    private static ThreadLocal<Object> threadLocal = new ThreadLocal<>();
    
    public void setValue(Object value) {
        threadLocal.set(value);
        // Thread dies, but ThreadLocal map still holds reference
    }
}
```

**Solution:**
```java
public class ThreadLocalFixed {
    private static ThreadLocal<Object> threadLocal = new ThreadLocal<>();
    
    public void setValue(Object value) {
        threadLocal.set(value);
    }
    
    public void cleanup() {
        threadLocal.remove();  // Remove when done
    }
}
```

### 6. Static Collections

**Problem:**
- Static collections grow
- Never cleared
- Accumulate forever

**Example:**
```java
public class StaticLeak {
    private static Map<String, Object> registry = new HashMap<>();
    
    public void register(String key, Object obj) {
        registry.put(key, obj);
        // Never removed!
    }
}
```

**Solution:**
```java
public class StaticFixed {
    private static Map<String, Object> registry = new HashMap<>();
    
    public void register(String key, Object obj) {
        registry.put(key, obj);
    }
    
    public void unregister(String key) {
        registry.remove(key);  // Remove when done
    }
}
```

---

## Detecting Memory Leaks

### Symptoms

**1. Memory Growth:**
- Memory usage increases over time
- Doesn't decrease after operations
- Grows continuously

**2. Performance Degradation:**
- Application slows down
- Garbage collection takes longer
- System becomes unresponsive

**3. Out of Memory Errors:**
- Application crashes
- "Out of memory" exceptions
- System runs out of memory

### Tools

**1. Memory Profilers:**
- **Java**: VisualVM, JProfiler, YourKit
- **Python**: memory_profiler, py-spy
- **JavaScript**: Chrome DevTools, heap snapshots
- **C/C++**: Valgrind, AddressSanitizer

**2. Monitoring:**
- **Metrics**: Track memory usage over time
- **Alerts**: Alert on memory growth
- **Dashboards**: Visualize memory trends

**3. Heap Dumps:**
- **Snapshot**: Take heap snapshot
- **Analyze**: Find objects using memory
- **Compare**: Compare snapshots over time

### Detection Techniques

**1. Memory Profiling:**
```python
# Python example
from memory_profiler import profile

@profile
def my_function():
    # Code that might leak
    data = [0] * 1000000
    # Check if memory is released
```

**2. Heap Analysis:**
```java
// Java example
// Take heap dump
jmap -dump:format=b,file=heap.bin <pid>

// Analyze with VisualVM or Eclipse MAT
```

**3. Monitoring:**
```python
import psutil
import time

def monitor_memory():
    process = psutil.Process()
    while True:
        memory = process.memory_info().rss / 1024 / 1024  # MB
        print(f"Memory: {memory} MB")
        time.sleep(1)
        # If memory keeps growing, there's a leak!
```

---

## Preventing Memory Leaks

### Best Practices

**1. Use Automatic Memory Management:**
- Use languages with GC (Java, Python, JavaScript)
- Let GC handle memory
- But still be careful with references

**2. Clear References:**
```python
# Good: Clear reference when done
data = load_large_data()
process(data)
data = None  # Clear reference
```

**3. Use Weak References:**
```python
import weakref

# Weak reference doesn't prevent GC
cache = weakref.WeakValueDictionary()

def get_data(key):
    if key not in cache:
        cache[key] = fetch_data(key)
    return cache[key]
    # Automatically removed when no strong references
```

**4. Limit Cache Size:**
```python
from collections import OrderedDict

class LimitedCache:
    def __init__(self, max_size=100):
        self.cache = OrderedDict()
        self.max_size = max_size
    
    def get(self, key):
        # Implementation with size limit
        pass
```

**5. Clean Up Resources:**
```python
# Use context managers
with open('file.txt') as f:
    data = f.read()
# Automatically closed

# Or try-finally
try:
    resource = acquire_resource()
    use_resource(resource)
finally:
    release_resource(resource)
```

**6. Remove Event Listeners:**
```javascript
// Always remove listeners
const handler = () => { /* ... */ };
element.addEventListener('click', handler);
// Later...
element.removeEventListener('click', handler);
```

---

## Fixing Memory Leaks

### Step 1: Identify the Leak

**Process:**
1. Monitor memory usage
2. Identify growth pattern
3. Take heap snapshots
4. Compare snapshots

### Step 2: Find the Source

**Techniques:**
1. Analyze heap dump
2. Find objects using memory
3. Trace references
4. Identify root cause

### Step 3: Fix the Leak

**Solutions:**
1. Remove unnecessary references
2. Clear collections
3. Unregister listeners
4. Use weak references
5. Limit cache sizes

### Step 4: Verify Fix

**Verification:**
1. Monitor memory after fix
2. Confirm memory stabilizes
3. Test under load
4. Verify no regression

---

## Summary

Memory leaks cause applications to consume increasing amounts of memory, eventually leading to failure. Understanding causes, detection, and prevention is essential.

**Key Takeaways:**
- Memory leak: Memory not released, accumulates
- Common causes: Forgotten references, event listeners, circular references, unlimited caches
- Detection: Profilers, monitoring, heap dumps
- Prevention: Clear references, use weak references, limit caches, clean up resources
- Fixing: Identify, find source, fix, verify

**Next Steps:**
- Use memory profilers
- Monitor memory usage
- Follow best practices
- Test for leaks
- Fix leaks promptly

