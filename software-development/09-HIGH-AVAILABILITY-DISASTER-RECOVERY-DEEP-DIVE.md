# High Availability and Disaster Recovery Deep Dive - Complete Understanding

## Table of Contents
1. [What is High Availability?](#what-is-high-availability)
2. [What is Disaster Recovery?](#what-is-disaster-recovery)
3. [High Availability Patterns](#high-availability-patterns)
4. [Disaster Recovery Strategies](#disaster-recovery-strategies)
5. [Backup Strategies](#backup-strategies)
6. [Failover Mechanisms](#failover-mechanisms)
7. [Measuring Availability](#measuring-availability)

---

## What is High Availability?

### Definition

**High Availability (HA)**: System design that ensures system remains operational and accessible for a high percentage of time, typically 99.9% or higher.

**Key Concept:**
- **Uptime**: System is available
- **Redundancy**: Multiple components
- **Failover**: Automatic switching
- **No single point of failure**

### Availability Levels

**Availability Percentages:**
```
99% (Two 9s):    87.6 hours downtime/year
99.9% (Three 9s): 8.76 hours downtime/year
99.99% (Four 9s): 52.56 minutes downtime/year
99.999% (Five 9s): 5.26 minutes downtime/year
```

**Real-World Examples:**
- **99%**: Basic systems
- **99.9%**: Business applications
- **99.99%**: Critical systems
- **99.999%**: Mission-critical systems

---

## What is Disaster Recovery?

### Definition

**Disaster Recovery (DR)**: Process of restoring systems and data after a disaster or major failure.

**Key Concepts:**
- **Disaster**: Major failure (fire, flood, cyber attack)
- **Recovery**: Restore operations
- **RTO**: Recovery Time Objective (how fast)
- **RPO**: Recovery Point Objective (how much data loss)

### RTO vs RPO

**RTO (Recovery Time Objective):**
- **What**: Maximum acceptable downtime
- **Question**: How fast must we recover?
- **Example**: 4 hours (must recover within 4 hours)

**RPO (Recovery Point Objective):**
- **What**: Maximum acceptable data loss
- **Question**: How much data can we lose?
- **Example**: 1 hour (can lose up to 1 hour of data)

**Example:**
```
Backup: Every hour
Disaster: Database corrupted at 2:30 PM
Last backup: 2:00 PM

RPO: 1 hour (lost 30 minutes of data, acceptable)
RTO: 4 hours (must recover by 6:30 PM)
```

---

## High Availability Patterns

### 1. Redundancy

**Concept:**
- Multiple components
- If one fails, others continue
- No single point of failure

**Example:**
```
Single Server:
  Server → [If fails, everything down]

Redundant Servers:
  Server 1 ──┐
  Server 2 ──┼──→ Load Balancer → Users
  Server 3 ──┘
  [If one fails, others continue]
```

### 2. Active-Passive (Hot Standby)

**Pattern:**
- Active server handles traffic
- Passive server ready to take over
- Automatic failover

**Example:**
```
Active Server (handling traffic)
  ↓
Passive Server (standby, ready)
  ↓
If active fails:
  Passive becomes active
  Traffic switches
```

**Benefits:**
- Fast failover
- No data loss
- Simple

**Drawbacks:**
- Wasted resources (passive idle)
- More expensive

### 3. Active-Active

**Pattern:**
- Multiple active servers
- All handle traffic
- Load distributed

**Example:**
```
Active Server 1 ──┐
Active Server 2 ──┼──→ Load Balancer → Users
Active Server 3 ──┘
All active, handling traffic
```

**Benefits:**
- Better resource utilization
- Higher capacity
- No wasted resources

**Drawbacks:**
- More complex
- State synchronization needed

### 4. Geographic Redundancy

**Pattern:**
- Servers in multiple locations
- Survive regional disasters
- Global availability

**Example:**
```
Region 1 (US East) ──┐
Region 2 (US West) ──┼──→ Global Load Balancer
Region 3 (Europe)  ───┘
```

**Benefits:**
- Survive regional disasters
- Lower latency (closer to users)
- Compliance (data in region)

---

## Disaster Recovery Strategies

### 1. Backup and Restore

**Strategy:**
- Regular backups
- Restore from backup when needed
- Simple, cost-effective

**Process:**
```
1. Regular backups (daily, hourly)
2. Store backups offsite
3. Test restore regularly
4. When disaster: Restore from backup
```

**RTO/RPO:**
- **RTO**: Hours to days (slow)
- **RPO**: Depends on backup frequency

### 2. Pilot Light

**Strategy:**
- Minimal infrastructure running
- Core data replicated
- Scale up when needed

**Example:**
```
Production: Full infrastructure
  ↓
Disaster Recovery: Minimal (pilot light)
  - Database replicated
  - No application servers
  ↓
When disaster: Scale up quickly
```

**RTO/RPO:**
- **RTO**: Minutes to hours
- **RPO**: Real-time (data replicated)

### 3. Warm Standby

**Strategy:**
- Scaled-down version running
- Data replicated
- Can scale up quickly

**Example:**
```
Production: Full capacity
  ↓
Warm Standby: Reduced capacity
  - Application servers (smaller)
  - Database replicated
  ↓
When disaster: Scale up
```

**RTO/RPO:**
- **RTO**: Minutes
- **RPO**: Real-time

### 4. Hot Standby (Multi-Site)

**Strategy:**
- Full duplicate running
- Real-time replication
- Instant failover

**Example:**
```
Site 1 (Primary): Full capacity
  ↓ (real-time replication)
Site 2 (Standby): Full capacity
  ↓
When disaster: Instant failover
```

**RTO/RPO:**
- **RTO**: Seconds to minutes
- **RPO**: Near zero

---

## Backup Strategies

### Backup Types

**1. Full Backup:**
- Complete copy of data
- Slow, large
- Complete recovery

**2. Incremental Backup:**
- Only changed data since last backup
- Fast, small
- Need full + all incrementals

**3. Differential Backup:**
- Changed data since last full backup
- Medium speed, medium size
- Need full + latest differential

### Backup Frequency

**Factors:**
- **RPO**: How much data can be lost?
- **Data change rate**: How often data changes?
- **Cost**: More frequent = more cost

**Examples:**
```
Critical: Every hour
Important: Every 6 hours
Standard: Daily
Archive: Weekly
```

### 3-2-1 Backup Rule

**Rule:**
- **3 copies**: Original + 2 backups
- **2 different media**: Different storage types
- **1 offsite**: One copy offsite

**Example:**
```
Original: Production database
Backup 1: Local backup (same location)
Backup 2: Cloud backup (offsite)
```

---

## Failover Mechanisms

### Automatic Failover

**How It Works:**
```
1. Monitor health
2. Detect failure
3. Automatically switch
4. Route traffic to backup
```

**Example:**
```
Active Server: Healthy
  ↓
Monitor: Checks health every 5 seconds
  ↓
Failure detected: No response
  ↓
Failover: Switch to standby
  ↓
Traffic: Routes to standby
```

### Manual Failover

**How It Works:**
```
1. Detect failure
2. Human decides
3. Manual switch
4. Verify
```

**Use When:**
- Need human verification
- Complex scenarios
- Risk of false positives

### Failover Time

**Factors:**
- **Detection time**: How fast to detect failure
- **Switch time**: How fast to switch
- **Recovery time**: How fast to recover

**Targets:**
- **Automatic**: Seconds to minutes
- **Manual**: Minutes to hours

---

## Measuring Availability

### Availability Calculation

**Formula:**
```
Availability = (Uptime / Total Time) × 100%
```

**Example:**
```
Total time: 365 days = 8,760 hours
Downtime: 8.76 hours
Uptime: 8,751.24 hours

Availability = (8,751.24 / 8,760) × 100% = 99.9%
```

### SLA (Service Level Agreement)

**SLA:**
- Contractual agreement
- Defines availability target
- Consequences if not met

**Example:**
```
SLA: 99.9% availability
If below: Service credit (refund)
Monitoring: Track continuously
```

### Monitoring Availability

**Metrics:**
- **Uptime**: Percentage of time up
- **Downtime**: Total downtime
- **MTBF**: Mean Time Between Failures
- **MTTR**: Mean Time To Repair

**Tools:**
- **Uptime monitoring**: Pingdom, UptimeRobot
- **Application monitoring**: New Relic, Datadog
- **Infrastructure monitoring**: CloudWatch, Prometheus

---

## Summary

High Availability and Disaster Recovery ensure systems remain operational and can recover from disasters.

**Key Takeaways:**
- High Availability: System remains operational (99.9%+)
- Disaster Recovery: Restore after disaster
- Patterns: Redundancy, active-passive, active-active, geographic
- Strategies: Backup/restore, pilot light, warm standby, hot standby
- Backup: 3-2-1 rule, full/incremental/differential
- Failover: Automatic or manual
- Measuring: Availability percentage, SLA, MTBF, MTTR

**Next Steps:**
- Define availability requirements
- Design for redundancy
- Plan disaster recovery
- Test regularly
- Monitor continuously

