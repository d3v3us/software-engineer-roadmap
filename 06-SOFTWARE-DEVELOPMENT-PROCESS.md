# Software Development Process - Comprehensive Guide

## Table of Contents
1. [Domain-Driven Design (DDD)](#domain-driven-design-ddd)
2. [Test-Driven Development (TDD)](#test-driven-development-tdd)
3. [Behavior-Driven Development (BDD)](#behavior-driven-development-bdd)

---

## Domain-Driven Design (DDD)

### What is DDD?

**Domain-Driven Design (DDD)** is a software development approach that focuses on modeling software to match a domain according to input from that domain's experts.

**Core Idea:**
- Software should reflect the business domain
- Domain experts and developers collaborate
- Code structure mirrors business structure

**Analogy:**
Instead of building software based on technical requirements, build it based on how the business actually works. The code becomes a reflection of the business domain.

### Key Concepts

**1. Domain:**
The sphere of knowledge or activity around which the application logic revolves.

**Example:**
- E-commerce domain: Products, Orders, Customers, Payments
- Banking domain: Accounts, Transactions, Loans, Interest

**2. Domain Model:**
Abstraction that represents the domain concepts and their relationships.

**3. Ubiquitous Language:**
Common language used by developers and domain experts to describe the domain.

**Example:**
```
Domain Expert: "When a customer places an order, we reserve inventory"
Developer: "So Order entity has a relationship with Inventory entity"
Both use same terms: Order, Customer, Inventory
```

**4. Bounded Context:**
Explicit boundary within which a domain model applies.

**Visual:**
```
┌─────────────────────┐  ┌─────────────────────┐
│  E-Commerce Context │  │   Shipping Context  │
│                     │  │                     │
│  Order              │  │  Shipment           │
│  Product            │  │  Delivery Address    │
│  Customer           │  │  Carrier            │
└─────────────────────┘  └─────────────────────┘
```

**Why Bounded Contexts?**
- Different contexts may have different models for same concept
- "Customer" in sales context ≠ "Customer" in shipping context
- Prevents model pollution

### Two Base Foundations of DDD

**1. Strategic Design:**
High-level design focusing on:
- **Bounded Contexts**: Boundaries of domain models
- **Context Mapping**: Relationships between contexts
- **Ubiquitous Language**: Shared vocabulary

**Strategic Patterns:**
- **Shared Kernel**: Shared code between contexts
- **Customer-Supplier**: One context depends on another
- **Conformist**: One context conforms to another's model
- **Anticorruption Layer**: Protects one context from another

**Visual:**
```
┌──────────────┐
│   Context A  │
│              │
│  ┌────────┐  │
│  │Shared  │  │ ← Shared Kernel
│  │Kernel  │  │
│  └────────┘  │
└──────┬───────┘
       │
       │ Customer-Supplier
       │
┌──────▼───────┐
│   Context B  │
└──────────────┘
```

**2. Tactical Design:**
Low-level design patterns for building domain models within a bounded context.

**Tactical Patterns:**

**a) Entity:**
Object with unique identity that persists over time.

```python
class Order:
    def __init__(self, order_id, customer_id):
        self.order_id = order_id  # Identity
        self.customer_id = customer_id
        self.items = []
    
    def add_item(self, product_id, quantity):
        # Identity remains same, state changes
        self.items.append(Item(product_id, quantity))
```

**b) Value Object:**
Object defined by its attributes, not identity. Immutable.

```python
class Money:
    def __init__(self, amount, currency):
        self.amount = amount
        self.currency = currency
    
    def __eq__(self, other):
        return (self.amount == other.amount and 
                self.currency == other.currency)
    
    # Two $10 bills are the same (value, not identity)
```

**c) Aggregate:**
Cluster of entities and value objects treated as a single unit.

```python
class Order:  # Aggregate Root
    def __init__(self, order_id):
        self.order_id = order_id
        self.items = []  # Entities within aggregate
        self.shipping_address = None  # Value object
    
    def add_item(self, item):
        # Business logic within aggregate
        if self.is_valid():
            self.items.append(item)
```

**d) Repository:**
Abstraction for accessing aggregates.

```python
class OrderRepository:
    def save(self, order):
        # Persist aggregate
        pass
    
    def find_by_id(self, order_id):
        # Retrieve aggregate
        pass
```

**e) Domain Service:**
Service that doesn't naturally fit in an entity or value object.

```python
class TransferService:
    def transfer(self, from_account, to_account, amount):
        # Business logic that involves multiple aggregates
        if self.is_valid_transfer(from_account, to_account, amount):
            from_account.debit(amount)
            to_account.credit(amount)
```

