# Programming Paradigm - Comprehensive Guide

## Table of Contents
1. [Object-Oriented Programming (OOP)](#object-oriented-programming-oop)
2. [Functional Programming (FP)](#functional-programming-fp)

---

## Object-Oriented Programming (OOP)

### What is OOP?

**Object-Oriented Programming** is a programming paradigm based on the concept of "objects" that contain data (attributes) and code (methods).

**Core Concept:**
- **Objects**: Instances of classes that encapsulate data and behavior
- **Classes**: Blueprints for creating objects
- **Encapsulation**: Bundling data and methods together
- **Abstraction**: Hiding complex implementation details

**Analogy:**
Think of a **car**:
- **Class**: Blueprint for a car
- **Object**: A specific car (e.g., "my red Toyota")
- **Attributes**: Color, model, speed
- **Methods**: Start, stop, accelerate

### Why Use OOP?

**Benefits:**

**1. Modularity:**
```
Code organized into classes
Each class has specific responsibility
Easy to understand and maintain
```

**2. Reusability:**
```
Create class once
Use it many times (create multiple objects)
```

**3. Maintainability:**
```
Changes isolated to specific classes
Easier to update and fix bugs
```

**4. Modeling Real World:**
```
Natural way to model real-world entities
Intuitive for many developers
```

### Four Principles of OOP

**1. Encapsulation**

**Definition**: Bundling data and methods that operate on that data within a single unit (class).

**Purpose**: Hide internal implementation, expose only necessary interface.

**Example:**
```python
class BankAccount:
    def __init__(self, balance):
        self._balance = balance  # Private (encapsulated)
    
    def deposit(self, amount):
        if amount > 0:
            self._balance += amount
    
    def get_balance(self):
        return self._balance
```

**Benefits:**
- Control access to data
- Prevent invalid states
- Change implementation without affecting users

**Visual:**
```
┌─────────────────────┐
│   BankAccount       │
│                     │
│  ┌───────────────┐  │
│  │ _balance      │  │ ← Encapsulated (private)
│  └───────────────┘  │
│                     │
│  deposit()          │ ← Public interface
│  get_balance()      │
└─────────────────────┘
```

**2. Inheritance**

**Definition**: Mechanism where a new class (child) inherits properties and methods from existing class (parent).

**Purpose**: Code reuse, establish relationships.

**Example:**
```python
class Animal:
    def __init__(self, name):
        self.name = name
    
    def speak(self):
        return "Some sound"

class Dog(Animal):  # Inherits from Animal
    def speak(self):  # Override method
        return "Woof!"

class Cat(Animal):  # Inherits from Animal
    def speak(self):  # Override method
        return "Meow!"
```

**Visual:**
```
        Animal (Parent)
           │
    ┌──────┴──────┐
    │             │
   Dog          Cat (Children)
```

**Benefits:**
- Code reuse
- Polymorphism
- Hierarchical organization

**3. Polymorphism**

**Definition**: Ability of different classes to be treated through the same interface.

**Types:**

**a) Method Overriding:**
```python
class Shape:
    def area(self):
        pass

class Circle(Shape):
    def area(self):
        return 3.14 * self.radius ** 2

class Rectangle(Shape):
    def area(self):
        return self.width * self.height

# Polymorphism in action
shapes = [Circle(5), Rectangle(3, 4)]
for shape in shapes:
    print(shape.area())  # Different implementation, same interface
```

**b) Method Overloading:**
```python
# Python doesn't support true overloading, but concept exists
class Calculator:
    def add(self, a, b):
        return a + b
    
    def add(self, a, b, c):  # In languages that support it
        return a + b + c
```

**Visual:**
```
Interface: area()
    │
    ├──→ Circle.area() → πr²
    ├──→ Rectangle.area() → w×h
    └──→ Triangle.area() → ½bh
```

**4. Abstraction**

**Definition**: Hiding complex implementation details, showing only essential features.

**Purpose**: Simplify interface, focus on what, not how.

**Example:**
```python
from abc import ABC, abstractmethod

class Vehicle(ABC):  # Abstract class
    @abstractmethod
    def start(self):
        pass
    
    @abstractmethod
    def stop(self):
        pass

class Car(Vehicle):
    def start(self):
        # Complex engine start logic
        print("Engine started")
    
    def stop(self):
        # Complex engine stop logic
        print("Engine stopped")
```

