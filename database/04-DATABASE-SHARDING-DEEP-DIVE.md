# Database Sharding Deep Dive - Complete Understanding

## Table of Contents
1. [What is Sharding and Why Do We Need It?](#what-is-sharding-and-why-do-we-need-it)
2. [Sharding Strategies - How to Split Data](#sharding-strategies---how-to-split-data)
3. [Shard Key Selection - The Critical Decision](#shard-key-selection---the-critical-decision)
4. [Sharding Architectures](#sharding-architectures)
5. [Querying Across Shards](#querying-across-shards)
6. [Rebalancing - Moving Data Between Shards](#rebalancing---moving-data-between-shards)
7. [Sharding Challenges and Solutions](#sharding-challenges-and-solutions)
8. [Sharding vs Partitioning](#sharding-vs-partitioning)
9. [Sharding Best Practices](#sharding-best-practices)

---

## What is Sharding and Why Do We Need It?

### The Problem

**Single Database Server:**
```
All data on one server
10 TB of data
1 million writes/second
```

**Problems:**
- **Storage limits**: Can't store more data
- **Performance limits**: Can't handle more writes
- **Single point of failure**: If server fails, everything fails
- **Can't scale**: Hardware limits reached

### The Solution: Sharding

**Sharding**: Split large database into smaller, manageable pieces (shards) distributed across multiple servers.

**Analogy:**
- **Library**: Too many books for one building
- **Solution**: Split into multiple buildings
- **Each building**: Has subset of books (A-F, G-M, N-Z)
- **Users**: Go to appropriate building

**Visual:**
```
Single Database (10 TB):
[All Data]
    ↓
Sharded (3 shards, 3.3 TB each):
[Shard 1] [Shard 2] [Shard 3]
```

### Benefits

**1. Horizontal Scaling:**
```
Add more servers → Add more shards
Unlimited scale (theoretically)
```

**2. Performance:**
```
Smaller datasets per server → Faster queries
Distributed writes → Higher throughput
```

**3. Availability:**
```
If one shard fails → Others continue
Partial failure, not total failure
```

**4. Cost:**
```
Use smaller, cheaper servers
Instead of one huge expensive server
```

---

## Sharding Strategies - How to Split Data

### 1. Range-Based Sharding

**How it works:**
- **Split by value ranges**
- **Each shard**: Handles range of values

**Example:**
```
User IDs:
Shard 1: 1 - 1,000,000
Shard 2: 1,000,001 - 2,000,000
Shard 3: 2,000,001 - 3,000,000
```

**Pros:**
- **Simple**: Easy to understand
- **Range queries**: Efficient for ranges
- **Easy to add shards**: Just add new range

**Cons:**
- **Hot spots**: Some ranges busier
- **Uneven distribution**: Data not evenly distributed
- **Rebalancing**: Hard to rebalance

**Use Case:**
- **Time-series data**: By date ranges
- **Sequential IDs**: Natural ordering
- **Known distribution**: Know how data distributes

### 2. Hash-Based Sharding

**How it works:**
- **Hash shard key** to get shard number
- **Distribute evenly** across shards

**Example:**
```
User ID: 12345
Hash(12345) = 789
789 % 3 shards = Shard 0

User ID: 67890
Hash(67890) = 456
456 % 3 shards = Shard 1
```

**Pros:**
- **Even distribution**: Data spread evenly
- **No hot spots**: Load balanced
- **Simple routing**: Hash function determines shard

**Cons:**
- **No range queries**: Can't query ranges efficiently
- **Rebalancing**: Must move data when adding shards
- **Hash collisions**: Rare but possible

**Use Case:**
- **Even distribution needed**: Uniform load
- **No range queries**: Don't need ranges
- **High write load**: Distribute writes evenly

### 3. Directory-Based Sharding

**How it works:**
- **Lookup table**: Maps shard key to shard
- **Query lookup table**: Find which shard

**Example:**
```
Lookup Table:
user_id → shard_id
1 → 0
2 → 1
3 → 2
4 → 0
...

Query: user_id = 1
Lookup: 1 → Shard 0
Route to: Shard 0
```

**Pros:**
- **Flexible**: Can move data easily
- **Easy rebalancing**: Update lookup table
- **Custom logic**: Can use any routing logic

**Cons:**
- **Lookup overhead**: Must query lookup table
- **Single point of failure**: Lookup table must be available
- **Consistency**: Lookup table must be consistent

**Use Case:**
- **Flexible routing**: Need custom logic
- **Frequent rebalancing**: Move data often
- **Complex requirements**: Can't use simple hash/range

### 4. Geographic Sharding

**How it works:**
- **Split by geography**: Region, country, etc.
- **Each shard**: Handles geographic region

**Example:**
```
Shard 1: North America (US, Canada)
Shard 2: Europe (UK, Germany, France)
Shard 3: Asia (China, Japan, India)
```

**Pros:**
- **Low latency**: Data near users
- **Compliance**: Data stays in region
- **Natural partitioning**: Geographic boundaries

**Cons:**
- **Uneven distribution**: Some regions busier
- **Cross-region queries**: Hard to query across regions

**Use Case:**
- **Global services**: Users worldwide
- **Data residency**: Legal requirements
- **Low latency**: Performance critical

---

## Shard Key Selection - The Critical Decision

### What is a Shard Key?

**Shard Key**: Column(s) used to determine which shard stores the row.

**Example:**
```
Table: users
Shard Key: user_id

user_id = 123 → Hash(123) % 3 = Shard 0
user_id = 456 → Hash(456) % 3 = Shard 1
```

### Shard Key Requirements

**1. High Cardinality:**
```
Many unique values
Prevents hot spots
```

**2. Even Distribution:**
```
Values spread evenly
No concentration
```

**3. Frequently Queried:**
```
Most queries filter by shard key
Enables efficient routing
```

**4. Rarely Changed:**
```
Shard key changes → Must move data
Expensive operation
```

### Good Shard Keys

**User ID:**
```
✅ High cardinality (many users)
✅ Even distribution (random IDs)
✅ Frequently queried (most queries by user)
✅ Rarely changed (user ID stable)
```

**Order ID:**
```
✅ High cardinality (many orders)
✅ Even distribution
✅ Frequently queried
✅ Rarely changed
```

### Bad Shard Keys

**Status (active/inactive):**
```
❌ Low cardinality (only 2 values)
❌ Uneven distribution (most active)
❌ Hot spots (all active in one shard)
```

**Created Date:**
```
❌ Time-based (all new data in one shard)
❌ Hot spots (recent dates busier)
❌ Uneven distribution
```

**Email:**
```
⚠️ High cardinality
⚠️ But: Can change (must move data)
⚠️ Privacy: Sensitive data
```

### Composite Shard Keys

**Multiple Columns:**
```
Shard Key: (user_id, order_id)

Benefits:
  - More granular distribution
  - Can query by user (all orders together)
  - Better for related data
```

**Example:**
```
Hash(user_id, order_id) → Shard
All orders for user_id in same shard (if user_id part of hash)
```

---

## Sharding Architectures

### 1. Application-Level Sharding

**How it works:**
- **Application determines** shard
- **Application routes** queries to shard
- **Database unaware** of sharding

**Example:**
```python
def get_shard(user_id):
    return hash(user_id) % num_shards

def get_user(user_id):
    shard = get_shard(user_id)
    return db_connections[shard].query(
        "SELECT * FROM users WHERE id = ?", user_id
    )
```

**Pros:**
- **Full control**: Application controls everything
- **Flexible**: Can implement any strategy
- **No database changes**: Works with any database

**Cons:**
- **Application complexity**: Must handle in code
- **Error-prone**: Easy to make mistakes
- **Maintenance**: More code to maintain

### 2. Database-Level Sharding

**How it works:**
- **Database handles** sharding
- **Application unaware** of shards
- **Database routes** automatically

**Example:**
```sql
-- Application just queries
SELECT * FROM users WHERE id = 123;

-- Database automatically routes to correct shard
```

**Pros:**
- **Simple application**: No sharding logic
- **Transparent**: Application doesn't know
- **Less error-prone**: Database handles it

**Cons:**
- **Database support needed**: Must support sharding
- **Less flexible**: Limited to database features
- **Vendor lock-in**: Tied to specific database

### 3. Proxy-Based Sharding

**How it works:**
- **Proxy sits** between application and databases
- **Proxy routes** queries to shards
- **Application queries** proxy (like single database)

**Example:**
```
Application → Proxy → Shard 1
                  → Shard 2
                  → Shard 3
```

**Pros:**
- **Application simple**: Queries proxy
- **Centralized logic**: Sharding in one place
- **Database agnostic**: Works with any database

**Cons:**
- **Additional component**: More infrastructure
- **Single point of failure**: Proxy must be available
- **Latency**: Extra hop

---

## Querying Across Shards

### The Challenge

**Problem:**
```
Query: SELECT * FROM users WHERE age > 30;
Users with age > 30 might be in any shard
```

**Solutions:**

### 1. Scatter-Gather

**How it works:**
- **Send query to all shards**
- **Gather results**
- **Merge and return**

**Example:**
```
Query Router:
  1. Send to Shard 1: SELECT * FROM users WHERE age > 30
  2. Send to Shard 2: SELECT * FROM users WHERE age > 30
  3. Send to Shard 3: SELECT * FROM users WHERE age > 30
  4. Gather: [Results from Shard 1, Shard 2, Shard 3]
  5. Merge: Combine results
  6. Return: Merged results
```

**Pros:**
- **Works for any query**: Can query all shards
- **Simple**: Straightforward implementation

**Cons:**
- **Slow**: Must query all shards
- **Expensive**: High resource usage
- **Network overhead**: Many queries

**Use Case:**
- **Rare queries**: Not frequent
- **Analytics**: Batch processing
- **Admin queries**: Administrative tasks

### 2. Query Routing

**How it works:**
- **If query has shard key**: Route to specific shard
- **If query doesn't have shard key**: Scatter-gather

**Example:**
```
Query: SELECT * FROM users WHERE user_id = 123;
  → Has shard key (user_id)
  → Route to Shard 0 (where user_id=123 is)

Query: SELECT * FROM users WHERE age > 30;
  → No shard key
  → Scatter-gather to all shards
```

**Pros:**
- **Efficient for shard key queries**: Single shard
- **Works for other queries**: Scatter-gather fallback

**Cons:**
- **Complex**: Must determine routing
- **Still slow for non-shard key**: Scatter-gather

### 3. Denormalization

**How it works:**
- **Duplicate data** across shards
- **Each shard** has complete view
- **No cross-shard queries**

**Example:**
```
User data sharded by user_id
But: Also store user data in each shard for queries
```

**Pros:**
- **Fast queries**: No cross-shard
- **Simple**: No scatter-gather

**Cons:**
- **Data duplication**: More storage
- **Consistency**: Must keep in sync
- **Writes**: Must update all shards

### 4. Materialized Views

**How it works:**
- **Pre-compute** cross-shard data
- **Store** in separate shard
- **Query** materialized view

**Example:**
```
Shard 1, 2, 3: User data
Materialized View Shard: Aggregated statistics
  - Total users
  - Users by age
  - etc.
```

**Pros:**
- **Fast reads**: Pre-computed
- **No scatter-gather**: Single query

**Cons:**
- **Stale data**: Not real-time
- **Maintenance**: Must update views
- **Storage**: Additional storage

---

## Rebalancing - Moving Data Between Shards

### Why Rebalancing?

**Reasons:**
- **Add new shards**: Distribute load
- **Remove shards**: Consolidate
- **Uneven distribution**: Balance load
- **Hot spots**: Move data from hot shards

### Rebalancing Strategies

**1. Offline Rebalancing:**
```
1. Stop writes
2. Move data
3. Update routing
4. Resume writes
```

**Pros:**
- **Simple**: No concurrent writes
- **Safe**: No conflicts

**Cons:**
- **Downtime**: Service unavailable
- **Slow**: Takes time

**2. Online Rebalancing:**
```
1. Start moving data (background)
2. Continue serving reads/writes
3. Dual-write: Write to old and new shard
4. When complete: Switch to new shard
5. Remove old data
```

**Pros:**
- **No downtime**: Service continues
- **Smooth**: Gradual migration

**Cons:**
- **Complex**: Must handle dual-writes
- **More resources**: Temporary duplication

### Rebalancing Process

**Step 1: Identify Data to Move**
```
Find data that should be in different shard
Based on shard key and current distribution
```

**Step 2: Copy Data**
```
Copy data from source shard to destination shard
Keep original (don't delete yet)
```

**Step 3: Update Routing**
```
Update routing table/lookup
Point to new shard
```

**Step 4: Dual-Write Period**
```
Write to both old and new shard
Ensure consistency
```

**Step 5: Switch Reads**
```
Start reading from new shard
Verify correctness
```

**Step 6: Remove Old Data**
```
After verification, remove from old shard
Complete migration
```

---

## Sharding Challenges and Solutions

### Challenge 1: Joins Across Shards

**Problem:**
```
Users sharded by user_id
Orders sharded by order_id
Query: SELECT * FROM users JOIN orders ON users.id = orders.user_id
→ Users and orders in different shards!
```

**Solutions:**

**1. Denormalize:**
```
Store user data in orders table
No join needed
```

**2. Application-Level Join:**
```
1. Query users shard
2. Get user IDs
3. Query orders shards for those user IDs
4. Join in application
```

**3. Avoid Joins:**
```
Design schema to avoid cross-shard joins
Use denormalization
```

### Challenge 2: Transactions Across Shards

**Problem:**
```
Transfer money:
  Debit account in Shard 1
  Credit account in Shard 2
Both must succeed or both fail
```

**Solutions:**

**1. Two-Phase Commit (2PC):**
```
Phase 1: Prepare (both shards ready)
Phase 2: Commit (both commit)
If any fails → Abort both
```

**2. Saga Pattern:**
```
1. Debit Shard 1 (commit)
2. Credit Shard 2 (commit)
3. If step 2 fails: Compensate (credit Shard 1 back)
```

**3. Avoid Cross-Shard Transactions:**
```
Design to keep transactions within shard
Use eventual consistency
```

### Challenge 3: Global Primary Keys

**Problem:**
```
Each shard generates IDs: 1, 2, 3, ...
Shard 1: ID 1
Shard 2: ID 1 (collision!)
```

**Solutions:**

**1. Composite Key:**
```
Primary Key: (shard_id, local_id)
Shard 1: (1, 1), (1, 2), (1, 3)
Shard 2: (2, 1), (2, 2), (2, 3)
All unique!
```

**2. UUID:**
```
Generate UUIDs (globally unique)
No collisions
```

**3. ID Generation Service:**
```
Central service generates IDs
Sequential, unique
```

**4. Twitter Snowflake:**
```
64-bit ID:
[Timestamp][Machine ID][Sequence]
Globally unique
```

### Challenge 4: Hot Spots

**Problem:**
```
One shard gets most traffic
Overloaded while others idle
```

**Solutions:**

**1. Better Shard Key:**
```
Choose shard key with even distribution
Avoid time-based, status-based
```

**2. Sub-sharding:**
```
Split hot shard into sub-shards
Further distribute load
```

**3. Caching:**
```
Cache hot data
Reduce database load
```

---

## Sharding vs Partitioning

### Partitioning

**What:**
- **Same server**: Multiple partitions on one server
- **Logical separation**: Data separated logically
- **No distribution**: All on one machine

**Example:**
```
Table partitioned by date:
  Partition 1: 2024-01 (same server)
  Partition 2: 2024-02 (same server)
  Partition 3: 2024-03 (same server)
```

### Sharding

**What:**
- **Different servers**: Each shard on different server
- **Physical separation**: Data on different machines
- **Distribution**: Data distributed

**Example:**
```
Shard 1: Server 1
Shard 2: Server 2
Shard 3: Server 3
```

### Comparison

| Aspect | Partitioning | Sharding |
|--------|--------------|----------|
| **Location** | Same server | Different servers |
| **Scale** | Limited by server | Unlimited |
| **Complexity** | Simpler | More complex |
| **Use Case** | Large table on one server | Scale beyond one server |

---

## Sharding Best Practices

### 1. Start Without Sharding

**Don't shard prematurely:**
- **Optimize first**: Indexes, queries, hardware
- **Replication**: Try read replicas first
- **Shard when needed**: Only when necessary

### 2. Choose Shard Key Carefully

**Critical decision:**
- **High cardinality**: Many unique values
- **Even distribution**: No hot spots
- **Query patterns**: Matches how you query
- **Stable**: Rarely changes

### 3. Plan for Rebalancing

**Will need to rebalance:**
- **Design for it**: Make rebalancing possible
- **Automate**: Don't do manually
- **Test**: Practice rebalancing

### 4. Minimize Cross-Shard Operations

**Avoid when possible:**
- **Design schema**: Keep related data together
- **Denormalize**: Duplicate if needed
- **Accept limitations**: Some queries won't work

### 5. Monitor and Measure

**Track metrics:**
- **Shard sizes**: Ensure even
- **Query performance**: Per shard
- **Hot spots**: Identify and fix
- **Rebalancing needs**: When to rebalance

---

## Summary

Database sharding is essential for scaling beyond single server limits. Understanding sharding strategies, shard key selection, querying across shards, and rebalancing is crucial for backend engineers.

**Key Takeaways:**
- Sharding splits database across multiple servers
- Strategies: Range, hash, directory, geographic
- Shard key selection is critical
- Cross-shard queries are expensive (scatter-gather)
- Rebalancing is necessary but complex
- Avoid sharding until necessary
- Design to minimize cross-shard operations
- Monitor and measure continuously

**Next Steps:**
- Understand when sharding is needed
- Choose appropriate sharding strategy
- Design schema for sharding
- Plan for rebalancing
- Monitor shard performance

