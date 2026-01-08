# Go ORM Patterns Deep Dive - Complete Understanding

## Table of Contents
1. [What are ORM Patterns in Go?](#what-are-orm-patterns-in-go)
2. [Why ORM Patterns Matter](#why-orm-patterns-matter)
3. [ORM Libraries](#orm-libraries)
4. [GORM Patterns](#gorm-patterns)
5. [SQLx Patterns](#sqlx-patterns)
6. [When to Use ORM vs Raw SQL](#when-to-use-orm-vs-raw-sql)
7. [ORM Best Practices](#orm-best-practices)
8. [Common Patterns](#common-patterns)
9. [Best Practices](#best-practices)

---

## What are ORM Patterns in Go?

### Definition

**ORM Patterns**: Patterns for using Object-Relational Mapping libraries in Go.

**Key Characteristics:**
- **Abstraction**: Database abstraction
- **Mapping**: Object-relational mapping
- **Convenience**: Convenient database access
- **Trade-offs**: Performance trade-offs

### Real-World Analogy

**ORM = Translator:**
- **Objects**: Go structs
- **Database**: SQL tables
- **ORM**: Translator
- **Mapping**: Automatic mapping

**Programming:**
- **Structs**: Go structs
- **Tables**: Database tables
- **ORM**: Mapping layer
- **Queries**: High-level queries

---

## Why ORM Patterns Matter?

### Benefits

**1. Productivity:**
```
High-level API
  ↓
ORM
  ↓
Faster development
```

**2. Type Safety:**
```
Type-safe queries
  ↓
ORM
  ↓
Compile-time safety
```

**3. Abstraction:**
```
Database abstraction
  ↓
ORM
  ↓
Database independence
```

---

## ORM Libraries

### GORM

**Popular ORM:**
- **Full-featured**: Full-featured ORM
- **Convenient**: Very convenient
- **Overhead**: Some overhead

**Installation:**
```bash
go get gorm.io/gorm
go get gorm.io/driver/postgres
```

### SQLx

**Lightweight:**
- **Lightweight**: Lightweight
- **SQL-like**: SQL-like queries
- **Fast**: Fast

**Installation:**
```bash
go get github.com/jmoiron/sqlx
```

---

## GORM Patterns

### Basic Usage

**Example:**
```go
import (
    "gorm.io/gorm"
    "gorm.io/driver/postgres"
)

type User struct {
    ID    uint   `gorm:"primaryKey"`
    Name  string
    Email string `gorm:"uniqueIndex"`
}

func main() {
    db, err := gorm.Open(postgres.Open("dsn"), &gorm.Config{})
    if err != nil {
        log.Fatal(err)
    }
    
    // Auto migrate
    db.AutoMigrate(&User{})
    
    // Create
    user := User{Name: "Alice", Email: "alice@example.com"}
    db.Create(&user)
    
    // Read
    var found User
    db.First(&found, user.ID)
    
    // Update
    db.Model(&found).Update("Name", "Bob")
    
    // Delete
    db.Delete(&found)
}
```

### Associations

**HasMany:**
```go
type User struct {
    ID    uint
    Name  string
    Posts []Post `gorm:"foreignKey:UserID"`
}

type Post struct {
    ID     uint
    Title  string
    UserID uint
    User   User
}
```

### Query Patterns

**Where:**
```go
db.Where("name = ?", "Alice").Find(&users)
db.Where("age > ?", 18).Find(&users)
```

**Preload:**
```go
db.Preload("Posts").Find(&users)
```

---

## SQLx Patterns

### Basic Usage

**Example:**
```go
import "github.com/jmoiron/sqlx"

type User struct {
    ID    int    `db:"id"`
    Name  string `db:"name"`
    Email string `db:"email"`
}

func main() {
    db, err := sqlx.Connect("postgres", "dsn")
    if err != nil {
        log.Fatal(err)
    }
    
    // Named queries
    users := []User{}
    db.Select(&users, "SELECT * FROM users WHERE age > $1", 18)
    
    // Named exec
    db.NamedExec("INSERT INTO users (name, email) VALUES (:name, :email)", 
        map[string]interface{}{
            "name":  "Alice",
            "email": "alice@example.com",
        })
}
```

### Struct Scanning

**Scan into struct:**
```go
var user User
db.Get(&user, "SELECT * FROM users WHERE id = $1", 1)
```

---

## When to Use ORM vs Raw SQL

### Use ORM When

**Use ORM when:**
- **Rapid development**: Need rapid development
- **Simple queries**: Simple queries
- **CRUD operations**: Standard CRUD
- **Team productivity**: Team productivity

### Use Raw SQL When

**Use raw SQL when:**
- **Complex queries**: Complex queries
- **Performance**: Performance critical
- **Control**: Need full control
- **Optimization**: Need optimization

---

## ORM Best Practices

### 1. Use Migrations

**Why:**
- **Version control**: Version control schema
- **Reproducibility**: Reproducible schema
- **Team collaboration**: Team collaboration

**Guidelines:**
- **Migrations**: Use migrations
- **Version**: Version migrations
- **Rollback**: Support rollback

### 2. Avoid N+1 Problem

**Why:**
- **Performance**: Better performance
- **Efficiency**: More efficient
- **Scalability**: Better scalability

**Guidelines:**
- **Preload**: Use preload
- **Eager loading**: Eager load associations
- **Batch**: Batch queries

### 3. Use Transactions

**Why:**
- **Consistency**: Data consistency
- **Atomicity**: Atomic operations
- **Reliability**: More reliable

**Guidelines:**
- **Transactions**: Use transactions
- **Error handling**: Handle errors
- **Rollback**: Rollback on error

### 4. Monitor Performance

**Why:**
- **Performance**: Monitor performance
- **Optimization**: Better optimization
- **Issues**: Identify issues

**Guidelines:**
- **Logging**: Log slow queries
- **Profiling**: Profile queries
- **Optimization**: Optimize queries

---

## Common Patterns

### Pattern 1: Repository Pattern

**Repository:**
```go
type UserRepository struct {
    db *gorm.DB
}

func (r *UserRepository) FindByID(id uint) (*User, error) {
    var user User
    err := r.db.First(&user, id).Error
    return &user, err
}
```

### Pattern 2: Service Layer

**Service:**
```go
type UserService struct {
    repo *UserRepository
}

func (s *UserService) GetUser(id uint) (*User, error) {
    return s.repo.FindByID(id)
}
```

---

## Best Practices

### 1. Choose Right Tool

**Why:**
- **Effectiveness**: More effective
- **Performance**: Better performance
- **Productivity**: Better productivity

**Guidelines:**
- **GORM**: Use for rapid development
- **SQLx**: Use for performance
- **Raw SQL**: Use for complex queries

### 2. Understand Trade-offs

**Why:**
- **Decisions**: Better decisions
- **Optimization**: Better optimization
- **Balance**: Balance trade-offs

**Guidelines:**
- **ORM overhead**: Understand overhead
- **Performance**: Consider performance
- **Balance**: Balance convenience and performance

### 3. Use Migrations

**Why:**
- **Schema management**: Better schema management
- **Version control**: Version control
- **Team collaboration**: Team collaboration

**Guidelines:**
- **Migrations**: Always use migrations
- **Version**: Version migrations
- **Document**: Document changes

### 4. Test Database Code

**Why:**
- **Correctness**: Ensure correctness
- **Reliability**: More reliable
- **Quality**: Better quality

**Guidelines:**
- **Tests**: Write tests
- **Integration tests**: Integration tests
- **Test data**: Use test data

---

## Summary

ORM patterns enable convenient database access in Go. Understanding ORM libraries, GORM patterns, SQLx patterns, when to use ORM vs raw SQL, best practices, and common patterns is crucial for database development.

**Key Takeaways:**
- **ORM patterns**: Patterns for using ORM libraries (abstraction, mapping, convenience, trade-offs)
- **ORM libraries**: GORM (full-featured, convenient, overhead), SQLx (lightweight, SQL-like, fast)
- **GORM patterns**: Basic usage (AutoMigrate, Create, Read, Update, Delete), associations (HasMany, BelongsTo), query patterns (Where, Preload)
- **SQLx patterns**: Basic usage (Connect, Select, NamedExec), struct scanning (Get, Select)
- **When to use ORM vs raw SQL**: Use ORM (rapid development, simple queries, CRUD) vs Use raw SQL (complex queries, performance, control)
- **ORM best practices**: Use migrations, avoid N+1 problem, use transactions, monitor performance
- **Common patterns**: Repository pattern, service layer
- **Best practices**: Choose right tool, understand trade-offs, use migrations, test database code

**ORM Benefits:**
- **Productivity**: Faster development
- **Type safety**: Type-safe queries
- **Abstraction**: Database abstraction

**Best Practices:**
- Choose right tool
- Understand trade-offs
- Use migrations
- Test database code

**Next Steps:**
- Learn ORM libraries
- Practice ORM patterns
- Understand trade-offs
- Apply best practices