**f) Factory:**
Creates complex aggregates or entities.

```python
class OrderFactory:
    def create_order(self, customer_id, items):
        # Complex creation logic
        order = Order(customer_id)
        for item in items:
            order.add_item(item)
        order.validate()
        return order
```

### DDD Benefits

**1. Better Communication:**
- Developers and domain experts speak same language
- Reduced misunderstandings

**2. Maintainable Code:**
- Code structure mirrors business structure
- Changes in business → clear changes in code

**3. Focus on Business Value:**
- Technical concerns separated from business logic
- Business logic is explicit and testable

**4. Scalable Architecture:**
- Bounded contexts enable independent development
- Teams can work on different contexts

### DDD Challenges

**1. Complexity:**
- Can be overkill for simple applications
- Requires domain expertise

**2. Learning Curve:**
- Team needs to understand DDD concepts
- Requires discipline

**3. Initial Overhead:**
- More upfront design
- Slower initial development

---

## Test-Driven Development (TDD)

### What is TDD?

**Test-Driven Development (TDD)** is a software development process where you write tests before writing the code that makes those tests pass.

**Process (Red-Green-Refactor):**

**1. Red: Write failing test**
```python
def test_add():
    assert add(2, 3) == 5
# Test fails (function doesn't exist)
```

**2. Green: Write minimal code to pass**
```python
def add(a, b):
    return a + b
# Test passes
```

**3. Refactor: Improve code while keeping tests green**
```python
def add(a, b):
    # Maybe add validation, error handling
    if not isinstance(a, (int, float)) or not isinstance(b, (int, float)):
        raise TypeError("Arguments must be numbers")
    return a + b
# Tests still pass
```

**Visual Cycle:**
```
┌─────────┐
│   Red   │ ← Write failing test
└────┬────┘
     │
     ▼
┌─────────┐
│  Green  │ ← Write code to pass
└────┬────┘
     │
     ▼
┌──────────┐
│ Refactor │ ← Improve code
└────┬─────┘
     │
     └─────┐
           │
           ▼
      (Repeat)
```

### TDD Benefits

**1. Better Design:**
- Writing tests first forces you to think about interface
- Code is more testable (by design)

**2. Confidence:**
- Tests provide safety net
- Refactor with confidence

**3. Documentation:**
- Tests serve as executable documentation
- Show how code should be used

**4. Fewer Bugs:**
- Catch bugs early
- Prevent regressions

**5. Faster Development:**
- Counterintuitive, but true
- Less debugging time
- Clear requirements

### TDD Example

**Feature: Calculate total price with tax**

**Step 1: Red (Write failing test)**
```python
def test_calculate_total_with_tax():
    assert calculate_total(100, 0.1) == 110
# Test fails: function doesn't exist
```

**Step 2: Green (Write minimal code)**
```python
def calculate_total(price, tax_rate):
    return price * (1 + tax_rate)
# Test passes
```

**Step 3: Refactor (Improve)**
```python
def calculate_total(price, tax_rate):
    if price < 0:
        raise ValueError("Price cannot be negative")
    if tax_rate < 0 or tax_rate > 1:
        raise ValueError("Tax rate must be between 0 and 1")
    return round(price * (1 + tax_rate), 2)
# Tests still pass (add more tests for edge cases)
```

**Step 4: Add more tests**
```python
def test_calculate_total_negative_price():
    with pytest.raises(ValueError):
        calculate_total(-100, 0.1)

def test_calculate_total_invalid_tax():
    with pytest.raises(ValueError):
        calculate_total(100, 1.5)
```

### TDD Best Practices

**1. Write Small Tests:**
- One concept per test
- Easy to understand and maintain

**2. Test Behavior, Not Implementation:**
```python
# Good: Tests behavior
def test_user_can_login():
    user = create_user("alice", "password123")
    assert login("alice", "password123") == True

# Bad: Tests implementation
def test_login_sets_session():
    # Too specific, breaks if implementation changes
    assert session.get("user_id") is not None
```

**3. Keep Tests Fast:**
- Unit tests should be fast
- Use mocks for slow operations (database, network)

**4. Test Edge Cases:**
- Null values
- Empty inputs
- Boundary conditions
- Error cases

**5. Maintain Test Quality:**
- Tests are code too
- Keep them clean and readable
- Refactor tests when needed

### TDD Challenges

**1. Learning Curve:**
- Requires discipline
- Can feel slow initially

**2. Over-Testing:**
- Testing implementation details
- Too many tests for simple code

