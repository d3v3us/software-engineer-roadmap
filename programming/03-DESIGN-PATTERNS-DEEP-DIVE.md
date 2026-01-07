# Design Patterns Deep Dive - Complete Understanding

## Table of Contents
1. [What are Design Patterns?](#what-are-design-patterns)
2. [Singleton Pattern](#singleton-pattern)
3. [Inversion of Control (IoC)](#inversion-of-control-ioc)
4. [Law of Demeter](#law-of-demeter)
5. [Active Record vs Data Mapper](#active-record-vs-data-mapper)
6. [Inheritance vs Composition](#inheritance-vs-composition)
7. [Anti-Corruption Layer](#anti-corruption-layer)
8. [Separation of Concerns](#separation-of-concerns)
9. [Don't Repeat Yourself (DRY)](#dont-repeat-yourself-dry)
10. [Dependency Hell](#dependency-hell)
11. [Globals Are Evil](#globals-are-evil)
12. [The Billion-Dollar Mistake](#the-billion-dollar-mistake)

---

## What are Design Patterns?

### Definition

**Design Patterns**: Reusable solutions to common problems in software design. They are templates for solving problems that occur repeatedly in software development.

**Key Concepts:**
- **Not code**: Patterns are ideas, not implementations
- **Proven solutions**: Solutions that have worked in the past
- **Language-agnostic**: Can be implemented in any language
- **Best practices**: Represent collective wisdom

### Why Design Patterns?

**Benefits:**
- **Reusability**: Solve common problems efficiently
- **Communication**: Common vocabulary for developers
- **Maintainability**: Well-understood solutions
- **Flexibility**: Patterns often provide flexibility

**Categories:**
- **Creational**: How objects are created
- **Structural**: How objects are composed
- **Behavioral**: How objects interact

---

## Singleton Pattern

### What is Singleton?

**Singleton Pattern**: Ensures a class has only one instance and provides global access to it.

**Use Case:**
- Database connections
- Logging
- Configuration management
- Caching

### Implementation

**Basic Singleton:**
```python
class Singleton:
    _instance = None
    
    def __new__(cls):
        if cls._instance is None:
            cls._instance = super().__new__(cls)
        return cls._instance

# Usage
s1 = Singleton()
s2 = Singleton()
print(s1 is s2)  # True (same instance)
```

### Problems with Singleton

**1. Global State:**
- Hidden dependencies
- Hard to test
- Tight coupling

**2. Thread Safety:**
- Not thread-safe by default
- Need synchronization

**3. Testing:**
- Hard to mock
- Tests affect each other

**When to Use:**
- Truly single resource (hardware, file system)
- Expensive to create
- Need global access

**When NOT to Use:**
- Most cases (prefer dependency injection)
- When you need multiple instances
- When testing is important

---

## Inversion of Control (IoC)

### What is IoC?

**Inversion of Control**: Principle where the control of object creation and dependency management is inverted from the class itself to an external framework or container.

**Traditional Approach (Control in Class):**
```python
class UserService:
    def __init__(self):
        self.db = Database()  # Creates dependency itself
        self.logger = Logger()  # Creates dependency itself
    
    def get_user(self, id):
        self.logger.log(f"Getting user {id}")
        return self.db.query(f"SELECT * FROM users WHERE id = {id}")
```

**IoC Approach (Control Outside):**
```python
class UserService:
    def __init__(self, db, logger):  # Dependencies injected
        self.db = db
        self.logger = logger
    
    def get_user(self, id):
        self.logger.log(f"Getting user {id}")
        return self.db.query(f"SELECT * FROM users WHERE id = {id}")

# Usage
db = Database()
logger = Logger()
user_service = UserService(db, logger)  # Dependencies provided externally
```

### Benefits

**1. Loose Coupling:**
- Classes don't create dependencies
- Dependencies provided externally

**2. Testability:**
- Easy to mock dependencies
- Test in isolation

**3. Flexibility:**
- Swap implementations easily
- Change behavior without changing code

**4. Single Responsibility:**
- Class focuses on its job
- Dependency management elsewhere

### Dependency Injection

**Dependency Injection**: Form of IoC where dependencies are injected into objects.

**Types:**
- **Constructor Injection**: Dependencies via constructor
- **Setter Injection**: Dependencies via setters
- **Interface Injection**: Dependencies via interface

---

## Law of Demeter

### What is Law of Demeter?

**Law of Demeter (LoD)**: Principle that states an object should only talk to its immediate friends, not to friends of friends.

**Also Known As:**
- "Don't talk to strangers"
- Principle of least knowledge

### The Rule

**Allowed:**
- Object's own methods
- Methods of objects passed as parameters
- Methods of objects created locally
- Methods of object's direct components

**Not Allowed:**
- Methods of objects returned by other methods
- Methods of objects accessed through chains

### Example

**Violation:**
```python
# Bad: Talking to friend's friend
user.get_address().get_city().get_name()

# Chain of calls:
# user → address → city → name
# Too many levels!
```

**Correct:**
```python
# Good: Direct method
user.get_city_name()

# Single call, object handles internal navigation
```

### Benefits

**1. Loose Coupling:**
- Objects don't know internal structure of other objects
- Changes don't cascade

**2. Encapsulation:**
- Internal details hidden
- Interface is clear

**3. Maintainability:**
- Changes isolated
- Easier to refactor

---

## Active Record vs Data Mapper

### Active Record Pattern

**Active Record**: Object that wraps a row in a database table, encapsulates database access, and adds domain logic.

**Characteristics:**
- Object represents both data and behavior
- Object knows how to save/load itself
- Direct database access

**Example:**
```python
class User(ActiveRecord):
    def __init__(self, name, email):
        self.name = name
        self.email = email
    
    def save(self):
        # Saves itself to database
        db.execute(f"INSERT INTO users (name, email) VALUES ({self.name}, {self.email})")
    
    @classmethod
    def find(cls, id):
        # Loads itself from database
        row = db.query(f"SELECT * FROM users WHERE id = {id}")
        return cls(row['name'], row['email'])

# Usage
user = User("Alice", "alice@example.com")
user.save()  # Object saves itself
```

**Pros:**
- Simple
- Direct
- Easy to understand

**Cons:**
- Tight coupling to database
- Hard to test
- Violates Single Responsibility

### Data Mapper Pattern

**Data Mapper**: Separates in-memory objects from database. Objects don't know about database.

**Characteristics:**
- Objects are pure domain objects
- Separate mapper handles database
- Objects don't know about persistence

**Example:**
```python
class User:
    def __init__(self, name, email):
        self.name = name
        self.email = email

class UserMapper:
    def save(self, user):
        db.execute(f"INSERT INTO users (name, email) VALUES ({user.name}, {user.email})")
    
    def find(self, id):
        row = db.query(f"SELECT * FROM users WHERE id = {id}")
        return User(row['name'], row['email'])

# Usage
user = User("Alice", "alice@example.com")
mapper = UserMapper()
mapper.save(user)  # Mapper handles persistence
```

**Pros:**
- Separation of concerns
- Easy to test
- Flexible

**Cons:**
- More complex
- More code
- More abstraction

### When to Use Each

**Active Record:**
- Simple applications
- Rapid prototyping
- Small projects

**Data Mapper:**
- Complex domain logic
- Need for testing
- Multiple data sources

---

## Inheritance vs Composition

### Inheritance

**Inheritance**: "Is-a" relationship. Child class inherits from parent class.

**Example:**
```python
class Animal:
    def speak(self):
        pass

class Dog(Animal):  # Dog IS-A Animal
    def speak(self):
        return "Woof"

class Cat(Animal):  # Cat IS-A Animal
    def speak(self):
        return "Meow"
```

**Pros:**
- Code reuse
- Polymorphism
- Clear hierarchy

**Cons:**
- Tight coupling
- Fragile base class problem
- Limited flexibility

### Composition

**Composition**: "Has-a" relationship. Object contains other objects.

**Example:**
```python
class Engine:
    def start(self):
        return "Engine started"

class Car:
    def __init__(self):
        self.engine = Engine()  # Car HAS-A Engine
    
    def start(self):
        return self.engine.start()
```

**Pros:**
- Loose coupling
- Flexible
- Easy to change

**Cons:**
- More objects
- More complex

### When to Use Each

**Inheritance:**
- True "is-a" relationship
- Need polymorphism
- Shared behavior

**Composition:**
- "Has-a" relationship
- Need flexibility
- Prefer composition over inheritance

---

## Anti-Corruption Layer

### What is Anti-Corruption Layer?

**Anti-Corruption Layer**: Layer that isolates your application from legacy systems or external systems with different models.

**Purpose:**
- Protect your domain model
- Translate between models
- Isolate legacy code

### Example

**Problem:**
```
Your Application (Modern Domain Model)
    ↓
Legacy System (Old Model)
    ↓
Corruption spreads!
```

**Solution:**
```
Your Application (Modern Domain Model)
    ↓
Anti-Corruption Layer (Translation)
    ↓
Legacy System (Old Model)
    ↓
Corruption contained!
```

**Implementation:**
```python
# Your domain model
class User:
    def __init__(self, name, email):
        self.name = name
        self.email = email

# Legacy system model
class LegacyUser:
    def __init__(self, full_name, email_address):
        self.full_name = full_name
        self.email_address = email_address

# Anti-Corruption Layer
class UserAdapter:
    @staticmethod
    def to_legacy(user):
        return LegacyUser(user.name, user.email)
    
    @staticmethod
    def from_legacy(legacy_user):
        return User(legacy_user.full_name, legacy_user.email_address)
```

### Benefits

**1. Isolation:**
- Legacy code doesn't affect your code
- Changes isolated

**2. Translation:**
- Converts between models
- Handles differences

**3. Gradual Migration:**
- Can migrate gradually
- Don't need to change everything at once

---

## Separation of Concerns

### What is Separation of Concerns?

**Separation of Concerns (SoC)**: Principle of separating a computer program into distinct sections, each addressing a separate concern.

**Key Idea:**
- Each part should do one thing
- Parts should be independent
- Changes to one part shouldn't affect others

### Example

**Bad (Mixed Concerns):**
```python
class User:
    def __init__(self, name, email):
        self.name = name
        self.email = email
    
    def save(self):
        # Database concern
        db.execute(f"INSERT INTO users (name, email) VALUES ({self.name}, {self.email})")
    
    def send_email(self):
        # Email concern
        email_service.send(self.email, "Welcome!")
    
    def validate(self):
        # Validation concern
        if not self.email or "@" not in self.email:
            raise ValueError("Invalid email")
```

**Good (Separated Concerns):**
```python
# Domain concern
class User:
    def __init__(self, name, email):
        self.name = name
        self.email = email

# Persistence concern
class UserRepository:
    def save(self, user):
        db.execute(f"INSERT INTO users (name, email) VALUES ({user.name}, {user.email})")

# Communication concern
class EmailService:
    def send_welcome(self, email):
        email_service.send(email, "Welcome!")

# Validation concern
class UserValidator:
    def validate(self, user):
        if not user.email or "@" not in user.email:
            raise ValueError("Invalid email")
```

### Benefits

**1. Maintainability:**
- Easy to find code
- Easy to change

**2. Testability:**
- Test each concern separately
- Mock dependencies easily

**3. Reusability:**
- Reuse concerns in different contexts
- Mix and match

---

## Don't Repeat Yourself (DRY)

### What is DRY?

**DRY Principle**: Every piece of knowledge must have a single, unambiguous, authoritative representation within a system.

**Key Idea:**
- Don't duplicate code
- Don't duplicate logic
- Single source of truth

### Example

**Violation:**
```python
def calculate_total(items):
    total = 0
    for item in items:
        total += item.price * item.quantity
    return total

def calculate_subtotal(items):
    subtotal = 0
    for item in items:
        subtotal += item.price * item.quantity
    return subtotal

# Same logic repeated!
```

**DRY:**
```python
def calculate_item_total(item):
    return item.price * item.quantity

def calculate_total(items):
    return sum(calculate_item_total(item) for item in items)

def calculate_subtotal(items):
    return calculate_total(items)  # Reuse

# Single source of truth
```

### Benefits

**1. Maintainability:**
- Change in one place
- No inconsistencies

**2. Consistency:**
- Same logic everywhere
- No bugs from differences

**3. Less Code:**
- Smaller codebase
- Easier to understand

---

## Dependency Hell

### What is Dependency Hell?

**Dependency Hell**: Situation where dependencies conflict with each other, making it difficult or impossible to install or use software.

### Causes

**1. Version Conflicts:**
```
App needs: Library A v1.0
App needs: Library B v2.0
Library B needs: Library A v2.0
→ Conflict! Can't have both versions
```

**2. Circular Dependencies:**
```
A depends on B
B depends on C
C depends on A
→ Circular! Can't resolve
```

**3. Too Many Dependencies:**
```
App
├── Library A
│   ├── Library X
│   ├── Library Y
│   └── Library Z
│       ├── Library M
│       └── Library N
└── Library B
    └── Library X (different version!)
→ Too complex!
```

### Solutions

**1. Dependency Management:**
- Use package managers
- Lock versions
- Resolve conflicts

**2. Minimize Dependencies:**
- Only add what you need
- Review dependencies regularly

**3. Use Interfaces:**
- Depend on interfaces, not implementations
- Swap implementations easily

---

## Globals Are Evil

### Why Globals Are Evil?

**Problems with Global Variables:**

**1. Hidden Dependencies:**
```python
# Global variable
config = {"debug": True}

def process_data(data):
    if config["debug"]:  # Hidden dependency!
        print("Processing:", data)
    # ...
```

**2. Hard to Test:**
```python
# Test
def test_process_data():
    # How to test with different config?
    # Must modify global state
    global config
    config = {"debug": False}
    # ...
```

**3. Concurrency Issues:**
```python
# Thread 1
config["value"] = 10

# Thread 2
config["value"] = 20  # Race condition!
```

**4. Tight Coupling:**
- Code depends on global state
- Hard to change
- Hard to reuse

### Alternatives

**1. Dependency Injection:**
```python
def process_data(data, config):
    if config["debug"]:
        print("Processing:", data)
```

**2. Configuration Object:**
```python
class Config:
    def __init__(self, debug=False):
        self.debug = debug

config = Config(debug=True)
process_data(data, config)
```

**3. Context/Environment:**
```python
class Context:
    def __init__(self, config):
        self.config = config

context = Context(config)
process_data(data, context)
```

---

## The Billion-Dollar Mistake

### What is the Billion-Dollar Mistake?

**The Billion-Dollar Mistake**: Null references, introduced by Tony Hoare in 1965, who called it his "billion-dollar mistake."

**The Problem:**
```python
def get_user(id):
    user = db.find_user(id)
    return user.name  # What if user is None?
    # → NullPointerException / AttributeError
```

### Why It's a Problem

**1. Runtime Errors:**
- Null pointer exceptions
- Hard to debug
- Crashes at runtime

**2. Defensive Programming:**
```python
def get_user(id):
    user = db.find_user(id)
    if user is None:
        return None  # Or raise exception?
    return user.name
```

**3. Propagation:**
```python
def process_user(id):
    user = get_user(id)  # Might be None
    if user is None:
        return None
    name = user.name  # Might be None
    if name is None:
        return None
    # ...
```

### Solutions

**1. Optional/Maybe Types:**
```python
from typing import Optional

def get_user(id) -> Optional[User]:
    return db.find_user(id)

# Forces handling of None
user = get_user(id)
if user is not None:
    process(user)
```

**2. Null Object Pattern:**
```python
class NullUser:
    def get_name(self):
        return "Unknown"

def get_user(id):
    user = db.find_user(id)
    return user if user else NullUser()
```

**3. Result Types:**
```python
from typing import Union

def get_user(id) -> Union[User, Error]:
    user = db.find_user(id)
    if user:
        return user
    return Error("User not found")
```

---

## Summary

Design patterns provide proven solutions to common problems. Understanding when and how to use them is essential for building maintainable software.

**Key Takeaways:**
- Design patterns are reusable solutions
- Singleton: One instance, but use carefully
- IoC: Invert control, inject dependencies
- Law of Demeter: Don't talk to strangers
- Active Record vs Data Mapper: Choose based on complexity
- Inheritance vs Composition: Prefer composition
- Anti-Corruption Layer: Isolate legacy systems
- Separation of Concerns: One concern per component
- DRY: Don't repeat yourself
- Avoid dependency hell
- Avoid globals
- Handle nulls carefully

**Next Steps:**
- Study specific patterns in depth
- Practice implementing patterns
- Understand trade-offs
- Know when NOT to use patterns

