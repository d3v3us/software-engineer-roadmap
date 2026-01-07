# Database Migrations Deep Dive - Complete Understanding

## Table of Contents
1. [What are Database Migrations?](#what-are-database-migrations)
2. [Why Do We Need Migrations?](#why-do-we-need-migrations)
3. [Migration Concepts](#migration-concepts)
4. [Types of Migrations](#types-of-migrations)
5. [Migration Tools](#migration-tools)
6. [Writing Migrations](#writing-migrations)
7. [Migration Best Practices](#migration-best-practices)
8. [Rolling Back Migrations](#rolling-back-migrations)
9. [Migration Strategies](#migration-strategies)
10. [Handling Data Migrations](#handling-data-migrations)
11. [Zero-Downtime Migrations](#zero-downtime-migrations)
12. [Common Migration Patterns](#common-migration-patterns)
13. [Migration Testing](#migration-testing)
14. [Troubleshooting Migrations](#troubleshooting-migrations)

---

## What are Database Migrations?

### Definition

**Database Migration**: Scripted changes to database schema that can be versioned, tracked, and applied consistently across environments.

**Key Characteristics:**
- **Versioned**: Each migration has a version/timestamp
- **Reversible**: Can be rolled back
- **Reproducible**: Can be applied to any environment
- **Tracked**: System tracks which migrations have been applied

### Real-World Analogy

**Migrations = Building Renovations:**
- **Blueprint**: Migration script (plan)
- **Versioned**: Each change has a version
- **Reversible**: Can undo changes
- **Tracked**: Know what changes were made
- **Consistent**: Same changes everywhere

**Without Migrations:**
```
Developer A: Manually changes database
Developer B: Doesn't know about changes
Production: Different from development
Result: Chaos, bugs, inconsistencies
```

**With Migrations:**
```
Migration script: Add column "email" to users table
All developers: Run same migration
Production: Run same migration
Result: Consistent schema everywhere
```

---

## Why Do We Need Migrations?

### Problems Without Migrations

**1. Manual Changes:**
```
Developer manually alters database
Other developers don't know
Production different from dev
Inconsistencies everywhere
```

**2. No History:**
```
Can't see what changed
Can't track schema evolution
Hard to debug issues
No audit trail
```

**3. No Rollback:**
```
Made a mistake?
Can't easily undo
Must manually fix
Risky and error-prone
```

**4. Environment Drift:**
```
Dev: Schema version 1
Staging: Schema version 2
Production: Schema version 3
All different!
```

### Benefits of Migrations

**1. Version Control:**
- **Tracked in Git**: Migrations in version control
- **History**: See all schema changes
- **Review**: Review changes before applying

**2. Consistency:**
- **Same everywhere**: Same schema in all environments
- **Reproducible**: Can recreate database from scratch
- **Predictable**: Know what schema you have

**3. Collaboration:**
- **Team knows**: Everyone sees changes
- **No conflicts**: Avoid manual change conflicts
- **Clear process**: Clear process for changes

**4. Safety:**
- **Reversible**: Can rollback if needed
- **Tested**: Can test migrations
- **Controlled**: Controlled deployment

---

## Migration Concepts

### Key Concepts

**1. Migration File:**
- **Script**: Script that changes schema
- **Versioned**: Has version/timestamp
- **Idempotent**: Can detect if already applied

**2. Migration Table:**
- **Tracking**: Tracks applied migrations
- **State**: Knows current schema version
- **History**: History of all migrations

**3. Up Migration:**
- **Forward**: Applies changes
- **Create**: Creates/modifies schema

**4. Down Migration:**
- **Reverse**: Reverses changes
- **Rollback**: Rolls back to previous state

### Migration Flow

```
1. Developer writes migration
   ↓
2. Commit to version control
   ↓
3. Run migration locally
   ↓
4. Test migration
   ↓
5. Deploy to staging
   ↓
6. Run migration in staging
   ↓
7. Deploy to production
   ↓
8. Run migration in production
```

### Migration Table

**Example:**
```sql
CREATE TABLE schema_migrations (
    version VARCHAR(255) PRIMARY KEY,
    applied_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP
);
```

**Contents:**
```
version          | applied_at
----------------|-------------------
20240101000001  | 2024-01-01 10:00:00
20240102000002  | 2024-01-02 11:00:00
20240103000003  | 2024-01-03 12:00:00
```

---

## Types of Migrations

### 1. Schema Migrations

**Structure Changes:**
- **Add table**: Create new table
- **Add column**: Add column to table
- **Modify column**: Change column type/size
- **Remove column**: Remove column
- **Add index**: Create index
- **Add constraint**: Add foreign key, unique, etc.

**Example:**
```sql
-- Add column
ALTER TABLE users ADD COLUMN email VARCHAR(255);

-- Add index
CREATE INDEX idx_users_email ON users(email);

-- Add foreign key
ALTER TABLE orders ADD CONSTRAINT fk_user 
    FOREIGN KEY (user_id) REFERENCES users(id);
```

### 2. Data Migrations

**Data Changes:**
- **Transform data**: Change data format
- **Backfill data**: Fill missing data
- **Clean data**: Remove invalid data
- **Move data**: Move data between tables

**Example:**
```sql
-- Backfill email from username
UPDATE users 
SET email = username || '@example.com' 
WHERE email IS NULL;

-- Transform status
UPDATE orders 
SET status = 'completed' 
WHERE status = 'done';
```

### 3. Seed Migrations

**Initial Data:**
- **Reference data**: Countries, categories, etc.
- **Default data**: Default users, settings
- **Test data**: Test data for development

**Example:**
```sql
-- Seed countries
INSERT INTO countries (code, name) VALUES
    ('US', 'United States'),
    ('CA', 'Canada'),
    ('UK', 'United Kingdom');
```

---

## Migration Tools

### Popular Tools

**1. Rails Migrations (Ruby on Rails):**
```ruby
class AddEmailToUsers < ActiveRecord::Migration[7.0]
  def change
    add_column :users, :email, :string
  end
end
```

**2. Django Migrations (Python):**
```python
from django.db import migrations, models

class Migration(migrations.Migration):
    dependencies = [
        ('app', '0001_initial'),
    ]
    
    operations = [
        migrations.AddField(
            model_name='user',
            name='email',
            field=models.CharField(max_length=255),
        ),
    ]
```

**3. Alembic (Python/SQLAlchemy):**
```python
def upgrade():
    op.add_column('users', sa.Column('email', sa.String(255)))

def downgrade():
    op.drop_column('users', 'email')
```

**4. Flyway (Java):**
```sql
-- V1__Add_email_to_users.sql
ALTER TABLE users ADD COLUMN email VARCHAR(255);
```

**5. Liquibase (Java):**
```xml
<changeSet id="1" author="developer">
    <addColumn tableName="users">
        <column name="email" type="varchar(255)"/>
    </addColumn>
</changeSet>
```

**6. TypeORM (TypeScript/Node.js):**
```typescript
export class AddEmailToUsers1234567890 implements MigrationInterface {
    public async up(queryRunner: QueryRunner): Promise<void> {
        await queryRunner.addColumn('users', new TableColumn({
            name: 'email',
            type: 'varchar',
            length: '255'
        }));
    }
    
    public async down(queryRunner: QueryRunner): Promise<void> {
        await queryRunner.dropColumn('users', 'email');
    }
}
```

---

## Writing Migrations

### Migration Structure

**Basic Structure:**
```
1. Up migration: Apply changes
2. Down migration: Reverse changes
3. Version/ID: Unique identifier
4. Description: What migration does
```

### Example: Add Column

**Up Migration:**
```sql
ALTER TABLE users ADD COLUMN email VARCHAR(255);
```

**Down Migration:**
```sql
ALTER TABLE users DROP COLUMN email;
```

### Example: Create Table

**Up Migration:**
```sql
CREATE TABLE orders (
    id SERIAL PRIMARY KEY,
    user_id INTEGER NOT NULL,
    total DECIMAL(10, 2) NOT NULL,
    created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
    FOREIGN KEY (user_id) REFERENCES users(id)
);

CREATE INDEX idx_orders_user_id ON orders(user_id);
```

**Down Migration:**
```sql
DROP INDEX IF EXISTS idx_orders_user_id;
DROP TABLE IF EXISTS orders;
```

### Example: Modify Column

**Up Migration:**
```sql
ALTER TABLE users 
ALTER COLUMN email TYPE VARCHAR(255),
ALTER COLUMN email SET NOT NULL;
```

**Down Migration:**
```sql
ALTER TABLE users 
ALTER COLUMN email TYPE VARCHAR(100),
ALTER COLUMN email DROP NOT NULL;
```

### Best Practices for Writing Migrations

**1. Make Reversible:**
```sql
-- Good: Has down migration
ALTER TABLE users ADD COLUMN email VARCHAR(255);
-- Down: ALTER TABLE users DROP COLUMN email;

-- Bad: No way to reverse
ALTER TABLE users DROP COLUMN old_column;
```

**2. Use Transactions:**
```sql
BEGIN;
ALTER TABLE users ADD COLUMN email VARCHAR(255);
-- If error, rollback automatically
COMMIT;
```

**3. Check Before Applying:**
```sql
-- Good: Check if column exists
DO $$
BEGIN
    IF NOT EXISTS (
        SELECT 1 FROM information_schema.columns 
        WHERE table_name = 'users' AND column_name = 'email'
    ) THEN
        ALTER TABLE users ADD COLUMN email VARCHAR(255);
    END IF;
END $$;
```

**4. Be Explicit:**
```sql
-- Good: Explicit
ALTER TABLE users ADD COLUMN email VARCHAR(255) NOT NULL DEFAULT '';

-- Bad: Implicit (might fail if data exists)
ALTER TABLE users ADD COLUMN email VARCHAR(255) NOT NULL;
```

---

## Migration Best Practices

### 1. One Change Per Migration

**Good:**
```
Migration 1: Add email column
Migration 2: Add phone column
Migration 3: Add index on email
```

**Bad:**
```
Migration 1: Add email, phone, address, index, foreign key
(Too many changes, hard to rollback)
```

### 2. Name Migrations Clearly

**Good:**
```
20240101000001_add_email_to_users.sql
20240102000002_add_phone_to_users.sql
20240103000003_create_orders_table.sql
```

**Bad:**
```
20240101000001_migration.sql
20240102000002_update.sql
20240103000003_fix.sql
```

### 3. Test Migrations

**Test:**
- **Up migration**: Test applying migration
- **Down migration**: Test rolling back
- **Data integrity**: Test data integrity
- **Performance**: Test performance impact

### 4. Review Migrations

**Before applying:**
- **Code review**: Review migration code
- **Understand impact**: Understand what it does
- **Check dependencies**: Check dependencies
- **Test locally**: Test locally first

### 5. Backup Before Production

**Always:**
- **Backup database**: Backup before migration
- **Test restore**: Test restore process
- **Have rollback plan**: Plan for rollback
- **Monitor**: Monitor during migration

---

## Rolling Back Migrations

### Why Rollback?

**Reasons:**
- **Bug in migration**: Migration has bug
- **Performance issue**: Migration causes performance issue
- **Breaking change**: Migration breaks application
- **Wrong migration**: Applied wrong migration

### Rollback Process

**1. Identify Migration:**
```
Check migration history
Find migration to rollback
```

**2. Run Down Migration:**
```
Run down migration
Reverse changes
```

**3. Verify:**
```
Check schema
Verify data
Test application
```

### Rollback Example

**Up Migration:**
```sql
ALTER TABLE users ADD COLUMN email VARCHAR(255);
```

**Down Migration:**
```sql
ALTER TABLE users DROP COLUMN email;
```

**Rollback:**
```bash
# Rails
rails db:rollback

# Django
python manage.py migrate app_name 0001

# Flyway
flyway undo

# Alembic
alembic downgrade -1
```

### Rollback Considerations

**1. Data Loss:**
```
Rolling back might lose data
Consider data migration
Backup before rollback
```

**2. Dependencies:**
```
Later migrations depend on this one
Must rollback in reverse order
Check dependencies
```

**3. Irreversible Changes:**
```
Some changes can't be reversed
Dropping table with data
Consider carefully
```

---

## Migration Strategies

### Strategy 1: Forward-Only Migrations

**Approach:**
- **Only up migrations**: No down migrations
- **Always forward**: Always move forward
- **Fix with new migration**: Fix issues with new migration

**Use When:**
- **Simple projects**: Simple projects
- **No rollback needed**: Don't need rollback
- **Rapid development**: Rapid development

### Strategy 2: Reversible Migrations

**Approach:**
- **Up and down**: Both up and down migrations
- **Can rollback**: Can rollback if needed
- **Safer**: Safer for production

**Use When:**
- **Production systems**: Production systems
- **Need rollback**: Might need rollback
- **Critical systems**: Critical systems

### Strategy 3: Blue-Green Migrations

**Approach:**
- **Two environments**: Blue and green
- **Migrate one**: Migrate one environment
- **Switch traffic**: Switch traffic when ready

**Use When:**
- **Zero downtime**: Need zero downtime
- **Large migrations**: Large, risky migrations
- **High availability**: High availability required

---

## Handling Data Migrations

### Data Migration Challenges

**1. Large Datasets:**
```
Millions of rows
Takes long time
Might timeout
```

**2. Data Integrity:**
```
Must maintain integrity
Validate data
Handle errors
```

**3. Performance:**
```
Slow queries
Lock tables
Impact users
```

### Data Migration Patterns

**1. Batch Processing:**
```sql
-- Process in batches
DO $$
DECLARE
    batch_size INTEGER := 1000;
    offset_val INTEGER := 0;
BEGIN
    LOOP
        UPDATE users 
        SET email = username || '@example.com'
        WHERE email IS NULL
        AND id > offset_val
        LIMIT batch_size;
        
        EXIT WHEN NOT FOUND;
        offset_val := offset_val + batch_size;
        COMMIT;
    END LOOP;
END $$;
```

**2. Background Jobs:**
```
Migration triggers background job
Job processes data
Migration completes quickly
```

**3. Two-Phase Migration:**
```
Phase 1: Add column (nullable)
Phase 2: Backfill data
Phase 3: Make NOT NULL
```

### Data Migration Best Practices

**1. Validate Data:**
```sql
-- Validate before migration
SELECT COUNT(*) FROM users WHERE email IS NULL;
-- Should be 0 before making NOT NULL
```

**2. Test on Sample:**
```sql
-- Test on sample first
UPDATE users 
SET email = username || '@example.com'
WHERE id < 100;  -- Test on 100 rows
```

**3. Monitor Progress:**
```sql
-- Track progress
SELECT 
    COUNT(*) as total,
    COUNT(email) as with_email,
    COUNT(*) - COUNT(email) as remaining
FROM users;
```

---

## Zero-Downtime Migrations

### Challenge

**Problem:**
```
Migration locks table
Application can't use table
Downtime for users
```

### Solution: Zero-Downtime Pattern

**Pattern:**
```
1. Add new column (nullable)
2. Backfill data (in background)
3. Update application (use new column)
4. Make NOT NULL (after all data backfilled)
5. Remove old column (after migration complete)
```

### Example: Rename Column

**Step 1: Add New Column**
```sql
ALTER TABLE users ADD COLUMN email_address VARCHAR(255);
```

**Step 2: Backfill Data**
```sql
UPDATE users SET email_address = email;
```

**Step 3: Update Application**
```python
# Application now uses email_address
user.email_address
```

**Step 4: Remove Old Column**
```sql
ALTER TABLE users DROP COLUMN email;
```

### Zero-Downtime Best Practices

**1. Additive Changes:**
```
Add, don't modify
Add new column, don't change old
Add new table, don't change old
```

**2. Gradual Migration:**
```
Migrate gradually
Not all at once
Test at each step
```

**3. Feature Flags:**
```
Use feature flags
Control migration
Rollback if needed
```

---

## Common Migration Patterns

### Pattern 1: Add Column

**Standard:**
```sql
ALTER TABLE users ADD COLUMN email VARCHAR(255);
```

**Zero-Downtime:**
```sql
-- Step 1: Add nullable
ALTER TABLE users ADD COLUMN email VARCHAR(255);

-- Step 2: Backfill
UPDATE users SET email = username || '@example.com';

-- Step 3: Make NOT NULL
ALTER TABLE users ALTER COLUMN email SET NOT NULL;
```

### Pattern 2: Remove Column

**Standard:**
```sql
ALTER TABLE users DROP COLUMN old_column;
```

**Zero-Downtime:**
```sql
-- Step 1: Stop using column in application
-- Step 2: Remove column
ALTER TABLE users DROP COLUMN old_column;
```

### Pattern 3: Rename Column

**Standard:**
```sql
ALTER TABLE users RENAME COLUMN email TO email_address;
```

**Zero-Downtime:**
```sql
-- Step 1: Add new column
ALTER TABLE users ADD COLUMN email_address VARCHAR(255);

-- Step 2: Copy data
UPDATE users SET email_address = email;

-- Step 3: Update application

-- Step 4: Remove old
ALTER TABLE users DROP COLUMN email;
```

### Pattern 4: Change Column Type

**Standard:**
```sql
ALTER TABLE users ALTER COLUMN age TYPE INTEGER;
```

**Zero-Downtime:**
```sql
-- Step 1: Add new column
ALTER TABLE users ADD COLUMN age_new INTEGER;

-- Step 2: Copy and convert
UPDATE users SET age_new = CAST(age AS INTEGER);

-- Step 3: Update application

-- Step 4: Remove old
ALTER TABLE users DROP COLUMN age;
ALTER TABLE users RENAME COLUMN age_new TO age;
```

---

## Migration Testing

### What to Test

**1. Up Migration:**
- **Applies correctly**: Migration applies without errors
- **Schema correct**: Schema is correct after
- **Data intact**: Data is intact
- **Performance**: Performance is acceptable

**2. Down Migration:**
- **Rolls back correctly**: Can rollback
- **Schema correct**: Schema correct after rollback
- **Data intact**: Data intact after rollback

**3. Idempotency:**
- **Can run twice**: Can run migration twice
- **No errors**: No errors on second run
- **Same result**: Same result

### Testing Strategies

**1. Test Database:**
```
Create test database
Run migration
Verify schema
Test application
```

**2. Integration Tests:**
```python
def test_migration():
    # Run migration
    run_migration('001_add_email')
    
    # Verify
    assert column_exists('users', 'email')
    
    # Test application
    user = User.create(email='test@example.com')
    assert user.email == 'test@example.com'
```

**3. Rollback Tests:**
```python
def test_rollback():
    # Run migration
    run_migration('001_add_email')
    
    # Rollback
    rollback_migration('001_add_email')
    
    # Verify
    assert not column_exists('users', 'email')
```

---

## Troubleshooting Migrations

### Common Issues

**1. Migration Already Applied:**
```
Error: Migration already applied
Solution: Check migration table
Skip if already applied
```

**2. Migration Fails:**
```
Error: Migration fails
Solution: Check error message
Fix migration
Rollback if needed
```

**3. Data Conflicts:**
```
Error: Constraint violation
Solution: Check data
Fix data before migration
```

**4. Lock Timeout:**
```
Error: Lock timeout
Solution: Reduce migration scope
Run during low traffic
```

### Debugging Steps

**1. Check Migration Status:**
```sql
SELECT * FROM schema_migrations 
ORDER BY applied_at DESC;
```

**2. Check Schema:**
```sql
-- PostgreSQL
\d users

-- MySQL
DESCRIBE users;
```

**3. Check Logs:**
```
Check migration logs
Check application logs
Check database logs
```

**4. Test Locally:**
```
Reproduce locally
Test migration
Debug issue
```

---

## Summary

Database migrations are essential for managing database schema changes in a controlled, versioned, and reproducible way.

**Key Takeaways:**
- **Migrations**: Versioned, tracked schema changes
- **Benefits**: Version control, consistency, collaboration, safety
- **Types**: Schema, data, seed migrations
- **Tools**: Rails, Django, Alembic, Flyway, Liquibase, TypeORM
- **Best practices**: One change per migration, clear names, test, review
- **Rollback**: Can rollback if needed
- **Zero-downtime**: Patterns for zero-downtime migrations
- **Testing**: Test migrations thoroughly

**Migration Process:**
1. Write migration
2. Test locally
3. Review
4. Deploy to staging
5. Test in staging
6. Deploy to production
7. Monitor

**Best Practices:**
- One change per migration
- Clear naming
- Test migrations
- Review before applying
- Backup before production
- Make reversible
- Use transactions
- Check before applying

**Next Steps:**
- Choose migration tool
- Set up migration process
- Write first migration
- Test migration process
- Apply best practices

