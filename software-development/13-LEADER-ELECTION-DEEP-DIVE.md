# Leader Election Deep Dive - Complete Understanding

## Table of Contents
1. [What is Leader Election?](#what-is-leader-election)
2. [Why Do We Need Leader Election?](#why-do-we-need-leader-election)
3. [Leader Election Requirements](#leader-election-requirements)
4. [Leader Election Algorithms](#leader-election-algorithms)
5. [Zookeeper-Based Leader Election](#zookeeper-based-leader-election)
6. [etcd-Based Leader Election](#etcd-based-leader-election)
7. [Redis-Based Leader Election](#redis-based-leader-election)
8. [Database-Based Leader Election](#database-based-leader-election)
9. [Raft Consensus for Leader Election](#raft-consensus-for-leader-election)
10. [Leader Failure and Re-election](#leader-failure-and-re-election)
11. [Split-Brain Problem](#split-brain-problem)
12. [Best Practices](#best-practices)
13. [Common Pitfalls](#common-pitfalls)

---

## What is Leader Election?

### Definition

**Leader Election**: Process of selecting a single leader from a group of nodes in a distributed system.

**Key Concept:**
- **Single leader**: Only one leader at a time
- **Automatic**: Election happens automatically
- **Failure handling**: Re-elects if leader fails

### Real-World Analogy

**Leader Election = Class President Election:**
- **Multiple candidates**: Multiple nodes
- **Voting process**: Election algorithm
- **One winner**: One leader selected
- **If leader leaves**: Re-election happens

**Without Leader Election:**
```
Multiple nodes
  ↓
All try to be leader
  ↓
Conflicts, chaos
  ↓
Inconsistent state
```

**With Leader Election:**
```
Multiple nodes
  ↓
Election process
  ↓
One leader selected
  ↓
Others are followers
  ↓
Consistent state
```

---

## Why Do We Need Leader Election?

### Use Cases

**1. Single Writer:**
```
Multiple database replicas
  ↓
Only leader can write
  ↓
Followers replicate from leader
  ↓
Prevents write conflicts
```

**2. Task Coordination:**
```
Multiple workers
  ↓
Only leader coordinates tasks
  ↓
Prevents duplicate work
  ↓
Efficient resource usage
```

**3. Configuration Management:**
```
Multiple nodes
  ↓
Only leader updates configuration
  ↓
Prevents conflicts
  ↓
Consistent configuration
```

**4. Scheduled Jobs:**
```
Multiple nodes
  ↓
Only leader runs scheduled jobs
  ↓
Prevents duplicate execution
  ↓
Reliable scheduling
```

### Example: Database Replication

**Problem Without Leader:**
```
Node A: Writes data
Node B: Writes different data
  ↓
Conflict: Which is correct?
  ↓
Data inconsistency
```

**Solution With Leader:**
```
Leader (Node A): Writes data
Followers (Node B, C): Replicate from leader
  ↓
No conflicts
  ↓
Consistent data
```

---

## Leader Election Requirements

### Essential Requirements

**1. Single Leader:**
- **Only one**: Only one leader at a time
- **Exclusive**: Leader is exclusive
- **No conflicts**: No multiple leaders

**2. Automatic Election:**
- **Automatic**: Election happens automatically
- **No manual**: No manual intervention
- **Fast**: Fast election process

**3. Failure Detection:**
- **Detect failure**: Detect leader failure
- **Fast detection**: Fast failure detection
- **Reliable**: Reliable detection

**4. Re-election:**
- **Re-elect**: Re-elect if leader fails
- **Fast re-election**: Fast re-election
- **No downtime**: Minimal downtime

---

## Leader Election Algorithms

### Algorithm 1: Bully Algorithm

**How It Works:**
```
1. Node with highest ID becomes leader
2. If leader fails, node with next highest ID starts election
3. Sends election message to all higher ID nodes
4. If no response, becomes leader
5. Otherwise, highest responding node becomes leader
```

**Example:**
```
Nodes: A(ID=1), B(ID=2), C(ID=3)
  ↓
Leader: C (highest ID)
  ↓
C fails
  ↓
B starts election
  ↓
B sends to C (no response - C is down)
  ↓
B becomes leader
```

**Pros:**
- **Simple**: Simple algorithm
- **Deterministic**: Deterministic leader

**Cons:**
- **Message overhead**: Many messages
- **Not fault-tolerant**: Assumes all nodes know each other

### Algorithm 2: Ring Algorithm

**How It Works:**
```
1. Nodes arranged in ring
2. Election message passed around ring
3. Each node adds its ID if higher
4. When message returns to initiator, highest ID is leader
```

**Example:**
```
Ring: A → B → C → A
  ↓
A starts election: [A]
  ↓
B adds: [A, B]
  ↓
C adds: [A, B, C]
  ↓
Returns to A: Leader is C
```

**Pros:**
- **Ordered**: Ordered election
- **Simple**: Simple algorithm

**Cons:**
- **Ring failure**: Ring breaks if node fails
- **Slow**: Slow election (O(n))

---

## Zookeeper-Based Leader Election

### How It Works

**Approach:**
```
1. Create ephemeral sequential node
2. Get all children, sort by sequence number
3. If this node is smallest → This is leader
4. Otherwise → Watch previous node
5. When previous node deleted → Check if leader
```

**Implementation:**
```python
from kazoo.client import KazooClient
from kazoo.exceptions import NodeExistsError

class LeaderElection:
    def __init__(self, zk_client, election_path):
        self.zk = zk_client
        self.election_path = election_path
        self.current_node = None
        self.is_leader = False
    
    def start_election(self):
        # Create ephemeral sequential node
        self.current_node = self.zk.create(
            f"{self.election_path}/node-",
            ephemeral=True,
            sequence=True
        )
        
        # Get all children and sort
        children = sorted(self.zk.get_children(self.election_path))
        my_node_name = self.current_node.split('/')[-1]
        my_index = children.index(my_node_name)
        
        if my_index == 0:
            # I'm first, I'm the leader
            self.is_leader = True
            return True
        
        # Watch previous node
        previous_node = f"{self.election_path}/{children[my_index - 1]}"
        event = self.zk.exists(previous_node, watch=True)
        
        if event is None:
            # Previous node already deleted, try again
            return self.start_election()
        
        # Wait for previous node to be deleted
        event.wait()
        return self.start_election()
    
    def is_leader_node(self):
        return self.is_leader
```

**Advantages:**
- **Automatic cleanup**: Ephemeral nodes auto-delete
- **Fair**: Sequential nodes ensure fairness
- **Reliable**: Zookeeper handles failures

---

## etcd-Based Leader Election

### How It Works

**Approach:**
```
1. Try to create key with TTL
2. If created → Leader
3. If exists → Watch key
4. When key deleted → Try again
```

**Implementation:**
```python
import etcd3

class EtcdLeaderElection:
    def __init__(self, etcd_client, election_key, ttl=30):
        self.etcd = etcd_client
        self.election_key = election_key
        self.ttl = ttl
        self.lease = None
        self.is_leader = False
    
    def start_election(self):
        # Create lease
        self.lease = self.etcd.lease(self.ttl)
        
        # Try to acquire leadership
        try:
            self.etcd.put(self.election_key, "leader", lease=self.lease)
            self.is_leader = True
            
            # Keep lease alive
            self.keep_alive()
            return True
        except:
            # Key exists, watch for deletion
            self.watch_leader()
            return False
    
    def keep_alive(self):
        # Keep lease alive
        while self.is_leader:
            self.lease.refresh()
            time.sleep(self.ttl / 2)
    
    def watch_leader(self):
        # Watch for leader deletion
        events, cancel = self.etcd.watch(self.election_key)
        for event in events:
            if event.type == etcd3.events.DELETE:
                # Leader deleted, try again
                self.start_election()
                break
```

---

## Redis-Based Leader Election

### How It Works

**Approach:**
```
1. Try to set key with value (process ID)
2. If set → Leader
3. If exists → Not leader
4. Watch key for expiration
5. When expired → Try again
```

**Implementation:**
```python
import redis
import time
import uuid

class RedisLeaderElection:
    def __init__(self, redis_client, election_key, ttl=30):
        self.redis = redis_client
        self.election_key = election_key
        self.ttl = ttl
        self.node_id = str(uuid.uuid4())
        self.is_leader = False
        self.pubsub = None
    
    def start_election(self):
        # Try to acquire leadership
        result = self.redis.set(
            self.election_key,
            self.node_id,
            nx=True,  # Only set if not exists
            ex=self.ttl  # Expire after TTL
        )
        
        if result:
            # I'm the leader
            self.is_leader = True
            self.keep_alive()
            return True
        else:
            # Not leader, watch for expiration
            self.watch_leader()
            return False
    
    def keep_alive(self):
        # Keep lock alive
        while self.is_leader:
            self.redis.expire(self.election_key, self.ttl)
            time.sleep(self.ttl / 2)
    
    def watch_leader(self):
        # Watch for key expiration
        self.pubsub = self.redis.pubsub()
        self.pubsub.psubscribe(f"__keyspace@0__:{self.election_key}")
        
        for message in self.pubsub.listen():
            if message['type'] == 'pmessage':
                if message['data'] == 'expired':
                    # Leader expired, try again
                    self.start_election()
                    break
```

---

## Database-Based Leader Election

### How It Works

**Approach:**
```
1. Try to insert row with node ID
2. If inserted → Leader
3. If exists → Not leader
4. Periodically check if leader is alive
5. If leader dead → Delete and try again
```

**Implementation:**
```python
def acquire_leadership(node_id, timeout=30):
    expires_at = datetime.now() + timedelta(seconds=timeout)
    
    try:
        # Try to insert
        db.execute("""
            INSERT INTO leaders 
            (node_id, acquired_at, expires_at)
            VALUES (?, ?, ?)
        """, (node_id, datetime.now(), expires_at))
        return True
    except IntegrityError:
        # Leader exists, check if expired
        leader = db.execute("""
            SELECT * FROM leaders 
            WHERE expires_at > ?
        """, (datetime.now(),)).fetchone()
        
        if leader is None:
            # Expired, try to acquire
            db.execute("DELETE FROM leaders")
            return acquire_leadership(node_id, timeout)
        return False

def keep_alive(node_id, timeout=30):
    while True:
        expires_at = datetime.now() + timedelta(seconds=timeout)
        db.execute("""
            UPDATE leaders 
            SET expires_at = ?
            WHERE node_id = ?
        """, (expires_at, node_id))
        time.sleep(timeout / 2)
```

---

## Raft Consensus for Leader Election

### How Raft Works

**Raft Leader Election:**
```
1. Nodes start as followers
2. If no leader heartbeat → Become candidate
3. Candidate requests votes
4. If majority votes → Become leader
5. Leader sends heartbeats
6. If leader fails → Re-election
```

**States:**
- **Follower**: Receives heartbeats from leader
- **Candidate**: Requesting votes
- **Leader**: Sending heartbeats

**Election Process:**
```
Follower (no heartbeat) → Candidate
  ↓
Request votes from all nodes
  ↓
If majority votes → Leader
  ↓
Send heartbeats to followers
```

---

## Leader Failure and Re-election

### Failure Detection

**Heartbeat-Based:**
```
Leader sends heartbeat every T seconds
  ↓
If no heartbeat for 2T seconds → Leader failed
  ↓
Start re-election
```

**Implementation:**
```python
class LeaderMonitor:
    def __init__(self, heartbeat_timeout=10):
        self.heartbeat_timeout = heartbeat_timeout
        self.last_heartbeat = None
    
    def receive_heartbeat(self):
        self.last_heartbeat = time.time()
    
    def is_leader_alive(self):
        if self.last_heartbeat is None:
            return False
        return time.time() - self.last_heartbeat < self.heartbeat_timeout
```

### Re-election Process

**Steps:**
```
1. Detect leader failure
2. Start election
3. Select new leader
4. Notify all nodes
5. New leader takes over
```

**Timeline:**
```
T0: Leader sends heartbeat
T1: Leader fails
T2: No heartbeat (timeout)
T3: Start election
T4: New leader elected
T5: New leader takes over
```

---

## Split-Brain Problem

### What is Split-Brain?

**Split-Brain**: Situation where network partition causes multiple leaders to be elected.

**Problem:**
```
Network partition
  ↓
Group A: Elects Leader A
Group B: Elects Leader B
  ↓
Two leaders!
  ↓
Conflicts, data corruption
```

### Prevention

**1. Quorum:**
```
Require majority (N/2 + 1) votes
  ↓
Only one group can have majority
  ↓
Only one leader
```

**2. Fencing:**
```
Use fencing tokens
  ↓
Only leader with valid token can act
  ↓
Prevents stale leaders
```

**3. STONITH (Shoot The Other Node In The Head):**
```
If split-brain detected
  ↓
Kill nodes in minority partition
  ↓
Only majority partition survives
```

---

## Best Practices

### 1. Use Quorum

**Why:**
- **Prevent split-brain**: Prevent split-brain
- **Majority rule**: Majority decides
- **Reliability**: More reliable

**Implementation:**
```python
quorum = len(nodes) // 2 + 1
if votes >= quorum:
    become_leader()
```

### 2. Fast Failure Detection

**Why:**
- **Quick recovery**: Quick recovery
- **Less downtime**: Less downtime
- **Better availability**: Better availability

**Implementation:**
```python
heartbeat_interval = 1  # 1 second
heartbeat_timeout = 3   # 3 seconds
```

### 3. Graceful Leader Transition

**Why:**
- **No data loss**: No data loss
- **Smooth transition**: Smooth transition
- **Better reliability**: Better reliability

**Implementation:**
```python
def transition_leadership():
    # Stop accepting new work
    stop_accepting()
    
    # Complete in-flight work
    wait_for_completion()
    
    # Transfer state to new leader
    transfer_state()
    
    # Step down
    step_down()
```

### 4. Monitor Leader Health

**Why:**
- **Early detection**: Early detection of issues
- **Proactive**: Proactive handling
- **Reliability**: More reliable

**Metrics:**
- **Heartbeat latency**: Heartbeat latency
- **Leader uptime**: Leader uptime
- **Re-election frequency**: Re-election frequency

---

## Common Pitfalls

### Pitfall 1: No Quorum

**Bad:**
```python
# Any node can become leader
if votes > 0:
    become_leader()
# Split-brain possible!
```

**Good:**
```python
# Require majority
quorum = len(nodes) // 2 + 1
if votes >= quorum:
    become_leader()
```

### Pitfall 2: Slow Failure Detection

**Bad:**
```python
heartbeat_timeout = 60  # 60 seconds
# Too slow, long downtime
```

**Good:**
```python
heartbeat_timeout = 3  # 3 seconds
# Fast detection, quick recovery
```

### Pitfall 3: Not Handling Split-Brain

**Bad:**
```python
# No split-brain prevention
# Multiple leaders possible
```

**Good:**
```python
# Use quorum
# Use fencing
# Prevent split-brain
```

---

## Summary

Leader election is essential for coordinating distributed systems. Understanding algorithms, implementations, and best practices is crucial for building reliable systems.

**Key Takeaways:**
- **Leader election**: Select single leader from group
- **Use cases**: Single writer, task coordination, configuration
- **Algorithms**: Bully, Ring, Zookeeper, etcd, Redis, Database
- **Requirements**: Single leader, automatic, failure detection
- **Split-brain**: Prevent with quorum, fencing
- **Best practices**: Quorum, fast detection, graceful transition

**Algorithms:**
- **Bully**: Highest ID wins
- **Ring**: Pass message around ring
- **Zookeeper**: Ephemeral sequential nodes
- **etcd**: Key with TTL
- **Redis**: Key with expiration
- **Database**: Row with expiration
- **Raft**: Consensus algorithm

**Best Practices:**
- Use quorum
- Fast failure detection
- Graceful transition
- Monitor health

**Common Pitfalls:**
- No quorum
- Slow detection
- Not handling split-brain

**Next Steps:**
- Choose algorithm
- Implement election
- Add monitoring
- Test failure scenarios
- Apply best practices

