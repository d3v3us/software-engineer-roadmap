# Database Backup and Recovery Deep Dive - Complete Understanding

## Table of Contents
1. [What is Database Backup?](#what-is-database-backup)
2. [Why Do We Need Backups?](#why-do-we-need-backups)
3. [Types of Backups](#types-of-backups)
4. [Backup Strategies](#backup-strategies)
5. [Full Backup](#full-backup)
6. [Incremental Backup](#incremental-backup)
7. [Differential Backup](#differential-backup)
8. [Continuous Backup](#continuous-backup)
9. [Backup Storage](#backup-storage)
10. [Recovery Strategies](#recovery-strategies)
11. [Point-in-Time Recovery (PITR)](#point-in-time-recovery-pitr)
12. [Disaster Recovery](#disaster-recovery)
13. [Backup Testing](#backup-testing)
14. [Best Practices](#best-practices)

---

## What is Database Backup?

### Definition

**Database Backup**: Copy of database data and structure that can be used to restore the database to a previous state.

**Key Concept:**
- **Copy of data**: Copy of database data
- **Restore capability**: Can restore from backup
- **Point in time**: Snapshot at point in time
- **Recovery**: Use for recovery

### Real-World Analogy

**Backup = Photo Copy:**
- **Original document**: Database
- **Photo copy**: Backup
- **If original lost**: Use photo copy
- **Restore**: Restore from copy

**Database:**
- **Database**: Original database
- **Backup**: Copy of database
- **If database lost**: Restore from backup
- **Recovery**: Recover data

---

## Why Do We Need Backups?

### Disaster Scenarios

**1. Hardware Failure:**
```
Database server fails
  ↓
Data lost
  ↓
Restore from backup
```

**2. Human Error:**
```
Accidental DELETE
  ↓
Data deleted
  ↓
Restore from backup
```

**3. Software Bug:**
```
Bug corrupts data
  ↓
Data corrupted
  ↓
Restore from backup
```

**4. Security Breach:**
```
Ransomware attack
  ↓
Data encrypted
  ↓
Restore from backup
```

### Benefits of Backups

**1. Data Protection:**
- **Protect data**: Protect against data loss
- **Recovery**: Can recover data
- **Business continuity**: Business continuity

**2. Compliance:**
- **Regulatory**: Regulatory requirements
- **Audit**: Audit requirements
- **Legal**: Legal requirements

**3. Peace of Mind:**
- **Confidence**: Confidence in data safety
- **Risk mitigation**: Risk mitigation
- **Disaster recovery**: Disaster recovery

---

## Types of Backups

### Backup Types

**1. Full Backup:**
- **All data**: All database data
- **Complete**: Complete copy
- **Largest**: Largest backup size

**2. Incremental Backup:**
- **Changes only**: Only changes since last backup
- **Smaller**: Smaller backup size
- **Faster**: Faster backup

**3. Differential Backup:**
- **Changes since full**: Changes since last full backup
- **Medium size**: Medium backup size
- **Faster restore**: Faster restore than incremental

**4. Continuous Backup:**
- **Continuous**: Continuous backup
- **Real-time**: Near real-time
- **Point-in-time**: Point-in-time recovery

---

## Backup Strategies

### Strategy 1: Full Backup Only

**Approach:**
```
Daily: Full backup
  ↓
Simple strategy
  ↓
Easy to restore
```

**Pros:**
- **Simple**: Simple strategy
- **Fast restore**: Fast restore (one backup)
- **Easy**: Easy to manage

**Cons:**
- **Large**: Large backup size
- **Slow**: Slow backup
- **Storage**: High storage cost

### Strategy 2: Full + Incremental

**Approach:**
```
Sunday: Full backup
Monday-Saturday: Incremental backup
  ↓
Smaller backups
  ↓
Faster backups
```

**Pros:**
- **Smaller**: Smaller backup size
- **Faster**: Faster backups
- **Efficient**: More efficient

**Cons:**
- **Slower restore**: Slower restore (need full + all incrementals)
- **More complex**: More complex
- **Dependency**: Incrementals depend on full

### Strategy 3: Full + Differential

**Approach:**
```
Sunday: Full backup
Monday-Saturday: Differential backup
  ↓
Medium backups
  ↓
Faster restore than incremental
```

**Pros:**
- **Faster restore**: Faster restore (full + one differential)
- **Medium size**: Medium backup size
- **Balance**: Balance between full and incremental

**Cons:**
- **Larger than incremental**: Larger than incremental
- **More complex**: More complex than full only

### Strategy 4: 3-2-1 Rule

**3-2-1 Rule:**
```
3 copies of data
2 different media types
1 off-site copy
```

**Example:**
```
1. Production database
2. Local backup (disk)
3. Remote backup (cloud)
  ↓
3 copies, 2 media (disk, cloud), 1 off-site
```

---

## Full Backup

### What is Full Backup?

**Full Backup**: Complete copy of entire database.

**Characteristics:**
- **All data**: All database data
- **All structure**: All database structure
- **Complete**: Complete snapshot

### Full Backup Process

**1. Lock Database (Optional):**
```
Lock database (read-only)
  ↓
Consistent snapshot
```

**2. Copy Data:**
```
Copy all data files
  ↓
Copy all tables
  ↓
Copy all indexes
```

**3. Copy Metadata:**
```
Copy schema
  ↓
Copy configuration
  ↓
Copy logs
```

### Full Backup Example

**PostgreSQL:**
```bash
# Full backup
pg_dump -F c -f backup.dump mydatabase

# Restore
pg_restore -d mydatabase backup.dump
```

**MySQL:**
```bash
# Full backup
mysqldump -u user -p database > backup.sql

# Restore
mysql -u user -p database < backup.sql
```

**MongoDB:**
```bash
# Full backup
mongodump --db mydatabase --out /backup

# Restore
mongorestore --db mydatabase /backup/mydatabase
```

---

## Incremental Backup

### What is Incremental Backup?

**Incremental Backup**: Backup of only changes since last backup (full or incremental).

**Characteristics:**
- **Changes only**: Only changed data
- **Small**: Much smaller than full
- **Fast**: Faster backup

### Incremental Backup Process

**1. Identify Changes:**
```
Compare with last backup
  ↓
Identify changed data
```

**2. Backup Changes:**
```
Backup only changed data
  ↓
Store increment
```

**3. Chain Incrementals:**
```
Full → Incremental 1 → Incremental 2 → ...
  ↓
Chain of backups
```

### Incremental Backup Example

**PostgreSQL (WAL Archiving):**
```sql
-- Enable WAL archiving
archive_mode = on
archive_command = 'cp %p /backup/wal/%f'

-- Full backup
pg_basebackup -D /backup/full

-- Incremental (WAL files)
-- WAL files are automatically archived
```

**MySQL (Binary Logs):**
```bash
# Full backup
mysqldump --all-databases > full_backup.sql

# Incremental (binary logs)
mysqlbinlog binlog.000001 > incremental.sql
```

---

## Differential Backup

### What is Differential Backup?

**Differential Backup**: Backup of all changes since last full backup.

**Characteristics:**
- **Since full**: All changes since full backup
- **Medium size**: Medium backup size
- **Faster restore**: Faster restore than incremental

### Differential vs Incremental

**Incremental:**
```
Full → Inc1 → Inc2 → Inc3
  ↓
Each incremental: Changes since previous
  ↓
Restore: Full + Inc1 + Inc2 + Inc3
```

**Differential:**
```
Full → Diff1 → Diff2 → Diff3
  ↓
Each differential: All changes since full
  ↓
Restore: Full + Diff3 (only need latest diff)
```

---

## Continuous Backup

### What is Continuous Backup?

**Continuous Backup**: Continuous backup of database changes.

**Characteristics:**
- **Continuous**: Continuous backup
- **Real-time**: Near real-time
- **Point-in-time**: Point-in-time recovery

### How Continuous Backup Works

**1. Transaction Logs:**
```
Every transaction logged
  ↓
Logs continuously backed up
  ↓
Can replay to any point
```

**2. Replication:**
```
Master → Replica
  ↓
Continuous replication
  ↓
Replica is backup
```

**3. WAL Archiving:**
```
Write-Ahead Logs archived
  ↓
Continuous archiving
  ↓
Point-in-time recovery
```

---

## Backup Storage

### Storage Options

**1. Local Storage:**
- **Same server**: Same server or local network
- **Fast**: Fast access
- **Risk**: Risk if server fails

**2. Remote Storage:**
- **Different location**: Different location
- **Safe**: Safe from local disasters
- **Slower**: Slower access

**3. Cloud Storage:**
- **Cloud**: Cloud storage (S3, etc.)
- **Scalable**: Scalable
- **Managed**: Managed service

### Storage Best Practices

**1. Multiple Locations:**
```
Local + Remote + Cloud
  ↓
Redundancy
  ↓
Safety
```

**2. Encryption:**
```
Encrypt backups
  ↓
Protect sensitive data
  ↓
Security
```

**3. Versioning:**
```
Keep multiple versions
  ↓
Can restore to different points
  ↓
Flexibility
```

---

## Recovery Strategies

### Recovery Types

**1. Full Recovery:**
```
Restore full backup
  ↓
Complete recovery
```

**2. Partial Recovery:**
```
Restore specific tables
  ↓
Partial recovery
```

**3. Point-in-Time Recovery:**
```
Restore to specific point in time
  ↓
Precise recovery
```

### Recovery Process

**1. Stop Database:**
```
Stop database
  ↓
Prevent conflicts
```

**2. Restore Backup:**
```
Restore backup files
  ↓
Restore data
```

**3. Apply Logs:**
```
Apply transaction logs
  ↓
Recover to point in time
```

**4. Verify:**
```
Verify data integrity
  ↓
Test database
```

**5. Start Database:**
```
Start database
  ↓
Resume operations
```

---

## Point-in-Time Recovery (PITR)

### What is PITR?

**Point-in-Time Recovery**: Recovery to specific point in time.

**Use Case:**
```
10:00 AM: Full backup
10:30 AM: Accidental DELETE
11:00 AM: Discover error
  ↓
Recover to 10:29 AM (before DELETE)
```

### How PITR Works

**1. Full Backup:**
```
Full backup at 10:00 AM
```

**2. Transaction Logs:**
```
Transaction logs from 10:00 AM to 10:29 AM
```

**3. Recovery:**
```
Restore full backup
  ↓
Apply logs up to 10:29 AM
  ↓
Recover to 10:29 AM
```

### PITR Implementation

**PostgreSQL:**
```bash
# Restore full backup
pg_restore -d mydatabase backup.dump

# Apply WAL logs to point in time
recovery_target_time = '2024-01-15 10:29:00'
```

**MySQL:**
```bash
# Restore full backup
mysql -u user -p database < full_backup.sql

# Apply binary logs to point in time
mysqlbinlog --stop-datetime="2024-01-15 10:29:00" binlog.* | mysql -u user -p database
```

---

## Disaster Recovery

### What is Disaster Recovery?

**Disaster Recovery**: Process of recovering from major disasters.

**Disaster Types:**
- **Hardware failure**: Server failure
- **Data center failure**: Data center failure
- **Natural disaster**: Natural disaster
- **Cyber attack**: Cyber attack

### Disaster Recovery Plan

**1. Backup Strategy:**
```
Regular backups
  ↓
Multiple locations
  ↓
Tested backups
```

**2. Recovery Procedures:**
```
Documented procedures
  ↓
Tested procedures
  ↓
Trained team
```

**3. RTO and RPO:**
```
RTO (Recovery Time Objective): How fast to recover
RPO (Recovery Point Objective): How much data loss acceptable
```

### RTO and RPO

**RTO (Recovery Time Objective):**
```
Time to recover
  ↓
Example: 4 hours
  ↓
System must be back in 4 hours
```

**RPO (Recovery Point Objective):**
```
Acceptable data loss
  ↓
Example: 1 hour
  ↓
Can lose up to 1 hour of data
```

---

## Backup Testing

### Why Test Backups?

**Problem:**
```
Backup exists
  ↓
Never tested
  ↓
When needed, backup corrupted
  ↓
Cannot restore
```

**Solution: Test Backups**

### Testing Process

**1. Regular Testing:**
```
Test backups regularly
  ↓
Monthly or quarterly
  ↓
Ensure backups work
```

**2. Restore Test:**
```
Restore backup to test environment
  ↓
Verify data integrity
  ↓
Test recovery process
```

**3. Document Results:**
```
Document test results
  ↓
Fix issues found
  ↓
Improve process
```

### Testing Checklist

**1. Backup Completeness:**
- [ ] All data backed up
- [ ] All tables present
- [ ] All indexes present

**2. Backup Integrity:**
- [ ] Backup not corrupted
- [ ] Can restore successfully
- [ ] Data matches original

**3. Recovery Speed:**
- [ ] Recovery within RTO
- [ ] Recovery process works
- [ ] Team trained

---

## Best Practices

### 1. Follow 3-2-1 Rule

**Why:**
- **Redundancy**: Multiple copies
- **Safety**: Safety from disasters
- **Best practice**: Industry best practice

**Implementation:**
```
3 copies: Production, local backup, remote backup
2 media: Disk, cloud
1 off-site: Remote location
```

### 2. Automate Backups

**Why:**
- **Consistency**: Consistent backups
- **No human error**: No human error
- **Reliability**: More reliable

**Implementation:**
```
Cron job or scheduler
  ↓
Automated backups
  ↓
No manual intervention
```

### 3. Encrypt Backups

**Why:**
- **Security**: Protect sensitive data
- **Compliance**: Compliance requirements
- **Best practice**: Security best practice

**Implementation:**
```
Encrypt backup files
  ↓
Store encryption keys securely
  ↓
Protect backups
```

### 4. Test Regularly

**Why:**
- **Verify**: Verify backups work
- **Confidence**: Confidence in recovery
- **Fix issues**: Fix issues early

**Implementation:**
```
Monthly restore tests
  ↓
Verify integrity
  ↓
Document results
```

### 5. Monitor Backups

**Why:**
- **Visibility**: Visibility into backup status
- **Alerting**: Alert on failures
- **Proactive**: Proactive management

**Metrics:**
- **Backup success rate**: Backup success rate
- **Backup size**: Backup size trends
- **Backup duration**: Backup duration

---

## Summary

Database backup and recovery are essential for data protection and business continuity. Understanding backup types, strategies, and recovery procedures is crucial for production systems.

**Key Takeaways:**
- **Backup**: Copy of database for recovery
- **Types**: Full, incremental, differential, continuous
- **Strategies**: Full only, full+incremental, full+differential, 3-2-1 rule
- **Recovery**: Full, partial, point-in-time
- **PITR**: Point-in-time recovery
- **Disaster recovery**: RTO and RPO
- **Best practices**: 3-2-1 rule, automate, encrypt, test, monitor

**Backup Types:**
- **Full**: Complete copy
- **Incremental**: Changes since last backup
- **Differential**: Changes since full backup
- **Continuous**: Continuous backup

**Recovery Types:**
- **Full recovery**: Complete restore
- **Partial recovery**: Specific tables
- **Point-in-time**: Recover to specific time

**Best Practices:**
- Follow 3-2-1 rule
- Automate backups
- Encrypt backups
- Test regularly
- Monitor backups

**Next Steps:**
- Design backup strategy
- Implement backups
- Test recovery
- Monitor backups
- Document procedures

