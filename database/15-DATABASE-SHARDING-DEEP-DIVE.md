# Database Sharding Deep Dive - Complete Understanding

## Table of Contents
1. [What is Database Sharding?](#what-is-database-sharding)
2. [Why Do We Need Sharding?](#why-do-we-need-sharding)
3. [Sharding vs Replication](#sharding-vs-replication)
4. [Sharding Strategies](#sharding-strategies)
5. [Horizontal Sharding](#horizontal-sharding)
6. [Vertical Sharding](#vertical-sharding)
7. [Shard Key Selection](#shard-key-selection)
8. [Sharding Architectures](#sharding-architectures)
9. [Shard Management](#shard-management)
10. [Cross-Shard Queries](#cross-shard-queries)
11. [Shard Rebalancing](#shard-rebalancing)
12. [Best Practices](#best-practices)
13. [Common Challenges](#common-challenges)

---

## What is Database Sharding?

### Definition

**Database Sharding**: Technique of splitting a large database into smaller, more manageable pieces called shards.

**Key Concept:**
- **Split database**: Split into multiple shards
- **Distribute data**: Distribute data across shards
- **Independent**: Each shard independent
- **Scalability**: Horizontal scalability

### Real-World Analogy

**Sharding = Library Branches:**
- **Central library**: Single large database
- **Branches**: Shards (smaller databases)
- **Books distributed**: Books distributed by location
- **Users go to branch**: Users query appropriate shard

**Database:**
- **Single database**: Single large database
- **Shards**: Multiple smaller databases
- **Data distributed**: Data distributed by shard key
- **Queries routed**: Queries routed to appropriate shard

---

## Why Do We Need Sharding?

### Problems with Single Database

**1. Size Limits:**
```
Single database
  ↓
Grows too large
  ↓
Performance degrades
  ↓
Hard to manage
```

**2. Performance:**
```
All queries → Single database
  ↓
High load
  ↓
Slow queries
  ↓
Bottleneck
```

**3. Scalability:**
```
Vertical scaling (bigger server)
  ↓
Expensive
  ↓
Limited
  ↓
Not sustainable
```

### Benefits of Sharding

**1. Scalability:**
- **Horizontal scaling**: Scale horizontally
- **Add shards**: Add more shards
- **Unlimited**: Unlimited scaling

**2. Performance:**
- **Distributed load**: Distribute load
- **Smaller databases**: Smaller, faster databases
- **Parallel queries**: Parallel queries

**3. Availability:**
- **Isolation**: Failure isolation
- **Partial failure**: One shard fails, others work
- **Better availability**: Better availability

---

## Sharding vs Replication

### Replication

**Replication:**
```
Master → Replicas (same data)
  ↓
Read scaling
  ↓
High availability
  ↓
Same data everywhere
```

**Use Case:**
- **Read scaling**: Scale reads
- **High availability**: High availability
- **Backup**: Backup

### Sharding

**Sharding:**
```
Database → Shards (different data)
  ↓
Write scaling
  ↓
Distributed data
  ↓
Different data in each shard
```

**Use Case:**
- **Write scaling**: Scale writes
- **Large datasets**: Very large datasets
- **Performance**: Better performance

### Combined

**Sharding + Replication:**
```
Shard 1 → Replicas
Shard 2 → Replicas
Shard 3 → Replicas
  ↓
Both read and write scaling
  ↓
High availability
```

---

## Sharding Strategies

### Strategy 1: Range-Based Sharding

**How It Works:**
```
Shard 1: user_id 1-1000000
Shard 2: user_id 1000001-2000000
Shard 3: user_id 2000001-3000000
```

**Pros:**
- **Simple**: Simple to implement
- **Easy queries**: Easy range queries
- **Sequential**: Sequential data

**Cons:**
- **Hot spots**: Hot spots (uneven distribution)
- **Rebalancing**: Hard to rebalance

### Strategy 2: Hash-Based Sharding

**How It Works:**
```
hash(user_id) % num_shards → shard_number
  ↓
Even distribution
  ↓
No hot spots
```

**Example:**
```
user_id = 123
hash(123) = 456789
456789 % 3 = 0
  ↓
Shard 0
```

**Pros:**
- **Even distribution**: Even distribution
- **No hot spots**: No hot spots
- **Simple**: Simple routing

**Cons:**
- **No range queries**: Hard range queries
- **Rebalancing**: Hard to rebalance

### Strategy 3: Directory-Based Sharding

**How It Works:**
```
Lookup table: user_id → shard
  ↓
Query lookup table
  ↓
Route to shard
```

**Pros:**
- **Flexible**: Flexible
- **Easy rebalancing**: Easy rebalancing
- **Custom routing**: Custom routing

**Cons:**
- **Lookup overhead**: Lookup overhead
- **Single point**: Lookup table single point
- **Complexity**: More complexity

### Strategy 4: Geographic Sharding

**How It Works:**
```
Shard by location
  ↓
US users → US shard
EU users → EU shard
Asia users → Asia shard
```

**Pros:**
- **Latency**: Low latency
- **Compliance**: Data locality (compliance)
- **Natural**: Natural distribution

**Cons:**
- **Uneven**: May be uneven
- **Migration**: User migration issues

---

## Horizontal Sharding

### What is Horizontal Sharding?

**Horizontal Sharding**: Split by rows (same schema, different data).

**Example:**
```
Original Table:
users: [1, 2, 3, 4, 5, 6, 7, 8, 9, 10]

Shard 1: [1, 2, 3, 4, 5]
Shard 2: [6, 7, 8, 9, 10]
```

**Schema:**
```
All shards have same schema
  ↓
users table in each shard
  ↓
Different rows in each
```

### Horizontal Sharding Benefits

**1. Scalability:**
- **Add shards**: Add more shards
- **Unlimited**: Unlimited scaling
- **Distribute load**: Distribute load

**2. Performance:**
- **Smaller tables**: Smaller tables per shard
- **Faster queries**: Faster queries
- **Parallel**: Parallel processing

---

## Vertical Sharding

### What is Vertical Sharding?

**Vertical Sharding**: Split by columns (different tables/schemas).

**Example:**
```
Original Table:
users: [id, name, email, profile_data, settings]

Shard 1 (frequent): [id, name, email]
Shard 2 (infrequent): [id, profile_data, settings]
```

**Schema:**
```
Different schemas per shard
  ↓
Split by access pattern
  ↓
Frequent vs infrequent
```

### Vertical Sharding Benefits

**1. Access Patterns:**
- **Frequent data**: Fast access to frequent data
- **Infrequent data**: Separate infrequent data
- **Optimization**: Optimize per shard

**2. Storage:**
- **Different storage**: Different storage per shard
- **Cost optimization**: Cost optimization
- **Efficiency**: More efficient

---

## Shard Key Selection

### What is Shard Key?

**Shard Key**: Field used to determine which shard stores data.

**Example:**
```
Shard key: user_id
  ↓
hash(user_id) → shard
  ↓
All user data in same shard
```

### Shard Key Requirements

**1. High Cardinality:**
```
Many distinct values
  ↓
Even distribution
  ↓
No hot spots
```

**2. Even Distribution:**
```
Values distributed evenly
  ↓
No hot spots
  ↓
Balanced load
```

**3. Query Patterns:**
```
Match query patterns
  ↓
Most queries use shard key
  ↓
Avoid cross-shard queries
```

### Good Shard Keys

**Examples:**
- **user_id**: User ID (high cardinality, even)
- **tenant_id**: Tenant ID (multi-tenant)
- **order_id**: Order ID (distributed)

### Bad Shard Keys

**Examples:**
- **status**: Status (low cardinality, uneven)
- **created_date**: Date (time-based, hot spots)
- **country**: Country (uneven distribution)

---

## Sharding Architectures

### Architecture 1: Application-Level Sharding

**How It Works:**
```
Application
  ↓
Shard router
  ↓
Routes to shards
```

**Pros:**
- **Control**: Full control
- **Flexible**: Flexible
- **Custom**: Custom logic

**Cons:**
- **Complexity**: Application complexity
- **Maintenance**: Maintenance burden

### Architecture 2: Database Proxy

**How It Works:**
```
Application → Proxy → Shards
  ↓
Proxy handles routing
  ↓
Transparent to application
```

**Pros:**
- **Transparent**: Transparent to app
- **Centralized**: Centralized logic
- **Easier**: Easier for application

**Cons:**
- **Proxy overhead**: Proxy overhead
- **Single point**: Proxy single point

### Architecture 3: Database Native

**How It Works:**
```
Database handles sharding
  ↓
Built-in sharding
  ↓
Automatic routing
```

**Examples:**
- **MongoDB**: MongoDB sharding
- **Citus**: PostgreSQL extension
- **Vitess**: MySQL sharding

**Pros:**
- **Automatic**: Automatic
- **Optimized**: Optimized
- **Less code**: Less application code

**Cons:**
- **Vendor lock-in**: Vendor specific
- **Less control**: Less control

---

## Shard Management

### Adding Shards

**Process:**
```
1. Create new shard
2. Rebalance data
3. Update routing
4. Verify
```

**Challenges:**
- **Data migration**: Data migration
- **Downtime**: Potential downtime
- **Complexity**: Complexity

### Removing Shards

**Process:**
```
1. Migrate data out
2. Update routing
3. Remove shard
4. Verify
```

**Challenges:**
- **Data migration**: Data migration
- **Routing updates**: Routing updates
- **Verification**: Verification

### Monitoring Shards

**Metrics:**
- **Size**: Shard size
- **Load**: Query load
- **Performance**: Query performance
- **Health**: Shard health

---

## Cross-Shard Queries

### Problem

**Cross-Shard Query:**
```
Query spans multiple shards
  ↓
Query each shard
  ↓
Combine results
  ↓
Slow and complex
```

**Example:**
```
SELECT * FROM orders WHERE total > 1000
  ↓
Query all shards
  ↓
Combine results
  ↓
Expensive
```

### Solutions

**1. Avoid Cross-Shard Queries:**
```
Design to avoid
  ↓
Use shard key in queries
  ↓
Single shard queries
```

**2. Fan-Out Queries:**
```
Query all shards
  ↓
Combine results
  ↓
Acceptable for some cases
```

**3. Materialized Views:**
```
Pre-compute cross-shard data
  ↓
Store in single shard
  ↓
Fast queries
```

---

## Shard Rebalancing

### Why Rebalance?

**Reasons:**
- **Uneven distribution**: Uneven data distribution
- **Hot spots**: Hot spots
- **New shards**: Adding new shards
- **Removing shards**: Removing shards

### Rebalancing Process

**1. Identify Imbalance:**
```
Measure shard sizes
  ↓
Identify imbalance
  ↓
Plan rebalancing
```

**2. Migrate Data:**
```
Move data between shards
  ↓
Maintain consistency
  ↓
Update routing
```

**3. Verify:**
```
Verify data
  ↓
Test queries
  ↓
Monitor performance
```

### Rebalancing Challenges

**1. Downtime:**
```
May require downtime
  ↓
Or complex online migration
```

**2. Consistency:**
```
Maintain consistency
  ↓
During migration
  ↓
Complex
```

**3. Performance:**
```
Migration overhead
  ↓
Performance impact
  ↓
During rebalancing
```

---

## Best Practices

### 1. Choose Right Shard Key

**Why:**
- **Distribution**: Even distribution
- **Query patterns**: Match query patterns
- **Performance**: Better performance

**Guidelines:**
- **High cardinality**: High cardinality
- **Even distribution**: Even distribution
- **Query patterns**: Match query patterns

### 2. Start Simple

**Why:**
- **Complexity**: Sharding adds complexity
- **Start small**: Start with simple approach
- **Evolve**: Evolve as needed

**Guidelines:**
- **Don't shard early**: Don't shard until needed
- **Start with replication**: Start with replication
- **Shard when needed**: Shard when needed

### 3. Plan for Cross-Shard Queries

**Why:**
- **Inevitable**: Some cross-shard queries inevitable
- **Performance**: Plan for performance
- **Architecture**: Design architecture

**Guidelines:**
- **Minimize**: Minimize cross-shard queries
- **Optimize**: Optimize when needed
- **Accept**: Accept some cross-shard queries

### 4. Monitor Shards

**Why:**
- **Health**: Monitor shard health
- **Performance**: Monitor performance
- **Imbalance**: Detect imbalance

**Metrics:**
- **Size**: Shard size
- **Load**: Query load
- **Performance**: Query performance
- **Errors**: Error rates

### 5. Plan for Rebalancing

**Why:**
- **Inevitable**: Rebalancing inevitable
- **Plan**: Plan for it
- **Tools**: Have tools ready

**Guidelines:**
- **Automation**: Automate rebalancing
- **Testing**: Test rebalancing
- **Monitoring**: Monitor during rebalancing

---

## Common Challenges

### Challenge 1: Hot Spots

**Problem:**
```
Uneven distribution
  ↓
Some shards overloaded
  ↓
Poor performance
```

**Solution:**
```
Choose better shard key
  ↓
Rebalance
  ↓
Monitor distribution
```

### Challenge 2: Cross-Shard Queries

**Problem:**
```
Queries span shards
  ↓
Slow and complex
  ↓
Poor performance
```

**Solution:**
```
Design to avoid
  ↓
Use shard key
  ↓
Accept some cross-shard
```

### Challenge 3: Rebalancing

**Problem:**
```
Hard to rebalance
  ↓
Downtime
  ↓
Complexity
```

**Solution:**
```
Plan for rebalancing
  ↓
Automate
  ↓
Test thoroughly
```

### Challenge 4: Joins Across Shards

**Problem:**
```
Joins across shards
  ↓
Very slow
  ↓
Complex
```

**Solution:**
```
Denormalize
  ↓
Avoid joins
  ↓
Materialized views
```

---

## Summary

Database sharding enables horizontal scaling for large datasets. Understanding strategies, shard key selection, and challenges is essential for building scalable systems.

**Key Takeaways:**
- **Sharding**: Split database into shards
- **Strategies**: Range-based, hash-based, directory-based, geographic
- **Horizontal**: Split by rows
- **Vertical**: Split by columns
- **Shard key**: Critical for distribution
- **Cross-shard queries**: Expensive, avoid when possible
- **Rebalancing**: Plan for rebalancing
- **Best practices**: Choose right key, start simple, monitor, plan

**Sharding Strategies:**
- **Range-based**: Simple, but hot spots
- **Hash-based**: Even distribution, no hot spots
- **Directory-based**: Flexible, but overhead
- **Geographic**: Natural, but may be uneven

**Best Practices:**
- Choose right shard key
- Start simple
- Plan for cross-shard queries
- Monitor shards
- Plan for rebalancing

**Common Challenges:**
- Hot spots
- Cross-shard queries
- Rebalancing
- Joins across shards

**Next Steps:**
- Evaluate need for sharding
- Choose sharding strategy
- Select shard key
- Design architecture
- Plan for challenges

