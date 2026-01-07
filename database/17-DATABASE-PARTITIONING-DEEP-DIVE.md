# Database Partitioning Deep Dive - Complete Understanding

## Table of Contents
1. [What is Database Partitioning?](#what-is-database-partitioning)
2. [Why Do We Need Partitioning?](#why-do-we-need-partitioning)
3. [Partitioning vs Sharding](#partitioning-vs-sharding)
4. [Types of Partitioning](#types-of-partitioning)
5. [Horizontal Partitioning](#horizontal-partitioning)
6. [Vertical Partitioning](#vertical-partitioning)
7. [Partitioning Strategies](#partitioning-strategies)
8. [Partition Key Selection](#partition-key-selection)
9. [Partition Pruning](#partition-pruning)
10. [Partition Maintenance](#partition-maintenance)
11. [Best Practices](#best-practices)
12. [Common Challenges](#common-challenges)

---

## What is Database Partitioning?

### Definition

**Database Partitioning**: Technique of dividing a large table into smaller, more manageable pieces called partitions.

**Key Concept:**
- **Divide table**: Divide large table
- **Smaller pieces**: Smaller, manageable pieces
- **Same schema**: Same schema across partitions
- **Performance**: Better performance

### Real-World Analogy

**Partitioning = Filing Cabinet:**
- **Large cabinet**: Large table
- **Drawers**: Partitions
- **Organized**: Organized by criteria (date, category)
- **Faster access**: Faster to find files

**Database:**
- **Large table**: Large table (millions of rows)
- **Partitions**: Smaller partitions
- **Organized**: Organized by partition key
- **Faster queries**: Faster queries

---

## Why Do We Need Partitioning?

### Problems with Large Tables

**1. Slow Queries:**
```
Large table (100 million rows)
  ↓
Full table scan
  ↓
Slow queries (minutes)
```

**2. Maintenance Issues:**
```
Large table
  ↓
Slow backups
  ↓
Slow index rebuilds
  ↓
Slow maintenance
```

**3. Storage Issues:**
```
Large table
  ↓
All data in one place
  ↓
Storage management
```

### Benefits of Partitioning

**1. Performance:**
- **Faster queries**: Faster queries (partition pruning)
- **Parallel processing**: Parallel processing
- **Better indexes**: Better index performance

**2. Maintenance:**
- **Faster maintenance**: Faster maintenance operations
- **Partition-level**: Partition-level operations
- **Easier management**: Easier management

**3. Storage:**
- **Archive old**: Archive old partitions
- **Storage optimization**: Storage optimization
- **Data lifecycle**: Data lifecycle management

---

## Partitioning vs Sharding

### Partitioning

**Partitioning:**
```
Single database
  ↓
Table divided into partitions
  ↓
Same database instance
  ↓
Logical division
```

**Characteristics:**
- **Single database**: Single database
- **Logical**: Logical division
- **Transparent**: Transparent to application
- **Same instance**: Same database instance

### Sharding

**Sharding:**
```
Multiple databases
  ↓
Data distributed across databases
  ↓
Different database instances
  ↓
Physical division
```

**Characteristics:**
- **Multiple databases**: Multiple databases
- **Physical**: Physical division
- **Application-aware**: Application must be aware
- **Different instances**: Different database instances

### Comparison

| Aspect | Partitioning | Sharding |
|--------|--------------|----------|
| **Location** | Same database | Different databases |
| **Division** | Logical | Physical |
| **Transparency** | Transparent | Application-aware |
| **Use Case** | Large table | Very large dataset |

---

## Types of Partitioning

### Type 1: Range Partitioning

**How It Works:**
```
Partition by range of values
  ↓
Partition 1: 0-1000
Partition 2: 1001-2000
Partition 3: 2001-3000
```

**Use Case:**
- **Date ranges**: Date-based partitioning
- **Sequential data**: Sequential data
- **Time-series**: Time-series data

**Example:**
```sql
CREATE TABLE orders (
    id INT,
    order_date DATE,
    amount DECIMAL
) PARTITION BY RANGE (YEAR(order_date)) (
    PARTITION p2020 VALUES LESS THAN (2021),
    PARTITION p2021 VALUES LESS THAN (2022),
    PARTITION p2022 VALUES LESS THAN (2023)
);
```

### Type 2: Hash Partitioning

**How It Works:**
```
Partition by hash of key
  ↓
hash(key) % num_partitions → partition
  ↓
Even distribution
```

**Use Case:**
- **Even distribution**: Even distribution needed
- **No natural range**: No natural range
- **Load balancing**: Load balancing

**Example:**
```sql
CREATE TABLE users (
    id INT,
    name VARCHAR(100)
) PARTITION BY HASH(id) PARTITIONS 4;
```

### Type 3: List Partitioning

**How It Works:**
```
Partition by list of values
  ↓
Partition 1: ['US', 'CA']
Partition 2: ['UK', 'FR']
Partition 3: ['JP', 'CN']
```

**Use Case:**
- **Discrete values**: Discrete values
- **Geographic**: Geographic partitioning
- **Categories**: Categories

**Example:**
```sql
CREATE TABLE sales (
    id INT,
    region VARCHAR(50),
    amount DECIMAL
) PARTITION BY LIST (region) (
    PARTITION p_americas VALUES IN ('US', 'CA', 'MX'),
    PARTITION p_europe VALUES IN ('UK', 'FR', 'DE'),
    PARTITION p_asia VALUES IN ('JP', 'CN', 'IN')
);
```

### Type 4: Composite Partitioning

**How It Works:**
```
Combine partitioning methods
  ↓
Range + Hash
Range + List
  ↓
Two-level partitioning
```

**Use Case:**
- **Complex requirements**: Complex requirements
- **Multiple dimensions**: Multiple dimensions
- **Fine-grained**: Fine-grained control

**Example:**
```sql
CREATE TABLE orders (
    id INT,
    order_date DATE,
    customer_id INT,
    amount DECIMAL
) PARTITION BY RANGE (YEAR(order_date))
SUBPARTITION BY HASH(customer_id) SUBPARTITIONS 4 (
    PARTITION p2020 VALUES LESS THAN (2021),
    PARTITION p2021 VALUES LESS THAN (2022)
);
```

---

## Horizontal Partitioning

### What is Horizontal Partitioning?

**Horizontal Partitioning**: Divide table by rows (same columns, different rows).

**Example:**
```
Original Table:
users: [row1, row2, row3, row4, row5, row6, row7, row8, row9, row10]

Partition 1: [row1, row2, row3, row4, row5]
Partition 2: [row6, row7, row8, row9, row10]
```

**Schema:**
```
All partitions have same schema
  ↓
Same columns
  ↓
Different rows
```

### Horizontal Partitioning Benefits

**1. Query Performance:**
- **Partition pruning**: Query only relevant partitions
- **Faster scans**: Faster table scans
- **Parallel processing**: Parallel processing

**2. Maintenance:**
- **Partition-level**: Partition-level operations
- **Faster backups**: Faster backups
- **Easier management**: Easier management

---

## Vertical Partitioning

### What is Vertical Partitioning?

**Vertical Partitioning**: Divide table by columns (different columns, same rows).

**Example:**
```
Original Table:
users: [id, name, email, profile_data, settings, metadata]

Partition 1 (frequent): [id, name, email]
Partition 2 (infrequent): [id, profile_data, settings, metadata]
```

**Schema:**
```
Different schemas per partition
  ↓
Split by access pattern
  ↓
Frequent vs infrequent
```

### Vertical Partitioning Benefits

**1. Access Patterns:**
- **Frequent data**: Fast access to frequent data
- **Infrequent data**: Separate infrequent data
- **Optimization**: Optimize per partition

**2. Storage:**
- **Different storage**: Different storage per partition
- **Cost optimization**: Cost optimization
- **Efficiency**: More efficient

---

## Partitioning Strategies

### Strategy 1: Date-Based Partitioning

**How It Works:**
```
Partition by date
  ↓
Partition 1: 2020
Partition 2: 2021
Partition 3: 2022
  ↓
Archive old partitions
```

**Use Case:**
- **Time-series data**: Time-series data
- **Logs**: Log tables
- **Historical data**: Historical data

**Example:**
```sql
CREATE TABLE events (
    id INT,
    event_date DATE,
    data TEXT
) PARTITION BY RANGE (event_date) (
    PARTITION p2020 VALUES LESS THAN ('2021-01-01'),
    PARTITION p2021 VALUES LESS THAN ('2022-01-01'),
    PARTITION p2022 VALUES LESS THAN ('2023-01-01')
);
```

### Strategy 2: Geographic Partitioning

**How It Works:**
```
Partition by region
  ↓
Partition 1: Americas
Partition 2: Europe
Partition 3: Asia
```

**Use Case:**
- **Geographic data**: Geographic data
- **Compliance**: Data locality requirements
- **Performance**: Performance by region

### Strategy 3: Hash-Based Partitioning

**How It Works:**
```
Partition by hash
  ↓
Even distribution
  ↓
No hot spots
```

**Use Case:**
- **Even distribution**: Even distribution needed
- **No natural key**: No natural partitioning key
- **Load balancing**: Load balancing

---

## Partition Key Selection

### What is Partition Key?

**Partition Key**: Column(s) used to determine which partition stores a row.

**Example:**
```
Partition key: order_date
  ↓
hash(order_date) → partition
  ↓
Row stored in determined partition
```

### Partition Key Requirements

**1. High Cardinality:**
```
Many distinct values
  ↓
Even distribution
  ↓
No hot spots
```

**2. Query Patterns:**
```
Match query patterns
  ↓
Most queries use partition key
  ↓
Partition pruning
```

**3. Even Distribution:**
```
Values distributed evenly
  ↓
No hot partitions
  ↓
Balanced load
```

### Good Partition Keys

**Examples:**
- **Date**: Date columns (for time-series)
- **User ID**: User ID (for user data)
- **Region**: Region (for geographic data)

### Bad Partition Keys

**Examples:**
- **Status**: Status (low cardinality)
- **Boolean**: Boolean fields
- **Sequential IDs**: Sequential IDs (uneven)

---

## Partition Pruning

### What is Partition Pruning?

**Partition Pruning**: Process of eliminating partitions that don't need to be scanned.

**How It Works:**
```
Query: SELECT * FROM orders WHERE order_date = '2022-01-15'
  ↓
Partition key: order_date
  ↓
Only scan partition p2022
  ↓
Skip other partitions
```

### Benefits

**1. Performance:**
- **Faster queries**: Much faster queries
- **Less I/O**: Less I/O operations
- **Better performance**: Better performance

**2. Efficiency:**
- **Resource usage**: Lower resource usage
- **Cost**: Lower cost
- **Scalability**: Better scalability

### Pruning Conditions

**Partition Pruning Works When:**
- **Partition key in WHERE**: Partition key in WHERE clause
- **Range queries**: Range queries on partition key
- **Equality**: Equality on partition key

**Partition Pruning Doesn't Work When:**
- **No partition key**: Partition key not in query
- **Functions**: Functions on partition key
- **Complex conditions**: Complex conditions

---

## Partition Maintenance

### Adding Partitions

**Process:**
```
1. Create new partition
2. Define partition bounds
3. Verify
```

**Example:**
```sql
ALTER TABLE orders
ADD PARTITION p2023 VALUES LESS THAN ('2024-01-01');
```

### Dropping Partitions

**Process:**
```
1. Drop partition
2. Data removed
3. Verify
```

**Example:**
```sql
ALTER TABLE orders
DROP PARTITION p2020;
```

### Merging Partitions

**Process:**
```
1. Merge partitions
2. Combine data
3. Verify
```

**Example:**
```sql
ALTER TABLE orders
MERGE PARTITIONS p2020, p2021 INTO p2020_2021;
```

### Splitting Partitions

**Process:**
```
1. Split partition
2. Redistribute data
3. Verify
```

**Example:**
```sql
ALTER TABLE orders
SPLIT PARTITION p2022 INTO (
    PARTITION p2022_q1 VALUES LESS THAN ('2022-04-01'),
    PARTITION p2022_q2 VALUES LESS THAN ('2022-07-01')
);
```

---

## Best Practices

### 1. Choose Right Partitioning Type

**Why:**
- **Requirements**: Based on requirements
- **Query patterns**: Query patterns
- **Data characteristics**: Data characteristics

**Guidelines:**
- **Time-series**: Range partitioning
- **Even distribution**: Hash partitioning
- **Discrete values**: List partitioning

### 2. Select Good Partition Key

**Why:**
- **Partition pruning**: Enable partition pruning
- **Even distribution**: Even distribution
- **Performance**: Better performance

**Guidelines:**
- **High cardinality**: High cardinality
- **Query patterns**: Match query patterns
- **Even distribution**: Even distribution

### 3. Plan Partition Size

**Why:**
- **Performance**: Optimal performance
- **Maintenance**: Easier maintenance
- **Balance**: Balance size and number

**Guidelines:**
- **Not too small**: Not too small (overhead)
- **Not too large**: Not too large (slow)
- **Optimal size**: Optimal size (millions of rows)

### 4. Monitor Partition Usage

**Why:**
- **Performance**: Monitor performance
- **Imbalance**: Detect imbalance
- **Optimization**: Optimize

**Metrics:**
- **Partition size**: Partition sizes
- **Query distribution**: Query distribution
- **Performance**: Query performance

### 5. Archive Old Partitions

**Why:**
- **Storage**: Reduce storage
- **Performance**: Better performance
- **Cost**: Lower cost

**Process:**
- **Archive**: Move to archive
- **Compress**: Compress old partitions
- **Remove**: Remove very old partitions

---

## Common Challenges

### Challenge 1: Partition Imbalance

**Problem:**
```
Uneven partition sizes
  ↓
Some partitions large
  ↓
Poor performance
```

**Solution:**
```
Choose better partition key
  ↓
Or use hash partitioning
  ↓
Rebalance partitions
```

### Challenge 2: Cross-Partition Queries

**Problem:**
```
Queries span partitions
  ↓
Scan multiple partitions
  ↓
Slower
```

**Solution:**
```
Design queries to use partition key
  ↓
Partition pruning
  ↓
Single partition queries
```

### Challenge 3: Partition Maintenance

**Problem:**
```
Partition maintenance complex
  ↓
Adding/dropping partitions
  ↓
Data migration
```

**Solution:**
```
Plan partition strategy
  ↓
Automate maintenance
  ↓
Monitor and adjust
```

### Challenge 4: Partition Key Changes

**Problem:**
```
Need to change partition key
  ↓
Requires table rebuild
  ↓
Downtime
```

**Solution:**
```
Choose partition key carefully
  ↓
Plan for future
  ↓
Minimize changes
```

---

## Summary

Database partitioning improves performance and manageability of large tables. Understanding partitioning types, strategies, and best practices is essential for scaling databases.

**Key Takeaways:**
- **Partitioning**: Divide large table into smaller partitions
- **Types**: Range, hash, list, composite
- **Horizontal**: Divide by rows
- **Vertical**: Divide by columns
- **Partition key**: Critical for performance
- **Partition pruning**: Eliminate unnecessary partitions
- **Maintenance**: Adding, dropping, merging, splitting
- **Best practices**: Right type, good key, plan size, monitor, archive

**Partitioning Types:**
- **Range**: By range of values
- **Hash**: By hash of key
- **List**: By list of values
- **Composite**: Combine methods

**Best Practices:**
- Choose right partitioning type
- Select good partition key
- Plan partition size
- Monitor partition usage
- Archive old partitions

**Common Challenges:**
- Partition imbalance
- Cross-partition queries
- Partition maintenance
- Partition key changes

**Next Steps:**
- Analyze table size and queries
- Choose partitioning strategy
- Implement partitioning
- Monitor performance
- Optimize and maintain

