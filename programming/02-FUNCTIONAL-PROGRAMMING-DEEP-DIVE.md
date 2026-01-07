# Functional Programming Deep Dive - Complete Understanding

## Table of Contents
1. [What is Functional Programming?](#what-is-functional-programming)
2. [Core Principles of FP](#core-principles-of-fp)
3. [Pure Functions - The Foundation](#pure-functions---the-foundation)
4. [Immutability - Data That Doesn't Change](#immutability---data-that-doesnt-change)
5. [First-Class and Higher-Order Functions](#first-class-and-higher-order-functions)
6. [Function Composition - Building Complex from Simple](#function-composition---building-complex-from-simple)
7. [Recursion - Replacing Loops](#recursion---replacing-loops)
8. [Lazy Evaluation - Computing on Demand](#lazy-evaluation---computing-on-demand)
9. [Monads, Functors, and Advanced Concepts](#monads-functors-and-advanced-concepts)
10. [FP vs OOP - When to Use What](#fp-vs-oop---when-to-use-what)

---

## What is Functional Programming?

### Definition

**Functional Programming (FP)**: Programming paradigm that treats computation as evaluation of mathematical functions and avoids changing state and mutable data.

**Key Characteristics:**
- **Functions as first-class citizens**: Functions are values
- **Immutability**: Data doesn't change
- **Pure functions**: No side effects
- **Declarative**: What to do, not how

### Imperative vs Functional

**Imperative (How):**
```python
# Step-by-step instructions
result = []
for x in numbers:
    if x > 0:
        result.append(x * 2)
```

**Functional (What):**
```python
# Declarative: what we want
result = [x * 2 for x in numbers if x > 0]
# Or: list(map(lambda x: x * 2, filter(lambda x: x > 0, numbers)))
```

### Why Functional Programming?

**Benefits:**
- **Predictable**: Same input → same output
- **Testable**: Easy to test (no side effects)
- **Concurrent**: No race conditions (immutable data)
- **Composable**: Build complex from simple
- **Mathematical**: Based on solid theory

---

## Core Principles of FP

### 1. Pure Functions

**Pure Function**: Function that:
- Always returns same output for same input
- Has no side effects

**Example:**
```python
# Pure
def add(a, b):
    return a + b

# Impure
counter = 0
def increment():
    global counter
    counter += 1  # Side effect
    return counter
```

### 2. Immutability

**Immutability**: Data cannot be changed after creation.

**Example:**
```python
# Mutable (not FP)
numbers = [1, 2, 3]
numbers.append(4)  # Modifies original

# Immutable (FP)
numbers = [1, 2, 3]
new_numbers = numbers + [4]  # Creates new list
```

### 3. Higher-Order Functions

**Higher-Order Function**: Function that takes functions as arguments or returns functions.

**Example:**
```python
def apply(func, x, y):
    return func(x, y)

result = apply(add, 2, 3)  # 5
```

---

## Pure Functions - The Foundation

### What Makes a Function Pure?

**Requirements:**
1. **Deterministic**: Same input → same output
2. **No side effects**: Doesn't modify external state
3. **No dependencies**: Doesn't depend on external state

### Pure Function Example

```python
def square(x):
    return x * x  # Pure: no side effects, deterministic

def get_current_time():
    return time.now()  # Impure: different output each time
```

### Benefits of Pure Functions

**1. Predictable:**
```
Same input always produces same output
Easy to reason about
```

**2. Testable:**
```
Just test input → output
No need to set up external state
```

**3. Cacheable:**
```
Can cache results
Same input → use cached result
```

**4. Parallelizable:**
```
No shared state
Can run in parallel safely
```

### Side Effects

**What are Side Effects?**
- **Modify global variables**
- **Modify arguments**
- **I/O operations** (print, file, network)
- **Throw exceptions** (sometimes)

**Why Avoid?**
- **Hard to test**: Must set up external state
- **Hard to reason**: Unexpected behavior
- **Not parallelizable**: Race conditions

---

## Immutability - Data That Doesn't Change

### What is Immutability?

**Immutability**: Once created, data cannot be modified.

**Mutable:**
```python
x = [1, 2, 3]
x.append(4)  # x is now [1, 2, 3, 4]
```

**Immutable:**
```python
x = (1, 2, 3)  # Tuple (immutable)
# x.append(4)  # Error! Can't modify
y = x + (4,)   # Create new tuple
```

### Why Immutability?

**Benefits:**
- **No side effects**: Can't accidentally modify
- **Thread-safe**: No race conditions
- **Predictable**: Data doesn't change unexpectedly
- **Easier debugging**: Data is what it is

**Costs:**
- **Memory**: Must create new objects
- **Performance**: Some overhead (usually minimal)

### Immutability Patterns

**1. Create New Instead of Modify:**
```python
# Mutable
def add_item(list, item):
    list.append(item)  # Modifies original

# Immutable
def add_item(list, item):
    return list + [item]  # Returns new list
```

**2. Persistent Data Structures:**
```
Share structure between versions
Efficient immutability
```

---

## First-Class and Higher-Order Functions

### First-Class Functions

**First-Class Function**: Function treated as value.

**Can:**
- **Assign to variable**
- **Pass as argument**
- **Return from function**
- **Store in data structures**

**Example:**
```python
# Assign to variable
add = lambda x, y: x + y

# Pass as argument
def apply(func, x, y):
    return func(x, y)

result = apply(add, 2, 3)  # 5

# Return from function
def make_multiplier(n):
    return lambda x: x * n

double = make_multiplier(2)
result = double(5)  # 10
```

### Higher-Order Functions

**Higher-Order Function**: Function that operates on functions.

**Common Examples:**

**1. Map:**
```python
# Apply function to each element
numbers = [1, 2, 3, 4]
squared = list(map(lambda x: x ** 2, numbers))
# [1, 4, 9, 16]
```

**2. Filter:**
```python
# Keep elements that satisfy condition
numbers = [1, 2, 3, 4, 5]
evens = list(filter(lambda x: x % 2 == 0, numbers))
# [2, 4]
```

**3. Reduce:**
```python
from functools import reduce

# Combine elements into single value
numbers = [1, 2, 3, 4]
sum = reduce(lambda x, y: x + y, numbers)
# 10
```

**4. Custom:**
```python
def apply_twice(func, value):
    return func(func(value))

result = apply_twice(lambda x: x * 2, 3)
# apply_twice(3) → 6 → 12
```

---

## Function Composition - Building Complex from Simple

### What is Function Composition?

**Function Composition**: Combining simple functions to build complex ones.

**Mathematical:**
```
f(g(x)) = (f ∘ g)(x)
```

**Code:**
```python
def compose(f, g):
    return lambda x: f(g(x))

# Example
add_one = lambda x: x + 1
multiply_two = lambda x: x * 2

add_then_multiply = compose(multiply_two, add_one)
result = add_then_multiply(3)  # (3 + 1) * 2 = 8
```

### Benefits

**1. Modularity:**
```
Small, focused functions
Combine as needed
```

**2. Reusability:**
```
Reuse simple functions
Build complex from simple
```

**3. Testability:**
```
Test simple functions
Complex function works if simple ones work
```

---

## Recursion - Replacing Loops

### What is Recursion?

**Recursion**: Function calls itself.

**Base Case**: Condition that stops recursion
**Recursive Case**: Function calls itself with modified input

### Recursion Example

```python
def factorial(n):
    if n == 0:  # Base case
        return 1
    else:  # Recursive case
        return n * factorial(n - 1)

factorial(5)
  = 5 * factorial(4)
  = 5 * 4 * factorial(3)
  = 5 * 4 * 3 * factorial(2)
  = 5 * 4 * 3 * 2 * factorial(1)
  = 5 * 4 * 3 * 2 * 1 * factorial(0)
  = 5 * 4 * 3 * 2 * 1 * 1
  = 120
```

### Tail Recursion

**Tail Recursion**: Recursive call is last operation.

**Optimization:**
```
Some languages optimize tail recursion
Convert to loop (no stack growth)
```

---

## Lazy Evaluation - Computing on Demand

### What is Lazy Evaluation?

**Lazy Evaluation**: Expressions not evaluated until value needed.

**Eager (Immediate):**
```python
numbers = [x * 2 for x in range(1000000)]  # Computes all immediately
```

**Lazy (On Demand):**
```python
numbers = (x * 2 for x in range(1000000))  # Generator, computes on demand

for num in numbers:
    print(num)
    if num > 10:
        break  # Stops early, doesn't compute rest
```

### Benefits

**1. Efficiency:**
```
Don't compute what you don't need
Save computation
```

**2. Infinite Sequences:**
```
Can work with infinite data
Only compute what's needed
```

**3. Memory:**
```
Don't store all values
Generate as needed
```

---

## Monads, Functors, and Advanced Concepts

### Functor

**Functor**: Type that can be mapped over.

**Has `map` function:**
```python
# List is a functor
numbers = [1, 2, 3]
squared = list(map(lambda x: x ** 2, numbers))
```

### Monad

**Monad**: Functor that can chain operations.

**Has `bind` (flatMap) function:**
```python
# Concept: Chain operations that return wrapped values
# Maybe/Option is a monad
def divide(a, b):
    return Some(a / b) if b != 0 else None

result = Some(10).bind(lambda x: divide(x, 2)).bind(lambda x: divide(x, 5))
```

**Note**: These are advanced concepts. Understanding the idea is more important than implementation details for most developers.

---

## FP vs OOP - When to Use What

### Functional Programming

**Use When:**
- **Data transformations**: Processing data
- **Concurrent programming**: Immutability helps
- **Mathematical operations**: Natural fit
- **Pipelines**: Data processing pipelines

### Object-Oriented Programming

**Use When:**
- **Modeling entities**: Real-world objects
- **Stateful systems**: Need to maintain state
- **GUI applications**: Natural object model
- **Large systems**: Organize with classes

### Hybrid Approach

**Best Practice:**
```
Use both paradigms
- OOP for structure and organization
- FP for data transformations
- Best of both worlds
```

**Example:**
```python
class User:
    def __init__(self, name, email):
        self.name = name
        self.email = email
    
    def to_dict(self):
        return {"name": self.name, "email": self.email}

# FP for transformations
users = [User("Alice", "alice@example.com"), ...]
user_dicts = list(map(lambda u: u.to_dict(), users))
```

---

## Summary

Functional Programming is a powerful paradigm emphasizing immutability, pure functions, and composition. Understanding FP principles helps write more predictable, testable, and maintainable code.

**Key Takeaways:**
- FP treats computation as function evaluation
- Pure functions: No side effects, deterministic
- Immutability: Data doesn't change
- Higher-order functions: Functions as values
- Function composition: Build complex from simple
- Recursion: Alternative to loops
- Lazy evaluation: Compute on demand
- FP and OOP can be combined
- Choose paradigm based on problem

**Next Steps:**
- Practice FP in your language
- Understand your language's FP features
- Learn FP libraries and tools
- Apply FP principles gradually
- Combine FP with OOP as needed

