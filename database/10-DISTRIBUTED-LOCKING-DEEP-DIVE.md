# Distributed Locking Deep Dive - Complete Understanding

## Table of Contents
1. [What is Distributed Locking?](#what-is-distributed-locking)
2. [Why Do We Need Distributed Locks?](#why-do-we-need-distributed-locks)
3. [Challenges of Distributed Locking](#challenges-of-distributed-locking)
4. [Distributed Locking Requirements](#distributed-locking-requirements)
5. [Distributed Locking Algorithms](#distributed-locking-algorithms)
6. [Redis-Based Distributed Locks](#redis-based-distributed-locks)
7. [Zookeeper-Based Distributed Locks](#zookeeper-based-distributed-locks)
8. [Database-Based Distributed Locks](#database-based-distributed-locks)
9. [etcd-Based Distributed Locks](#etcd-based-distributed-locks)
10. [Lock Timeout and Deadlocks](#lock-timeout-and-deadlocks)
11. [Lock Reentrancy](#lock-reentrancy)
12. [Best Practices](#best-practices)
13. [Common Pitfalls](#common-pitfalls)

---

## What is Distributed Locking?

### Definition

**Distributed Lock**: Synchronization mechanism that allows only one process/thread to access a shared resource at a time across multiple machines.

**Key Concept:**
- **Mutual exclusion**: Only one process can hold the lock
- **Distributed**: Works across multiple machines
- **Coordination**: Coordinates access to shared resources

### Real-World Analogy

**Distributed Lock = Conference Room Booking:**
- **Resource**: Conference room (shared resource)
- **Lock**: Booking the room (acquiring lock)
- **Multiple people**: Multiple processes want to use room
- **Only one**: Only one person can book at a time
- **Release**: Release booking when done (release lock)

**Without Lock:**
```
Process A: Check if resource available → Yes
Process B: Check if resource available → Yes (at same time!)
Process A: Use resource
Process B: Use resource (conflict!)
```

**With Lock:**
```
Process A: Acquire lock → Success
Process B: Acquire lock → Wait (lock held by A)
Process A: Use resource
Process A: Release lock
Process B: Acquire lock → Success
Process B: Use resource
```

---

## Why Do We Need Distributed Locks?

### Use Cases

**1. Preventing Race Conditions:**
```
Multiple processes updating same data
  ↓
Without lock: Race condition, data corruption
  ↓
With lock: Sequential access, data integrity
```

**2. Ensuring Idempotency:**
```
Process payment only once
  ↓
Without lock: Might process twice
  ↓
With lock: Process exactly once
```

**3. Resource Coordination:**
```
Only one process can perform maintenance
  ↓
Without lock: Multiple processes try
  ↓
With lock: Only one proceeds
```

**4. Leader Election:**
```
Only one process is leader
  ↓
Without lock: Multiple leaders
  ↓
With lock: Single leader
```

### Example: Payment Processing

**Problem Without Lock:**
```python
# Process A
balance = get_balance()  # 100
balance -= 50
save_balance(balance)    # 50

# Process B (at same time)
balance = get_balance()  # 100 (before A saved)
balance -= 30
save_balance(balance)    # 70 (overwrites A's change!)

# Result: Should be 20, but is 70 (wrong!)
```

**Solution With Lock:**
```python
# Process A
with distributed_lock("balance"):
    balance = get_balance()  # 100
    balance -= 50
    save_balance(balance)    # 50

# Process B (waits for lock)
with distributed_lock("balance"):
    balance = get_balance()  # 50 (after A)
    balance -= 30
    save_balance(balance)    # 20

# Result: 20 (correct!)
```

---

## Challenges of Distributed Locking

### Challenge 1: Network Partitions

**Problem:**
```
Process A acquires lock
  ↓
Network partition occurs
  ↓
Process A can't communicate with lock server
  ↓
Lock server thinks A is dead
  ↓
Lock expires, Process B acquires lock
  ↓
Now both A and B think they have lock!
```

**Solution:**
- **Fencing tokens**: Use increasing tokens
- **Lease-based locks**: Time-based leases
- **Heartbeats**: Keep lock alive with heartbeats

### Challenge 2: Clock Skew

**Problem:**
```
Machine A: Clock = 10:00:00
Machine B: Clock = 10:00:05 (5 seconds ahead)
  ↓
Lock expires at 10:00:10 (according to A)
But B thinks it's already 10:00:10
  ↓
B acquires lock before A releases it
```

**Solution:**
- **Server time**: Use server time, not client time
- **Clock synchronization**: Synchronize clocks (NTP)
- **Lease extension**: Extend lease before expiration

### Challenge 3: Single Point of Failure

**Problem:**
```
Lock server fails
  ↓
All locks lost
  ↓
System can't coordinate
```

**Solution:**
- **High availability**: Run lock server in HA mode
- **Replication**: Replicate lock state
- **Consensus**: Use consensus algorithm (Raft)

### Challenge 4: Performance

**Problem:**
```
Every operation needs lock
  ↓
High contention
  ↓
Performance degradation
```

**Solution:**
- **Fine-grained locks**: Lock smaller resources
- **Lock-free algorithms**: Use lock-free data structures when possible
- **Optimistic locking**: Use version numbers instead

---

## Distributed Locking Requirements

### Essential Requirements

**1. Mutual Exclusion:**
- **Only one**: Only one process can hold lock
- **Exclusive**: Lock is exclusive
- **Atomic**: Lock acquisition is atomic

**2. Deadlock Prevention:**
- **Timeout**: Locks must timeout
- **No circular waits**: Prevent circular dependencies
- **Ordering**: Acquire locks in consistent order

**3. Fault Tolerance:**
- **Survive failures**: Lock server failures
- **Recovery**: Recover from failures
- **Consistency**: Maintain consistency

**4. Performance:**
- **Low latency**: Fast lock acquisition
- **High throughput**: Handle many requests
- **Scalability**: Scale with load

---

## Distributed Locking Algorithms

### Algorithm 1: Simple Lock

**Basic Approach:**
```
1. Try to set key with value
2. If key doesn't exist → Lock acquired
3. If key exists → Lock not acquired
4. Release by deleting key
```

**Implementation:**
```python
def acquire_lock(lock_key, timeout=30):
    # Try to set key (only if not exists)
    result = redis.set(lock_key, "locked", nx=True, ex=timeout)
    return result  # True if acquired, False if not

def release_lock(lock_key):
    redis.delete(lock_key)
```

**Problem:**
- **Not safe**: Process might release wrong lock
- **No ownership**: Can't verify ownership

### Algorithm 2: Lock with Ownership

**Better Approach:**
```
1. Set key with unique value (process ID)
2. Only release if value matches
3. Prevents releasing someone else's lock
```

**Implementation:**
```python
import uuid

def acquire_lock(lock_key, timeout=30):
    lock_value = str(uuid.uuid4())
    result = redis.set(lock_key, lock_value, nx=True, ex=timeout)
    if result:
        return lock_value  # Return value for release
    return None

def release_lock(lock_key, lock_value):
    # Lua script: delete only if value matches
    script = """
    if redis.call("get", KEYS[1]) == ARGV[1] then
        return redis.call("del", KEYS[1])
    else
        return 0
    end
    """
    redis.eval(script, 1, lock_key, lock_value)
```

### Algorithm 3: Redlock (Redis Distributed Lock)

**Redlock Algorithm:**
```
1. Get current time
2. Try to acquire lock on N/2+1 nodes
3. Calculate elapsed time
4. If elapsed < lock validity time and majority acquired → Success
5. Otherwise → Release all locks and retry
```

**Implementation:**
```python
def acquire_redlock(lock_key, timeout=30, retry_count=3):
    lock_value = str(uuid.uuid4())
    quorum = len(redis_nodes) // 2 + 1
    start_time = time.time()
    
    for attempt in range(retry_count):
        acquired = 0
        for node in redis_nodes:
            try:
                if node.set(lock_key, lock_value, nx=True, ex=timeout):
                    acquired += 1
            except:
                continue
        
        elapsed = time.time() - start_time
        lock_validity = timeout - elapsed - clock_drift
        
        if acquired >= quorum and lock_validity > 0:
            return lock_value
        
        # Release all locks
        release_all_locks(lock_key, lock_value)
        time.sleep(random.uniform(0, 0.1))
    
    return None
```

---

## Redis-Based Distributed Locks

### Basic Redis Lock

**Simple Implementation:**
```python
import redis
import time
import uuid

class RedisLock:
    def __init__(self, redis_client, lock_key, timeout=30):
        self.redis = redis_client
        self.lock_key = lock_key
        self.timeout = timeout
        self.lock_value = None
    
    def acquire(self):
        self.lock_value = str(uuid.uuid4())
        result = self.redis.set(
            self.lock_key,
            self.lock_value,
            nx=True,  # Only set if not exists
            ex=self.timeout  # Expire after timeout
        )
        return result
    
    def release(self):
        if not self.lock_value:
            return False
        
        # Lua script to ensure atomic release
        script = """
        if redis.call("get", KEYS[1]) == ARGV[1] then
            return redis.call("del", KEYS[1])
        else
            return 0
        end
        """
        return self.redis.eval(script, 1, self.lock_key, self.lock_value)
    
    def __enter__(self):
        if not self.acquire():
            raise LockAcquisitionError("Failed to acquire lock")
        return self
    
    def __exit__(self, exc_type, exc_val, exc_tb):
        self.release()
```

**Usage:**
```python
redis_client = redis.Redis()
lock = RedisLock(redis_client, "my-lock", timeout=30)

with lock:
    # Critical section
    do_something()
```

### Lock Extension

**Extending Lock:**
```python
def extend_lock(self, additional_time):
    script = """
    if redis.call("get", KEYS[1]) == ARGV[1] then
        return redis.call("expire", KEYS[1], ARGV[2])
    else
        return 0
    end
    """
    return self.redis.eval(
        script, 1, self.lock_key, self.lock_value, additional_time
    )
```

---

## Zookeeper-Based Distributed Locks

### Zookeeper Lock Implementation

**How It Works:**
```
1. Create ephemeral sequential node
2. Get all children, sort by sequence number
3. If this node is smallest → Lock acquired
4. Otherwise → Watch previous node
5. When previous node deleted → Try again
```

**Implementation:**
```python
from kazoo.client import KazooClient
from kazoo.exceptions import NodeExistsError

class ZookeeperLock:
    def __init__(self, zk_client, lock_path):
        self.zk = zk_client
        self.lock_path = lock_path
        self.lock_node = None
    
    def acquire(self, timeout=None):
        # Create ephemeral sequential node
        self.lock_node = self.zk.create(
            f"{self.lock_path}/lock-",
            ephemeral=True,
            sequence=True
        )
        
        # Get all children and sort
        children = sorted(self.zk.get_children(self.lock_path))
        my_index = children.index(self.lock_node.split('/')[-1])
        
        if my_index == 0:
            # I'm first, lock acquired
            return True
        
        # Watch previous node
        previous_node = f"{self.lock_path}/{children[my_index - 1]}"
        event = self.zk.exists(previous_node, watch=True)
        
        if event is None:
            # Previous node already deleted, try again
            return self.acquire(timeout)
        
        # Wait for previous node to be deleted
        event.wait(timeout=timeout)
        return self.acquire(timeout)
    
    def release(self):
        if self.lock_node:
            self.zk.delete(self.lock_node)
            self.lock_node = None
```

**Advantages:**
- **Automatic cleanup**: Ephemeral nodes auto-delete
- **Fair**: Sequential nodes ensure fairness
- **Reliable**: Zookeeper handles failures

---

## Database-Based Distributed Locks

### Database Lock Table

**Approach:**
```
Create lock table
  ↓
Insert row to acquire lock
  ↓
Delete row to release lock
```

**Implementation:**
```sql
CREATE TABLE distributed_locks (
    lock_name VARCHAR(255) PRIMARY KEY,
    lock_holder VARCHAR(255) NOT NULL,
    acquired_at TIMESTAMP NOT NULL,
    expires_at TIMESTAMP NOT NULL
);
```

**Acquire Lock:**
```python
def acquire_lock(lock_name, holder_id, timeout=30):
    expires_at = datetime.now() + timedelta(seconds=timeout)
    
    try:
        db.execute("""
            INSERT INTO distributed_locks 
            (lock_name, lock_holder, acquired_at, expires_at)
            VALUES (?, ?, ?, ?)
        """, (lock_name, holder_id, datetime.now(), expires_at))
        return True
    except IntegrityError:
        # Lock already exists
        # Check if expired
        lock = db.execute("""
            SELECT * FROM distributed_locks 
            WHERE lock_name = ? AND expires_at > ?
        """, (lock_name, datetime.now())).fetchone()
        
        if lock is None:
            # Expired, try to acquire
            return acquire_lock(lock_name, holder_id, timeout)
        return False
```

**Release Lock:**
```python
def release_lock(lock_name, holder_id):
    db.execute("""
        DELETE FROM distributed_locks 
        WHERE lock_name = ? AND lock_holder = ?
    """, (lock_name, holder_id))
```

### Database-Specific Locks

**PostgreSQL Advisory Locks:**
```sql
-- Acquire lock
SELECT pg_advisory_lock(12345);

-- Release lock
SELECT pg_advisory_unlock(12345);
```

**MySQL GET_LOCK:**
```sql
-- Acquire lock
SELECT GET_LOCK('my_lock', 30);

-- Release lock
SELECT RELEASE_LOCK('my_lock');
```

---

## etcd-Based Distributed Locks

### etcd Lock Implementation

**How It Works:**
```
1. Create key with TTL
2. Watch for key deletion
3. If key deleted → Lock released
```

**Implementation:**
```python
import etcd3

class EtcdLock:
    def __init__(self, etcd_client, lock_key, ttl=30):
        self.etcd = etcd_client
        self.lock_key = lock_key
        self.ttl = ttl
        self.lease = None
    
    def acquire(self):
        # Create lease
        self.lease = self.etcd.lease(self.ttl)
        
        # Try to acquire lock
        try:
            self.etcd.put(self.lock_key, "locked", lease=self.lease)
            return True
        except:
            return False
    
    def release(self):
        if self.lease:
            self.etcd.delete(self.lock_key)
            self.lease.revoke()
```

---

## Lock Timeout and Deadlocks

### Lock Timeout

**Why Timeout?**
- **Prevent deadlocks**: Process crashes → lock never released
- **Automatic cleanup**: Lock expires automatically
- **Fail fast**: Don't wait forever

**Implementation:**
```python
# Redis: Set expiration
redis.set(lock_key, value, ex=30)  # Expires in 30 seconds

# Zookeeper: Ephemeral node (auto-deletes on disconnect)

# Database: Expires_at timestamp
```

### Deadlock Prevention

**1. Timeout:**
```
Lock must timeout
  ↓
Prevents permanent deadlock
```

**2. Lock Ordering:**
```
Always acquire locks in same order
  ↓
Prevents circular waits
```

**3. Try-Lock:**
```
Try to acquire, don't wait
  ↓
Fail fast if can't acquire
```

---

## Lock Reentrancy

### What is Reentrancy?

**Reentrant Lock**: Lock that can be acquired multiple times by same process.

**Example:**
```python
def function_a():
    with lock:
        function_b()  # Also needs lock

def function_b():
    with lock:  # Reentrant: same process, OK
        do_something()
```

**Implementation:**
```python
class ReentrantLock:
    def __init__(self, lock_key):
        self.lock_key = lock_key
        self.holder = None
        self.count = 0
    
    def acquire(self, holder_id):
        if self.holder == holder_id:
            # Same holder, increment count
            self.count += 1
            return True
        
        if self.holder is None:
            # No holder, acquire
            self.holder = holder_id
            self.count = 1
            return True
        
        return False  # Lock held by someone else
    
    def release(self, holder_id):
        if self.holder != holder_id:
            return False
        
        self.count -= 1
        if self.count == 0:
            self.holder = None
        return True
```

---

## Best Practices

### 1. Always Use Timeout

**Why:**
- **Prevent deadlocks**: Locks expire automatically
- **Fail fast**: Don't wait forever
- **Recovery**: System recovers from failures

**Implementation:**
```python
lock = acquire_lock("resource", timeout=30)
```

### 2. Verify Ownership

**Why:**
- **Safety**: Don't release someone else's lock
- **Correctness**: Ensure correct behavior

**Implementation:**
```python
lock_value = acquire_lock("resource")
# ... use resource ...
release_lock("resource", lock_value)  # Verify value
```

### 3. Keep Lock Time Short

**Why:**
- **Reduce contention**: Less time locked
- **Better performance**: More throughput
- **Lower risk**: Less chance of deadlock

**Guideline:**
- **Short operations**: 1-5 seconds
- **Medium operations**: 10-30 seconds
- **Long operations**: Consider alternative (sagas, etc.)

### 4. Use Fencing Tokens

**Why:**
- **Prevent stale locks**: Detect stale locks
- **Safety**: Ensure only one process acts

**Implementation:**
```python
fence_token = acquire_lock_with_token("resource")
# Include token in all operations
process_payment(fence_token, amount)
```

### 5. Monitor Lock Contention

**Metrics:**
- **Acquisition time**: How long to acquire
- **Wait time**: How long waiting
- **Failure rate**: How often acquisition fails

---

## Common Pitfalls

### Pitfall 1: Not Using Timeout

**Bad:**
```python
lock = acquire_lock("resource")  # No timeout!
# If process crashes, lock never released
```

**Good:**
```python
lock = acquire_lock("resource", timeout=30)
```

### Pitfall 2: Releasing Wrong Lock

**Bad:**
```python
lock_value = acquire_lock("resource")
# ... use resource ...
release_lock("resource")  # No value check!
# Might release someone else's lock
```

**Good:**
```python
lock_value = acquire_lock("resource")
# ... use resource ...
release_lock("resource", lock_value)  # Verify value
```

### Pitfall 3: Long-Lived Locks

**Bad:**
```python
with lock:
    process_large_file()  # Takes 10 minutes!
    # Lock held for 10 minutes
    # High contention, poor performance
```

**Good:**
```python
# Break into smaller operations
for chunk in file_chunks:
    with lock:
        process_chunk(chunk)  # Lock held briefly
```

### Pitfall 4: Ignoring Clock Skew

**Bad:**
```python
expires_at = local_time() + timeout
# If local clock is wrong, lock expires incorrectly
```

**Good:**
```python
expires_at = server_time() + timeout
# Use server time
```

---

## Summary

Distributed locking is essential for coordinating access to shared resources in distributed systems. Understanding the challenges, algorithms, and best practices is crucial for building reliable systems.

**Key Takeaways:**
- **Distributed lock**: Mutual exclusion across machines
- **Use cases**: Race conditions, idempotency, coordination
- **Challenges**: Network partitions, clock skew, failures
- **Algorithms**: Simple lock, ownership, Redlock
- **Implementations**: Redis, Zookeeper, Database, etcd
- **Best practices**: Timeout, ownership, short locks, monitoring

**Requirements:**
- Mutual exclusion
- Deadlock prevention
- Fault tolerance
- Performance

**Best Practices:**
- Always use timeout
- Verify ownership
- Keep lock time short
- Use fencing tokens
- Monitor contention

**Common Pitfalls:**
- No timeout
- Releasing wrong lock
- Long-lived locks
- Ignoring clock skew

**Next Steps:**
- Choose lock implementation
- Implement with timeout
- Add monitoring
- Test failure scenarios
- Apply best practices

