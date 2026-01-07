# Scheduling Algorithms Deep Dive - Complete Understanding

## Table of Contents
1. [What is Process Scheduling?](#what-is-process-scheduling)
2. [Scheduling Criteria - What Makes a Good Scheduler?](#scheduling-criteria---what-makes-a-good-scheduler)
3. [First-Come-First-Served (FCFS)](#first-come-first-served-fcfs)
4. [Shortest Job First (SJF)](#shortest-job-first-sjf)
5. [Round Robin (RR)](#round-robin-rr)
6. [Priority Scheduling](#priority-scheduling)
7. [Multilevel Queue Scheduling](#multilevel-queue-scheduling)
8. [Multilevel Feedback Queue](#multilevel-feedback-queue)
9. [Real-Time Scheduling](#real-time-scheduling)
10. [Scheduling in Modern Systems](#scheduling-in-modern-systems)

---

## What is Process Scheduling?

### The Problem

**CPU Can Only Run One Process at a Time:**
```
Multiple processes want CPU
CPU can only run one
Which one to run?
```

**Solution: Scheduler**
```
OS scheduler decides
Which process runs next
When to switch
```

### Scheduler Goals

**1. Fairness:**
```
All processes get CPU time
No starvation
```

**2. Efficiency:**
```
Maximize CPU utilization
Minimize idle time
```

**3. Response Time:**
```
Interactive processes respond quickly
Good user experience
```

**4. Throughput:**
```
Maximize processes completed per time
```

---

## Scheduling Criteria - What Makes a Good Scheduler?

### Metrics

**1. CPU Utilization:**
```
Percentage of time CPU is busy
Higher is better
```

**2. Throughput:**
```
Number of processes completed per time unit
Higher is better
```

**3. Turnaround Time:**
```
Time from submission to completion
Lower is better
```

**4. Waiting Time:**
```
Time process spends waiting in ready queue
Lower is better
```

**5. Response Time:**
```
Time from submission to first response
Lower is better (for interactive)
```

---

## First-Come-First-Served (FCFS)

### How It Works

**Principle:** First process to arrive runs first.

**Implementation:**
```
Ready queue: FIFO (First In First Out)
Process at front runs
When done, next process runs
```

### Example

**Processes:**
```
P1: Arrival 0, Burst 24
P2: Arrival 1, Burst 3
P3: Arrival 2, Burst 3
```

**Timeline:**
```
0──────24──────27──────30
P1      P2      P3

Waiting times:
P1: 0
P2: 23 (waited for P1)
P3: 25 (waited for P1 and P2)

Average: (0 + 23 + 25) / 3 = 16
```

### Pros and Cons

**Pros:**
- **Simple**: Easy to implement
- **Fair**: First come, first served

**Cons:**
- **Convoy effect**: Short process waits behind long process
- **Poor for interactive**: Long response time
- **No preemption**: Can't interrupt long process

---

## Shortest Job First (SJF)

### How It Works

**Principle:** Process with shortest burst time runs first.

**Implementation:**
```
Sort processes by burst time
Shortest runs first
```

### Example

**Processes:**
```
P1: Burst 6
P2: Burst 8
P3: Burst 7
P4: Burst 3
```

**Timeline:**
```
0──3──9──16──23
P4  P1  P3  P2

Waiting times:
P4: 0 (shortest)
P1: 3
P3: 9
P2: 16

Average: (0 + 3 + 9 + 16) / 4 = 7
```

### Pros and Cons

**Pros:**
- **Optimal**: Minimizes average waiting time
- **Efficient**: Short jobs finish quickly

**Cons:**
- **Requires knowledge**: Need to know burst time
- **Starvation**: Long jobs may never run
- **Not practical**: Can't know burst time in advance

### Preemptive SJF (Shortest Remaining Time First)

**How it works:**
```
When new process arrives
Compare with currently running
If new process shorter → Preempt
```

**Example:**
```
Time 0: P1 starts (burst 6)
Time 2: P2 arrives (burst 3)
  → P2 shorter → Preempt P1
Time 5: P2 finishes, P1 resumes
```

---

## Round Robin (RR)

### How It Works

**Principle:** Each process gets time quantum (time slice).

**Implementation:**
```
Process runs for time quantum
If not done → Move to back of queue
Next process runs
Cycle continues
```

### Example

**Time Quantum = 4:**
```
Processes: P1 (24), P2 (3), P3 (3)

Timeline:
0──4──7──10──14──18──22──26──30
P1  P2  P3   P1   P1   P1   P1   P1

Waiting times:
P1: 6 (waited for P2, P3, and own slices)
P2: 4
P3: 7

Average: (6 + 4 + 7) / 3 = 5.67
```

### Time Quantum Trade-off

**Small Quantum:**
```
More context switches
Better response time
Higher overhead
```

**Large Quantum:**
```
Fewer context switches
Worse response time
Lower overhead
Approaches FCFS
```

**Optimal:**
```
Usually 10-100ms
Balance response time and overhead
```

### Pros and Cons

**Pros:**
- **Fair**: Every process gets CPU time
- **Good response time**: Interactive processes respond quickly
- **No starvation**: All processes eventually run

**Cons:**
- **Higher waiting time**: Than SJF
- **Performance depends on quantum**: Must choose carefully

---

## Priority Scheduling

### How It Works

**Principle:** Each process has priority. Higher priority runs first.

**Implementation:**
```
Sort by priority
Highest priority runs first
```

### Example

**Processes:**
```
P1: Priority 3
P2: Priority 1 (highest)
P3: Priority 2
P4: Priority 4 (lowest)
```

**Timeline:**
```
0──1──3──13──18
P2  P3  P1   P4
```

### Priority Assignment

**Static:**
```
Priority assigned at creation
Doesn't change
```

**Dynamic:**
```
Priority changes over time
Aging: Increase priority of waiting processes
Prevents starvation
```

### Pros and Cons

**Pros:**
- **Flexible**: Can prioritize important processes
- **Adaptive**: Dynamic priority prevents starvation

**Cons:**
- **Starvation**: Low priority may never run (without aging)
- **Indefinite blocking**: If high priority processes keep arriving

---

## Multilevel Queue Scheduling

### How It Works

**Principle:** Different queues for different process types.

**Structure:**
```
Queue 0: System processes (highest priority)
Queue 1: Interactive processes
Queue 2: Batch processes (lowest priority)
```

**Scheduling:**
```
1. Process in higher queue runs first
2. Within queue, use queue's algorithm
3. Lower queues only run if higher empty
```

### Example

```
Queue 0 (System, RR, q=8ms): P1
Queue 1 (Interactive, RR, q=16ms): P2, P3
Queue 2 (Batch, FCFS): P4, P5

Scheduling:
P1 runs (Queue 0)
P2, P3 run (Queue 1, round robin)
P4, P5 run (Queue 2, only if Queue 0 and 1 empty)
```

### Pros and Cons

**Pros:**
- **Categorization**: Different treatment for different types
- **Flexible**: Different algorithms per queue

**Cons:**
- **Starvation**: Lower queues may starve
- **Rigid**: Process stuck in queue

---

## Multilevel Feedback Queue

### How It Works

**Principle:** Processes can move between queues based on behavior.

**Rules:**
```
1. New process starts in highest priority queue
2. If uses full time quantum → Demote to lower queue
3. If gives up CPU (I/O) → Stay or promote
4. Lower queues have longer time quanta
```

### Example

```
Queue 0: RR, q=8ms (highest priority)
Queue 1: RR, q=16ms
Queue 2: FCFS (lowest priority)

Process behavior:
- CPU-bound: Uses full quantum → Demoted
- I/O-bound: Gives up CPU → Stays or promoted
```

### Benefits

**1. Adapts to Process Behavior:**
```
I/O-bound: Stays in high priority (responsive)
CPU-bound: Moves to lower priority (efficient)
```

**2. Prevents Starvation:**
```
Aging: Increase priority over time
Eventually all processes run
```

**3. Good Performance:**
```
Interactive processes: Fast response
CPU-bound: Efficient execution
```

---

## Real-Time Scheduling

### Real-Time Systems

**Requirements:**
```
Tasks must complete by deadline
Missing deadline is failure
```

### Scheduling Algorithms

**1. Rate Monotonic:**
```
Higher frequency → Higher priority
Static priorities
```

**2. Earliest Deadline First (EDF):**
```
Earlier deadline → Higher priority
Dynamic priorities
Optimal for single processor
```

---

## Scheduling in Modern Systems

### Linux CFS (Completely Fair Scheduler)

**Principle:**
```
Fair share of CPU time
Virtual runtime tracking
```

### Windows Scheduler

**Features:**
```
Priority-based
Multiprocessor support
Real-time support
```

---

## Summary

Process scheduling is fundamental to operating systems. Understanding different algorithms, their trade-offs, and when to use each is essential for backend engineers.

**Key Takeaways:**
- Scheduler decides which process runs
- Different algorithms for different goals
- FCFS: Simple but can have convoy effect
- SJF: Optimal but requires knowledge
- Round Robin: Fair, good for interactive
- Priority: Flexible but can starve
- Multilevel: Categorize processes
- Multilevel Feedback: Adapts to behavior
- Choose algorithm based on system goals

**Next Steps:**
- Understand your OS's scheduler
- Monitor process scheduling
- Tune scheduler parameters if needed
- Consider scheduling in application design

