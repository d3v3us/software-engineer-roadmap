# Closures and Generics Deep Dive - Complete Understanding

## Table of Contents
1. [Closures - Functions with Memory](#closures---functions-with-memory)
2. [Generics - Type Parameters](#generics---type-parameters)
3. [Type Erasure - When Types Disappear](#type-erasure---when-types-disappear)
4. [Practical Applications](#practical-applications)

---

## Closures - Functions with Memory

### What is a Closure?

**Closure**: Function that has access to variables from its outer (enclosing) scope even after the outer function has finished executing.

**Key Concept:**
- Function "closes over" variables
- Maintains reference to outer scope
- Variables persist beyond outer function execution

### Simple Example

**Without Closure:**
```python
def outer():
    x = 10
    def inner():
        print(x)  # Error: x is not defined
    return inner

func = outer()
func()  # Error!
```

**With Closure:**
```python
def outer():
    x = 10
    def inner():
        print(x)  # Accesses x from outer scope
    return inner

func = outer()
func()  # Prints: 10 ✓
# x still exists even though outer() finished!
```

### How Closures Work

**Step-by-Step:**
```
1. outer() is called
2. x = 10 is created in outer's scope
3. inner() is defined (captures x)
4. outer() returns inner
5. outer() finishes, but x persists
6. inner() still has access to x
```

**Visual:**
```
outer() scope:
  x = 10
  └── inner() (captures x)
      └── Can access x even after outer() finishes
```

### Closure with Mutable State

**Example:**
```python
def counter():
    count = 0  # Captured variable
    
    def increment():
        nonlocal count  # Modify outer variable
        count += 1
        return count
    
    return increment

c1 = counter()
print(c1())  # 1
print(c1())  # 2
print(c1())  # 3

c2 = counter()  # New closure, new state
print(c2())  # 1 (independent)
```

**Key Point:**
- Each closure has its own captured state
- `c1` and `c2` are independent
- State persists between calls

### Common Use Cases

**1. Function Factories:**
```python
def multiplier(factor):
    def multiply(x):
        return x * factor
    return multiply

double = multiplier(2)
triple = multiplier(3)

print(double(5))  # 10
print(triple(5))  # 15
```

**2. Callbacks:**
```python
def create_handler(prefix):
    def handler(message):
        print(f"{prefix}: {message}")
    return handler

error_handler = create_handler("ERROR")
info_handler = create_handler("INFO")

error_handler("Something went wrong")  # ERROR: Something went wrong
info_handler("Process started")        # INFO: Process started
```

**3. Data Privacy:**
```python
def create_account(initial_balance):
    balance = initial_balance  # Private variable
    
    def deposit(amount):
        nonlocal balance
        balance += amount
        return balance
    
    def withdraw(amount):
        nonlocal balance
        if amount <= balance:
            balance -= amount
            return balance
        else:
            raise ValueError("Insufficient funds")
    
    def get_balance():
        return balance
    
    return {
        'deposit': deposit,
        'withdraw': withdraw,
        'get_balance': get_balance
    }

account = create_account(100)
account['deposit'](50)  # 150
# balance is not directly accessible (encapsulated)
```

### Closure Gotchas

**1. Late Binding:**
```python
# Problem
functions = []
for i in range(3):
    functions.append(lambda: print(i))

for f in functions:
    f()  # Prints: 2, 2, 2 (not 0, 1, 2!)
```

**Why?**
- All closures capture the same variable `i`
- When called, `i` is already 2
- Late binding: value evaluated when called

**Solution:**
```python
# Fix: Capture value, not variable
functions = []
for i in range(3):
    functions.append(lambda x=i: print(x))  # Default parameter captures value

for f in functions:
    f()  # Prints: 0, 1, 2 ✓
```

**2. Mutable Captures:**
```python
def outer():
    items = []  # Mutable
    
    def add(item):
        items.append(item)
        return items
    
    return add

adder = outer()
print(adder(1))  # [1]
print(adder(2))  # [1, 2]
# All closures share same list!
```

---

## Generics - Type Parameters

### What are Generics?

**Generics**: Feature that allows types to be parameters. Write code that works with multiple types.

**Also Known As:**
- Type parameters
- Parametric polymorphism
- Templates (C++)

### The Problem Without Generics

**Without Generics:**
```java
// Must write separate code for each type
class IntList {
    private int[] items;
    void add(int item) { ... }
    int get(int index) { ... }
}

class StringList {
    private String[] items;
    void add(String item) { ... }
    String get(int index) { ... }
}

// Duplicate code for each type!
```

### Solution: Generics

**With Generics:**
```java
// One implementation for all types
class List<T> {
    private T[] items;
    void add(T item) { ... }
    T get(int index) { ... }
}

// Usage
List<Integer> intList = new List<>();
List<String> stringList = new List<>();
List<User> userList = new List<>();
```

**Benefits:**
- Code reuse
- Type safety
- No casting needed

### Generic Syntax by Language

**Java:**
```java
class Box<T> {
    private T value;
    
    public void set(T value) {
        this.value = value;
    }
    
    public T get() {
        return value;
    }
}

Box<Integer> intBox = new Box<>();
Box<String> stringBox = new Box<>();
```

**C#:**
```csharp
class Box<T> {
    private T value;
    
    public void Set(T value) {
        this.value = value;
    }
    
    public T Get() {
        return value;
    }
}
```

**TypeScript:**
```typescript
class Box<T> {
    private value: T;
    
    set(value: T): void {
        this.value = value;
    }
    
    get(): T {
        return this.value;
    }
}
```

**Python (Type Hints):**
```python
from typing import TypeVar, Generic

T = TypeVar('T')

class Box(Generic[T]):
    def __init__(self):
        self.value: T = None
    
    def set(self, value: T) -> None:
        self.value = value
    
    def get(self) -> T:
        return self.value
```

### Bounded Generics

**Constraint Types:**
```java
// T must extend Number
class Calculator<T extends Number> {
    T add(T a, T b) {
        // Can use Number methods
        return a + b;
    }
}

Calculator<Integer> calc1 = new Calculator<>();  // OK
Calculator<String> calc2 = new Calculator<>();   // Error!
```

**Multiple Bounds:**
```java
// T must extend A and implement B
class Example<T extends A & B> {
    // ...
}
```

### Generic Methods

**Methods Can Be Generic Too:**
```java
class Utils {
    public static <T> T getFirst(List<T> list) {
        return list.get(0);
    }
    
    public static <T extends Comparable<T>> T max(T a, T b) {
        return a.compareTo(b) > 0 ? a : b;
    }
}
```

### Wildcards

**Java Wildcards:**
```java
// ? extends T: Upper bound (read-only)
List<? extends Number> numbers;  // Can read as Number

// ? super T: Lower bound (write-only)
List<? super Integer> integers;  // Can write Integer

// ?: Unbounded (read as Object)
List<?> anything;  // Can read as Object
```

---

## Type Erasure - When Types Disappear

### What is Type Erasure?

**Type Erasure**: Process where generic type information is removed at runtime. Types exist only at compile time.

### Example: Java

**At Compile Time:**
```java
List<Integer> intList = new ArrayList<>();
List<String> stringList = new ArrayList<>();
```

**At Runtime (After Erasure):**
```java
List intList = new ArrayList();      // Just List
List stringList = new ArrayList();   // Just List
// Type information is gone!
```

### Why Type Erasure?

**Reasons:**
- **Backward compatibility**: Generics added later, must work with old code
- **Performance**: No runtime overhead
- **Simplicity**: JVM doesn't need to know about generics

### Consequences

**1. Cannot Check Generic Type at Runtime:**
```java
List<Integer> list = new ArrayList<>();

// This doesn't work:
if (list instanceof List<Integer>) {  // Error!
    // ...
}

// This works:
if (list instanceof List) {  // OK (raw type)
    // ...
}
```

**2. Type Information Lost:**
```java
List<Integer> intList = new ArrayList<>();
List<String> stringList = new ArrayList<>();

// At runtime, both are just List
// Cannot distinguish between List<Integer> and List<String>
```

**3. Array Creation:**
```java
// Cannot create generic arrays
T[] array = new T[10];  // Error!

// Workaround: Use Object[] and cast
T[] array = (T[]) new Object[10];
```

### Type Erasure in Different Languages

**Java:**
- ✅ Type erasure (types removed at runtime)
- Types only at compile time

**C#:**
- ❌ No type erasure (types preserved at runtime)
- Can check types at runtime
- More overhead

**TypeScript:**
- ✅ Type erasure (compiles to JavaScript)
- Types only at compile time
- No runtime type information

**C++:**
- ❌ No type erasure (templates, not generics)
- Code generated for each type
- More code, but faster

### Working with Type Erasure

**1. Use Type Tokens:**
```java
class TypeToken<T> {
    private Class<T> type;
    
    public TypeToken(Class<T> type) {
        this.type = type;
    }
    
    public Class<T> getType() {
        return type;
    }
}

TypeToken<List<String>> token = new TypeToken<List<String>>(List.class);
// Can get type information at runtime
```

**2. Reflection:**
```java
// Can inspect generic types through reflection
Method method = ...;
Type returnType = method.getGenericReturnType();
// Can extract type parameters
```

---

## Practical Applications

### Closures in Practice

**1. Event Handlers:**
```javascript
function createButtonHandler(buttonId) {
    let clickCount = 0;
    
    return function() {
        clickCount++;
        console.log(`Button ${buttonId} clicked ${clickCount} times`);
    };
}

const handler1 = createButtonHandler("btn1");
const handler2 = createButtonHandler("btn2");
// Each has independent click count
```

**2. Memoization:**
```python
def memoize(func):
    cache = {}  # Captured in closure
    
    def wrapper(*args):
        if args not in cache:
            cache[args] = func(*args)
        return cache[args]
    
    return wrapper

@memoize
def fibonacci(n):
    if n < 2:
        return n
    return fibonacci(n-1) + fibonacci(n-2)
```

**3. Partial Application:**
```python
def partial(func, *args):
    def wrapper(*more_args):
        return func(*args, *more_args)
    return wrapper

def multiply(x, y):
    return x * y

double = partial(multiply, 2)
print(double(5))  # 10
```

### Generics in Practice

**1. Collections:**
```java
// Type-safe collections
List<String> names = new ArrayList<>();
names.add("Alice");
String name = names.get(0);  // No casting needed
```

**2. Algorithms:**
```java
public static <T> void swap(List<T> list, int i, int j) {
    T temp = list.get(i);
    list.set(i, list.get(j));
    list.set(j, temp);
}
// Works with any type
```

**3. Builders:**
```java
class Builder<T> {
    private T instance;
    
    public Builder<T> setValue(T value) {
        this.instance = value;
        return this;
    }
    
    public T build() {
        return instance;
    }
}
```

---

## Summary

Closures and generics are powerful features that enable more flexible and reusable code.

**Key Takeaways:**
- Closures: Functions that capture outer scope variables
- Generics: Type parameters for code reuse
- Type erasure: Generic types removed at runtime (in some languages)
- Use closures for stateful functions, callbacks, encapsulation
- Use generics for type-safe, reusable code
- Understand type erasure limitations

**Next Steps:**
- Practice writing closures
- Learn generic syntax in your language
- Understand type erasure in your language
- Apply in real projects

