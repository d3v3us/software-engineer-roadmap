# Code Design Principles Deep Dive - Complete Understanding

## Table of Contents
1. [High Cohesion, Loose Coupling](#high-cohesion-loose-coupling)
2. [DRY (Don't Repeat Yourself)](#dry-dont-repeat-yourself)
3. [Refactoring](#refactoring)
4. [Code Comments](#code-comments)
5. [Design vs Architecture](#design-vs-architecture)
6. [Early Testing](#early-testing)
7. [Domain Logic in Stored Procedures](#domain-logic-in-stored-procedures)

---

## High Cohesion, Loose Coupling

### What is Cohesion?

**Cohesion**: Measure of how closely related the responsibilities of a module are.

**High Cohesion:**
- All parts work together toward a single purpose
- Related functionality grouped together
- Clear, focused responsibility

**Low Cohesion:**
- Parts have unrelated responsibilities
- Unclear purpose
- Hard to understand

### Example

**Low Cohesion:**
```python
class UserManager:
    def create_user(self, name, email):
        # User creation
        pass
    
    def send_email(self, to, subject, body):
        # Email sending (unrelated!)
        pass
    
    def calculate_tax(self, amount):
        # Tax calculation (unrelated!)
        pass
```

**High Cohesion:**
```python
class UserService:
    def create_user(self, name, email):
        # User creation
        pass
    
    def update_user(self, user_id, data):
        # User update
        pass
    
    def delete_user(self, user_id):
        # User deletion
        pass
# All methods related to user management
```

### What is Coupling?

**Coupling**: Measure of how dependent modules are on each other.

**Loose Coupling:**
- Modules are independent
- Changes in one don't affect others
- Easy to modify

**Tight Coupling:**
- Modules depend heavily on each other
- Changes cascade
- Hard to modify

### Example

**Tight Coupling:**
```python
class UserService:
    def __init__(self):
        self.db = MySQLDatabase()  # Specific implementation
        self.logger = FileLogger()  # Specific implementation
    
    def get_user(self, id):
        self.logger.log(f"Getting user {id}")
        return self.db.query(f"SELECT * FROM users WHERE id = {id}")
```

**Loose Coupling:**
```python
class UserService:
    def __init__(self, db, logger):  # Depend on interfaces
        self.db = db
        self.logger = logger
    
    def get_user(self, id):
        self.logger.log(f"Getting user {id}")
        return self.db.query(f"SELECT * FROM users WHERE id = {id}")

# Can use any database or logger implementation
```

### Benefits

**High Cohesion:**
- Easier to understand
- Easier to maintain
- Easier to test
- Easier to reuse

**Loose Coupling:**
- Independent modules
- Easy to change
- Easy to test
- Easy to reuse

---

## DRY (Don't Repeat Yourself)

### What is DRY?

**DRY Principle**: Every piece of knowledge must have a single, unambiguous, authoritative representation within a system.

**Key Points:**
- Don't duplicate code
- Don't duplicate logic
- Don't duplicate data
- Single source of truth

### Example

**Violation:**
```python
def calculate_order_total(order):
    total = 0
    for item in order.items:
        total += item.price * item.quantity
    if order.customer.is_vip:
        total *= 0.9  # 10% discount
    return total

def calculate_cart_total(cart):
    total = 0
    for item in cart.items:
        total += item.price * item.quantity
    if cart.customer.is_vip:
        total *= 0.9  # Same discount logic!
    return total
```

**DRY:**
```python
def calculate_item_total(item):
    return item.price * item.quantity

def apply_vip_discount(total, is_vip):
    return total * 0.9 if is_vip else total

def calculate_total(items, is_vip=False):
    subtotal = sum(calculate_item_total(item) for item in items)
    return apply_vip_discount(subtotal, is_vip)

# Reuse in both cases
order_total = calculate_total(order.items, order.customer.is_vip)
cart_total = calculate_total(cart.items, cart.customer.is_vip)
```

### When NOT to DRY

**Sometimes duplication is OK:**
- Different contexts (similar but different)
- Premature abstraction
- Over-abstraction makes code harder to understand

**Example:**
```python
# These look similar but serve different purposes
def calculate_area(radius):
    return 3.14159 * radius * radius

def calculate_distance(x1, y1, x2, y2):
    return ((x2 - x1) ** 2 + (y2 - y1) ** 2) ** 0.5

# Don't force them into same abstraction!
```

---

## Refactoring

### What is Refactoring?

**Refactoring**: Process of restructuring existing code without changing its external behavior.

**Key Points:**
- Improve code structure
- Don't change functionality
- Make code easier to understand
- Make code easier to maintain

### When to Refactor

**1. Code Smells:**
- Long methods
- Duplicate code
- Large classes
- Too many parameters

**2. Before Adding Features:**
- Clean code first
- Easier to add features

**3. After Understanding:**
- Now that you understand, improve it

**4. Regularly:**
- Continuous improvement
- Technical debt management

### Refactoring Techniques

**1. Extract Method:**
```python
# Before
def process_order(order):
    total = 0
    for item in order.items:
        total += item.price * item.quantity
    if order.customer.is_vip:
        total *= 0.9
    # ... more code

# After
def calculate_subtotal(items):
    return sum(item.price * item.quantity for item in items)

def apply_discount(total, is_vip):
    return total * 0.9 if is_vip else total

def process_order(order):
    total = calculate_subtotal(order.items)
    total = apply_discount(total, order.customer.is_vip)
    # ... more code
```

**2. Extract Class:**
```python
# Before
class Order:
    def __init__(self, items, customer):
        self.items = items
        self.customer = customer
    
    def calculate_total(self):
        # Calculation logic
        pass
    
    def apply_discount(self):
        # Discount logic
        pass
    
    def send_email(self):
        # Email logic
        pass

# After
class Order:
    def __init__(self, items, customer):
        self.items = items
        self.customer = customer
        self.calculator = OrderCalculator()
        self.notifier = OrderNotifier()

class OrderCalculator:
    def calculate_total(self, order):
        pass

class OrderNotifier:
    def send_email(self, order):
        pass
```

**3. Rename:**
```python
# Before
def calc(x, y):
    return x + y

# After
def calculate_sum(first_number, second_number):
    return first_number + second_number
```

### Refactoring Safety

**1. Tests:**
- Have tests before refactoring
- Tests ensure behavior doesn't change

**2. Small Steps:**
- Refactor in small increments
- Each step should work

**3. Version Control:**
- Commit frequently
- Easy to revert if needed

---

## Code Comments

### When to Comment

**Good Comments:**
- **Why**, not what
- Explain complex logic
- Document assumptions
- Warn about gotchas

**Bad Comments:**
- Obvious code
- Outdated information
- Commented-out code

### Example

**Bad:**
```python
# Increment counter
counter += 1

# Get user by ID
user = get_user(id)
```

**Good:**
```python
# Increment counter to track retry attempts
# Max 3 retries to prevent infinite loops
counter += 1

# Use cached user if available to avoid database hit
# Cache expires after 5 minutes
user = get_user(id)
```

### Self-Documenting Code

**Better than comments:**
```python
# Bad
def calc(x, y):
    return x * y * 0.1

# Good (self-documenting)
def calculate_tax_amount(price, tax_rate):
    return price * tax_rate

# Even better
def calculate_sales_tax(item_price, tax_rate_percentage):
    tax_rate_decimal = tax_rate_percentage / 100
    return item_price * tax_rate_decimal
```

---

## Design vs Architecture

### Design

**Design**: Low-level structure of code. How individual components are implemented.

**Focus:**
- Classes and methods
- Data structures
- Algorithms
- Code organization

**Example:**
- How to implement a user service
- What data structure to use
- How to structure a function

### Architecture

**Architecture**: High-level structure of system. How components interact.

**Focus:**
- System components
- Component interactions
- System boundaries
- Technology choices

**Example:**
- Microservices vs monolith
- Database choice
- Communication patterns

### Relationship

```
Architecture
    ↓
  Design
    ↓
  Code
```

**Architecture** defines the big picture.
**Design** implements the details.
**Code** is the actual implementation.

---

## Early Testing

### Why Test Early?

**Benefits:**
- Catch bugs early
- Cheaper to fix
- Better design
- Documentation

### Test-Driven Development (TDD)

**Process:**
1. Write test (Red)
2. Write code (Green)
3. Refactor (Refactor)

**Benefits:**
- Forces good design
- Tests as documentation
- Confidence to refactor

### Testing Pyramid

```
        /\
       /  \  E2E Tests (few)
      /____\
     /      \  Integration Tests (some)
    /________\
   /          \  Unit Tests (many)
  /____________\
```

**Many unit tests:**
- Fast
- Isolated
- Catch most bugs

**Some integration tests:**
- Test interactions
- Slower

**Few E2E tests:**
- Test full system
- Slowest
- Most expensive

---

## Domain Logic in Stored Procedures

### What are Stored Procedures?

**Stored Procedures**: Database procedures that contain business logic.

**Example:**
```sql
CREATE PROCEDURE CalculateOrderTotal
    @OrderId INT
AS
BEGIN
    DECLARE @Total DECIMAL(10,2)
    
    SELECT @Total = SUM(Price * Quantity)
    FROM OrderItems
    WHERE OrderId = @OrderId
    
    IF @Total > 1000
        SET @Total = @Total * 0.9  -- 10% discount
    
    UPDATE Orders
    SET Total = @Total
    WHERE Id = @OrderId
END
```

### Pros

**1. Performance:**
- Execute on database
- Less network traffic
- Database optimizations

**2. Consistency:**
- Logic in one place
- Enforced by database

**3. Security:**
- Can control access
- SQL injection protection

### Cons

**1. Language Lock-in:**
- Tied to database
- Hard to migrate

**2. Testing:**
- Hard to test
- Need database

**3. Version Control:**
- Hard to version
- Hard to review

**4. Business Logic:**
- Logic in database
- Hard to change
- Not in application code

### When to Use

**Good For:**
- Data-intensive operations
- Performance-critical
- Simple logic

**Not Good For:**
- Complex business logic
- Logic that changes often
- Logic that needs testing

### Best Practice

**Keep business logic in application:**
- Easier to test
- Easier to change
- Easier to version
- Language flexibility

**Use stored procedures for:**
- Data operations
- Performance optimization
- Simple calculations

---

## Summary

Code design principles guide us in writing maintainable, understandable software.

**Key Takeaways:**
- High cohesion: Related functionality together
- Loose coupling: Independent modules
- DRY: Don't repeat yourself
- Refactor regularly
- Comment why, not what
- Design vs Architecture: Different levels
- Test early
- Keep business logic in application code

**Next Steps:**
- Apply principles in your code
- Practice refactoring
- Write self-documenting code
- Test your code

