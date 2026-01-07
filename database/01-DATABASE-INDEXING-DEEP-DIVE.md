# Database Indexing Deep Dive - Complete Understanding

## Table of Contents
1. [What is an Index and Why Do We Need It?](#what-is-an-index-and-why-do-we-need-it)
2. [How Indexing Works Internally - Data Structures](#how-indexing-works-internally---data-structures)
3. [B-Tree and B+ Tree - The Foundation of Indexing](#b-tree-and-b-tree---the-foundation-of-indexing)
4. [Types of Indexes](#types-of-indexes)
5. [Composite Indexes - Multi-Column Indexing](#composite-indexes---multi-column-indexing)
6. [Index Selection and Query Optimization](#index-selection-and-query-optimization)
7. [When to Use Indexes - Best Practices](#when-to-use-indexes---best-practices)
8. [Index Maintenance and Overhead](#index-maintenance-and-overhead)
9. [Indexing Strategies for Different Data Types](#indexing-strategies-for-different-data-types)
10. [Common Indexing Mistakes and How to Avoid Them](#common-indexing-mistakes-and-how-to-avoid-them)

---

## What is an Index and Why Do We Need It?

### The Problem Without Indexes

**Scenario: Finding a book in a library without a catalog**

Imagine you need to find a book titled "Database Design" in a library with 1 million books:

**Without Index (Full Table Scan):**
```
1. Start at first shelf
2. Check every book
3. Continue until found
4. Worst case: Check all 1 million books
5. Time: O(n) - Linear search
```

**With Index (Like library catalog):**
```
1. Look up "Database Design" in catalog
2. Catalog says: "Shelf 5, Row 3, Book 12"
3. Go directly to that location
4. Time: O(log n) - Logarithmic search
```

### Real Database Example

**Table: `users` with 10 million rows**

**Query without index:**
```sql
SELECT * FROM users WHERE email = 'alice@example.com';
```

**What happens:**
```
Database must:
1. Read every row (10 million rows)
2. Check if email matches
3. Return matching row
4. Time: Very slow (seconds or minutes)
```

**Query with index:**
```sql
-- Index exists on email column
SELECT * FROM users WHERE email = 'alice@example.com';
```

**What happens:**
```
Database:
1. Look up 'alice@example.com' in index
2. Index says: "Row at position 5,234,567"
3. Go directly to that row
4. Time: Very fast (milliseconds)
```

### What is an Index?

**Definition:**
An **index** is a data structure that improves the speed of data retrieval operations on a database table. It's like a book's index - instead of reading the entire book to find a topic, you look it up in the index and go directly to the page.

**Key Characteristics:**
- **Separate data structure**: Stored separately from table data
- **Sorted**: Organized for fast lookup
- **Points to data**: Contains pointers to actual rows
- **Trade-off**: Faster reads, slower writes (must maintain index)

### Analogy: Phone Book vs Random List

**Without Index (Random List):**
```
Names in random order:
- Bob: 555-1234
- Alice: 555-5678
- Charlie: 555-9012
- ...

To find Alice:
- Check first name: Bob (no)
- Check second name: Alice (yes!)
- Average: Check half the list
```

**With Index (Phone Book - Sorted):**
```
Names in alphabetical order:
- Alice: 555-5678
- Bob: 555-1234
- Charlie: 555-9012
- ...

To find Alice:
- Open to middle: "M" names
- Go left: "A" names
- Find Alice quickly
- Average: Check log(n) names
```

### Performance Comparison

**Table with 1 million rows:**

| Operation | Without Index | With Index | Improvement |
|-----------|---------------|------------|-------------|
| **Find by ID** | 1,000,000 reads | ~20 reads | 50,000x faster |
| **Find by email** | 1,000,000 reads | ~20 reads | 50,000x faster |
| **Insert** | 1 write | 2 writes (table + index) | 2x slower |
| **Update** | 1 read + 1 write | 2 reads + 2 writes | 2x slower |
| **Delete** | 1 read + 1 write | 2 reads + 2 writes | 2x slower |

**Key Insight:**
- **Reads**: Indexes make reads MUCH faster
- **Writes**: Indexes make writes slightly slower
- **Trade-off**: Optimize for your use case (read-heavy vs write-heavy)

---

## How Indexing Works Internally - Data Structures

### Why Not Simple Arrays or Hash Tables?

**Array (Sorted):**
```
Pros:
  - Binary search: O(log n)
  - Simple

Cons:
  - Insert/delete: O(n) - must shift elements
  - Not good for frequent updates
```

**Hash Table:**
```
Pros:
  - Lookup: O(1) average
  - Fast

Cons:
  - Range queries: O(n) - must scan all
  - Not sorted
  - Collision handling overhead
```

**B-Tree (What databases use):**
```
Pros:
  - Lookup: O(log n)
  - Range queries: O(log n + k) where k = results
  - Insert/delete: O(log n)
  - Sorted
  - Disk-friendly

Cons:
  - More complex
  - More memory overhead
```

### Why B-Tree for Databases?

**Key Requirements for Database Indexes:**

**1. Must be Disk-Friendly:**
- Data doesn't fit in memory
- Must minimize disk I/O
- B-Tree nodes match disk page size

**2. Must Support Range Queries:**
```sql
WHERE age BETWEEN 20 AND 30
WHERE name > 'Alice'
```
- Hash tables can't do this efficiently
- B-Tree maintains sorted order

**3. Must Handle Frequent Updates:**
- Inserts, updates, deletes
- B-Tree handles this well (O(log n))

**4. Must Scale:**
- Millions/billions of rows
- B-Tree depth grows logarithmically

---

## B-Tree and B+ Tree - The Foundation of Indexing

### Understanding B-Tree Structure

**B-Tree Properties:**
- **Balanced**: All leaf nodes at same level
- **Ordered**: Keys sorted within nodes
- **Branching Factor**: Each node has multiple children (not just 2)
- **Min/Max Keys**: Each node has minimum and maximum number of keys

**Visual B-Tree (Order 3):**
```
                    [50]
                   /    \
              [20, 30]  [70, 80]
             /   |   \   |   |   \
        [10] [25] [35] [60] [75] [90]
```

**Key Concepts:**

**1. Internal Nodes:**
- Store keys and pointers to children
- Keys act as separators
- Don't store actual data (in B+ Tree)

**2. Leaf Nodes:**
- Store keys and data (or pointers to data)
- All leaf nodes at same level
- Linked together (in B+ Tree)

**3. Search Process:**
```
Find key 25:
1. Start at root [50]
2. 25 < 50 → Go left to [20, 30]
3. 20 < 25 < 30 → Go to middle child
4. Found at leaf [25]
```

### B+ Tree - Optimized for Databases

**B+ Tree Improvements over B-Tree:**

**1. Data Only in Leaves:**
```
B-Tree: Data in all nodes
B+ Tree: Data only in leaf nodes
```

**Why:**
- Internal nodes can store more keys (no data)
- Fewer levels
- Faster searches

**2. Leaf Nodes Linked:**
```
Leaf 1 → Leaf 2 → Leaf 3 → ...
```

**Why:**
- Range queries are fast
- Sequential scan is efficient
- No need to traverse tree for ranges

**Visual B+ Tree:**
```
                    [50]
                   /    \
              [20, 30]  [70, 80]
             /   |   \   |   |   \
        [10→] [25→] [35→] [60→] [75→] [90→]
         ↓     ↓     ↓     ↓     ↓     ↓
        Data  Data  Data  Data  Data  Data
         ↓     ↓     ↓     ↓     ↓     ↓
        →→→→→→→→→→→→→→→→→→→→→→→→→→→→→→→→
        (Linked leaf nodes)
```

### B+ Tree Operations

#### Search

**Process:**
```
1. Start at root
2. Compare key with node keys
3. Follow appropriate pointer
4. Repeat until leaf
5. Search leaf node
6. Return data (or pointer)
```

**Complexity: O(log n)**
- Height of tree: log(n)
- Each level: One disk read
- Total: log(n) disk reads

**Example:**
```
Find key 25 in tree with 1 million keys:
Height: log₂(1,000,000) ≈ 20 levels
Disk reads: ~20 (very fast!)
```

#### Insert

**Process:**
```
1. Search for insertion point
2. Insert into leaf node
3. If leaf full → Split
4. Propagate split upward if needed
```

**Example:**
```
Insert 15 into [10, 20] (full):
1. Split: [10, 15] and [20]
2. Promote 15 to parent
3. Update parent node
```

**Complexity: O(log n)**
- Search: O(log n)
- Insert: O(1) if space
- Split: O(log n) worst case

#### Delete

**Process:**
```
1. Search for key
2. Delete from leaf
3. If leaf underflow → Merge or redistribute
4. Propagate changes upward if needed
```

**Complexity: O(log n)**

### Why B+ Tree Depth Matters

**Tree with 1 million keys:**
```
Branching factor: 100 (each node has up to 100 children)
Height: log₁₀₀(1,000,000) ≈ 3 levels

Level 0 (root): 1 node
Level 1: 100 nodes
Level 2: 10,000 nodes (leaves)
```

**Disk Reads:**
- Search: 3 disk reads (one per level)
- Very fast!

**Compare to binary tree:**
```
Binary tree: log₂(1,000,000) ≈ 20 levels
B+ Tree: 3 levels
Much better!
```

---

## Types of Indexes

### 1. Primary Key Index

**Definition:**
- Automatically created for primary key
- Unique, not null
- Clustered (in most databases)

**Example:**
```sql
CREATE TABLE users (
    id INT PRIMARY KEY,  -- Index automatically created
    name VARCHAR(100)
);
```

**Characteristics:**
- **Clustered**: Data physically ordered by primary key
- **Unique**: No duplicates
- **Fast**: Direct access by primary key

**Visual:**
```
Index (B+ Tree):
        [50]
       /    \
   [25]      [75]
  /   \     /   \
[10] [30] [60] [90]

Data (physically ordered by id):
[10] → [25] → [30] → [50] → [60] → [75] → [90]
```

### 2. Secondary Index (Non-Clustered)

**Definition:**
- Index on non-primary key column
- Points to primary key (or row ID)
- Separate from data

**Example:**
```sql
CREATE INDEX idx_email ON users(email);
```

**How it works:**
```
Index (B+ Tree on email):
        [m@example.com]
       /                \
[alice@...]          [zoe@...]

Points to primary keys:
alice@example.com → id=5
m@example.com → id=10
zoe@example.com → id=20

Then lookup by id in primary key index
```

**Two-Step Process:**
```
1. Search email index → Get id
2. Search primary key index → Get row
```

### 3. Unique Index

**Definition:**
- Ensures uniqueness
- Like primary key, but can have nulls (usually)

**Example:**
```sql
CREATE UNIQUE INDEX idx_email ON users(email);
```

**Use Case:**
- Email addresses (unique)
- Usernames (unique)
- Social security numbers (unique)

### 4. Composite Index (Multi-Column)

**Definition:**
- Index on multiple columns
- Order matters!

**Example:**
```sql
CREATE INDEX idx_name_age ON users(name, age);
```

**How it works:**
```
Index sorted by (name, age):
(alice, 20)
(alice, 25)
(alice, 30)
(bob, 20)
(bob, 25)
(charlie, 20)
```

**Leftmost Prefix Rule:**
```
Index on (name, age) can be used for:
✅ WHERE name = 'alice'
✅ WHERE name = 'alice' AND age = 25
❌ WHERE age = 25  (age is second column)
```

**Why?**
- Index is sorted by name first, then age
- To use for age alone, would need separate index

### 5. Partial Index

**Definition:**
- Index on subset of rows
- Only indexes rows matching condition

**Example:**
```sql
CREATE INDEX idx_active_users ON users(email) 
WHERE status = 'active';
```

**Use Case:**
- Most queries filter by status='active'
- Smaller index (only active users)
- Faster queries

### 6. Covering Index

**Definition:**
- Index contains all columns needed for query
- No need to access table data

**Example:**
```sql
-- Query
SELECT name, email FROM users WHERE email = 'alice@example.com';

-- Covering index
CREATE INDEX idx_email_name ON users(email, name);
```

**Why Fast:**
```
Without covering index:
1. Search email index → Get id
2. Lookup by id → Get row
3. Extract name, email

With covering index:
1. Search email index → Get name, email
2. Done! (no table lookup)
```

### 7. Hash Index

**Definition:**
- Uses hash function
- O(1) average lookup
- No range queries

**Use Case:**
- Exact match only
- No sorting needed
- Memory databases (Redis)

**Limitation:**
```
Can't do:
WHERE age > 25  (range query)
WHERE name LIKE 'A%'  (prefix search)

Can do:
WHERE id = 123  (exact match)
```

---

## Composite Indexes - Multi-Column Indexing

### Understanding Composite Index Order

**Index: (name, age, city)**

**How it's sorted:**
```
1. First by name (alphabetical)
2. Then by age (if names equal)
3. Then by city (if name and age equal)

Data:
(alice, 20, NYC)
(alice, 25, LA)
(alice, 25, SF)
(bob, 20, NYC)
(bob, 30, LA)
```

### Leftmost Prefix Rule - Deep Dive

**Index: (A, B, C)**

**Can use index for:**
```
✅ WHERE A = ?
✅ WHERE A = ? AND B = ?
✅ WHERE A = ? AND B = ? AND C = ?
✅ WHERE A > ?  (range on first column)
✅ WHERE A = ? AND B > ?  (equality + range)
```

**Cannot use index efficiently for:**
```
❌ WHERE B = ?  (skipped first column)
❌ WHERE C = ?  (skipped first two columns)
❌ WHERE B = ? AND C = ?  (skipped first column)
❌ WHERE A > ? AND B = ?  (range then equality - inefficient)
```

**Why?**
- Index is sorted: A first, then B, then C
- To use B alone, would need to scan all A values
- Defeats purpose of index

### Composite Index Examples

**Example 1: Name and Age**
```sql
CREATE INDEX idx_name_age ON users(name, age);

-- ✅ Uses index
SELECT * FROM users WHERE name = 'alice';

-- ✅ Uses index
SELECT * FROM users WHERE name = 'alice' AND age = 25;

-- ❌ Doesn't use index efficiently
SELECT * FROM users WHERE age = 25;
```

**Example 2: Status and Created Date**
```sql
CREATE INDEX idx_status_created ON orders(status, created_at);

-- ✅ Uses index
SELECT * FROM orders WHERE status = 'pending';

-- ✅ Uses index
SELECT * FROM orders 
WHERE status = 'pending' 
ORDER BY created_at;

-- ❌ Doesn't use index efficiently
SELECT * FROM orders ORDER BY created_at;
```

### Choosing Composite Index Order

**Rule of Thumb:**
1. **Most selective first**: Column that filters most rows
2. **Equality before range**: Equality columns first
3. **Query patterns**: Order by how queries filter

**Example:**
```
Queries:
1. WHERE status = 'active' AND age > 25
2. WHERE status = 'active' AND city = 'NYC'

Index: (status, age, city)
- status: Most selective (filters most)
- age: Range query
- city: Equality query

Better: (status, city, age)
- status: Equality
- city: Equality (before range)
- age: Range
```

### Composite Index for Sorting

**Index: (status, created_at)**

**Query:**
```sql
SELECT * FROM orders 
WHERE status = 'pending' 
ORDER BY created_at;
```

**Why Fast:**
- Index already sorted by (status, created_at)
- For status='pending', created_at is already sorted
- No need to sort results!

**Without Index:**
```
1. Filter by status
2. Sort by created_at
3. Slow!
```

**With Index:**
```
1. Index scan (already sorted)
2. Fast!
```

---

## Index Selection and Query Optimization

### How Database Chooses Index

**Query Optimizer Process:**

**1. Parse Query:**
```sql
SELECT * FROM users WHERE email = 'alice@example.com';
```

**2. Identify Available Indexes:**
```
- Primary key index (id)
- Index on email
- Index on name
```

**3. Estimate Cost:**
```
Option 1: Full table scan
  Cost: 1,000,000 rows scanned

Option 2: Use email index
  Cost: ~20 index reads + 1 table read

Option 3: Use name index
  Cost: Not applicable (wrong column)
```

**4. Choose Best Plan:**
```
Choose: Email index (lowest cost)
```

### EXPLAIN Plan - Understanding Query Execution

**PostgreSQL Example:**
```sql
EXPLAIN SELECT * FROM users WHERE email = 'alice@example.com';
```

**Output:**
```
Index Scan using idx_email on users
  (cost=0.42..8.44 rows=1 width=64)
  Index Cond: (email = 'alice@example.com'::text)
```

**Reading the Plan:**
- **Index Scan**: Using index (good!)
- **cost=0.42..8.44**: Estimated cost
- **rows=1**: Expected 1 row
- **Index Cond**: Condition used for index

**Full Table Scan:**
```
Seq Scan on users
  (cost=0.00..18334.00 rows=1 width=64)
  Filter: (email = 'alice@example.com'::text)
```

**Reading:**
- **Seq Scan**: Sequential scan (bad!)
- **cost=0.00..18334.00**: High cost
- **Filter**: Filtering after scan (inefficient)

### Index Selectivity

**Selectivity**: How unique values are

**High Selectivity (Good for Index):**
```
Email: 1,000,000 unique values in 1,000,000 rows
Selectivity: 100%
→ Excellent for index
```

**Low Selectivity (Bad for Index):**
```
Status: 3 values (active, inactive, pending) in 1,000,000 rows
Selectivity: 0.0003%
→ Poor for index (unless partial index)
```

**Rule:**
- **High selectivity** → Good index candidate
- **Low selectivity** → May not help much

**Exception:**
- Low selectivity + frequent filter → Still useful
- Example: WHERE status='active' (filters 90% of rows)

### When Index is Used

**✅ Index Used:**
```sql
-- Equality
WHERE email = 'alice@example.com'

-- Range
WHERE age > 25
WHERE age BETWEEN 20 AND 30

-- Prefix (for strings)
WHERE name LIKE 'Alice%'

-- IN clause
WHERE id IN (1, 2, 3)

-- Composite (leftmost prefix)
WHERE name = 'alice' AND age = 25
```

**❌ Index NOT Used:**
```sql
-- Function on column
WHERE UPPER(email) = 'ALICE@EXAMPLE.COM'

-- Expression
WHERE age + 1 > 25

-- Suffix (for strings)
WHERE name LIKE '%Alice'

-- Negation (sometimes)
WHERE email != 'alice@example.com'
```

### Index-Only Scans

**When all data is in index:**
```sql
-- Covering index
CREATE INDEX idx_email_name ON users(email, name);

-- Query uses only indexed columns
SELECT email, name FROM users WHERE email = 'alice@example.com';
```

**Result:**
- **Index-only scan**: No table access needed
- **Very fast**: Only index reads

---

## When to Use Indexes - Best Practices

### When to Create Indexes

**1. Foreign Keys:**
```sql
CREATE INDEX idx_user_id ON orders(user_id);
```
- **Why**: JOINs are faster
- **Always**: Index foreign keys

**2. Frequently Filtered Columns:**
```sql
CREATE INDEX idx_status ON orders(status);
```
- **Why**: WHERE clauses are faster
- **When**: Column appears in WHERE often

**3. Frequently Sorted Columns:**
```sql
CREATE INDEX idx_created_at ON orders(created_at);
```
- **Why**: ORDER BY is faster
- **When**: Column appears in ORDER BY often

**4. Join Columns:**
```sql
CREATE INDEX idx_department_id ON employees(department_id);
```
- **Why**: JOINs are faster
- **When**: Column used in JOINs

**5. Unique Constraints:**
```sql
CREATE UNIQUE INDEX idx_email ON users(email);
```
- **Why**: Enforces uniqueness, fast lookups
- **When**: Need unique values

### When NOT to Create Indexes

**1. Small Tables:**
```
Table with 100 rows
→ Index overhead > benefit
→ Full scan is fast enough
```

**2. Write-Heavy Tables:**
```
Table with 1000 inserts/second
→ Each insert updates index
→ Slows down writes significantly
```

**3. Low Selectivity Columns:**
```
Column with 2 values in 1 million rows
→ Index doesn't help much
→ May even slow down (maintenance overhead)
```

**4. Rarely Queried Columns:**
```
Column queried once per month
→ Index maintenance cost > query benefit
```

**5. Frequently Updated Columns:**
```
Column updated in every transaction
→ Index must be updated every time
→ High maintenance cost
```

### Index Maintenance Cost

**Every INSERT:**
```
1. Insert into table
2. Insert into all indexes
3. Update index structure if needed
```

**Every UPDATE:**
```
1. Update table
2. If indexed column changed:
   a. Remove old entry from index
   b. Insert new entry into index
```

**Every DELETE:**
```
1. Delete from table
2. Delete from all indexes
```

**Cost:**
- **1 index**: ~2x write cost
- **5 indexes**: ~6x write cost
- **Trade-off**: More indexes = slower writes

### Index Strategy

**Read-Heavy Application:**
```
Many SELECT queries, few INSERT/UPDATE
→ Create many indexes
→ Optimize for reads
```

**Write-Heavy Application:**
```
Many INSERT/UPDATE, few SELECT
→ Create few indexes
→ Optimize for writes
```

**Balanced Application:**
```
Mix of reads and writes
→ Create indexes on frequently queried columns
→ Monitor performance
→ Adjust as needed
```

---

## Index Maintenance and Overhead

### Index Fragmentation

**Problem:**
- Inserts and deletes create gaps
- Index becomes fragmented
- Performance degrades

**Solution:**
```sql
-- Rebuild index (PostgreSQL)
REINDEX INDEX idx_email;

-- Rebuild index (MySQL)
ALTER TABLE users DROP INDEX idx_email;
CREATE INDEX idx_email ON users(email);
```

### Index Statistics

**Database maintains statistics:**
- Number of distinct values
- Data distribution
- Used by query optimizer

**Update Statistics:**
```sql
-- PostgreSQL
ANALYZE users;

-- MySQL
ANALYZE TABLE users;
```

**When:**
- After bulk inserts
- Periodically (automatic in most databases)

### Index Size

**Indexes take space:**
```
Table: 1 GB
Index: ~200 MB (20% of table size)
Multiple indexes: Can be larger than table!
```

**Monitor:**
```sql
-- PostgreSQL
SELECT 
    schemaname,
    tablename,
    indexname,
    pg_size_pretty(pg_relation_size(indexname::regclass)) AS size
FROM pg_indexes
WHERE tablename = 'users';
```

---

## Indexing Strategies for Different Data Types

### Integer Indexing

**Characteristics:**
- Fast comparison
- Small size (4-8 bytes)
- Excellent for indexing

**Example:**
```sql
CREATE INDEX idx_user_id ON orders(user_id);
```

### String Indexing

**Challenges:**
- Variable length
- Comparison is slower
- Larger size

**Strategies:**

**1. Full Index:**
```sql
CREATE INDEX idx_email ON users(email);
```
- Index entire string
- Good for exact matches

**2. Prefix Index:**
```sql
CREATE INDEX idx_name ON users(name(10));
```
- Index first 10 characters
- Smaller index
- Good for LIKE 'prefix%'

**3. Hash Index:**
```sql
CREATE INDEX idx_email_hash ON users(email) USING HASH;
```
- Hash string to fixed size
- Fast lookups
- No range queries

### Date/Time Indexing

**Characteristics:**
- Fixed size (8 bytes)
- Natural ordering
- Excellent for indexing

**Example:**
```sql
CREATE INDEX idx_created_at ON orders(created_at);
```

**Use Cases:**
- Range queries: WHERE created_at BETWEEN ...
- Sorting: ORDER BY created_at
- Time-series data

### Boolean Indexing

**Low Selectivity:**
```
Only 2 values: true/false
Selectivity: 50%
```

**When Useful:**
- Partial index
- Filters large portion of data

**Example:**
```sql
-- Partial index
CREATE INDEX idx_active ON users(email) WHERE active = true;
```

---

## Common Indexing Mistakes and How to Avoid Them

### Mistake 1: Too Many Indexes

**Problem:**
```
10 indexes on table
Every INSERT updates 10 indexes
Very slow writes
```

**Solution:**
- Only index what's needed
- Monitor query patterns
- Remove unused indexes

### Mistake 2: Wrong Column Order in Composite Index

**Problem:**
```sql
CREATE INDEX idx_age_name ON users(age, name);

-- Query
SELECT * FROM users WHERE name = 'alice';
-- Index not used (wrong order)
```

**Solution:**
```sql
CREATE INDEX idx_name_age ON users(name, age);
-- Now index can be used
```

### Mistake 3: Indexing Low-Selectivity Columns

**Problem:**
```sql
CREATE INDEX idx_gender ON users(gender);
-- Only 2 values, doesn't help much
```

**Solution:**
- Use partial index if filtering by gender
- Or skip index if not frequently queried

### Mistake 4: Not Indexing Foreign Keys

**Problem:**
```sql
-- No index on user_id
SELECT * FROM orders WHERE user_id = 123;
-- Full table scan
```

**Solution:**
```sql
CREATE INDEX idx_user_id ON orders(user_id);
-- Always index foreign keys
```

### Mistake 5: Functions on Indexed Columns

**Problem:**
```sql
CREATE INDEX idx_email ON users(email);

-- Query
SELECT * FROM users WHERE UPPER(email) = 'ALICE@EXAMPLE.COM';
-- Index not used (function on column)
```

**Solution:**
```sql
-- Option 1: Function-based index
CREATE INDEX idx_email_upper ON users(UPPER(email));

-- Option 2: Store uppercase in separate column
ALTER TABLE users ADD COLUMN email_upper VARCHAR(255);
CREATE INDEX idx_email_upper ON users(email_upper);
```

### Mistake 6: Ignoring Index Maintenance

**Problem:**
- Fragmented indexes
- Outdated statistics
- Poor query performance

**Solution:**
- Regular index maintenance
- Update statistics
- Monitor index usage

---

## Summary

Database indexing is crucial for performance. Understanding how indexes work, when to use them, and common pitfalls is essential for backend engineers.

**Key Takeaways:**
- Indexes dramatically speed up queries (O(log n) vs O(n))
- B+ Tree is the standard data structure
- Composite indexes: Order matters (leftmost prefix rule)
- Indexes speed up reads but slow down writes
- Choose indexes based on query patterns
- Monitor and maintain indexes
- Avoid common mistakes (too many, wrong order, etc.)

**Next Steps:**
- Analyze your queries (EXPLAIN plans)
- Identify missing indexes
- Remove unused indexes
- Monitor index performance
- Understand your database's indexing features