**3. Maintenance:**
- Tests need maintenance too
- Can become outdated

**4. Not Always Applicable:**
- UI testing can be difficult
- Exploratory work may not benefit

---

## Behavior-Driven Development (BDD)

### What is BDD?

**Behavior-Driven Development (BDD)** extends TDD by writing tests in a natural language that non-programmers can read.

**Focus:**
- **Behavior**: What the system should do
- **Business Value**: Why the feature exists
- **Collaboration**: Developers, QA, business analysts work together

### BDD Structure

**Given-When-Then format:**

```
Given [initial context]
When [event occurs]
Then [expected outcome]
```

**Example:**
```
Feature: User Login

Scenario: Successful login
  Given a user exists with username "alice" and password "password123"
  When the user logs in with username "alice" and password "password123"
  Then the user should be logged in
  And the user should see the dashboard

Scenario: Failed login with wrong password
  Given a user exists with username "alice" and password "password123"
  When the user logs in with username "alice" and password "wrongpassword"
  Then the user should see an error message "Invalid credentials"
  And the user should not be logged in
```

### BDD Tools

**1. Cucumber (Ruby, Java, JavaScript):**
```gherkin
Feature: Calculator
  Scenario: Add two numbers
    Given I have a calculator
    When I add 2 and 3
    Then the result should be 5
```

**2. Behave (Python):**
```python
# steps.py
from behave import given, when, then

@given('I have a calculator')
def step_impl(context):
    context.calculator = Calculator()

@when('I add {a:d} and {b:d}')
def step_impl(context, a, b):
    context.result = context.calculator.add(a, b)

@then('the result should be {expected:d}')
def step_impl(context, expected):
    assert context.result == expected
```

**3. SpecFlow (.NET):**
Similar to Cucumber for .NET

### BDD Benefits

**1. Shared Understanding:**
- Business and technical teams use same language
- Requirements are executable

**2. Living Documentation:**
- Scenarios serve as documentation
- Always up-to-date (they're tests)

**3. Focus on Behavior:**
- Tests describe what system does
- Not how it's implemented

**4. Collaboration:**
- Business analysts can write scenarios
- Developers implement steps
- QA validates behavior

### BDD vs TDD

**TDD:**
- Technical focus
- Developer-written tests
- Unit level
- Code-centric

**BDD:**
- Business focus
- Business-readable scenarios
- Integration/acceptance level
- Behavior-centric

**Relationship:**
- BDD can use TDD for implementation
- BDD scenarios can drive TDD cycles
- Complementary, not competing

**Visual:**
```
BDD (High Level)
  ↓
Feature: User Login
  ↓
TDD (Implementation)
  ↓
test_login_success()
test_login_failure()
  ↓
Code
```

### BDD Best Practices

**1. Write Clear Scenarios:**
- Use domain language
- Be specific but not overly detailed

**2. Keep Scenarios Independent:**
- Each scenario should be runnable alone
- Don't depend on other scenarios

**3. Focus on Behavior:**
- Describe what, not how
- Avoid implementation details

**4. Use Background for Common Setup:**
```gherkin
Background:
  Given the system is running
  And I am logged in as an admin
```

**5. Use Data Tables:**
```gherkin
Scenario: Calculate totals
  Given the following products:
    | name  | price |
    | Apple | 1.00  |
    | Banana| 0.50  |
  When I calculate the total
  Then the total should be 1.50
```

### BDD Challenges

**1. Tool Learning:**
- Need to learn BDD tools
- Can be complex for simple cases

**2. Maintenance:**
- Scenarios need maintenance
- Can become outdated

**3. Overhead:**
- More setup than unit tests
- May be overkill for simple features

**4. Misuse:**
- Writing scenarios that test implementation
- Too technical for business users

---

## Summary

Software development processes help create better software through structured approaches.

**DDD Key Takeaways:**
- Model software to match business domain
- Use ubiquitous language
- Define bounded contexts
- Strategic and tactical design patterns

**TDD Key Takeaways:**
- Write tests before code
- Red-Green-Refactor cycle
- Tests provide confidence and documentation
- Better design through testability

**BDD Key Takeaways:**
- Focus on behavior, not implementation
- Business-readable scenarios
- Collaboration between teams
- Living documentation

**In Practice:**
- These approaches complement each other
- Use what works for your team and project
- Adapt to your context
- Focus on delivering value

**Combining Approaches:**
```
BDD: Define behavior (high level)
  ↓
TDD: Implement with tests (unit level)
  ↓
DDD: Structure code to match domain
  ↓
Working software that delivers value
```

