# Domain-Driven Design (DDD) Deep Dive - Complete Understanding

## Table of Contents
1. [What is Domain-Driven Design?](#what-is-domain-driven-design)
2. [The Core Philosophy - Ubiquitous Language](#the-core-philosophy---ubiquitous-language)
3. [Strategic Design - Bounded Contexts](#strategic-design---bounded-contexts)
4. [Context Mapping - Relationships Between Contexts](#context-mapping---relationships-between-contexts)
5. [Tactical Design - Building Blocks](#tactical-design---building-blocks)
6. [Entities vs Value Objects](#entities-vs-value-objects)
7. [Aggregates - Consistency Boundaries](#aggregates---consistency-boundaries)
8. [Domain Services - Cross-Aggregate Logic](#domain-services---cross-aggregate-logic)
9. [Repositories - Data Access Abstraction](#repositories---data-access-abstraction)
10. [Domain Events - Communication Between Bounded Contexts](#domain-events---communication-between-bounded-contexts)
11. [Implementing DDD - Practical Guide](#implementing-ddd---practical-guide)
12. [DDD Patterns and Anti-Patterns](#ddd-patterns-and-anti-patterns)

---

## What is Domain-Driven Design?

### Definition

**Domain-Driven Design (DDD)**: Software development approach that focuses on modeling software to match a domain according to input from domain experts.

**Core Principle:**
- **Domain is central**: Business domain drives design
- **Collaboration**: Developers and domain experts work together
- **Ubiquitous language**: Shared vocabulary
- **Model reflects domain**: Code structure mirrors business

### The Problem DDD Solves

**Traditional Approach:**
```
Technical requirements → Code
Business logic scattered
Hard to understand business
Changes are difficult
```

**DDD Approach:**
```
Domain model → Code
Business logic explicit
Easy to understand business
Changes align with domain
```

### Real-World Analogy

**DDD = Architectural Blueprint:**
- **Traditional**: Build house based on materials available
- **DDD**: Understand how family lives → Design house for their needs
- **Result**: House that fits the family, not just a structure

---

## The Core Philosophy - Ubiquitous Language

### What is Ubiquitous Language?

**Ubiquitous Language**: Common language used by developers and domain experts to describe the domain.

**Key Points:**
- **Shared vocabulary**: Same terms for same concepts
- **No translation**: No need to translate between business and tech
- **Living language**: Evolves with domain understanding
- **In code**: Use same terms in code

### Example

**Without Ubiquitous Language:**
```
Domain Expert: "When customer places order, we reserve inventory"
Developer: "So User entity calls OrderService.createOrder() which updates InventoryDAO"
→ Translation needed, misunderstandings
```

**With Ubiquitous Language:**
```
Domain Expert: "When customer places order, we reserve inventory"
Developer: "So Order entity reserves Inventory"
→ Same language, clear understanding
```

### Building Ubiquitous Language

**Process:**
```
1. Talk with domain experts
2. Identify key concepts
3. Agree on terms
4. Use terms in code
5. Refine as understanding grows
```

**Example - E-commerce:**
```
Key Terms:
- Customer (not User)
- Order (not Transaction)
- Product (not Item)
- Inventory (not Stock)
- Reservation (not Hold)
```

---

## Strategic Design - Bounded Contexts

### What is a Bounded Context?

**Bounded Context**: Explicit boundary within which a domain model applies.

**Key Concept:**
- **Model is valid within context**: Same term may mean different things in different contexts
- **Clear boundaries**: Know where model applies
- **Independent**: Can evolve independently

### Why Bounded Contexts?

**Problem Without Bounded Contexts:**
```
"Customer" means different things:
- Sales: Customer is prospect, lead, opportunity
- Shipping: Customer is delivery address, contact info
- Billing: Customer is payment method, credit limit

Single model tries to represent all
→ Becomes bloated, confusing
```

**Solution: Bounded Contexts**
```
Sales Context: Customer = Sales entity
Shipping Context: Customer = Delivery entity
Billing Context: Customer = Billing entity

Each context has its own model
Clear and focused
```

### Visual Representation

```
┌─────────────────────┐  ┌─────────────────────┐
│  Sales Context      │  │  Shipping Context   │
│                     │  │                     │
│  Customer           │  │  Customer           │
│  (Sales data)       │  │  (Address data)     │
│                     │  │                     │
│  Order              │  │  Shipment           │
│  Product            │  │  Delivery            │
└─────────────────────┘  └─────────────────────┘
```

### Identifying Bounded Contexts

**Indicators:**
- **Different teams**: Different teams work on different areas
- **Different models**: Same concept means different things
- **Different data**: Different data needs
- **Different processes**: Different business processes

---

## Context Mapping - Relationships Between Contexts

### The Need for Context Mapping

**Problem:**
```
Multiple bounded contexts
Need to work together
How do they relate?
```

**Solution: Context Mapping**
```
Define relationships between contexts
Understand dependencies
Plan integration
```

### Context Relationship Patterns

**1. Shared Kernel:**
```
Two contexts share some code
Must coordinate changes
Use sparingly
```

**2. Customer-Supplier:**
```
Upstream context (supplier)
Downstream context (customer)
Customer depends on supplier
Supplier must consider customer needs
```

**3. Conformist:**
```
Downstream conforms to upstream
No influence on upstream
Use when upstream is external/legacy
```

**4. Anticorruption Layer:**
```
Protect context from another
Translate between models
Isolate from legacy systems
```

**5. Separate Ways:**
```
No relationship
Independent development
```

**6. Partnership:**
```
Two contexts coordinate
Shared development
Mutual dependency
```

### Visual Context Map

```
        [Sales Context]
              │
              │ Customer-Supplier
              ↓
        [Shipping Context]
              │
              │ Shared Kernel
              ↓
        [Billing Context]
              │
              │ Anticorruption Layer
              ↓
        [Legacy System]
```

---

## Tactical Design - Building Blocks

### Overview

**Tactical Design**: Patterns for building domain models within bounded context.

**Building Blocks:**
- **Entity**: Object with identity
- **Value Object**: Object defined by attributes
- **Aggregate**: Cluster of related objects
- **Domain Service**: Operation that doesn't fit in entity/value object
- **Repository**: Abstraction for data access
- **Factory**: Creates complex objects
- **Domain Event**: Something that happened in domain

---

## Entities vs Value Objects

### Entity

**Definition**: Object with unique identity that persists over time.

**Characteristics:**
- **Has identity**: Unique identifier
- **Mutable**: Can change state
- **Tracked**: Tracked by identity, not attributes

**Example:**
```python
class Order:
    def __init__(self, order_id):
        self.order_id = order_id  # Identity
        self.items = []
        self.status = "pending"
    
    def add_item(self, product_id, quantity):
        # State changes, but identity remains
        self.items.append(Item(product_id, quantity))
```

**Key Point:**
```
Two orders with same items but different IDs
→ Different entities
Identity matters, not just attributes
```

### Value Object

**Definition**: Object defined by its attributes, not identity.

**Characteristics:**
- **No identity**: Defined by values
- **Immutable**: Cannot change (create new instead)
- **Equality by value**: Two with same values are equal

**Example:**
```python
class Money:
    def __init__(self, amount, currency):
        self.amount = amount
        self.currency = currency
    
    def __eq__(self, other):
        return (self.amount == other.amount and 
                self.currency == other.currency)

# Two $10 bills are the same (value, not identity)
money1 = Money(10, "USD")
money2 = Money(10, "USD")
money1 == money2  # True (same value)
```

**Key Point:**
```
Two Money objects with same amount and currency
→ Same value object
Attributes matter, not identity
```

### When to Use Each

**Use Entity When:**
- **Identity matters**: Need to track specific instance
- **State changes**: Object changes over time
- **Lifecycle**: Object created, modified, deleted

**Use Value Object When:**
- **Attributes define it**: Value is what matters
- **Immutable**: Doesn't change
- **No lifecycle**: Just a value

---

## Aggregates - Consistency Boundaries

### What is an Aggregate?

**Aggregate**: Cluster of entities and value objects treated as a single unit.

**Key Concepts:**
- **Consistency boundary**: Changes within aggregate are consistent
- **Aggregate root**: Single entry point to aggregate
- **Invariants**: Business rules enforced within aggregate

### Aggregate Example

**Order Aggregate:**
```python
class Order:  # Aggregate Root
    def __init__(self, order_id, customer_id):
        self.order_id = order_id
        self.customer_id = customer_id
        self.items = []  # Entities within aggregate
        self.shipping_address = None  # Value object
        self.total = 0
    
    def add_item(self, product_id, quantity, price):
        # Business rule: Can't add items to completed order
        if self.status == "completed":
            raise ValueError("Cannot add items to completed order")
        
        item = OrderItem(product_id, quantity, price)
        self.items.append(item)
        self.total += item.subtotal()
    
    def complete(self):
        # Business rule: Must have items
        if not self.items:
            raise ValueError("Order must have items")
        
        self.status = "completed"
```

**Invariants:**
- **Order must have items**: Enforced in `complete()`
- **Can't modify completed order**: Enforced in `add_item()`
- **Total matches items**: Calculated from items

### Aggregate Rules

**1. One Aggregate Root:**
```
Only aggregate root can be accessed from outside
Other entities accessed through root
```

**2. Consistency Within Aggregate:**
```
All changes within aggregate are consistent
No partial updates
```

**3. References Between Aggregates:**
```
Reference by ID, not direct object reference
Maintains aggregate boundaries
```

**Example:**
```python
# Good: Reference by ID
class Order:
    def __init__(self, order_id, customer_id):  # ID, not Customer object
        self.customer_id = customer_id

# Bad: Direct reference
class Order:
    def __init__(self, order_id, customer):  # Direct reference
        self.customer = customer  # Violates aggregate boundary
```

---

## Domain Services - Cross-Aggregate Logic

### What is a Domain Service?

**Domain Service**: Operation that doesn't naturally fit in an entity or value object.

**When to Use:**
- **Involves multiple aggregates**: Logic spans aggregates
- **Stateless operation**: No entity to attach to
- **Domain concept**: Important business operation

### Domain Service Example

**Transfer Service:**
```python
class TransferService:
    def transfer(self, from_account, to_account, amount):
        # Business logic involving two aggregates
        if not self.is_valid_transfer(from_account, to_account, amount):
            raise ValueError("Invalid transfer")
        
        from_account.debit(amount)
        to_account.credit(amount)
    
    def is_valid_transfer(self, from_account, to_account, amount):
        # Cross-aggregate validation
        return (from_account.has_sufficient_balance(amount) and
                from_account != to_account and
                amount > 0)
```

**Why Domain Service?**
- **Involves two aggregates**: FromAccount and ToAccount
- **No single entity**: Doesn't belong to one account
- **Important domain concept**: Transfer is business operation

---

## Repositories - Data Access Abstraction

### What is a Repository?

**Repository**: Abstraction for accessing aggregates.

**Purpose:**
- **Hide persistence**: Application doesn't know about database
- **Collection-like interface**: Treat like in-memory collection
- **Aggregate-oriented**: Load/save entire aggregates

### Repository Example

```python
class OrderRepository:
    def save(self, order):
        # Persist aggregate
        # Implementation hidden
        pass
    
    def find_by_id(self, order_id):
        # Retrieve aggregate by ID
        # Returns Order aggregate
        pass
    
    def find_by_customer(self, customer_id):
        # Find orders for customer
        # Returns list of Order aggregates
        pass
```

**Usage:**
```python
# Application code
repo = OrderRepository()
order = repo.find_by_id(123)
order.add_item(product_id, quantity)
repo.save(order)
```

**Benefits:**
- **Testable**: Can mock repository
- **Flexible**: Can change persistence implementation
- **Domain-focused**: Code thinks in domain terms

---

## Domain Events - Communication Between Bounded Contexts

### What are Domain Events?

**Domain Event**: Something that happened in the domain that other parts of the system might care about.

**Purpose:**
- **Decouple contexts**: Contexts communicate via events
- **Asynchronous**: Don't need immediate response
- **Loose coupling**: Contexts don't directly depend on each other

### Domain Event Example

```python
class OrderCompletedEvent:
    def __init__(self, order_id, customer_id, total):
        self.order_id = order_id
        self.customer_id = customer_id
        self.total = total
        self.occurred_at = datetime.now()

class Order:
    def complete(self):
        self.status = "completed"
        # Publish domain event
        event = OrderCompletedEvent(
            self.order_id,
            self.customer_id,
            self.total
        )
        DomainEventPublisher.publish(event)
```

**Other Contexts Listen:**
```
Billing Context: Listens to OrderCompletedEvent → Create invoice
Shipping Context: Listens to OrderCompletedEvent → Prepare shipment
Analytics Context: Listens to OrderCompletedEvent → Update statistics
```

---

## Implementing DDD - Practical Guide

### Step-by-Step Process

**1. Understand Domain:**
```
Talk with domain experts
Identify key concepts
Build ubiquitous language
```

**2. Identify Bounded Contexts:**
```
Find natural boundaries
Different models for different areas
```

**3. Model Within Context:**
```
Identify entities and value objects
Define aggregates
Create domain services
```

**4. Implement:**
```
Write code using ubiquitous language
Implement aggregates
Create repositories
```

**5. Integrate Contexts:**
```
Define context relationships
Implement integration (events, APIs)
```

---

## DDD Patterns and Anti-Patterns

### Patterns

**1. Aggregate Pattern:**
```
Cluster related objects
Enforce invariants
Maintain consistency
```

**2. Repository Pattern:**
```
Abstract data access
Collection-like interface
```

**3. Domain Event Pattern:**
```
Publish domain events
Decouple contexts
```

### Anti-Patterns

**1. Anemic Domain Model:**
```
Entities are just data containers
Business logic in services
→ Not DDD!
```

**2. God Object:**
```
One entity does everything
Too much responsibility
→ Violates aggregate boundaries
```

**3. Leaky Abstraction:**
```
Repository exposes database details
Application knows about SQL
→ Breaks abstraction
```

---

## Summary

Domain-Driven Design aligns software with business domain. Understanding bounded contexts, tactical patterns, and implementation is essential for building maintainable software.

**Key Takeaways:**
- DDD focuses on domain, not technology
- Ubiquitous language bridges business and code
- Bounded contexts define model boundaries
- Entities have identity, value objects don't
- Aggregates enforce consistency
- Domain services handle cross-aggregate logic
- Repositories abstract data access
- Domain events decouple contexts
- Strategic design (contexts) and tactical design (patterns) work together

**Next Steps:**
- Understand your domain
- Identify bounded contexts
- Model with domain experts
- Implement tactical patterns
- Integrate contexts
- Refine as understanding grows