**Visual:**
```
User sees:
  vehicle.start()
  vehicle.stop()

Hidden:
  Engine ignition
  Fuel injection
  Spark plugs
  Transmission
  ...
```

### Composition vs Inheritance

**Inheritance: "is-a" relationship**

**Example:**
```python
class Car(Vehicle):  # Car IS-A Vehicle
    pass
```

**Composition: "has-a" relationship**

**Example:**
```python
class Engine:
    def start(self):
        print("Engine started")

class Car:
    def __init__(self):
        self.engine = Engine()  # Car HAS-A Engine
    
    def start(self):
        self.engine.start()
```

**Comparison:**

| Aspect | Inheritance | Composition |
|--------|-------------|-------------|
| **Relationship** | is-a | has-a |
| **Flexibility** | Less flexible | More flexible |
| **Coupling** | Tight coupling | Loose coupling |
| **Reusability** | Limited | High |
| **Runtime Changes** | No | Yes |

**When to use:**

**Inheritance:**
- Clear "is-a" relationship
- Need polymorphism
- Shared behavior

**Composition:**
- "has-a" relationship
- Need flexibility
- Avoid deep hierarchies

**Favor Composition over Inheritance** (Design principle)

### Interface vs Abstract Class

**Interface:**
- **Definition**: Contract that defines what methods a class must implement
- **Implementation**: No implementation, only method signatures
- **Multiple Inheritance**: Can implement multiple interfaces
- **Use**: Define contracts

**Example (Java-like):**
```java
interface Flyable {
    void fly();
}

interface Swimmable {
    void swim();
}

class Duck implements Flyable, Swimmable {
    public void fly() { ... }
    public void swim() { ... }
}
```

**Abstract Class:**
- **Definition**: Class that cannot be instantiated, may have some implementation
- **Implementation**: Can have both abstract and concrete methods
- **Multiple Inheritance**: Usually single inheritance
- **Use**: Share code, define common behavior

**Example:**
```python
from abc import ABC, abstractmethod

class Animal(ABC):
    def __init__(self, name):
        self.name = name  # Concrete implementation
    
    @abstractmethod
    def speak(self):  # Abstract method
        pass
```

**Comparison:**

| Aspect | Interface | Abstract Class |
|--------|-----------|----------------|
| **Implementation** | None | Some/all |
| **Inheritance** | Multiple | Single (usually) |
| **Fields** | Constants only | Any fields |
| **Use Case** | Contract | Shared code |

### Class Constructor

**Constructor**: Special method called when object is created.

**Purpose:**
- Initialize object state
- Set initial values
- Allocate resources

**Example:**
```python
class Person:
    def __init__(self, name, age):  # Constructor
        self.name = name
        self.age = age

person = Person("Alice", 30)  # Constructor called
```

**Why Constructor Doesn't Return Value?**

**Reason**: Constructor's job is to initialize object, not return value. Object creation and initialization are separate from returning.

**What happens:**
```
1. Memory allocated for object
2. Constructor called to initialize
3. Reference to object returned (implicitly)
```

**If constructor returned value:**
- Confusing (what would it return?)
- Breaks object creation pattern
- Not needed (object reference is what we want)

### Access Modifiers

**Access Modifiers**: Control visibility and accessibility of class members.

**Types:**

**1. Public:**
- **Access**: Accessible from anywhere
- **Python**: No prefix (default)
- **Java/C++**: `public`

```python
class Example:
    def public_method(self):  # Public
        return "Accessible from anywhere"
```

**2. Private:**
- **Access**: Only within same class
- **Python**: `__` prefix (name mangling)
- **Java/C++**: `private`

```python
class Example:
    def __private_method(self):  # Private
        return "Only accessible in this class"
```

**3. Protected:**
- **Access**: Same class and subclasses
- **Python**: `_` prefix (convention)
- **Java/C++**: `protected`

```python
class Example:
    def _protected_method(self):  # Protected
        return "Accessible in class and subclasses"
```

**Visual:**
```
┌─────────────────────┐
│      Class          │
│                     │
│  public_method()    │ ← Public (anywhere)
│  _protected_method()│ ← Protected (class + subclasses)
│  __private_method() │ ← Private (class only)
└─────────────────────┘
```

**Differences:**

| Modifier | Access Level | Use Case |
|----------|--------------|----------|
| **Public** | Anywhere | Interface, API |
| **Protected** | Class + Subclasses | Internal implementation, extensibility |
| **Private** | Class only | Internal details, encapsulation |

