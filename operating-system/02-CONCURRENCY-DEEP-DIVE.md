# Concurrency and Synchronization Deep Dive - Complete Understanding

## Table of Contents
1. [The Concurrency Problem - Why We Need Synchronization](#the-concurrency-problem---why-we-need-synchronization)
2. [Race Conditions - When Things Go Wrong](#race-conditions---when-things-go-wrong)
3. [Critical Sections - Protecting Shared Resources](#critical-sections---protecting-shared-resources)
4. [Mutex - Mutual Exclusion](#mutex---mutual-exclusion)
5. [Semaphore - Generalizing Locks](#semaphore---generalizing-locks)
6. [Read-Write Locks - Optimizing for Reads](#read-write-locks---optimizing-for-reads)
7. [Spinlock - Busy Waiting](#spinlock---busy-waiting)
8. [Deadlock - When Everything Stops](#deadlock---when-everything-stops)
9. [Deadlock Prevention and Avoidance](#deadlock-prevention-and-avoidance)
10. [Condition Variables - Waiting for Events](#condition-variables---waiting-for-events)
11. [Atomic Operations - Lock-Free Programming](#atomic-operations---lock-free-programming)
12. [Memory Ordering - The Hidden Complexity](#memory-ordering---the-hidden-complexity)

---

## The Concurrency Problem - Why We Need Synchronization

### What is Concurrency?

**Concurrency**: Multiple threads executing simultaneously, potentially accessing shared resources.

**Analogy:**
- **Single-threaded**: One person working alone
- **Concurrent**: Multiple people working together
- **Problem**: What if they need the same tool?

### The Shared Resource Problem

**Scenario:**
```
Two threads, one shared variable:
Thread 1: counter = counter + 1
Thread 2: counter = counter + 1

Expected: counter increases by 2
Reality: Might only increase by 1!
```

**Why?**
```
Thread 1: Read counter = 5
Thread 2: Read counter = 5  (before Thread 1 writes)
Thread 1: Write counter = 6
Thread 2: Write counter = 6  (overwrites Thread 1's write!)
Result: 6 (should be 7)
```

**Visual Timeline:**
```
Time →
Thread 1: Read(5) ──────────── Write(6)
Thread 2:        Read(5) ──────────── Write(6)
Result: Lost update! (both read 5, both write 6)
```

### Why Concurrency Exists

**Benefits:**
- **Performance**: Utilize multiple CPU cores
- **Responsiveness**: UI stays responsive while processing
- **Efficiency**: Overlap I/O with computation

**Costs:**
- **Complexity**: Hard to reason about
- **Bugs**: Race conditions, deadlocks
- **Synchronization overhead**: Locks, waits

### Real-World Analogy: Bank Account

**Scenario:**
```
Two people withdrawing from same account:
Account balance: $100
Person A: Withdraw $50
Person B: Withdraw $60
```

**Without Synchronization:**
```
Person A: Read balance = $100
Person B: Read balance = $100
Person A: Calculate: $100 - $50 = $50, Write $50
Person B: Calculate: $100 - $60 = $40, Write $40
Result: $40 (should be -$10, but both got money!)
```

**With Synchronization:**
```
Person A: Lock account → Read $100 → Withdraw $50 → Write $50 → Unlock
Person B: Wait for lock → Lock account → Read $50 → Can't withdraw $60 → Unlock
Result: $50 (correct!)
```

---

## Race Conditions - When Things Go Wrong

### What is a Race Condition?

**Race Condition**: Behavior depends on timing of events. Outcome is non-deterministic.

**Key Characteristics:**
- **Non-deterministic**: Same input, different outputs
- **Timing-dependent**: Depends on execution order
- **Hard to reproduce**: May work 99% of time, fail 1%

### Types of Race Conditions

**1. Data Race:**
```
Multiple threads access same memory location
At least one is a write
No synchronization
```

**Example:**
```c
// Thread 1
counter++;

// Thread 2
counter++;

// Result: Unpredictable
```

**2. Time-of-Check-Time-of-Use (TOCTOU):**
```
Check condition
(Time passes, condition changes)
Use based on old check
```

**Example:**
```c
// Thread 1
if (balance >= amount) {  // Check: $100 >= $50
    // Thread 2 withdraws $60 here
    balance -= amount;    // Use: Now balance is $40, but we thought it was $100
}
```

**3. Lost Update:**
```
Read value
Modify value
Write value
(Another thread does same, overwrites)
```

### Race Condition Examples

**Example 1: Counter Increment**
```c
int counter = 0;

// Thread 1
void increment() {
    counter = counter + 1;
}

// Thread 2
void increment() {
    counter = counter + 1;
}
```

**What Can Happen:**
```
Thread 1: Read counter (0)
Thread 2: Read counter (0)
Thread 1: Write counter (1)
Thread 2: Write counter (1)  // Lost update!
Result: 1 (should be 2)
```

**Example 2: Linked List Insert**
```c
// Thread 1
node->next = head;
head = node;

// Thread 2 (at same time)
node->next = head;
head = node;
```

**What Can Happen:**
```
Thread 1: node1->next = head
Thread 2: node2->next = head
Thread 1: head = node1
Thread 2: head = node2
Result: node1 lost! (only node2 in list)
```

### Why Race Conditions Are Dangerous

**1. Non-Deterministic Bugs:**
- Works most of the time
- Fails occasionally
- Hard to reproduce
- Hard to debug

**2. Security Issues:**
- TOCTOU can lead to vulnerabilities
- Unauthorized access
- Data corruption

**3. Data Corruption:**
- Incorrect results
- Lost updates
- Inconsistent state

---

## Critical Sections - Protecting Shared Resources

### What is a Critical Section?

**Critical Section**: Code that accesses shared resources and must not be executed by more than one thread at a time.

**Characteristics:**
- **Mutual Exclusion**: Only one thread at a time
- **Atomic**: Appears to execute as single operation
- **Protected**: Must use synchronization

**Example:**
```c
// Critical section
void withdraw(int amount) {
    // Start critical section
    if (balance >= amount) {
        balance -= amount;
    }
    // End critical section
}
```

### Requirements for Critical Sections

**1. Mutual Exclusion:**
- Only one thread in critical section at a time
- Other threads must wait

**2. Progress:**
- If no thread in critical section, allow one to enter
- Don't block unnecessarily

**3. Bounded Waiting:**
- Thread waiting to enter will eventually get chance
- No starvation

**4. Performance:**
- Minimal overhead
- Don't block other threads unnecessarily

### Protecting Critical Sections

**Solutions:**
1. **Locks (Mutex)**: Simple, most common
2. **Semaphores**: More flexible
3. **Atomic operations**: Lock-free
4. **Transactional memory**: Hardware support

---

## Mutex - Mutual Exclusion

### What is a Mutex?

**Mutex (Mutual Exclusion)**: Lock that ensures only one thread can execute critical section at a time.

**Analogy:**
- **Bathroom key**: Only one person can have it
- **When you have key**: You can use bathroom
- **When you don't**: You must wait
- **When done**: Return key

### How Mutex Works

**Basic Operations:**
```
lock(mutex):   Acquire lock (wait if locked)
unlock(mutex): Release lock (wake waiting threads)
```

**Example:**
```c
mutex_t mutex = MUTEX_INIT;

void critical_section() {
    lock(&mutex);      // Acquire lock
    // Critical section code
    counter++;
    unlock(&mutex);    // Release lock
}
```

### Mutex Properties

**1. Binary:**
- Two states: Locked or Unlocked
- Only one thread can hold lock

**2. Ownership:**
- Thread that locks must unlock
- Prevents accidental unlocking

**3. Blocking:**
- If locked, thread blocks (waits)
- Wakes when unlocked

**4. Reentrant vs Non-Reentrant:**
- **Non-reentrant**: Can't lock twice (deadlock)
- **Reentrant**: Same thread can lock multiple times

### Mutex Implementation

**Simple Implementation (Pseudocode):**
```
mutex_lock(mutex):
    while (mutex->locked) {
        wait();  // Block until unlocked
    }
    mutex->locked = true;
    mutex->owner = current_thread;

mutex_unlock(mutex):
    mutex->locked = false;
    mutex->owner = NULL;
    wake_one_waiting_thread();
```

**Problem: Race Condition in Lock!**
```
Thread 1: Check locked (false)
Thread 2: Check locked (false)  // Both see unlocked!
Thread 1: Set locked = true
Thread 2: Set locked = true     // Both have lock!
```

**Solution: Atomic Operations**
```
Use hardware atomic operations:
  - Test-and-Set
  - Compare-and-Swap
  - Load-Link/Store-Conditional
```

### Mutex Usage Patterns

**Pattern 1: Protect Shared Variable**
```c
int counter = 0;
mutex_t mutex;

void increment() {
    lock(&mutex);
    counter++;
    unlock(&mutex);
}
```

**Pattern 2: Protect Data Structure**
```c
list_t *list;
mutex_t list_mutex;

void add_item(item_t *item) {
    lock(&list_mutex);
    list_append(list, item);
    unlock(&list_mutex);
}
```

**Pattern 3: Protect Multiple Operations**
```c
void transfer(account_t *from, account_t *to, int amount) {
    lock(&from->mutex);
    lock(&to->mutex);
    from->balance -= amount;
    to->balance += amount;
    unlock(&to->mutex);
    unlock(&from->mutex);
}
```

### Common Mutex Mistakes

**1. Forgetting to Unlock:**
```c
lock(&mutex);
// ... code ...
// Forgot unlock!
// Deadlock: Other threads wait forever
```

**2. Double Locking:**
```c
lock(&mutex);
lock(&mutex);  // Deadlock! (if non-reentrant)
```

**3. Wrong Mutex:**
```c
lock(&mutex1);
// ... code ...
unlock(&mutex2);  // Wrong mutex!
```

**4. Holding Lock Too Long:**
```c
lock(&mutex);
slow_operation();  // Blocks other threads
unlock(&mutex);
```

---

## Semaphore - Generalizing Locks

### What is a Semaphore?

**Semaphore**: Synchronization primitive that allows N threads to access resource (not just 1).

**Analogy:**
- **Mutex**: One key (bathroom)
- **Semaphore**: Multiple keys (parking lot with N spaces)

**Types:**
- **Binary Semaphore**: Like mutex (0 or 1)
- **Counting Semaphore**: Can have value > 1

### Semaphore Operations

**wait() (P - Proberen "to test"):**
```
Decrement count
If count < 0: Block (wait)
```

**signal() (V - Verhogen "to increment"):**
```
Increment count
If count <= 0: Wake one waiting thread
```

### Semaphore Implementation

**Structure:**
```
semaphore_t {
    int count;           // Available resources
    queue_t wait_queue;  // Waiting threads
}
```

**Operations:**
```
sem_wait(sem):
    sem->count--;
    if (sem->count < 0) {
        block_current_thread();
        add_to_wait_queue(sem->wait_queue);
    }

sem_signal(sem):
    sem->count++;
    if (sem->count <= 0) {
        thread = remove_from_wait_queue(sem->wait_queue);
        wake_thread(thread);
    }
```

### Semaphore Use Cases

**1. Resource Pool:**
```c
// 10 database connections
semaphore_t db_connections = 10;

void use_database() {
    sem_wait(&db_connections);  // Acquire connection
    // Use database
    sem_signal(&db_connections); // Release connection
}
```

**2. Producer-Consumer:**
```c
semaphore_t empty = 10;  // Empty slots
semaphore_t full = 0;    // Full slots
mutex_t mutex;           // Protect buffer

void producer() {
    while (true) {
        item = produce();
        sem_wait(&empty);    // Wait for empty slot
        lock(&mutex);
        buffer_add(item);
        unlock(&mutex);
        sem_signal(&full);   // Signal item available
    }
}

void consumer() {
    while (true) {
        sem_wait(&full);     // Wait for item
        lock(&mutex);
        item = buffer_remove();
        unlock(&mutex);
        sem_signal(&empty);  // Signal slot empty
        consume(item);
    }
}
```

**3. Barrier Synchronization:**
```c
// Wait for N threads to reach point
semaphore_t barrier = 0;
int arrived = 0;
mutex_t mutex;

void barrier_wait() {
    lock(&mutex);
    arrived++;
    if (arrived == N) {
        unlock(&mutex);
        for (int i = 0; i < N-1; i++) {
            sem_signal(&barrier);
        }
    } else {
        unlock(&mutex);
        sem_wait(&barrier);
    }
}
```

### Semaphore vs Mutex

| Aspect | Mutex | Semaphore |
|--------|-------|-----------|
| **Values** | 0 or 1 | 0 to N |
| **Ownership** | Yes (must unlock) | No |
| **Use Case** | Mutual exclusion | Resource counting |
| **Complexity** | Simpler | More flexible |

**When to Use:**
- **Mutex**: One thread at a time (critical section)
- **Semaphore**: N threads at a time (resource pool)

---

## Read-Write Locks - Optimizing for Reads

### The Problem with Mutex

**Scenario:**
```
Many readers, few writers
Mutex: Only one reader at a time
→ Unnecessary serialization
```

**Example:**
```
10 threads reading data
Mutex: Only 1 can read at a time
→ 9 threads wait unnecessarily
```

### Read-Write Lock Solution

**Read-Write Lock**: Allows multiple readers OR one writer.

**Rules:**
- **Multiple readers**: Can read simultaneously
- **One writer**: Exclusive access
- **Readers block writers**: Writers wait for all readers
- **Writers block readers**: Readers wait for writer

### How Read-Write Lock Works

**Operations:**
```
read_lock(rwlock):   Acquire read lock
read_unlock(rwlock): Release read lock
write_lock(rwlock):  Acquire write lock (exclusive)
write_unlock(rwlock): Release write lock
```

**Implementation:**
```
read_write_lock_t {
    int readers;        // Number of readers
    mutex_t mutex;      // Protect readers count
    semaphore_t writer; // Writer semaphore
}

read_lock(rwlock):
    lock(&rwlock->mutex);
    rwlock->readers++;
    if (rwlock->readers == 1) {
        sem_wait(&rwlock->writer);  // First reader blocks writers
    }
    unlock(&rwlock->mutex);

read_unlock(rwlock):
    lock(&rwlock->mutex);
    rwlock->readers--;
    if (rwlock->readers == 0) {
        sem_signal(&rwlock->writer); // Last reader allows writers
    }
    unlock(&rwlock->mutex);

write_lock(rwlock):
    sem_wait(&rwlock->writer);  // Block until no readers/writers

write_unlock(rwlock):
    sem_signal(&rwlock->writer);
```

### Read-Write Lock Use Cases

**1. Database:**
```c
// Many SELECT queries (reads)
// Few UPDATE queries (writes)
read_write_lock_t db_lock;

void read_query() {
    read_lock(&db_lock);
    // Execute SELECT
    read_unlock(&db_lock);
}

void write_query() {
    write_lock(&db_lock);
    // Execute UPDATE
    write_unlock(&db_lock);
}
```

**2. Cache:**
```c
// Many cache reads
// Occasional cache updates
read_write_lock_t cache_lock;
```

**3. Configuration:**
```c
// Many threads read config
// Rarely update config
read_write_lock_t config_lock;
```

### Read-Write Lock Trade-offs

**Pros:**
- **Better concurrency**: Multiple readers
- **Performance**: Faster for read-heavy workloads

**Cons:**
- **More complex**: Harder to implement correctly
- **Writer starvation**: Readers can block writers indefinitely
- **Overhead**: More overhead than mutex

**When to Use:**
- **Read-heavy**: Many more reads than writes
- **Reads are fast**: Don't hold lock long
- **Writes are rare**: Infrequent updates

---

## Spinlock - Busy Waiting

### What is a Spinlock?

**Spinlock**: Lock that **spins** (busy waits) instead of blocking.

**Difference from Mutex:**
- **Mutex**: Blocks thread (yields CPU)
- **Spinlock**: Spins in loop (uses CPU)

### How Spinlock Works

**Implementation:**
```
spinlock_t {
    volatile int locked;
}

spin_lock(spinlock):
    while (test_and_set(&spinlock->locked)) {
        // Spin (busy wait)
    }

spin_unlock(spinlock):
    spinlock->locked = 0;
```

**Visual:**
```
Thread 1: Has lock
Thread 2: while (locked) { spin... spin... spin... }
Thread 1: Unlocks
Thread 2: Exits loop, acquires lock
```

### When to Use Spinlock

**Use Spinlock When:**
- **Short critical section**: Lock held briefly
- **Multicore system**: Other cores can make progress
- **Low contention**: Lock rarely contended
- **Can't block**: Interrupt handlers, kernel code

**Don't Use Spinlock When:**
- **Long critical section**: Wastes CPU
- **Single core**: No benefit, wastes CPU
- **High contention**: Many threads spinning
- **User space**: Usually better to block

### Spinlock vs Mutex Performance

**Short Critical Section (1 microsecond):**
```
Spinlock: ~1 microsecond (no context switch)
Mutex: ~10 microseconds (context switch overhead)
→ Spinlock faster
```

**Long Critical Section (1 millisecond):**
```
Spinlock: Wastes CPU for 1ms
Mutex: Blocks, other threads run
→ Mutex better
```

**Rule of Thumb:**
- **Critical section < context switch time**: Use spinlock
- **Critical section > context switch time**: Use mutex

---

## Deadlock - When Everything Stops

### What is Deadlock?

**Deadlock**: Two or more threads are blocked forever, waiting for each other.

**Conditions (All must be true):**
1. **Mutual Exclusion**: Resources can't be shared
2. **Hold and Wait**: Hold resource while waiting for another
3. **No Preemption**: Can't take resource away
4. **Circular Wait**: Circular chain of waiting

### Deadlock Example

**Classic Example:**
```
Thread 1: Lock A, then Lock B
Thread 2: Lock B, then Lock A
```

**What Happens:**
```
Time →
Thread 1: Lock A ✓
Thread 2: Lock B ✓
Thread 1: Try Lock B → Wait (Thread 2 has it)
Thread 2: Try Lock A → Wait (Thread 1 has it)
→ Deadlock! Both waiting forever
```

**Visual:**
```
Thread 1 ──(waiting for)──> Resource B
    ↑                           ↓
    │                      Thread 2
    │                           ↓
Resource A <──(waiting for)─────┘
```

### Real-World Deadlock Example

**Bank Transfer:**
```c
void transfer(account_t *from, account_t *to, int amount) {
    lock(&from->mutex);
    lock(&to->mutex);
    from->balance -= amount;
    to->balance += amount;
    unlock(&to->mutex);
    unlock(&from->mutex);
}

// Thread 1
transfer(account_A, account_B, 100);

// Thread 2
transfer(account_B, account_A, 50);
```

**Deadlock:**
```
Thread 1: Lock account_A ✓
Thread 2: Lock account_B ✓
Thread 1: Try Lock account_B → Wait
Thread 2: Try Lock account_A → Wait
→ Deadlock!
```

### Detecting Deadlock

**Resource Allocation Graph:**
```
Nodes: Processes and Resources
Edges: 
  - Process → Resource (waiting for)
  - Resource → Process (held by)

Deadlock if: Cycle in graph
```

**Example:**
```
P1 → R1 (P1 waiting for R1)
R1 → P2 (R1 held by P2)
P2 → R2 (P2 waiting for R2)
R2 → P1 (R2 held by P1)
→ Cycle! Deadlock!
```

---

## Deadlock Prevention and Avoidance

### Deadlock Prevention

**Prevent one of the four conditions:**

**1. Prevent Mutual Exclusion:**
- Make resources shareable
- **Problem**: Not always possible (printers, files)

**2. Prevent Hold and Wait:**
- **All-or-Nothing**: Request all resources at once
- **Release Before Request**: Release all before requesting new

**Example:**
```c
// Bad (hold and wait)
lock(&mutex1);
lock(&mutex2);  // Holds mutex1, waits for mutex2

// Good (all at once)
lock(&mutex1);
lock(&mutex2);  // Both acquired before use
```

**3. Allow Preemption:**
- Take resource away if needed
- **Problem**: Complex, may lose work

**4. Prevent Circular Wait:**
- **Order resources**: Always request in same order
- **Example**: Always lock mutex1 before mutex2

**Example:**
```c
// Bad (different orders)
Thread 1: lock(A), lock(B)
Thread 2: lock(B), lock(A)  // Different order!

// Good (same order)
Thread 1: lock(A), lock(B)
Thread 2: lock(A), lock(B)  // Same order!
```

### Deadlock Avoidance

**Banker's Algorithm:**
- System knows maximum resources each process needs
- Before granting, check if safe state exists
- Only grant if safe

**Safe State:**
- System can allocate resources to all processes
- No deadlock possible
- All processes can complete

**Unsafe State:**
- Deadlock possible (but not guaranteed)
- System should avoid

**Example:**
```
Resources: 10 units
Process A: Needs max 5, currently has 2
Process B: Needs max 4, currently has 2
Process C: Needs max 7, currently has 3

Available: 10 - 2 - 2 - 3 = 3

Can grant to A? (needs 3 more)
  After: Available = 0
  Can B or C complete? No
  → Unsafe, don't grant
```

### Deadlock Detection and Recovery

**Detection:**
- Build resource allocation graph
- Check for cycles
- If cycle → Deadlock

**Recovery:**
1. **Process Termination**: Kill one or more processes
2. **Resource Preemption**: Take resource, give to another

---

## Condition Variables - Waiting for Events

### What is a Condition Variable?

**Condition Variable**: Allows threads to wait for condition to become true.

**Use Case:**
- Thread needs to wait for condition
- Another thread will signal when condition is true

### How Condition Variables Work

**Operations:**
```
wait(cond, mutex):    Wait for condition (releases mutex)
signal(cond):         Wake one waiting thread
broadcast(cond):      Wake all waiting threads
```

**Important:**
- Always used with mutex
- Must hold mutex when calling wait/signal

### Condition Variable Example

**Producer-Consumer:**
```c
queue_t queue;
mutex_t mutex;
condition_t not_empty;
condition_t not_full;

void producer() {
    while (true) {
        item = produce();
        lock(&mutex);
        while (queue_full(&queue)) {
            wait(&not_full, &mutex);  // Wait for space
        }
        enqueue(&queue, item);
        signal(&not_empty);  // Signal item available
        unlock(&mutex);
    }
}

void consumer() {
    while (true) {
        lock(&mutex);
        while (queue_empty(&queue)) {
            wait(&not_empty, &mutex);  // Wait for item
        }
        item = dequeue(&queue);
        signal(&not_full);  // Signal space available
        unlock(&mutex);
        consume(item);
    }
}
```

### Why Condition Variables?

**Without Condition Variables:**
```c
// Busy waiting (wastes CPU)
while (queue_empty(&queue)) {
    unlock(&mutex);
    sleep(1);  // Waste time
    lock(&mutex);
}
```

**With Condition Variables:**
```c
// Efficient waiting
while (queue_empty(&queue)) {
    wait(&not_empty, &mutex);  // Blocks until signaled
}
```

---

## Atomic Operations - Lock-Free Programming

### What are Atomic Operations?

**Atomic Operation**: Operation that completes entirely or not at all. No intermediate state visible.

**Characteristics:**
- **Indivisible**: Can't be interrupted
- **Visible**: All threads see same result
- **Fast**: Usually single CPU instruction

### Atomic Operations Examples

**1. Atomic Increment:**
```c
// Not atomic (3 steps)
counter = counter + 1;
  Read counter
  Add 1
  Write counter

// Atomic (1 step)
atomic_increment(&counter);
```

**2. Test-and-Set:**
```c
// Atomically: Read value, set to 1, return old value
int test_and_set(int *lock) {
    int old = *lock;
    *lock = 1;
    return old;
}
```

**3. Compare-and-Swap (CAS):**
```c
// Atomically: If *ptr == old, set to new, return success
bool compare_and_swap(int *ptr, int old, int new) {
    if (*ptr == old) {
        *ptr = new;
        return true;
    }
    return false;
}
```

### Lock-Free Data Structures

**Lock-Free Stack (Simplified):**
```c
typedef struct node {
    int data;
    struct node *next;
} node_t;

node_t *head = NULL;

void push(int data) {
    node_t *new = malloc(sizeof(node_t));
    new->data = data;
    do {
        new->next = head;
    } while (!compare_and_swap(&head, new->next, new));
}

int pop() {
    node_t *old_head;
    do {
        old_head = head;
        if (old_head == NULL) return -1;
    } while (!compare_and_swap(&head, old_head, old_head->next));
    return old_head->data;
}
```

**Benefits:**
- **No locks**: No deadlocks
- **Better performance**: No blocking
- **Scalability**: Works well with many threads

**Costs:**
- **Complexity**: Hard to implement correctly
- **ABA problem**: Value changes back to original
- **Memory management**: Hard to free nodes safely

---

## Memory Ordering - The Hidden Complexity

### The Problem

**Modern CPUs:**
- **Out-of-order execution**: Execute instructions in different order
- **Speculative execution**: Execute before knowing if needed
- **Caches**: Multiple levels, different cores see different views

**Result:**
- Instructions may not execute in program order
- Memory operations may be reordered
- **Visibility**: Changes may not be immediately visible

### Memory Ordering Models

**1. Sequential Consistency:**
- All operations appear to execute in program order
- All threads see same order
- **Simple, but slow**

**2. Relaxed Ordering:**
- Operations can be reordered
- **Fast, but complex**

**3. Acquire-Release:**
- **Acquire**: All operations after acquire see operations before release
- **Release**: All operations before release visible after acquire
- **Balance**: Performance and correctness

### Example: Memory Reordering

**Code:**
```c
// Thread 1
x = 1;
y = 2;

// Thread 2
if (y == 2) {
    assert(x == 1);  // Might fail!
}
```

**What Can Happen:**
```
Thread 1: CPU reorders
  y = 2;  (executes first)
  x = 1;  (executes second)

Thread 2 sees:
  y = 2 (sees update)
  x = 0 (doesn't see update yet)
→ Assert fails!
```

**Solution: Memory Barriers**
```c
// Thread 1
x = 1;
memory_barrier();  // Ensure x=1 visible before y=2
y = 2;
```

---

## Summary

Concurrency and synchronization are essential for building efficient, correct multi-threaded programs. Understanding race conditions, locks, deadlocks, and atomic operations is crucial for backend engineers.

**Key Takeaways:**
- Race conditions occur when multiple threads access shared data
- Critical sections must be protected (mutex, semaphore, etc.)
- Mutex provides mutual exclusion
- Semaphore allows N threads to access resource
- Read-write locks optimize for read-heavy workloads
- Deadlocks occur when threads wait circularly
- Prevent deadlocks by ordering resources
- Condition variables allow efficient waiting
- Atomic operations enable lock-free programming
- Memory ordering affects visibility of changes

**Next Steps:**
- Practice with synchronization primitives
- Learn your language's concurrency features
- Understand lock-free data structures
- Profile concurrent programs
- Test for race conditions and deadlocks

