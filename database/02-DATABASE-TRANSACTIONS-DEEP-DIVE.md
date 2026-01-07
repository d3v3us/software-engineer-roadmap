# Database Transactions Deep Dive - Complete Understanding

## Table of Contents
1. [What is a Transaction?](#what-is-a-transaction)
2. [ACID Properties - The Foundation](#acid-properties---the-foundation)
3. [Transaction Isolation Levels](#transaction-isolation-levels)
4. [Concurrency Problems - What Can Go Wrong](#concurrency-problems---what-can-go-wrong)
5. [How Databases Implement Isolation](#how-databases-implement-isolation)
6. [Locking Mechanisms](#locking-mechanisms)
7. [Multi-Version Concurrency Control (MVCC)](#multi-version-concurrency-control-mvcc)
8. [Distributed Transactions](#distributed-transactions)
9. [Transaction Logs and Recovery](#transaction-logs-and-recovery)
10. [Optimistic vs Pessimistic Locking](#optimistic-vs-pessimistic-locking)

---

## What is a Transaction?

### Definition

**Transaction**: A sequence of database operations that execute as a single unit. Either all operations succeed, or all fail.

**Key Characteristics:**
- **Atomic**: All or nothing
- **Consistent**: Database remains in valid state
- **Isolated**: Concurrent transactions don't interfere
- **Durable**: Changes persist

### Real-World Analogy: Bank Transfer

**Scenario:**
```
Transfer $100 from Account A to Account B
```

**Operations:**
```
1. Debit Account A: $100
2. Credit Account B: $100
```

**Without Transaction:**
```
Step 1: Debit Account A → $100 deducted ✓
Step 2: Credit Account B → System crashes!
Result: $100 lost! (deducted but not credited)
```

**With Transaction:**
```
BEGIN TRANSACTION
  Debit Account A: $100
  Credit Account B: $100
COMMIT

If any step fails → ROLLBACK (undo everything)
Result: Either both succeed or both fail
```

### Transaction Example

**SQL:**
```sql
BEGIN TRANSACTION;

UPDATE accounts SET balance = balance - 100 WHERE id = 1;
UPDATE accounts SET balance = balance + 100 WHERE id = 2;

-- If both succeed:
COMMIT;

-- If any fails:
ROLLBACK;
```

**What Happens:**
```
BEGIN: Start transaction
  → Database creates transaction context
  → Changes are temporary (not visible to others)

UPDATE 1: Debit account 1
  → Change recorded in transaction log
  → Not yet committed

UPDATE 2: Credit account 2
  → Change recorded in transaction log
  → Not yet committed

COMMIT: Make changes permanent
  → Write changes to database
  → Release locks
  → Transaction complete

ROLLBACK: Undo changes
  → Discard all changes
  → Release locks
  → Database unchanged
```

---

## ACID Properties - The Foundation

### A - Atomicity

**Definition**: All operations in transaction execute, or none execute.

**Analogy:**
- **Light switch**: On or off, no in-between
- **Transaction**: Complete or nothing

**How It Works:**
```
Transaction starts
  → Record all changes in log
  → Don't apply to database yet

If all operations succeed:
  → Apply all changes (COMMIT)
  
If any operation fails:
  → Discard all changes (ROLLBACK)
```

**Example:**
```sql
BEGIN;
  INSERT INTO orders (user_id, total) VALUES (1, 100);
  INSERT INTO order_items (order_id, product_id, quantity) VALUES (1, 5, 2);
  -- If this fails, previous INSERT is rolled back
COMMIT;
```

**Implementation:**
- **Write-Ahead Logging (WAL)**: Log changes before applying
- **Two-Phase Commit**: Prepare then commit
- **Undo Log**: Record how to undo changes

### C - Consistency

**Definition**: Transaction brings database from one valid state to another valid state.

**Constraints:**
- **Primary keys**: Must be unique
- **Foreign keys**: Must reference existing rows
- **Check constraints**: Values must satisfy conditions
- **Business rules**: Custom validation

**Example:**
```sql
-- Constraint: balance >= 0
BEGIN;
  UPDATE accounts SET balance = balance - 150 WHERE id = 1;
  -- If balance becomes negative → Constraint violation
  -- Transaction fails, database unchanged
ROLLBACK;
```

**How It Works:**
```
Before transaction: Valid state
  → All constraints satisfied

During transaction: Temporary state
  → Constraints checked
  → If violation → Rollback

After transaction: Valid state
  → All constraints satisfied
```

### I - Isolation

**Definition**: Concurrent transactions don't interfere with each other.

**Goal:**
- Each transaction sees consistent snapshot
- Changes not visible until committed
- No interference between transactions

**Levels:**
- **Read Uncommitted**: See uncommitted changes
- **Read Committed**: See only committed changes
- **Repeatable Read**: Same reads return same values
- **Serializable**: Complete isolation

**Example:**
```
Transaction A: SELECT balance FROM accounts WHERE id = 1;
Transaction B: UPDATE accounts SET balance = 200 WHERE id = 1; COMMIT;
Transaction A: SELECT balance FROM accounts WHERE id = 1;

Isolation level determines what Transaction A sees:
  - Read Uncommitted: Might see 200 (uncommitted)
  - Read Committed: Sees old value, then 200
  - Repeatable Read: Always sees old value
  - Serializable: Complete isolation
```

### D - Durability

**Definition**: Committed changes persist even after system failure.

**How It Works:**
```
COMMIT
  → Write changes to disk
  → Write to transaction log
  → Flush to disk (fsync)

System crash
  → On restart, read transaction log
  → Replay committed transactions
  → Database restored to last committed state
```

**Guarantees:**
- **Power failure**: Changes persist
- **System crash**: Changes persist
- **Disk failure**: With replication, changes persist

**Implementation:**
- **Transaction log**: Write all changes
- **Write-ahead logging**: Log before applying
- **Checkpoints**: Periodic snapshots
- **Recovery**: Replay log on restart

---

## Transaction Isolation Levels

### Why Different Levels?

**Trade-off:**
- **Higher isolation**: More correct, but slower
- **Lower isolation**: Faster, but may see inconsistencies

**Choose based on:**
- **Correctness requirements**: How strict?
- **Performance requirements**: How fast?
- **Application needs**: What can you tolerate?

### Isolation Levels (Weakest to Strongest)

#### 1. Read Uncommitted

**Characteristics:**
- **Dirty reads**: Can read uncommitted data
- **No isolation**: See changes before commit
- **Fastest**: No locking overhead

**Example:**
```
Transaction A: BEGIN; UPDATE accounts SET balance = 200 WHERE id = 1;
Transaction B: SELECT balance FROM accounts WHERE id = 1;
  → Sees 200 (uncommitted!)
Transaction A: ROLLBACK;
Transaction B: Sees wrong data!
```

**Use Case:**
- **Rarely used**: Too permissive
- **Statistics**: Approximate counts OK
- **Not recommended**: For most applications

#### 2. Read Committed

**Characteristics:**
- **No dirty reads**: Only see committed data
- **Non-repeatable reads**: Same query, different results
- **Phantom reads**: New rows appear
- **Default in many databases**

**Example:**
```
Transaction A: BEGIN;
Transaction A: SELECT balance FROM accounts WHERE id = 1;  -- Returns 100
Transaction B: UPDATE accounts SET balance = 200 WHERE id = 1; COMMIT;
Transaction A: SELECT balance FROM accounts WHERE id = 1;  -- Returns 200 (different!)
```

**Use Case:**
- **Most applications**: Good balance
- **Web applications**: Usually sufficient
- **Default choice**: For many systems

#### 3. Repeatable Read

**Characteristics:**
- **No dirty reads**: Only committed data
- **No non-repeatable reads**: Same query, same results
- **Phantom reads**: New rows can appear
- **Snapshot isolation**: See consistent snapshot

**Example:**
```
Transaction A: BEGIN;
Transaction A: SELECT balance FROM accounts WHERE id = 1;  -- Returns 100
Transaction B: UPDATE accounts SET balance = 200 WHERE id = 1; COMMIT;
Transaction A: SELECT balance FROM accounts WHERE id = 1;  -- Still returns 100
```

**Use Case:**
- **Financial systems**: Need consistent reads
- **Reporting**: Consistent snapshots
- **Analytics**: Repeatable results

#### 4. Serializable

**Characteristics:**
- **Complete isolation**: As if transactions ran serially
- **No anomalies**: No dirty reads, non-repeatable reads, or phantoms
- **Slowest**: Most locking, potential deadlocks
- **Highest correctness**: Most strict

**Example:**
```
Transaction A: SELECT * FROM accounts WHERE balance > 100;
Transaction B: INSERT INTO accounts VALUES (5, 200);
  → Blocked until Transaction A commits
Transaction A: COMMIT;
Transaction B: Can now insert
```

**Use Case:**
- **Critical systems**: Maximum correctness
- **Financial transactions**: Must be perfect
- **When correctness > performance**

### Isolation Level Comparison

| Level | Dirty Read | Non-Repeatable Read | Phantom Read | Performance |
|-------|------------|---------------------|--------------|-------------|
| **Read Uncommitted** | Yes | Yes | Yes | Fastest |
| **Read Committed** | No | Yes | Yes | Fast |
| **Repeatable Read** | No | No | Yes | Medium |
| **Serializable** | No | No | No | Slowest |

---

## Concurrency Problems - What Can Go Wrong

### 1. Dirty Read

**Definition**: Read uncommitted data from another transaction.

**Example:**
```
Transaction A: BEGIN;
Transaction A: UPDATE accounts SET balance = 200 WHERE id = 1;
Transaction B: SELECT balance FROM accounts WHERE id = 1;
  → Reads 200 (uncommitted)
Transaction A: ROLLBACK;
Transaction B: Has wrong data (200, but should be 100)
```

**Problem:**
- Data may be rolled back
- Based decisions on invalid data

**Prevention:**
- **Read Committed** or higher
- Don't read uncommitted data

### 2. Non-Repeatable Read

**Definition**: Same query returns different results in same transaction.

**Example:**
```
Transaction A: BEGIN;
Transaction A: SELECT balance FROM accounts WHERE id = 1;  -- Returns 100
Transaction B: UPDATE accounts SET balance = 200 WHERE id = 1; COMMIT;
Transaction A: SELECT balance FROM accounts WHERE id = 1;  -- Returns 200 (different!)
```

**Problem:**
- Inconsistent view within transaction
- Decisions based on changing data

**Prevention:**
- **Repeatable Read** or higher
- Lock rows read

### 3. Phantom Read

**Definition**: New rows appear in subsequent queries.

**Example:**
```
Transaction A: BEGIN;
Transaction A: SELECT COUNT(*) FROM accounts WHERE balance > 100;  -- Returns 5
Transaction B: INSERT INTO accounts VALUES (6, 150); COMMIT;
Transaction A: SELECT COUNT(*) FROM accounts WHERE balance > 100;  -- Returns 6 (new row!)
```

**Problem:**
- Count changes within transaction
- Inconsistent results

**Prevention:**
- **Serializable** isolation
- Range locks

### 4. Lost Update

**Definition**: Two transactions update same row, one update is lost.

**Example:**
```
Transaction A: BEGIN;
Transaction A: SELECT balance FROM accounts WHERE id = 1;  -- 100
Transaction B: BEGIN;
Transaction B: SELECT balance FROM accounts WHERE id = 1;  -- 100
Transaction A: UPDATE accounts SET balance = 150 WHERE id = 1; COMMIT;  -- balance + 50
Transaction B: UPDATE accounts SET balance = 120 WHERE id = 1; COMMIT;  -- balance + 20
Result: 120 (Transaction A's +50 is lost!)
```

**Problem:**
- Updates overwrite each other
- Data loss

**Prevention:**
- **Optimistic locking**: Version numbers
- **Pessimistic locking**: Lock before update
- **Atomic operations**: UPDATE balance = balance + 50

### 5. Write Skew

**Definition**: Two transactions read same data, make different updates based on it.

**Example:**
```
Constraint: At least one doctor must be on call

Transaction A: SELECT on_call FROM doctors WHERE id = 1;  -- true
Transaction B: SELECT on_call FROM doctors WHERE id = 2;  -- true
Transaction A: UPDATE doctors SET on_call = false WHERE id = 1; COMMIT;
Transaction B: UPDATE doctors SET on_call = false WHERE id = 2; COMMIT;

Result: No doctors on call! (Constraint violated)
```

**Problem:**
- Both transactions see valid state
- Combined result violates constraint

**Prevention:**
- **Serializable** isolation
- **Application-level checks**: Re-check before commit

---

## How Databases Implement Isolation

### Locking

**Concept:**
- **Lock data** before accessing
- **Other transactions wait** if data locked
- **Release locks** when done

**Types:**

**1. Shared Lock (Read Lock):**
```
Multiple readers can hold
Writers must wait
```

**2. Exclusive Lock (Write Lock):**
```
Only one writer
No readers, no other writers
```

**Lock Granularity:**
- **Row-level**: Lock individual rows
- **Table-level**: Lock entire table
- **Page-level**: Lock data pages

### Multi-Version Concurrency Control (MVCC)

**Concept:**
- **Keep multiple versions** of data
- **Each transaction sees snapshot** at start time
- **No locking for reads**: Read old versions

**How It Works:**
```
Time →
Version 1: balance = 100 (committed at T1)
Version 2: balance = 200 (committed at T2)
Version 3: balance = 150 (uncommitted at T3)

Transaction started at T1:
  → Sees Version 1 (balance = 100)
  → Doesn't see Version 2 or 3

Transaction started at T2:
  → Sees Version 2 (balance = 200)
  → Doesn't see Version 3
```

**Benefits:**
- **Readers don't block writers**: Read old versions
- **Writers don't block readers**: Write new versions
- **High concurrency**: Fewer conflicts

**Costs:**
- **Storage**: Keep multiple versions
- **Cleanup**: Remove old versions
- **Complexity**: More complex implementation

---

## Locking Mechanisms

### Two-Phase Locking (2PL)

**Phase 1: Growing Phase**
```
Acquire locks
Cannot release locks
```

**Phase 2: Shrinking Phase**
```
Release locks
Cannot acquire new locks
```

**Why Two Phases?**
- **Ensures serializability**: Locks held until done
- **Prevents conflicts**: Can't release and re-acquire

**Example:**
```
BEGIN;
  Lock row 1 (growing)
  Lock row 2 (growing)
  Update row 1
  Update row 2
  Unlock row 1 (shrinking)
  Unlock row 2 (shrinking)
COMMIT;
```

### Deadlock Detection

**How Deadlocks Occur:**
```
Transaction A: Lock row 1
Transaction B: Lock row 2
Transaction A: Try lock row 2 → Wait
Transaction B: Try lock row 1 → Wait
→ Deadlock!
```

**Detection:**
- **Wait-for graph**: Build graph of waiting transactions
- **Cycle detection**: If cycle exists → Deadlock
- **Timeout**: If wait too long → Assume deadlock

**Resolution:**
- **Kill one transaction**: Rollback, release locks
- **Retry**: Transaction can retry

---

## Multi-Version Concurrency Control (MVCC)

### How MVCC Works

**Version Storage:**
```
Row versions stored with timestamps:
Row (id=1):
  Version 1: balance=100, created_at=T1, deleted_at=NULL
  Version 2: balance=200, created_at=T2, deleted_at=NULL
  Version 3: balance=150, created_at=T3, deleted_at=NULL
```

**Transaction Visibility:**
```
Transaction started at T2:
  → Sees versions created_at <= T2 AND deleted_at > T2
  → Sees Version 2 (balance=200)
```

**Update Process:**
```
UPDATE accounts SET balance = 150 WHERE id = 1;

1. Create new version: balance=150, created_at=now
2. Mark old version: deleted_at=now
3. Commit: New version visible
```

### MVCC Benefits

**1. Readers Don't Block Writers:**
```
Reader: Reads old version
Writer: Writes new version
→ No conflict!
```

**2. Writers Don't Block Readers:**
```
Writer: Locks row, creates new version
Reader: Reads old version
→ No blocking!
```

**3. Snapshot Isolation:**
```
Transaction sees consistent snapshot
All reads from same point in time
```

### MVCC Implementation (PostgreSQL)

**System Columns:**
```
xmin: Transaction ID that created version
xmax: Transaction ID that deleted version
```

**Visibility Rules:**
```
Version visible if:
  xmin < current_transaction_id (created before)
  AND (xmax is NULL OR xmax > current_transaction_id) (not deleted)
  AND created by committed transaction
```

---

## Distributed Transactions

### The Challenge

**Problem:**
```
Transaction spans multiple databases:
  Database A: Debit account
  Database B: Credit account
  
Both must succeed or both fail
```

**Challenges:**
- **Network failures**: Connection lost
- **Partial failures**: One database fails
- **Consistency**: All must agree

### Two-Phase Commit (2PC)

**Phase 1: Prepare**
```
Coordinator: "Can you commit?"
Database A: "Yes, ready" (prepare)
Database B: "Yes, ready" (prepare)
```

**Phase 2: Commit**
```
If all prepared:
  Coordinator: "Commit!"
  Database A: Commits
  Database B: Commits
Else:
  Coordinator: "Abort!"
  Database A: Aborts
  Database B: Aborts
```

**Problems:**
- **Blocking**: If coordinator fails, all wait
- **Slow**: Multiple round trips
- **Complex**: Recovery is hard

### Saga Pattern

**Alternative to 2PC:**
- **Compensating transactions**: Undo if needed
- **No locks**: Each step commits independently
- **Eventual consistency**: May be temporarily inconsistent

**Example:**
```
1. Debit Account A (commit)
2. Credit Account B (commit)
3. If step 2 fails:
   Compensate: Credit Account A (undo step 1)
```

---

## Transaction Logs and Recovery

### Write-Ahead Logging (WAL)

**Principle:**
- **Log changes before applying** to database
- **Write log to disk** (durable)
- **Apply to database** later

**Why?**
- **Durability**: Log is durable, database may not be
- **Recovery**: Can replay log
- **Performance**: Sequential writes (faster)

### Transaction Log Structure

**Log Entries:**
```
[T1, BEGIN]
[T1, UPDATE accounts, id=1, old_balance=100, new_balance=150]
[T1, UPDATE accounts, id=2, old_balance=50, new_balance=100]
[T1, COMMIT]
```

### Recovery Process

**After Crash:**
```
1. Read transaction log
2. Find committed transactions
3. Replay committed transactions
4. Find uncommitted transactions
5. Rollback uncommitted transactions
```

**Example:**
```
Log:
  [T1, BEGIN]
  [T1, UPDATE accounts, id=1, balance=150]
  [T1, COMMIT]
  [T2, BEGIN]
  [T2, UPDATE accounts, id=2, balance=200]
  [Crash - T2 not committed]

Recovery:
  Replay T1 (committed) → balance=150
  Rollback T2 (uncommitted) → balance unchanged
```

---

## Optimistic vs Pessimistic Locking

### Pessimistic Locking

**Strategy:**
- **Lock before reading**
- **Assume conflicts will occur**
- **Prevent conflicts**

**Example:**
```sql
BEGIN;
SELECT * FROM accounts WHERE id = 1 FOR UPDATE;  -- Lock row
-- Other transactions wait
UPDATE accounts SET balance = 150 WHERE id = 1;
COMMIT;  -- Release lock
```

**Pros:**
- **Prevents conflicts**: Locks prevent issues
- **Simple**: Straightforward

**Cons:**
- **Blocks**: Other transactions wait
- **Deadlocks**: Can cause deadlocks
- **Lower concurrency**: Locks reduce parallelism

### Optimistic Locking

**Strategy:**
- **Don't lock**
- **Assume no conflicts**
- **Detect conflicts, retry if needed**

**Example:**
```sql
-- Read with version
SELECT id, balance, version FROM accounts WHERE id = 1;
-- Returns: id=1, balance=100, version=5

-- Update with version check
UPDATE accounts 
SET balance = 150, version = 6 
WHERE id = 1 AND version = 5;

-- If version changed → Update fails (retry)
```

**Pros:**
- **High concurrency**: No blocking
- **No deadlocks**: No locks held
- **Better performance**: When conflicts rare

**Cons:**
- **Retries needed**: If conflict detected
- **More complex**: Must handle retries
- **Worse when conflicts common**: Many retries

### When to Use Each

**Pessimistic:**
- **High conflict rate**: Conflicts are common
- **Critical data**: Must prevent conflicts
- **Simple code**: Easier to reason about

**Optimistic:**
- **Low conflict rate**: Conflicts are rare
- **High concurrency needed**: Many readers
- **Can tolerate retries**: Retry is acceptable

---

## Summary

Database transactions ensure data consistency and reliability. Understanding ACID properties, isolation levels, concurrency problems, and implementation mechanisms is essential for backend engineers.

**Key Takeaways:**
- Transactions ensure atomicity, consistency, isolation, durability
- Isolation levels trade correctness for performance
- Concurrency problems: dirty reads, non-repeatable reads, phantoms, lost updates
- Locking and MVCC implement isolation
- Distributed transactions are complex (2PC, Saga)
- Transaction logs enable recovery
- Choose optimistic or pessimistic locking based on conflict rate

**Next Steps:**
- Understand your database's isolation level
- Choose appropriate isolation for your use case
- Handle deadlocks and retries
- Monitor transaction performance
- Test concurrent scenarios