---

## Functional Programming (FP)

### What is Functional Programming?

**Functional Programming** is a programming paradigm that treats computation as evaluation of mathematical functions and avoids changing state and mutable data.

**Core Principles:**
- **Functions as first-class citizens**
- **Immutability**: Data doesn't change
- **Pure functions**: No side effects
- **Higher-order functions**: Functions that take/return functions

**Analogy:**
- **Imperative**: "How to do" (step-by-step instructions)
- **Functional**: "What to do" (declarative, mathematical)

**Example:**
```python
# Imperative (how)
result = []
for x in numbers:
    if x > 0:
        result.append(x * 2)

# Functional (what)
result = [x * 2 for x in numbers if x > 0]
# Or: list(map(lambda x: x * 2, filter(lambda x: x > 0, numbers)))
```

### Why Use Functional Programming?

**Benefits:**

**1. Predictability:**
```
Pure functions always return same output for same input
Easier to reason about
```

**2. Testability:**
```
No side effects
Easy to test (just test input → output)
```

**3. Concurrency:**
```
Immutable data
No race conditions
Easier parallelization
```

**4. Composability:**
```
Small functions combine into larger ones
Build complex behavior from simple parts
```

### FP Characteristics

#### Immutability

**Definition**: Data cannot be changed after creation. Instead, create new data.

**Why Important:**
- **No side effects**: Can't accidentally modify data
- **Thread safety**: No race conditions
- **Predictability**: Data doesn't change unexpectedly

**Example:**
```python
# Mutable (bad in FP)
numbers = [1, 2, 3]
numbers.append(4)  # Modifies original
# numbers = [1, 2, 3, 4]

# Immutable (good in FP)
numbers = [1, 2, 3]
new_numbers = numbers + [4]  # Creates new list
# numbers = [1, 2, 3] (unchanged)
# new_numbers = [1, 2, 3, 4]
```

**Visual:**
```
Mutable:
[1, 2, 3] → modify → [1, 2, 3, 4]
(same object)

Immutability:
[1, 2, 3] → create new → [1, 2, 3, 4]
(different object)
```

#### Pure Functions

