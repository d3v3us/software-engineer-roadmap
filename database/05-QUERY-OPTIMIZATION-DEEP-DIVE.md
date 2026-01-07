# Database Query Optimization Deep Dive - Complete Understanding

## Table of Contents
1. [What is Query Optimization?](#what-is-query-optimization)
2. [Query Execution Process](#query-execution-process)
3. [Query Planner and Optimizer](#query-planner-and-optimizer)
4. [Understanding EXPLAIN Plans](#understanding-explain-plans)
5. [Common Query Patterns and Optimizations](#common-query-patterns-and-optimizations)
6. [Join Optimization](#join-optimization)
7. [Aggregation Optimization](#aggregation-optimization)
8. [Subquery Optimization](#subquery-optimization)
9. [Query Rewriting Techniques](#query-rewriting-techniques)
10. [Monitoring and Profiling Queries](#monitoring-and-profiling-queries)

---

## What is Query Optimization?

### The Problem

**Slow Queries:**
```
Query takes 10 seconds
Users wait
System slow
Poor experience
```

### The Goal

**Query Optimization:**
```
Make queries faster
Reduce execution time
Improve performance
Better user experience
```

### Optimization Levels

**1. Query-Level:**
```
Optimize individual queries
Rewrite queries
Add indexes
```

**2. Schema-Level:**
```
Design schema efficiently
Normalize/denormalize
Partition tables
```

**3. System-Level:**
```
Database configuration
Memory settings
Connection pooling
```

---

## Query Execution Process

### Steps

**1. Parse:**
```
Parse SQL syntax
Check syntax errors
Build parse tree
```

**2. Validate:**
```
Check table/column existence
Check permissions
Validate data types
```

**3. Optimize:**
```
Generate execution plans
Estimate costs
Choose best plan
```

**4. Execute:**
```
Run chosen plan
Return results
```

### Visual Flow

```
SQL Query
  ↓
Parser → Parse Tree
  ↓
Validator → Validated Tree
  ↓
Optimizer → Execution Plan
  ↓
Executor → Results
```

---

## Query Planner and Optimizer

### What Does Optimizer Do?

**Optimizer:**
```
1. Analyzes query
2. Generates multiple execution plans
3. Estimates cost of each plan
4. Chooses cheapest plan
```

### Cost Estimation

**Factors:**
```
- Disk I/O: Reading from disk
- CPU: Processing data
- Memory: Buffer usage
- Network: For distributed queries
```

**Example:**
```
Plan A: Full table scan
  Cost: 1000 (read all rows)

Plan B: Index scan
  Cost: 10 (read index + few rows)

Optimizer chooses: Plan B (cheaper)
```

### Optimization Techniques

**1. Predicate Pushdown:**
```
Apply WHERE filters early
Reduce data processed
```

**2. Projection Pushdown:**
```
Select only needed columns
Reduce data transferred
```

**3. Join Reordering:**
```
Join smaller tables first
Reduce intermediate results
```

**4. Index Selection:**
```
Choose best index
Consider selectivity
```

---

## Understanding EXPLAIN Plans

### What is EXPLAIN?

**EXPLAIN**: Shows how database will execute query.

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

### Reading EXPLAIN Output

**Key Information:**

**1. Operation Type:**
```
Index Scan: Using index (good!)
Seq Scan: Full table scan (bad!)
```

**2. Cost:**
```
cost=0.42..8.44
  Start cost: 0.42
  Total cost: 8.44
Lower is better
```

**3. Rows:**
```
rows=1
Expected rows returned
```

**4. Width:**
```
width=64
Average row size in bytes
```

### Good vs Bad Plans

**Good Plan (Index Scan):**
```
Index Scan using idx_email
  Cost: 0.42..8.44
  Fast: Uses index
```

**Bad Plan (Full Table Scan):**
```
Seq Scan on users
  Cost: 0.00..18334.00
  Slow: Scans all rows
```

---

## Common Query Patterns and Optimizations

### Pattern 1: WHERE Clause Optimization

**Problem:**
```sql
SELECT * FROM users WHERE UPPER(email) = 'ALICE@EXAMPLE.COM';
```
- **Function on column**: Can't use index
- **Full table scan**: Slow

**Solution:**
```sql
-- Option 1: Store uppercase
ALTER TABLE users ADD COLUMN email_upper VARCHAR(255);
CREATE INDEX idx_email_upper ON users(email_upper);
SELECT * FROM users WHERE email_upper = 'ALICE@EXAMPLE.COM';

-- Option 2: Function-based index
CREATE INDEX idx_email_upper ON users(UPPER(email));
SELECT * FROM users WHERE UPPER(email) = 'ALICE@EXAMPLE.COM';
```

### Pattern 2: LIKE Queries

**Problem:**
```sql
SELECT * FROM users WHERE name LIKE '%alice%';
```
- **Suffix wildcard**: Can't use index efficiently
- **Full table scan**: Slow

**Solution:**
```sql
-- Prefix only (can use index)
SELECT * FROM users WHERE name LIKE 'alice%';

-- Full-text search
CREATE INDEX idx_name_fts ON users USING gin(to_tsvector('english', name));
SELECT * FROM users WHERE to_tsvector('english', name) @@ to_tsquery('alice');
```

### Pattern 3: ORDER BY with LIMIT

**Problem:**
```sql
SELECT * FROM users ORDER BY name LIMIT 10 OFFSET 1000000;
```
- **Must sort all**: Before skipping
- **Very slow**: For large offsets

**Solution:**
```sql
-- Cursor-based pagination
SELECT * FROM users WHERE name > 'last_seen_name' ORDER BY name LIMIT 10;

-- Index on name helps
CREATE INDEX idx_name ON users(name);
```

### Pattern 4: COUNT(*)

**Problem:**
```sql
SELECT COUNT(*) FROM users;
```
- **Full table scan**: Must count all rows
- **Slow**: For large tables

**Solution:**
```sql
-- Approximate count (if acceptable)
SELECT reltuples FROM pg_class WHERE relname = 'users';

-- Maintain counter table
CREATE TABLE user_count (count INT);
UPDATE user_count SET count = count + 1; -- On insert
```

---

## Join Optimization

### Join Types

**1. Nested Loop Join:**
```
For each row in outer table:
  For each row in inner table:
    If match → Output
```
- **Cost**: O(n * m)
- **Use**: Small tables

**2. Hash Join:**
```
1. Build hash table from smaller table
2. Probe hash table with larger table
3. Output matches
```
- **Cost**: O(n + m)
- **Use**: Large tables, equality joins

**3. Merge Join:**
```
1. Sort both tables
2. Merge sorted lists
3. Output matches
```
- **Cost**: O(n log n + m log m)
- **Use**: Sorted data, range joins

### Join Optimization

**1. Join Order:**
```
Join smaller tables first
Reduce intermediate results
```

**2. Indexes:**
```
Index join columns
Fast lookups
```

**3. Filter Early:**
```
Apply WHERE before JOIN
Reduce data joined
```

---

## Aggregation Optimization

### GROUP BY Optimization

**Problem:**
```sql
SELECT department, COUNT(*) 
FROM employees 
GROUP BY department;
```
- **Must scan all rows**: To group
- **Sorting**: May need to sort

**Optimization:**
```
1. Use index on GROUP BY column
2. Hash aggregation (if supported)
3. Filter before grouping
```

### HAVING vs WHERE

**WHERE:**
```
Filters before grouping
More efficient
```

**HAVING:**
```
Filters after grouping
Less efficient
```

**Example:**
```sql
-- Better: Filter before grouping
SELECT department, COUNT(*) 
FROM employees 
WHERE salary > 50000
GROUP BY department;

-- Worse: Filter after grouping
SELECT department, COUNT(*) 
FROM employees 
GROUP BY department
HAVING AVG(salary) > 50000;
```

---

## Subquery Optimization

### Correlated vs Non-Correlated

**Non-Correlated:**
```sql
SELECT * FROM users 
WHERE id IN (SELECT user_id FROM orders);
```
- **Executed once**: Subquery independent
- **Can be optimized**: Convert to JOIN

**Correlated:**
```sql
SELECT * FROM users u
WHERE EXISTS (
  SELECT 1 FROM orders o 
  WHERE o.user_id = u.id
);
```
- **Executed per row**: Depends on outer query
- **Harder to optimize**: May be slow

### Subquery to JOIN

**Convert when possible:**
```sql
-- Subquery
SELECT * FROM users 
WHERE id IN (SELECT user_id FROM orders);

-- JOIN (often faster)
SELECT DISTINCT u.* 
FROM users u
JOIN orders o ON u.id = o.user_id;
```

---

## Query Rewriting Techniques

### 1. Avoid SELECT *

**Problem:**
```sql
SELECT * FROM users;
```
- **Fetches all columns**: Even if not needed
- **Wasteful**: More data transferred

**Solution:**
```sql
SELECT id, name, email FROM users;
-- Only fetch what you need
```

### 2. Use EXISTS Instead of COUNT

**Problem:**
```sql
SELECT * FROM users 
WHERE (SELECT COUNT(*) FROM orders WHERE user_id = users.id) > 0;
```
- **Counts all**: Even if just need existence
- **Slow**: Unnecessary work

**Solution:**
```sql
SELECT * FROM users 
WHERE EXISTS (SELECT 1 FROM orders WHERE user_id = users.id);
-- Stops at first match
```

### 3. Avoid Functions in WHERE

**Problem:**
```sql
SELECT * FROM users WHERE YEAR(created_at) = 2024;
```
- **Function on column**: Can't use index
- **Full table scan**: Slow

**Solution:**
```sql
SELECT * FROM users 
WHERE created_at >= '2024-01-01' AND created_at < '2025-01-01';
-- Can use index on created_at
```

---

## Monitoring and Profiling Queries

### Slow Query Log

**Enable:**
```
log_slow_queries = ON
long_query_time = 2  # Log queries > 2 seconds
```

**Analyze:**
```
1. Identify slow queries
2. Understand why slow
3. Optimize
4. Monitor improvement
```

### Query Profiling

**Tools:**
```
- EXPLAIN ANALYZE: Actual execution time
- Query profilers: Detailed analysis
- Database monitoring: Track performance
```

### Metrics to Monitor

**1. Execution Time:**
```
How long query takes
Target: < 100ms for most queries
```

**2. Rows Examined:**
```
How many rows processed
Should match rows returned (if using index)
```

**3. Index Usage:**
```
Are indexes being used?
Missing indexes?
```

---

## Summary

Query optimization is crucial for database performance. Understanding execution plans, optimization techniques, and monitoring is essential for backend engineers.

**Key Takeaways:**
- Optimizer chooses execution plan
- EXPLAIN shows how query executes
- Indexes dramatically improve performance
- Avoid functions in WHERE clauses
- Filter early, select only needed columns
- Convert subqueries to JOINs when possible
- Monitor slow queries
- Profile and measure improvements

**Next Steps:**
- Analyze your slow queries
- Use EXPLAIN to understand execution
- Add missing indexes
- Rewrite inefficient queries
- Monitor query performance
- Continuously optimize

