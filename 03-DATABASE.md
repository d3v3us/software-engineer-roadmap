# Database - Comprehensive Guide

## Table of Contents
1. [SQL vs NoSQL vs NewSQL](#sql-vs-nosql-vs-newsql)
2. [SQL Injection and Prepared Statements](#sql-injection-and-prepared-statements)
3. [Indexing](#indexing)
4. [Query Optimization](#query-optimization)
5. [Database Replication](#database-replication)
6. [Database Sharding](#database-sharding)
7. [Transactions and Concurrency](#transactions-and-concurrency)

---

## SQL vs NoSQL vs NewSQL

### Relational Database (SQL)

**SQL databases** store data in tables with rows and columns, enforcing relationships between tables.

**Characteristics:**
- **Structured**: Fixed schema
- **ACID**: Atomicity, Consistency, Isolation, Durability
- **Relationships**: Foreign keys, joins
- **Schema**: Must define structure before inserting data

**Example Schema:**
```
Users Table:
┌────┬──────────┬─────────────┐
│ id │ name     │ email       │
├────┼──────────┼─────────────┤
│ 1  │ Alice    │ alice@...   │
│ 2  │ Bob      │ bob@...     │
└────┴──────────┴─────────────┘

Orders Table:
┌────┬─────────┬──────────┐
│ id │ user_id │ total    │
├────┼─────────┼──────────┤
│ 1  │ 1       │ 100.00   │
│ 2  │ 1       │ 50.00    │
└────┴─────────┴──────────┘
```

**Pros:**
- ACID guarantees
- Complex queries (joins, aggregations)
- Data integrity (constraints, foreign keys)
- Mature ecosystem

**Cons:**
- Hard to scale horizontally
- Schema changes are expensive
- Can be overkill for simple use cases

### NoSQL Databases

**NoSQL** databases store data in flexible, non-relational formats.

**Types:**

**1. Document Databases (MongoDB, CouchDB)**
```json
{
  "id": 1,
  "name": "Alice",
  "email": "alice@...",
  "orders": [
    {"id": 1, "total": 100.00},
    {"id": 2, "total": 50.00}
  ]
}
```

**2. Key-Value Stores (Redis, DynamoDB)**
```
key: "user:1"
value: {"name": "Alice", "email": "alice@..."}
```

**3. Column-Family (Cassandra, HBase)**
```
Row Key: user:1
Columns:
  name: "Alice"
  email: "alice@..."
  age: 30
```

**4. Graph Databases (Neo4j)**
```
Nodes: Users, Products
Edges: BOUGHT, LIKES, FRIENDS_WITH
```

**Characteristics:**
- **Flexible Schema**: No fixed structure
- **BASE**: Basically Available, Soft state, Eventual consistency
- **Horizontal Scaling**: Easy to scale across servers
- **High Performance**: Optimized for specific use cases

**Pros:**
- Easy horizontal scaling
- Flexible schema
- High performance for specific workloads
- Good for unstructured data

**Cons:**
- Eventual consistency (not always consistent)
- Limited query capabilities
- No joins (usually)
- Less mature tooling

### NewSQL

**What is NewSQL?**

**NewSQL** is a class of modern relational database management systems that combine the ACID guarantees and SQL interface of traditional SQL databases with the horizontal scalability and distributed architecture of NoSQL databases.

**The Problem NewSQL Solves:**

**Traditional SQL:**
- ACID guarantees ✓
- SQL interface ✓
- Complex queries ✓
- **But**: Hard to scale horizontally (must scale vertically - bigger hardware)

**NoSQL:**
- Horizontal scaling ✓
- High performance ✓
- **But**: No ACID (eventual consistency), limited query capabilities

**NewSQL Solution:**
- ACID guarantees ✓
- SQL interface ✓
- Horizontal scaling ✓
- Distributed architecture ✓

**How NewSQL Works:**

**1. Distributed Architecture:**
```
Traditional SQL:
┌─────────────────┐
│  Single Server  │
│   (All Data)    │
└─────────────────┘

NewSQL:
┌──────┐  ┌──────┐  ┌──────┐
│Node 1│  │Node 2│  │Node 3│
│Shard │  │Shard │  │Shard │
└──────┘  └──────┘  └──────┘
    └──────┬──────┘
    Distributed
```

**2. Auto-Sharding:**
- Data automatically distributed across nodes
- No manual sharding required
- System handles data placement

**3. ACID Across Nodes:**
- Transactions can span multiple nodes
- Maintains ACID guarantees even in distributed setup
- Uses distributed consensus algorithms (like Raft, Paxos)

**4. SQL Interface:**
- Standard SQL queries
- Works like traditional SQL database
- No need to learn new query language

**Examples of NewSQL Databases:**

**1. Google Spanner:**
- **Developer**: Google
- **Features**: Global distribution, strong consistency
- **Use Case**: Global applications needing strong consistency
- **Notable**: First globally distributed database with external consistency

**2. CockroachDB:**
- **Developer**: Cockroach Labs
- **Features**: PostgreSQL-compatible, automatic sharding, multi-region
- **Use Case**: Cloud-native applications needing SQL with scale
- **Notable**: Open source, inspired by Spanner

**3. TiDB:**
- **Developer**: PingCAP
- **Features**: MySQL-compatible, horizontal scaling, HTAP (Hybrid Transactional/Analytical Processing)
- **Use Case**: Large-scale applications needing MySQL compatibility with scale
- **Notable**: Popular in Asia, open source

**4. Amazon Aurora:**
- **Developer**: Amazon
- **Features**: MySQL/PostgreSQL compatible, cloud-native
- **Use Case**: AWS-based applications
- **Notable**: Serverless option available

**Characteristics:**

**1. SQL Interface:**
- Standard SQL queries
- Joins, transactions, ACID
- Works like traditional SQL

**2. ACID Transactions:**
- Full ACID guarantees
- Even across distributed nodes
- Strong consistency

**3. Horizontal Scaling:**
- Add nodes to scale
- Automatic data distribution
- No downtime for scaling

**4. Distributed Architecture:**
- Data replicated across nodes
- High availability
- Fault tolerance

**5. Auto-Sharding:**
- Automatic data partitioning
- No manual sharding logic needed
- System handles distribution

**When to Use NewSQL:**

**Good For:**
- Need SQL features (joins, transactions) at scale
- Want ACID guarantees in distributed system
- Need horizontal scaling
- Existing SQL applications that need to scale

**Not Good For:**
- Simple applications (overkill)
- Very high write throughput (NoSQL might be better)
- Cost-sensitive (can be expensive)

**NewSQL vs SQL vs NoSQL:**

| Feature | SQL | NoSQL | NewSQL |
|---------|-----|-------|--------|
| **ACID** | ✓ | ✗ (eventual) | ✓ |
| **SQL Interface** | ✓ | ✗ | ✓ |
| **Horizontal Scaling** | ✗ | ✓ | ✓ |
| **Complex Queries** | ✓ | Limited | ✓ |
| **Consistency** | Strong | Eventual | Strong |
| **Use Case** | Traditional apps | High scale, simple data | High scale, complex queries |

**Use Case Example:**

**Scenario**: E-commerce platform with millions of users

**Requirements:**
- Need complex queries (user orders, product recommendations)
- Need transactions (order processing)
- Need to scale horizontally (millions of users)
- Need strong consistency (inventory, payments)

**Solution: NewSQL**
- SQL interface for complex queries
- ACID for transactions
- Horizontal scaling for millions of users
- Strong consistency for critical operations

### Scaling: SQL vs NoSQL

**SQL Vertical Scaling:**
```
Single Server:
┌─────────────────┐
│   Database      │
│   (Bigger CPU,  │
│    More RAM)    │
└─────────────────┘
```

**Limitations:**
- Hardware limits
- Expensive
- Single point of failure

**NoSQL Horizontal Scaling:**
```
Multiple Servers:
┌─────────┐  ┌─────────┐  ┌─────────┐
│ Node 1  │  │ Node 2  │  │ Node 3  │
└─────────┘  └─────────┘  └─────────┘
```

**Advantages:**
- Add more servers as needed
- Cost-effective
- Better fault tolerance

### Database Normalization (3 Normal Forms)

**Goal**: Eliminate redundancy and ensure data integrity.

**1st Normal Form (1NF):**
- Each column contains atomic values (no arrays/lists)
- Each row is unique

**Before 1NF:**
```
┌────┬──────────┬──────────────────┐
│ id │ name     │ phone            │
├────┼──────────┼──────────────────┤
│ 1  │ Alice    │ 123, 456, 789    │
└────┴──────────┴──────────────────┘
```

**After 1NF:**
```
┌────┬──────────┬───────┐
│ id │ name     │ phone │
├────┼──────────┼───────┤
│ 1  │ Alice    │ 123   │
│ 1  │ Alice    │ 456   │
│ 1  │ Alice    │ 789   │
└────┴──────────┴───────┘
```

**2nd Normal Form (2NF):**
- Must be in 1NF
- All non-key attributes fully dependent on primary key

**Before 2NF:**
```
Orders:
┌────┬─────────┬──────────┬──────────┐
│ id │ user_id │ username │ total    │
├────┼─────────┼──────────┼──────────┤
│ 1  │ 1       │ Alice    │ 100.00   │
└────┴─────────┴──────────┴──────────┘
```
Problem: `username` depends on `user_id`, not `id`.

**After 2NF:**
```
Orders:              Users:
┌────┬─────────┬────┐  ┌────┬──────────┐
│ id │ user_id │total│  │ id │ username │
├────┼─────────┼────┤  ├────┼──────────┤
│ 1  │ 1       │100  │  │ 1  │ Alice    │
└────┴─────────┴────┘  └────┴──────────┘
```

**3rd Normal Form (3NF):**
- Must be in 2NF
- No transitive dependencies (non-key attribute depends on another non-key attribute)

**Before 3NF:**
```
Employees:
┌────┬──────────┬─────────┬──────────┐
│ id │ name     │ dept_id │ dept_loc │
├────┼──────────┼─────────┼──────────┤
│ 1  │ Alice    │ 1       │ NYC      │
└────┴──────────┴─────────┴──────────┘
```
Problem: `dept_loc` depends on `dept_id`, not `id`.

**After 3NF:**
```
Employees:          Departments:
┌────┬──────────┬────┐  ┌────┬──────────┐
│ id │ name     │d_id│  │ id │ location │
├────┼──────────┼────┤  ├────┼──────────┤
│ 1  │ Alice    │ 1  │  │ 1  │ NYC      │
└────┴──────────┴────┘  └────┴──────────┘
```

### ACID Properties (SQL)

**ACID** ensures reliable database transactions.

**1. Atomicity**
- All or nothing
- If any part fails, entire transaction rolls back

**Example:**
```
Transfer $100 from Account A to Account B:
1. Debit Account A: $100
2. Credit Account B: $100

If step 2 fails → Rollback step 1
Result: Either both succeed or both fail
```

**2. Consistency**
- Database remains in valid state
- Constraints are maintained

**Example:**
```
Constraint: Account balance >= 0
Transaction that would make balance negative → Rejected
```

**3. Isolation**
- Concurrent transactions don't interfere
- Each transaction sees consistent snapshot

**Example:**
```
Transaction A reads balance: $100
Transaction B updates balance: $150
Transaction A reads again: Still $100 (until A commits)
```

**4. Durability**
- Committed changes persist even after system failure

**Example:**
```
Transaction commits
System crashes
After restart: Changes are still there
```

### BASE Properties (NoSQL)

**What is BASE?**

**BASE** (Basically Available, Soft state, Eventual consistency) is a design philosophy for distributed systems that prioritizes availability and performance over strict consistency. It's often described as the opposite of ACID.

**Why BASE?**

In distributed systems, maintaining strict consistency (ACID) across all nodes can be:
- **Slow**: Must wait for all nodes to agree
- **Unavailable**: System may reject requests during network issues
- **Expensive**: Requires complex coordination

BASE trades strict consistency for:
- **Availability**: System always responds
- **Performance**: Faster responses
- **Scalability**: Easier to scale

**1. Basically Available**

**What it means:**
- System remains available (responds to requests) even during failures
- May return degraded or stale data, but always responds
- No request is rejected due to system state

**Example:**
```
Node A: Available
Node B: Available
Node C: Down (network partition)

User reads from Node A: Gets data ✓
User reads from Node B: Gets data ✓
System doesn't reject requests, even if some nodes are down
```

**Contrast with ACID:**
- **ACID**: May reject requests if can't guarantee consistency
- **BASE**: Always responds, even with potentially stale data

**2. Soft State**

**What it means:**
- System state may change without new input
- State is "soft" because it's not immediately consistent
- State will eventually become consistent, but not immediately

**Example:**
```
Initial State:
  Node A: x = 10
  Node B: x = 10
  Node C: x = 10

Write to Node A: x = 20
Current State (immediately after write):
  Node A: x = 20 (updated)
  Node B: x = 10 (not yet updated - soft state)
  Node C: x = 10 (not yet updated - soft state)

Later (after replication):
  Node A: x = 20
  Node B: x = 20 (eventually updated)
  Node C: x = 20 (eventually updated)
```

**Key Point:**
State is "soft" because it's in transition. It will harden (become consistent) eventually.

**3. Eventual Consistency**

**What it means:**
- System will become consistent eventually
- Not immediately consistent
- All nodes will eventually have the same data

**Example Timeline:**
```
Time 0: Write x = 20 to Node A
  Node A: x = 20
  Node B: x = 10 (old value)
  Node C: x = 10 (old value)
  → Inconsistent

Time 1: Replication to Node B
  Node A: x = 20
  Node B: x = 20 (updated)
  Node C: x = 10 (old value)
  → Still inconsistent

Time 2: Replication to Node C
  Node A: x = 20
  Node B: x = 20
  Node C: x = 20 (updated)
  → Eventually consistent!
```

**Real-World Example: Social Media Feed**

**Scenario:** User posts a photo

**With BASE (Eventual Consistency):**
```
1. User posts photo → Saved to Node A
2. User's friends immediately check feed:
   - Friend 1 (reads from Node A): Sees photo ✓
   - Friend 2 (reads from Node B): Doesn't see photo yet (eventual)
3. Replication happens (seconds later)
4. Friend 2 refreshes: Now sees photo ✓
5. Eventually, all nodes have the photo
```

**Trade-off:**
- **ACID**: All friends see photo immediately (consistent), but system may be slower
- **BASE**: Some friends may see photo later (eventual), but system is faster and more available

**BASE Summary:**

| Property | Meaning | Benefit |
|----------|---------|---------|
| **Basically Available** | Always responds | High availability |
| **Soft State** | State in transition | Flexibility |
| **Eventual Consistency** | Consistent eventually | Performance, scalability |

**When to Use BASE:**
- High availability required
- Performance is critical
- Can tolerate temporary inconsistency
- Examples: Social media, content delivery, analytics

### CAP Theorem

**What is the CAP Theorem?**

**CAP Theorem** (also known as Brewer's Theorem) states that in a distributed system, you can only guarantee **2 out of 3** properties:

**C - Consistency**: All nodes see the same data simultaneously
**A - Availability**: System remains operational and responds to requests
**P - Partition Tolerance**: System continues to operate despite network failures (partitions)

**Why Only 2 of 3?**

When a network partition occurs (nodes can't communicate), you must choose:
- **Consistency**: Reject requests to maintain consistency (sacrifice availability)
- **Availability**: Accept requests with potentially inconsistent data (sacrifice consistency)

You cannot have both consistency and availability during a partition.

**Visual Representation:**
```
        Consistency
            /\
           /  \
          /    \
         /      \
        /        \
Availability ─── Partition Tolerance
```

**The Three Properties Explained:**

**1. Consistency (C):**
- All nodes have the same data at the same time
- Read always returns the most recent write
- No stale data

**Example:**
```
Write x = 10 to Node A
Immediately read from any node:
  Node A: x = 10 ✓
  Node B: x = 10 ✓
  Node C: x = 10 ✓
All nodes see same value immediately
```

**2. Availability (A):**
- System responds to every request
- No request is rejected
- System is always operational

**Example:**
```
User makes request
System always responds (even if data is stale)
Never returns error due to system state
```

**3. Partition Tolerance (P):**
- System continues operating despite network failures
- Nodes can be separated (partitioned) and system still works
- Required for distributed systems

**Example:**
```
Network partition:
  Node A ──X── Node B (can't communicate)

System still operates:
  Node A: Continues serving requests
  Node B: Continues serving requests
```

**The Three Choices:**

**1. CP (Consistency + Partition Tolerance)**
- **Choose**: Consistency and Partition Tolerance
- **Sacrifice**: Availability

**How it works:**
- During partition, system rejects requests to maintain consistency
- Prevents inconsistent data, but system becomes unavailable

**Example:**
```
Network partition occurs:
  Node A ──X── Node B

User tries to write:
  System: "Cannot guarantee consistency, request rejected"
  → System unavailable during partition
```

**Real-World Examples:**
- Traditional SQL databases (PostgreSQL, MySQL in strict mode)
- Distributed databases with strong consistency (MongoDB with strong consistency mode)
- Financial systems (must be consistent)

**2. AP (Availability + Partition Tolerance)**
- **Choose**: Availability and Partition Tolerance
- **Sacrifice**: Consistency

**How it works:**
- During partition, system accepts requests
- May return stale or inconsistent data
- System remains available

**Example:**
```
Network partition occurs:
  Node A ──X── Node B

User reads from Node A:
  System: Returns data (may be stale) ✓
  → System available, but may be inconsistent
```

**Real-World Examples:**
- NoSQL databases (Cassandra, DynamoDB)
- Content delivery networks (CDN)
- Social media feeds (can show stale data)

**3. CA (Consistency + Availability)**
- **Choose**: Consistency and Availability
- **Sacrifice**: Partition Tolerance

**How it works:**
- Only works when there's no network partition
- If partition occurs, must choose CP or AP
- Not practical for distributed systems

**Example:**
```
Single node system:
  No partitions possible
  Can have both consistency and availability
```

**Real-World Examples:**
- Single-server databases
- Not practical for distributed systems (partitions will occur)

**CAP Theorem in Practice:**

**Most Distributed Systems Choose:**
- **CP**: When consistency is critical (financial, healthcare)
- **AP**: When availability is critical (social media, content delivery)

**Important Notes:**

**1. CAP is about trade-offs during partitions:**
- When no partition: Can have all three
- During partition: Must choose 2

**2. "Choose 2" doesn't mean ignore the third:**
- CP systems still try to be available when possible
- AP systems still try to be consistent when possible

**3. Modern systems are more nuanced:**
- Many systems provide tunable consistency
- Can choose consistency level per operation
- Example: Strong consistency for critical operations, eventual for others

**CAP Theorem Summary:**

| Choice | Guarantees | Sacrifices | Use Case |
|--------|-----------|------------|----------|
| **CP** | Consistency, Partition Tolerance | Availability | Financial systems, critical data |
| **AP** | Availability, Partition Tolerance | Consistency | Social media, content delivery |
| **CA** | Consistency, Availability | Partition Tolerance | Single node (not distributed) |

**Real-World Example: E-commerce System**

**Scenario:** User adds item to cart

**CP System (Strong Consistency):**
```
During network partition:
  User adds item to cart
  System: "Cannot guarantee consistency, please try again"
  → Rejects request, maintains consistency
```

**AP System (High Availability):**
```
During network partition:
  User adds item to cart
  System: "Item added" (may not be immediately visible everywhere)
  → Accepts request, remains available
  → Eventually consistent across all nodes
```

**AP (Availability + Partition Tolerance)**
- Example: NoSQL (Cassandra, DynamoDB)
- Sacrifice: Consistency (eventual consistency)

**CA (Consistency + Availability)**
- Not possible in distributed systems
- Only works if no network partitions (single node)

**Real-World:**
- Most systems choose **AP** (availability + partition tolerance)
- Accept eventual consistency for better availability

---

## SQL Injection and Prepared Statements

### What is SQL Injection?

**SQL Injection**: Attacker injects malicious SQL code into application input.

**Vulnerable Code:**
```python
query = "SELECT * FROM users WHERE username = '" + username + "'"
# If username = "admin' OR '1'='1"
# Query becomes:
# SELECT * FROM users WHERE username = 'admin' OR '1'='1'
# This returns all users!
```

**Attack Examples:**

**1. Authentication Bypass:**
```
Input: admin' OR '1'='1
Query: SELECT * FROM users WHERE username = 'admin' OR '1'='1'
Result: Returns all users (bypasses authentication)
```

**2. Data Extraction:**
```
Input: ' UNION SELECT password FROM users WHERE username='admin'--
Query: SELECT * FROM products WHERE name = '' UNION SELECT password FROM users WHERE username='admin'--
Result: Returns admin's password
```

**3. Data Deletion:**
```
Input: '; DROP TABLE users;--
Query: SELECT * FROM products WHERE name = ''; DROP TABLE users;--
Result: Deletes users table!
```

### How to Prevent SQL Injection

**Solution: Parameterized Statements (Prepared Statements)**

**How it works:**
1. **Compile**: SQL template is compiled once
2. **Parameter Binding**: User input is bound as parameters
3. **Execute**: Execute with parameters

**Example:**
```python
# Prepared Statement
query = "SELECT * FROM users WHERE username = ?"
stmt = prepare(query)
stmt.bind_param(1, username)  # username is treated as data, not SQL
result = stmt.execute()
```

**Why it works:**
- SQL structure is fixed (compiled)
- Parameters are treated as **data**, not SQL code
- Database escapes/encodes parameters automatically

**Visual:**
```
Normal Query:
SQL String: "SELECT * FROM users WHERE username = 'admin' OR '1'='1'"
           └─────────────────────────────────────────────────────┘
           Everything is SQL code

Prepared Statement:
SQL Template: "SELECT * FROM users WHERE username = ?"
              └───────────────────────────────┘ └─┘
              Fixed SQL structure          Parameter (data)
              
Parameter: "admin' OR '1'='1"
           └─────────────────┘
           Treated as string data, not SQL
```

### How Many Requests for Prepared Statement?

**Two requests:**
1. **Prepare/Compile**: Send SQL template, get statement handle
2. **Execute**: Send parameters, get results

**First Request (Prepare):**
```
Client → Database: PREPARE stmt FROM "SELECT * FROM users WHERE username = ?"
Database: Statement compiled, returns handle
```

**Subsequent Requests (Execute):**
```
Client → Database: EXECUTE stmt USING 'alice'
Database: Returns results
```

**Reusing Prepared Statements:**
- ✅ **Yes, you can reuse!**
- Compile once, execute many times
- **Performance benefit**: No recompilation overhead
- **Security benefit**: Always parameterized

**Example:**
```python
# Prepare once
stmt = prepare("SELECT * FROM users WHERE username = ?")

# Execute many times with different parameters
for username in usernames:
    result = stmt.execute(username)  # Fast!
```

---

## Indexing

### What is an Index?

**Index** is like a book's index. Instead of reading entire book to find a topic, you check the index to find the page number.

**Without Index:**
```
Find user with email = "alice@example.com"
→ Scan entire table (Full Table Scan)
→ O(n) complexity
```

**With Index:**
```
Find user with email = "alice@example.com"
→ Look up in index
→ Get row location
→ O(log n) complexity
```

### How Indexing Works Internally

**Data Structure: B-Tree (or B+ Tree)**

**Why B-Tree?**
- **Balanced**: All leaf nodes at same level
- **Efficient**: O(log n) search, insert, delete
- **Disk-friendly**: Minimizes disk I/O (important for databases)

**B-Tree Structure:**
```
                    [50]
                   /    \
              [20]        [80]
             /   \       /    \
        [10] [30] [60] [90]
         │    │    │    │
        Leaf nodes point to data rows
```

**Search Process:**
```
Find value 30:
1. Start at root [50]
2. 30 < 50 → Go left to [20]
3. 30 > 20 → Go right to [30]
4. Found! → Follow pointer to data row
```

### Composite Index

**Composite Index**: Index on multiple columns.

**Example:**
```sql
CREATE INDEX idx_name_age ON users(name, age);
```

**How it works:**
- Indexes are ordered by first column, then second, etc.
- Like sorting by last name, then first name

**Query:**
```sql
SELECT * FROM users WHERE name = 'Alice' AND age = 30;
```
✅ Can use index (both columns in WHERE)

```sql
SELECT * FROM users WHERE age = 30;
```
❌ Cannot use index efficiently (age is second column)

**Rule**: Leftmost prefix rule
- Index on (A, B, C) can be used for:
  - (A)
  - (A, B)
  - (A, B, C)
- Cannot be used for:
  - (B)
  - (C)
  - (B, C)

### How to Know Query Uses Index

**1. EXPLAIN Plan:**
```sql
EXPLAIN SELECT * FROM users WHERE email = 'alice@example.com';
```

**Output:**
```
type: ref          (uses index)
key: idx_email     (index name)
rows: 1            (rows examined)
```

**2. Query Performance:**
- Fast query → Likely using index
- Slow query → Likely full table scan

**3. Database Monitoring:**
- Check slow query log
- Monitor index usage statistics

### Index with WHERE Clauses

**Equality: `WHERE age = 5`**
```
Index: [1][2][3][4][5][6][7]...
                    ↑
              Direct lookup
Complexity: O(log n)
```

**Range: `WHERE age > 5`**
```
Index: [1][2][3][4][5][6][7][8]...
                    ↑
              Start here, scan right
Complexity: O(log n + k) where k = number of matching rows
```

**Next Record Complexity:**
- **B+ Tree**: O(1) - Leaf nodes linked
- **B-Tree**: O(log n) - Need to traverse tree

**Visual:**
```
B+ Tree (leaf nodes linked):
[5] → [6] → [7] → [8] → ...
 ↑
Start here, follow links
```

### Indexing with Strings

**Challenges:**
- Strings can be long
- Comparison is slower than integers
- Prefix indexing often used

**Solutions:**

**1. Full Index:**
- Index entire string
- Good for exact matches
- Larger index size

**2. Prefix Index:**
```sql
CREATE INDEX idx_name ON users(name(10));  -- First 10 characters
```
- Smaller index
- Good for LIKE queries with prefix
- Cannot use for suffix searches

**3. Hash Index:**
- Hash string to fixed size
- Fast lookups
- Cannot use for range queries

---

## Query Optimization

### Query Complexity

**How to measure:**
- **Time Complexity**: How execution time grows with data size
- **Space Complexity**: Memory usage
- **I/O Complexity**: Disk reads/writes

**Factors:**
- Table size
- Index usage
- Join operations
- Aggregations
- Sorting

### SQL Query Optimizer

**What optimizer does:**
1. **Parse**: Parse SQL query
2. **Analyze**: Check table statistics, indexes
3. **Plan**: Generate execution plans
4. **Choose**: Select best plan (lowest cost)
5. **Execute**: Run chosen plan

**Optimization Techniques:**

**1. Index Selection:**
- Choose best index for query
- Consider index selectivity

**2. Join Order:**
- Join smaller tables first
- Use indexes for joins

**3. Predicate Pushdown:**
- Apply WHERE filters early
- Reduce data processed

**4. Projection Pushdown:**
- Select only needed columns
- Reduce data transferred

### Query Comparisons

**`WHERE id = 'a' AND id = 'b' AND id = 'c'`**
```
This is impossible! (id cannot be 'a' AND 'b' AND 'c')
Optimizer: Returns empty result immediately
Complexity: O(1) - No table scan
```

**`WHERE id IN ('a', 'b', 'c')`**
```
Use index lookup for each value
Complexity: O(3 * log n) = O(log n)
Much faster than multiple OR conditions
```

### Complex Query: ORDER BY with LIMIT/OFFSET

**Query:**
```sql
SELECT * FROM abc ORDER BY name LIMIT 10 OFFSET 1000000;
```

**Problem:**
1. Must sort entire table (O(n log n))
2. Then skip 1,000,000 rows
3. Then take 10 rows

**Complexity:**
- Sorting: O(n log n)
- Total: Very slow for large tables

**Optimization Strategies:**

**1. Index on name:**
```sql
CREATE INDEX idx_name ON abc(name);
```
- Can use index for sorting
- Still need to skip offset (problematic)

**2. Cursor-based Pagination:**
```sql
-- Instead of OFFSET
SELECT * FROM abc WHERE name > 'last_seen_name' ORDER BY name LIMIT 10;
```
- No offset needed
- Much faster

**3. Materialized View:**
- Pre-sort and store
- Query materialized view

### COUNT(*) Complexity

**Question**: What's the complexity of `COUNT(*)`?

**Answer**: Depends on implementation:

**1. Full Table Scan:**
```sql
SELECT COUNT(*) FROM users;
```
- O(n) - Must scan all rows
- Slow for large tables

**2. Index Scan:**
- If index exists, count index entries
- Still O(n) but faster (index smaller than table)

**3. Approximate Count:**
- Some databases store row count
- O(1) but approximate

**Optimization:**
- Maintain counter table
- Update on insert/delete
- Trade-off: Counter may be slightly stale

### Avoiding Full Table Scan

**Ways to avoid:**

**1. Use Indexes:**
```sql
-- Bad (full scan)
SELECT * FROM users WHERE email = 'alice@example.com';
-- No index on email

-- Good (index scan)
CREATE INDEX idx_email ON users(email);
SELECT * FROM users WHERE email = 'alice@example.com';
```

**2. Use WHERE with Indexed Columns:**
```sql
-- Uses index
SELECT * FROM users WHERE id = 123;

-- Full scan
SELECT * FROM users WHERE name LIKE '%alice%';
```

**3. Limit Results:**
```sql
-- Still full scan, but stops early
SELECT * FROM users WHERE name LIKE '%alice%' LIMIT 10;
```

### JOIN Complexity

**Types of Joins:**

**1. INNER JOIN:**
```sql
SELECT * FROM users u
INNER JOIN orders o ON u.id = o.user_id;
```

**Complexity:**
- **Nested Loop**: O(n * m) - For each row in users, scan orders
- **Hash Join**: O(n + m) - Build hash table, probe
- **Merge Join**: O(n log n + m log m) - Sort both, merge

**2. LEFT/RIGHT OUTER JOIN:**
- Similar complexity to INNER JOIN
- Additional work to include unmatched rows

**Optimization:**
- Index on join columns
- Choose join algorithm based on table sizes
- Filter early (WHERE before JOIN)

---

## Database Replication

### What is Replication?

**Replication**: Copying data from one database (master) to others (slaves/replicas).

**Why replicate?**
- **High Availability**: If master fails, slave can take over
- **Read Scaling**: Distribute read queries across slaves
- **Backup**: Slaves serve as backups
- **Geographic Distribution**: Place slaves near users

### Replication Architecture

**Master-Slave (Primary-Replica):**
```
Master (Primary)
    │
    ├──→ Slave 1 (Replica)
    ├──→ Slave 2 (Replica)
    └──→ Slave 3 (Replica)
```

**Characteristics:**
- **Master**: Handles writes
- **Slaves**: Handle reads (and can be promoted to master)
- **One-way**: Data flows master → slaves

### Binary Log (Binlog)

**Binary Log**: Log of all changes (writes) to database.

**What's logged:**
- INSERT statements
- UPDATE statements
- DELETE statements
- Schema changes

**Format:**
```
Time: 10:00:01 | Query: INSERT INTO users VALUES (1, 'Alice')
Time: 10:00:02 | Query: UPDATE users SET name='Bob' WHERE id=1
Time: 10:00:03 | Query: DELETE FROM users WHERE id=1
```

### Master-Slave Synchronization

**Process:**

**1. Master writes to binlog:**
```
Master executes: INSERT INTO users VALUES (1, 'Alice')
Master writes to binlog: [INSERT INTO users VALUES (1, 'Alice')]
```

**2. Slave reads binlog:**
```
Slave connects to master
Slave requests: "Give me changes after position X"
Master sends: Binlog entries
```

**3. Slave applies changes:**
```
Slave receives: INSERT INTO users VALUES (1, 'Alice')
Slave executes: INSERT INTO users VALUES (1, 'Alice')
Slave is now in sync
```

**Visual Timeline:**
```
Master:  [Write] → [Binlog] → [Send to Slave]
Slave:   [Receive] → [Apply] → [In Sync]
```

### Slave of Slave (Cascading Replication)

**Question**: Can a slave be a slave of another slave?

**Answer**: Yes! This is called **cascading replication**.

**Architecture:**
```
Master
  │
  └──→ Slave 1 (also acts as master)
         │
         └──→ Slave 2
```

**Why use it?**
- Reduce load on master
- Geographic distribution
- Network optimization

**Trade-off:**
- Increased latency (more hops)
- More complex setup

---

## Database Sharding

### What is Sharding?

**Sharding**: Splitting a large database into smaller, manageable pieces (shards).

**Analogy**: Like splitting a large library into multiple smaller libraries, each containing books for a specific range (A-F, G-M, N-Z).

**Why shard?**
- **Scale**: Single database can't handle load
- **Performance**: Smaller databases are faster
- **Storage**: Distribute data across servers

### When to Shard

**Indicators:**
- Database size too large for single server
- Query performance degrading
- Write throughput bottleneck
- Storage limits approaching

**Before Sharding:**
- Optimize queries
- Add indexes
- Use read replicas
- Consider if sharding is really needed

### Sharding Strategies

**1. Range-Based Sharding:**
```
Shard 1: user_id 1-1000000
Shard 2: user_id 1000001-2000000
Shard 3: user_id 2000001-3000000
```

**Pros:**
- Simple to implement
- Easy range queries

**Cons:**
- Uneven distribution (hot spots)
- Difficult to rebalance

**2. Hash-Based Sharding:**
```
hash(user_id) % 3 → Shard number
```

**Example:**
```
user_id = 123
hash(123) = 456789
456789 % 3 = 0 → Shard 0
```

**Pros:**
- Even distribution
- No hot spots

**Cons:**
- Cannot do range queries across shards
- Rebalancing requires moving data

**3. Directory-Based Sharding:**
```
Lookup table:
user_id → shard_id
```

**Pros:**
- Flexible
- Easy to rebalance

**Cons:**
- Lookup overhead
- Single point of failure (lookup service)

### Ensuring Global Primary Key Uniqueness

**Problem**: Each shard could generate same ID.

**Solutions:**

**1. UUID/GUID:**
```
Shard 1: user_id = uuid1()
Shard 2: user_id = uuid2()
Shard 3: user_id = uuid3()
All unique globally
```

**2. Composite Key:**
```
Primary Key: (shard_id, local_id)
Shard 1: (1, 1), (1, 2), (1, 3)
Shard 2: (2, 1), (2, 2), (2, 3)
All unique
```

**3. ID Generation Service:**
```
Central service generates IDs:
Shard 1 requests: ID → 1
Shard 2 requests: ID → 2
Shard 3 requests: ID → 3
```

**4. Twitter Snowflake:**
```
64-bit ID:
[Timestamp][Machine ID][Sequence]
Ensures uniqueness across machines
```

### Sharding Implementation

**Same Server, Different Tables:**
```
Database: myapp
├── users_shard_1
├── users_shard_2
└── users_shard_3
```

**Different Servers:**
```
Server 1: users_shard_1
Server 2: users_shard_2
Server 3: users_shard_3
```

### Querying Across Shards

**Problem**: Query needs data from multiple shards.

**Example:**
```sql
SELECT * FROM users WHERE age > 30;
-- Users with age > 30 might be in any shard
```

**Solutions:**

**1. Scatter-Gather:**
```
Query Router:
  1. Send query to all shards
  2. Gather results
  3. Merge and return
```

**Visual:**
```
Query Router
    ├──→ Shard 1: SELECT * FROM users WHERE age > 30
    ├──→ Shard 2: SELECT * FROM users WHERE age > 30
    └──→ Shard 3: SELECT * FROM users WHERE age > 30
         ↓
    Merge Results
         ↓
    Return to Client
```

**2. Query Routing:**
```
If query has shard key:
  Route to specific shard
Else:
  Scatter-gather
```

**Challenges:**
- **Joins across shards**: Very expensive
- **Transactions across shards**: Complex (distributed transactions)
- **Aggregations**: Need to aggregate results from all shards

---

## Transactions and Concurrency

### What is a Transaction?

**Transaction**: Group of operations that execute as a single unit.

**Properties (ACID):**
- **Atomicity**: All or nothing
- **Consistency**: Valid state
- **Isolation**: Concurrent transactions don't interfere
- **Durability**: Changes persist

**Example:**
```sql
BEGIN TRANSACTION;
  UPDATE accounts SET balance = balance - 100 WHERE id = 1;
  UPDATE accounts SET balance = balance + 100 WHERE id = 2;
COMMIT;
```

If either update fails → Both roll back.

### How Rollback Works

**Rollback**: Undo changes made by transaction.

**Mechanism: Transaction Log (Write-Ahead Logging)**

**Process:**
```
1. Before writing to database:
   Write change to transaction log

2. Write to database

3. If commit:
   Mark transaction as committed in log
   
4. If rollback:
   Read log backwards
   Undo each change
```

**Example:**
```
Transaction Log:
[T1, UPDATE accounts, id=1, old_balance=200, new_balance=100]
[T1, UPDATE accounts, id=2, old_balance=50, new_balance=150]

If rollback:
  Undo: Set id=1 balance back to 200
  Undo: Set id=2 balance back to 50
```

### Concurrency Problems

**1. Dirty Read:**
```
Transaction A reads uncommitted data from Transaction B
```

**Example:**
```
T1: UPDATE accounts SET balance = 200 WHERE id = 1;
T2: SELECT balance FROM accounts WHERE id = 1;  -- Reads 200
T1: ROLLBACK;  -- Balance back to 100
T2: Has wrong data (200 instead of 100)
```

**2. Dirty Write:**
```
Transaction A overwrites uncommitted data from Transaction B
```

**Example:**
```
T1: UPDATE accounts SET balance = 200 WHERE id = 1;
T2: UPDATE accounts SET balance = 300 WHERE id = 1;
T1: COMMIT;  -- Balance = 200
T2: COMMIT;  -- Balance = 300 (T1's change lost)
```

**3. Read Skew (Non-Repeatable Read):**
```
Transaction reads same data twice, gets different values
```

**Example:**
```
T1: SELECT balance FROM accounts WHERE id = 1;  -- Returns 100
T2: UPDATE accounts SET balance = 200 WHERE id = 1; COMMIT;
T1: SELECT balance FROM accounts WHERE id = 1;  -- Returns 200
T1: Sees different value in same transaction
```

**4. Phantom Read:**
```
Transaction sees new rows inserted by another transaction
```

**Example:**
```
T1: SELECT COUNT(*) FROM accounts WHERE balance > 100;  -- Returns 5
T2: INSERT INTO accounts VALUES (6, 150); COMMIT;
T1: SELECT COUNT(*) FROM accounts WHERE balance > 100;  -- Returns 6
T1: Sees new row that didn't exist before
```

**5. Write Skew:**
```
Two transactions read same data, make different updates based on it
```

**Example:**
```
Two doctors on call:
T1: SELECT on_call FROM doctors WHERE id = 1;  -- Returns true
T2: SELECT on_call FROM doctors WHERE id = 2;  -- Returns true
T1: UPDATE doctors SET on_call = false WHERE id = 1;
T2: UPDATE doctors SET on_call = false WHERE id = 2;
Result: No doctors on call! (Should have at least one)
```

**6. Lost Update:**
```
Two transactions update same row, one update is lost
```

**Example:**
```
T1: SELECT balance FROM accounts WHERE id = 1;  -- 100
T2: SELECT balance FROM accounts WHERE id = 1;  -- 100
T1: UPDATE accounts SET balance = 150 WHERE id = 1;  -- balance + 50
T2: UPDATE accounts SET balance = 120 WHERE id = 1;  -- balance + 20
Result: Balance = 120 (T1's +50 is lost)
```

### Transaction Isolation Levels

**Isolation Levels** (from weakest to strongest):

**1. Read Uncommitted:**
- Allows dirty reads
- No isolation
- Fastest but least safe

**2. Read Committed:**
- Prevents dirty reads
- Allows non-repeatable reads, phantom reads
- Default in many databases

**3. Repeatable Read:**
- Prevents dirty reads, non-repeatable reads
- Allows phantom reads
- Locks rows read

**4. Serializable:**
- Prevents all problems
- Highest isolation
- Slowest (locks entire tables)

### How Transactions Handle Concurrent Requests

**Mechanisms:**

**1. Locking:**
```
Transaction locks data it accesses
Other transactions must wait
```

**Types:**
- **Shared Lock (Read Lock)**: Multiple readers allowed
- **Exclusive Lock (Write Lock)**: Only one writer, no readers

**2. Multi-Version Concurrency Control (MVCC):**
```
Each transaction sees snapshot of data
No locking needed for reads
```

**How it works:**
```
Time →
T1 starts: Sees snapshot at time T
T2 starts: Sees snapshot at time T
T1 updates row → Creates new version
T2 still sees old version
```

**3. Timestamp Ordering:**
```
Each transaction has timestamp
Order transactions by timestamp
```

### Avoiding Race Conditions

**1. Pessimistic Locking:**
```
Lock data before reading
Prevents others from modifying
```

**Example:**
```sql
SELECT * FROM accounts WHERE id = 1 FOR UPDATE;
-- Row is locked, others must wait
UPDATE accounts SET balance = balance + 100 WHERE id = 1;
```

**2. Optimistic Locking:**
```
Read data with version number
Update only if version unchanged
```

**Example:**
```sql
-- Read
SELECT id, balance, version FROM accounts WHERE id = 1;
-- version = 5

-- Update (only if version still 5)
UPDATE accounts 
SET balance = 150, version = 6 
WHERE id = 1 AND version = 5;

-- If version changed → Update fails (retry)
```

**3. Atomic Operations:**
```
Use database atomic operations
```

**Example:**
```sql
-- Atomic increment
UPDATE accounts SET balance = balance + 100 WHERE id = 1;
-- No race condition possible
```

### Distributed Transactions

**Problem**: Transaction needs to access multiple databases.

**Example:**
```
Transfer money:
  Debit from Database A
  Credit to Database B
Both must succeed or both fail
```

**Solutions:**

**1. Two-Phase Commit (2PC):**
```
Phase 1 (Prepare):
  Coordinator: "Can you commit?"
  Database A: "Yes, ready"
  Database B: "Yes, ready"

Phase 2 (Commit):
  Coordinator: "Commit!"
  Database A: Commits
  Database B: Commits
```

**Problems:**
- Blocking (if coordinator fails, all wait)
- Slow (multiple round trips)

**2. Saga Pattern:**
```
Instead of 2PC, use compensating transactions
```

**Example:**
```
1. Debit Database A
2. Credit Database B
3. If step 2 fails:
   Compensate: Credit Database A (undo step 1)
```

**3. Try-Confirm-Cancel (TCC):**
```
Try Phase: Reserve resources
Confirm Phase: Commit if all succeeded
Cancel Phase: Release resources if any failed
```

**Example:**
```
Try:
  Reserve $100 in Account A
  Reserve $100 in Account B

If both succeed:
  Confirm: Actually transfer
Else:
  Cancel: Release reservations
```

---

## Summary

Databases are the foundation of backend systems. Understanding SQL vs NoSQL, indexing, query optimization, replication, sharding, and transactions is essential for building scalable, reliable systems.

**Key Takeaways:**
- SQL provides ACID, NoSQL provides scalability
- Indexes dramatically improve query performance
- Replication provides availability and read scaling
- Sharding enables horizontal scaling
- Transactions ensure data consistency
- Choose the right tool for your use case