**Definition**: Function that:
1. Always returns same output for same input
2. Has no side effects (doesn't modify external state)

**Why Important:**
- **Predictable**: Same input → same output
- **Testable**: Easy to test
- **Composable**: Can combine safely
- **Cacheable**: Can memoize results

**Example:**
```python
# Pure function
def add(a, b):
    return a + b  # No side effects, deterministic

# Impure function
counter = 0
def increment():
    global counter
    counter += 1  # Side effect: modifies global state
    return counter
```

**Pure Function Benefits:**
```
Input: (2, 3)
Output: 5 (always)

Can cache:
  add(2, 3) → 5 (cache this)
  add(2, 3) → 5 (use cache, no recomputation)
```

#### First-Class Functions

**Definition**: Functions are treated as values - can be assigned, passed as arguments, returned.

**Example:**
```python
# Assign function to variable
add = lambda x, y: x + y

# Pass function as argument
def apply(func, x, y):
    return func(x, y)

result = apply(add, 2, 3)  # 5

# Return function
def make_multiplier(n):
    return lambda x: x * n

double = make_multiplier(2)
result = double(5)  # 10
```

**Why Important:**
- Enables higher-order functions
- Enables function composition
- More flexible code

### Lazy Evaluation

**Definition**: Expressions are not evaluated until their value is needed.

**Benefits:**
- **Efficiency**: Don't compute what you don't need
- **Infinite sequences**: Can work with infinite data
- **Memory**: Only compute what's used

**Example:**
```python
# Eager evaluation (immediate)
numbers = [x * 2 for x in range(1000000)]  # Computes all immediately

# Lazy evaluation (on demand)
numbers = (x * 2 for x in range(1000000))  # Generator, computes on demand

# Only computes when iterated
for num in numbers:  # Computes one at a time
    print(num)
    if num > 10:
        break  # Stops early, doesn't compute rest
```

**Visual:**
```
Eager:
[0, 2, 4, 6, 8, ...] ← All computed immediately

Lazy:
(0, 2, 4, 6, 8, ...) ← Computed on demand
```

### Higher-Order Functions

**Definition**: Functions that take other functions as arguments or return functions.

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

**4. Custom Higher-Order Function:**
```python
def apply_twice(func, value):
    return func(func(value))

result = apply_twice(lambda x: x * 2, 3)
# apply_twice(3) → 6 → 12
```

### Currying and Partial Application

**Currying:**
Converting function with multiple arguments into sequence of functions, each taking one argument.

**Example:**
```python
# Normal function
def add(a, b, c):
    return a + b + c

# Curried version
def add_curried(a):
    def inner(b):
        def innermost(c):
            return a + b + c
        return innermost
    return inner

# Usage
result = add_curried(1)(2)(3)  # 6
```

**Or with lambda:**
```python
add_curried = lambda a: lambda b: lambda c: a + b + c
```

**Partial Application:**
Fixing some arguments of a function, creating new function with fewer arguments.

**Example:**
```python
from functools import partial

def multiply(a, b, c):
    return a * b * c

# Partial application: fix first argument
multiply_by_2 = partial(multiply, 2)
result = multiply_by_2(3, 4)  # 2 * 3 * 4 = 24
```

**Difference:**
- **Currying**: Always one argument at a time
- **Partial Application**: Can fix any number of arguments

### Pure vs Impure Functions

**Pure Functions:**
```python
def pure_add(a, b):
    return a + b  # No side effects, deterministic
```

**Characteristics:**
- Same input → same output
- No side effects
- No external dependencies
- Easier to test and reason about

**Impure Functions:**
```python
import random

def impure_add(a):
    return a + random.randint(1, 10)  # Non-deterministic
```

**Characteristics:**
- May return different output for same input
- Has side effects or dependencies
- Harder to test
- But sometimes necessary (I/O, randomness)

### FlatMap vs Map

**Map:**
Applies function to each element, returns list of results.

```python
numbers = [1, 2, 3]
squared = list(map(lambda x: x ** 2, numbers))
# [1, 4, 9]
```

**FlatMap:**
Applies function that returns list, then flattens result.

```python
def get_factors(n):
    return [i for i in range(1, n + 1) if n % i == 0]

numbers = [6, 8, 9]

# Map (nested lists)
factors_map = list(map(get_factors, numbers))
# [[1, 2, 3, 6], [1, 2, 4, 8], [1, 3, 9]]

# FlatMap (flattened)
factors_flatmap = [f for n in numbers for f in get_factors(n)]
# [1, 2, 3, 6, 1, 2, 4, 8, 1, 3, 9]
```

**Visual:**
```
Map:
[1, 2, 3] → [f(1), f(2), f(3)] → [[a, b], [c], [d, e]]

FlatMap:
[1, 2, 3] → [f(1), f(2), f(3)] → [a, b, c, d, e] (flattened)
```

### Functor, Applicative, Monoid, Monad

**These are advanced FP concepts. Brief overview:**

**1. Functor:**
Type that can be mapped over (has `map` function).

```python
# List is a functor
numbers = [1, 2, 3]
squared = list(map(lambda x: x ** 2, numbers))
```

**2. Applicative:**
Functor that can apply function wrapped in context.

```python
# Concept: Apply function in context to value in context
# [f, g] <*> [1, 2] → [f(1), f(2), g(1), g(2)]
```

**3. Monoid:**
Type with:
- Associative binary operation
- Identity element

```python
# Example: Addition
# Associative: (a + b) + c = a + (b + c)
# Identity: 0 (a + 0 = a)
```

**4. Monad:**
Type that can chain operations (has `bind`/`flatMap`).

```python
# Concept: Chain operations that return wrapped values
# Maybe/Option is a monad
def divide(a, b):
    return Some(a / b) if b != 0 else None

result = Some(10).bind(lambda x: divide(x, 2)).bind(lambda x: divide(x, 5))
```

**These are deep topics. For interviews, understanding the concepts is more important than implementation details.**

---

## Summary

Understanding both OOP and FP paradigms is crucial for backend development. Each has its strengths and use cases.

**OOP Key Takeaways:**
- Encapsulation, Inheritance, Polymorphism, Abstraction
- Composition over inheritance
- Interfaces vs abstract classes
- Access modifiers control visibility

**FP Key Takeaways:**
- Immutability prevents side effects
- Pure functions are predictable and testable
- Higher-order functions enable composition
- First-class functions provide flexibility

**In Practice:**
- Many languages support both paradigms
- Use OOP for modeling entities and relationships
- Use FP for data transformations and computations
- Best code often combines both approaches

