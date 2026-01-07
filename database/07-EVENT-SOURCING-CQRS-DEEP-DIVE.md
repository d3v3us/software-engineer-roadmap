# Event Sourcing and CQRS Deep Dive - Complete Understanding

## Table of Contents
1. [Event Sourcing - Storing Events, Not State](#event-sourcing---storing-events-not-state)
2. [CQRS - Command Query Responsibility Segregation](#cqrs---command-query-responsibility-segregation)
3. [Event Sourcing + CQRS Together](#event-sourcing--cqrs-together)
4. [Saga Pattern - Distributed Transactions](#saga-pattern---distributed-transactions)
5. [Implementation Examples](#implementation-examples)
6. [When to Use](#when-to-use)

---

## Event Sourcing - Storing Events, Not State

### What is Event Sourcing?

**Event Sourcing**: Store all changes as a sequence of events, rather than storing current state.

**Traditional Approach:**
```
Current State:
  User: {id: 1, name: "Alice", email: "alice@example.com"}
  
Update: Change email to "alice.new@example.com"
  User: {id: 1, name: "Alice", email: "alice.new@example.com"}
  
Old state lost!
```

**Event Sourcing Approach:**
```
Events:
  1. UserCreated(id=1, name="Alice", email="alice@example.com")
  2. EmailChanged(id=1, old_email="alice@example.com", new_email="alice.new@example.com")
  
Current State: Replay events to get current state
  Replay event 1: User created
  Replay event 2: Email changed
  Result: {id: 1, name: "Alice", email: "alice.new@example.com"}
  
All history preserved!
```

### Key Concepts

**1. Events are Immutable:**
- Events never change
- Append-only
- Historical record

**2. State is Derived:**
- Current state = replay all events
- Can rebuild state from events
- No separate state storage

**3. Event Store:**
- Database for events
- Append-only log
- Time-ordered sequence

### Event Sourcing Example

**Bank Account:**
```
Events:
  1. AccountOpened(account_id=1, initial_balance=0)
  2. MoneyDeposited(account_id=1, amount=100)
  3. MoneyDeposited(account_id=1, amount=50)
  4. MoneyWithdrawn(account_id=1, amount=30)
  5. MoneyDeposited(account_id=1, amount=20)

Current Balance: Replay events
  0 + 100 + 50 - 30 + 20 = 140
```

**Benefits:**
- **Audit trail**: Complete history
- **Time travel**: See state at any time
- **Debugging**: Understand what happened
- **Replay**: Rebuild state

### Event Store

**Structure:**
```
Event Store:
  - Stream ID (e.g., account_id)
  - Event Number (sequence)
  - Event Type
  - Event Data
  - Timestamp
```

**Example:**
```
Stream: account-1
Events:
  1. AccountOpened {balance: 0}
  2. MoneyDeposited {amount: 100}
  3. MoneyDeposited {amount: 50}
  4. MoneyWithdrawn {amount: 30}
```

### Replaying Events

**Process:**
```
1. Load all events for stream
2. Start with initial state
3. Apply each event in order
4. Result: Current state
```

**Optimization:**
- **Snapshots**: Save state at intervals
- **Replay from snapshot**: Faster recovery
- **Example**: Snapshot every 100 events

---

## CQRS - Command Query Responsibility Segregation

### What is CQRS?

**CQRS**: Separate read and write models. Use different models for commands (writes) and queries (reads).

**Traditional Approach:**
```
Single Model:
  Read: SELECT * FROM users WHERE id = 1
  Write: UPDATE users SET email = '...' WHERE id = 1
  
Same model for both
```

**CQRS Approach:**
```
Command Model (Write):
  - Optimized for writes
  - Normalized
  - ACID transactions
  
Query Model (Read):
  - Optimized for reads
  - Denormalized
  - Fast queries
```

### Why CQRS?

**Problem:**
- Read and write have different requirements
- Optimizing for one hurts the other
- Can't optimize both

**Solution:**
- Separate models
- Optimize each independently
- Best of both worlds

### CQRS Architecture

**Visual:**
```
Command Side:
  Command → Command Handler → Write Model → Database
  
Query Side:
  Query → Query Handler → Read Model → Database
```

**Example:**
```
Command: UpdateUserEmail(user_id=1, email="new@example.com")
  → Command Handler
  → Write to normalized User table
  → Update read model (denormalized)

Query: GetUserProfile(user_id=1)
  → Query Handler
  → Read from denormalized UserProfile table
  → Fast, optimized for reads
```

### Read Model Synchronization

**Problem:**
- Write model updates
- Read model must sync
- How to keep in sync?

**Solutions:**

**1. Synchronous:**
```
Write → Update write model
  → Immediately update read model
  → Return
```

**2. Asynchronous (Event-Driven):**
```
Write → Update write model
  → Publish event
  → Event handler updates read model
  → Eventually consistent
```

**3. Event Sourcing + CQRS:**
```
Command → Event Store
  → Event published
  → Read model updated from event
  → Perfect sync
```

---

## Event Sourcing + CQRS Together

### Combined Architecture

**Flow:**
```
Command → Command Handler
  → Validate
  → Create Event
  → Append to Event Store
  → Publish Event
  
Event → Event Handlers
  → Update Read Models
  → Send Notifications
  → Trigger Side Effects
```

**Benefits:**
- **Event Sourcing**: Complete history, audit trail
- **CQRS**: Optimized reads and writes
- **Together**: Best of both

### Example: E-commerce Order

**Command:**
```
CreateOrder(user_id=1, items=[...])
  → Command Handler validates
  → Creates OrderCreated event
  → Appends to event store
```

**Event:**
```
OrderCreated {
  order_id: 123,
  user_id: 1,
  items: [...],
  total: 100.00,
  timestamp: "2024-01-15T10:30:00Z"
}
```

**Event Handlers:**
```
1. Update Order Read Model (for queries)
2. Update User Order History (denormalized)
3. Send Notification
4. Update Inventory
5. Trigger Payment
```

**Query:**
```
GetUserOrders(user_id=1)
  → Query Read Model (denormalized, fast)
  → Returns orders quickly
```

---

## Saga Pattern - Distributed Transactions

### What is Saga Pattern?

**Saga Pattern**: Manage distributed transactions using a sequence of local transactions with compensating actions.

**Problem:**
- ACID transactions don't work across services
- Need distributed transaction
- But can't use 2PC (blocking, slow)

**Solution: Saga**
- Sequence of local transactions
- Each has compensating action
- If step fails, compensate previous steps

### Saga Types

**1. Choreography (Decentralized):**

**How It Works:**
- Each service knows what to do next
- Services communicate via events
- No central coordinator

**Example:**
```
Order Service: Create order → Publish OrderCreated
  ↓
Payment Service: Receives OrderCreated → Charge → Publish PaymentCharged
  ↓
Inventory Service: Receives PaymentCharged → Reserve → Publish InventoryReserved
  ↓
Order Service: Receives InventoryReserved → Confirm order
```

**If Payment Fails:**
```
Payment Service: Publish PaymentFailed
  ↓
Order Service: Receives PaymentFailed → Cancel order
```

**2. Orchestration (Centralized):**

**How It Works:**
- Central orchestrator coordinates
- Orchestrator tells services what to do
- Services report back

**Example:**
```
Orchestrator:
  1. Call Order Service: Create order
  2. Call Payment Service: Charge
  3. Call Inventory Service: Reserve
  4. Call Order Service: Confirm
  
If step 2 fails:
  Orchestrator:
    1. Call Order Service: Cancel order
    (Compensate step 1)
```

### Saga Implementation

**Orchestration Example:**
```python
class OrderSaga:
    def execute(self, order_data):
        steps = [
            self.create_order,
            self.charge_payment,
            self.reserve_inventory,
            self.confirm_order
        ]
        
        compensations = [
            None,  # No compensation for create_order
            self.cancel_order,  # Compensate create_order
            self.refund_payment,  # Compensate charge_payment
            self.release_inventory  # Compensate reserve_inventory
        ]
        
        completed = []
        try:
            for i, step in enumerate(steps):
                result = step(order_data)
                completed.append(i)
        except Exception as e:
            # Compensate in reverse order
            for i in reversed(completed):
                if compensations[i]:
                    compensations[i](order_data)
            raise e
```

### Saga vs 2PC

**2PC (Two-Phase Commit):**
- **Blocking**: All participants wait
- **Slow**: Multiple round trips
- **Centralized**: Coordinator needed
- **ACID**: Strong consistency

**Saga:**
- **Non-blocking**: No waiting
- **Fast**: Local transactions
- **Flexible**: Choreography or orchestration
- **Eventual**: Eventual consistency

---

## Implementation Examples

### Event Sourcing with EventStore

```python
class EventStore:
    def append(self, stream_id, event):
        # Append event to stream
        event_number = self.get_next_event_number(stream_id)
        self.store_event(stream_id, event_number, event)
    
    def get_events(self, stream_id, from_event=0):
        # Get all events from stream
        return self.load_events(stream_id, from_event)
    
    def get_current_state(self, stream_id):
        # Replay events to get current state
        events = self.get_events(stream_id)
        state = None
        for event in events:
            state = self.apply_event(state, event)
        return state
```

### CQRS Implementation

```python
# Command Side
class CreateUserCommand:
    def __init__(self, name, email):
        self.name = name
        self.email = email

class UserCommandHandler:
    def handle(self, command):
        # Validate
        if not self.is_valid(command):
            raise ValidationError()
        
        # Create event
        event = UserCreated(name=command.name, email=command.email)
        
        # Save to event store
        event_store.append("users", event)
        
        # Publish event
        event_bus.publish(event)

# Query Side
class GetUserQuery:
    def __init__(self, user_id):
        self.user_id = user_id

class UserQueryHandler:
    def handle(self, query):
        # Read from optimized read model
        return read_model.get_user(query.user_id)
```

---

## When to Use

### Event Sourcing

**Use When:**
- Need complete audit trail
- Need to replay events
- Need time travel
- Complex domain with many events

**Don't Use When:**
- Simple CRUD operations
- Don't need history
- Performance critical (replay overhead)

### CQRS

**Use When:**
- Read and write patterns differ
- High read/write ratio
- Need to scale reads independently
- Complex queries

**Don't Use When:**
- Simple application
- Read and write patterns similar
- Overhead not worth it

### Saga

**Use When:**
- Distributed transactions needed
- Can't use ACID
- Services are independent
- Eventual consistency acceptable

**Don't Use When:**
- Can use ACID transactions
- Strong consistency required
- Simple operations

---

## Summary

Event Sourcing, CQRS, and Saga are powerful patterns for building complex, distributed systems.

**Key Takeaways:**
- Event Sourcing: Store events, derive state, complete history
- CQRS: Separate read/write models, optimize independently
- Event Sourcing + CQRS: Best of both, event-driven
- Saga: Distributed transactions with compensation
- Use when appropriate, not always

**Next Steps:**
- Understand your requirements
- Choose appropriate patterns
- Implement carefully
- Test thoroughly
- Monitor and adjust

