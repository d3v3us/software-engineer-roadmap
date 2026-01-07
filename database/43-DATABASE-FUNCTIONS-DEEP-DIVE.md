# Database Functions Deep Dive - Complete Understanding

## Table of Contents
1. [What are Database Functions?](#what-are-database-functions)
2. [Why Database Functions Matter](#why-database-functions-matter)
3. [Function Types](#function-types)
4. [Built-in Functions](#built-in-functions)
5. [User-Defined Functions](#user-defined-functions)
6. [Aggregate Functions](#aggregate-functions)
7. [Window Functions](#window-functions)
8. [Best Practices](#best-practices)

---

## What are Database Functions?

### Definition

**Database Functions**: Predefined or user-defined operations in database.

**Key Concepts:**
- **Operations**: Database operations
- **Reusability**: Reusable code
- **Performance**: Optimized performance
- **Abstraction**: Data abstraction

### Real-World Analogy

**Database Functions = Calculator Functions:**
- **Calculator**: Database
- **Functions**: Built-in operations
- **Operations**: Mathematical operations
- **Reusability**: Reusable operations

**Database:**
- **Database**: Database system
- **Functions**: Database functions
- **Operations**: Data operations
- **Reusability**: Code reusability

---

## Why Database Functions Matter?

### Impact of Functions

**1. Code Reusability:**
```
Reusable functions
  ↓
Code reuse
  ↓
Efficiency
```

**2. Performance:**
```
Optimized functions
  ↓
Better performance
  ↓
Faster execution
```

**3. Consistency:**
```
Consistent operations
  ↓
Standardized logic
  ↓
Data quality
```

### Benefits of Functions

**1. Reusability:**
- **Code reuse**: Reuse code across queries
- **Consistency**: Consistent operations
- **Maintainability**: Easier maintenance

**2. Performance:**
- **Optimized**: Optimized by database
- **Efficient**: Efficient execution
- **Fast**: Faster than custom code

**3. Abstraction:**
- **Data abstraction**: Abstract data operations
- **Simplification**: Simplify queries
- **Readability**: Better readability

---

## Function Types

### Type 1: Scalar Functions

**What:**
```
Single value
  ↓
Return single value
  ↓
Per row
```

**Examples:**
- **UPPER()**: Convert to uppercase
- **LOWER()**: Convert to lowercase
- **LENGTH()**: Get string length
- **ROUND()**: Round number

### Type 2: Aggregate Functions

**What:**
```
Multiple values
  ↓
Return single value
  ↓
Across rows
```

**Examples:**
- **COUNT()**: Count rows
- **SUM()**: Sum values
- **AVG()**: Average values
- **MAX()**: Maximum value
- **MIN()**: Minimum value

### Type 3: Window Functions

**What:**
```
Window of rows
  ↓
Per row result
  ↓
Partition-based
```

**Examples:**
- **ROW_NUMBER()**: Row number
- **RANK()**: Rank values
- **LAG()**: Previous value
- **LEAD()**: Next value

---

## Built-in Functions

### String Functions

**1. UPPER/LOWER:**
```sql
SELECT UPPER('hello') AS upper_case;
-- Returns: HELLO

SELECT LOWER('HELLO') AS lower_case;
-- Returns: hello
```

**2. LENGTH:**
```sql
SELECT LENGTH('hello') AS length;
-- Returns: 5
```

**3. SUBSTRING:**
```sql
SELECT SUBSTRING('hello', 1, 3) AS substring;
-- Returns: hel
```

### Numeric Functions

**1. ROUND:**
```sql
SELECT ROUND(3.14159, 2) AS rounded;
-- Returns: 3.14
```

**2. ABS:**
```sql
SELECT ABS(-10) AS absolute;
-- Returns: 10
```

**3. CEIL/FLOOR:**
```sql
SELECT CEIL(3.14) AS ceiling;
-- Returns: 4

SELECT FLOOR(3.14) AS floor;
-- Returns: 3
```

### Date Functions

**1. NOW/CURRENT_DATE:**
```sql
SELECT NOW() AS current_time;
SELECT CURRENT_DATE AS current_date;
```

**2. DATE_ADD/DATE_SUB:**
```sql
SELECT DATE_ADD(NOW(), INTERVAL 1 DAY) AS tomorrow;
SELECT DATE_SUB(NOW(), INTERVAL 1 MONTH) AS last_month;
```

**3. EXTRACT:**
```sql
SELECT EXTRACT(YEAR FROM NOW()) AS year;
SELECT EXTRACT(MONTH FROM NOW()) AS month;
```

---

## User-Defined Functions

### What are User-Defined Functions?

**User-Defined Functions**: Custom functions created by users.

**Types:**

**1. Scalar Functions:**
```
Return single value
  ↓
Per row
  ↓
Custom logic
```

**2. Table-Valued Functions:**
```
Return table
  ↓
Multiple rows
  ↓
Table result
```

### User-Defined Function Example

**PostgreSQL:**
```sql
CREATE OR REPLACE FUNCTION calculate_discount(
    price DECIMAL,
    discount_percent DECIMAL
) RETURNS DECIMAL AS $$
BEGIN
    RETURN price * (1 - discount_percent / 100);
END;
$$ LANGUAGE plpgsql;

-- Usage
SELECT calculate_discount(100, 10) AS discounted_price;
-- Returns: 90
```

**MySQL:**
```sql
DELIMITER $$

CREATE FUNCTION calculate_discount(
    price DECIMAL(10,2),
    discount_percent DECIMAL(5,2)
) RETURNS DECIMAL(10,2)
DETERMINISTIC
BEGIN
    RETURN price * (1 - discount_percent / 100);
END$$

DELIMITER ;
```

---

## Aggregate Functions

### Common Aggregate Functions

**1. COUNT:**
```sql
SELECT COUNT(*) FROM users;
SELECT COUNT(DISTINCT email) FROM users;
```

**2. SUM:**
```sql
SELECT SUM(amount) FROM orders;
```

**3. AVG:**
```sql
SELECT AVG(price) FROM products;
```

**4. MAX/MIN:**
```sql
SELECT MAX(price) FROM products;
SELECT MIN(price) FROM products;
```

### Aggregate with GROUP BY

**Example:**
```sql
SELECT 
    category,
    COUNT(*) AS product_count,
    AVG(price) AS avg_price,
    SUM(quantity) AS total_quantity
FROM products
GROUP BY category;
```

---

## Window Functions

### What are Window Functions?

**Window Functions**: Functions that operate on window of rows.

**Characteristics:**
- **Window**: Window of rows
- **Partition**: Partition by column
- **Order**: Order within partition
- **Per row**: Result per row

### Window Function Examples

**1. ROW_NUMBER:**
```sql
SELECT 
    id,
    name,
    ROW_NUMBER() OVER (ORDER BY created_at) AS row_num
FROM users;
```

**2. RANK:**
```sql
SELECT 
    product_id,
    sales,
    RANK() OVER (ORDER BY sales DESC) AS rank
FROM product_sales;
```

**3. LAG/LEAD:**
```sql
SELECT 
    date,
    sales,
    LAG(sales) OVER (ORDER BY date) AS previous_sales,
    LEAD(sales) OVER (ORDER BY date) AS next_sales
FROM daily_sales;
```

**4. SUM OVER:**
```sql
SELECT 
    date,
    sales,
    SUM(sales) OVER (ORDER BY date) AS running_total
FROM daily_sales;
```

---

## Best Practices

### 1. Use Built-in Functions

**Why:**
- **Performance**: Optimized performance
- **Reliability**: Reliable functions
- **Maintenance**: Less maintenance

**Guidelines:**
- **Prefer built-in**: Prefer built-in functions
- **Documentation**: Check documentation
- **Compatibility**: Consider compatibility

### 2. Optimize User-Defined Functions

**Why:**
- **Performance**: Better performance
- **Efficiency**: More efficient
- **Scalability**: Better scalability

**Guidelines:**
- **Efficient logic**: Write efficient logic
- **Avoid heavy operations**: Avoid heavy operations
- **Index usage**: Consider index usage

### 3. Document Functions

**Why:**
- **Understanding**: Better understanding
- **Maintenance**: Easier maintenance
- **Usage**: Easier to use

**Guidelines:**
- **Document purpose**: Document function purpose
- **Document parameters**: Document parameters
- **Document return**: Document return value

### 4. Test Functions

**Why:**
- **Correctness**: Ensure correctness
- **Reliability**: Ensure reliability
- **Quality**: Higher quality

**Guidelines:**
- **Unit tests**: Write unit tests
- **Edge cases**: Test edge cases
- **Performance**: Test performance

---

## Summary

Database functions are essential for data operations. Understanding function types, built-in functions, user-defined functions, aggregate functions, window functions, and best practices is crucial for effective database development.

**Key Takeaways:**
- **Database functions**: Predefined or user-defined operations
- **Function types**: Scalar functions (single value), aggregate functions (multiple values), window functions (window of rows)
- **Built-in functions**: String, numeric, date functions (UPPER, LOWER, ROUND, NOW, etc.)
- **User-defined functions**: Custom functions (scalar, table-valued)
- **Aggregate functions**: COUNT, SUM, AVG, MAX, MIN (with GROUP BY)
- **Window functions**: ROW_NUMBER, RANK, LAG, LEAD, SUM OVER (partition-based)
- **Best practices**: Use built-in functions, optimize user-defined functions, document functions, test functions

**Function Types:**
- **Scalar**: Single value per row
- **Aggregate**: Single value across rows
- **Window**: Per row with window context

**Best Practices:**
- Use built-in functions
- Optimize user-defined functions
- Document functions
- Test functions

**Next Steps:**
- Understand function types
- Learn built-in functions
- Create user-defined functions
- Apply best practices

