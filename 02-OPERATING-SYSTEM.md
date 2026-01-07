# Operating System - Comprehensive Guide

## Table of Contents
1. [Processes and Threads](#processes-and-threads)
2. [Scheduling Algorithms](#scheduling-algorithms)
3. [Concurrency and Synchronization](#concurrency-and-synchronization)
4. [Memory Management](#memory-management)
5. [File Systems](#file-systems)
6. [System Calls](#system-calls)
7. [Caching](#caching)
8. [Sorting Algorithms](#sorting-algorithms)

---

## Processes and Threads

### What is a Process?

A **process** is like a complete, isolated workspace. Think of it as a separate office building where a company operates independently.

**Characteristics:**
- Has its own memory space (isolated from other processes)
- Contains one or more threads
- Has its own file descriptors, environment variables
- Can communicate with other processes via IPC (Inter-Process Communication)

**Visual Representation:**
```
Process A (Memory Space A)
├── Thread 1
├── Thread 2
└── Resources (files, network connections)

Process B (Memory Space B)
├── Thread 1
└── Resources

Process C (Memory Space C)
└── Thread 1
```

### What is a Thread?

A **thread** is like a worker within a process. Multiple threads in the same process share the same memory space.

**Analogy:**
- **Process** = Office building
- **Thread** = Worker in the building
- Multiple workers (threads) can work in the same building (process) and share resources

**Characteristics:**
- Shares memory with other threads in the same process
- Has its own stack (for local variables)
- Shares heap (for dynamic memory)
- Lightweight compared to processes

### Process vs Thread Comparison

| Aspect | Process | Thread |
|--------|---------|--------|
| **Memory** | Isolated | Shared |
| **Creation Cost** | High | Low |
| **Communication** | IPC (slow) | Shared memory (fast) |
| **Crash Impact** | Only that process | Can affect entire process |
| **Context Switch** | Expensive | Cheaper |

**Visual Comparison:**
```
Process Model:
[Process A Memory]  [Process B Memory]  [Process C Memory]
     ↓                   ↓                   ↓
  Isolated            Isolated            Isolated

Thread Model:
[Shared Process Memory]
  ├── Thread 1 Stack
  ├── Thread 2 Stack
  └── Thread 3 Stack
```

### Process States

A process goes through different states during its lifetime:

```
NEW → READY → RUNNING → WAITING → TERMINATED
      ↑         ↓          ↓
      └─────────┴──────────┘
```

**State Descriptions:**
1. **NEW**: Process is being created
2. **READY**: Process is ready to run, waiting for CPU
3. **RUNNING**: Process is executing on CPU
4. **WAITING**: Process is waiting for I/O or event
5. **TERMINATED**: Process has finished execution

**State Transitions:**
- **NEW → READY**: Process initialized, added to ready queue
- **READY → RUNNING**: Scheduler selects process, CPU allocated
- **RUNNING → READY**: Time slice expired or preempted
- **RUNNING → WAITING**: Process needs I/O or waits for event
- **WAITING → READY**: I/O complete or event occurred
- **RUNNING → TERMINATED**: Process finished execution

### Context Switching

**Context switching** is like switching between different tasks. The OS saves the current process's state and loads another process's state.

**What gets saved:**
- CPU registers (program counter, stack pointer, etc.)
- Process state
- Memory management information
- Open file descriptors

**Cost:**
- Time to save/restore state
- Cache misses (new process data not in cache)
- TLB (Translation Lookaside Buffer) flush

**Visual:**
```
Time →
Process A: [Running] [Saved] [Running] [Saved]
Process B: [Ready]  [Running] [Saved] [Running]
           ↑         ↑
      Context Switch
```

---

## Scheduling Algorithms

### What is Process Scheduling?

The CPU can only run one process at a time (on a single core). The **scheduler** decides which process runs next.

**Goal**: Maximize CPU utilization, minimize waiting time, ensure fairness

### First-Come-First-Served (FCFS)

**Principle**: First process to arrive gets CPU first.

**Example:**
```
Process | Arrival Time | Burst Time
--------|--------------|------------
P1      | 0            | 24
P2      | 1            | 3
P3      | 2            | 3

Timeline:
0──────24──────27──────30
P1      P2      P3

Average Waiting Time: (0 + 23 + 25) / 3 = 16
```

**Pros:**
- Simple to implement
- Fair (first come, first served)

**Cons:**
- **Convoy Effect**: Short process waits behind long process
- Poor for interactive systems

### Shortest Job First (SJF)

**Principle**: Process with shortest burst time runs first.

**Example:**
```
Process | Arrival Time | Burst Time
--------|--------------|------------
P1      | 0            | 6
P2      | 2            | 8
P3      | 4            | 7
P4      | 5            | 3

Timeline:
0──6──9──16──23
P1  P4  P3  P2
```

**Pros:**
- Minimizes average waiting time
- Optimal for minimizing turnaround time

**Cons:**
- Requires knowing burst time in advance (not practical)
- Can starve long processes

### Round Robin (RR)

**Principle**: Each process gets a time quantum (time slice). If not finished, it goes to back of queue.

**Example (Time Quantum = 4):**
```
Process | Burst Time
--------|------------
P1      | 24
P2      | 3
P3      | 3

Timeline:
0──4──7──10──14──18──22──26──30
P1  P2  P3   P1   P1   P1   P1   P1
```

**Pros:**
- Fair (every process gets CPU time)
- Good for interactive systems
- Prevents starvation

**Cons:**
- Higher average waiting time than SJF
- Performance depends on time quantum size

**Time Quantum Trade-off:**
- **Too small**: Too many context switches (overhead)
- **Too large**: Approaches FCFS (poor response time)
- **Optimal**: Usually 10-100ms

### Priority Scheduling

**Principle**: Each process has a priority. Higher priority runs first.

**Example:**
```
Process | Priority | Burst Time
--------|----------|------------
P1      | 3        | 10
P2      | 1        | 1  (highest priority)
P3      | 2        | 2
P4      | 4        | 5  (lowest priority)

Timeline:
0──1──3──13──18
P2  P3  P1   P4
```

**Problem**: **Starvation** - Low priority processes may never run.

**Solution**: **Aging** - Increase priority of waiting processes over time.

### Multilevel Queue Scheduling

Different types of processes in different queues:

```
┌─────────────────┐
│ System Queue    │ (Highest Priority)
│ (RR, q=8ms)     │
├─────────────────┤
│ Interactive     │
│ (RR, q=16ms)    │
├─────────────────┤
│ Batch           │
│ (FCFS)          │
└─────────────────┘
```

**Scheduling:**
1. Process in higher queue runs first
2. Within same queue, use queue's algorithm
3. Lower queue only runs if higher queues are empty

### Multilevel Feedback Queue

**Most complex, most practical**

**Features:**
- Multiple queues with different priorities
- Processes can move between queues
- Prevents starvation (aging)

**Example:**
```
Queue 0: RR, q=8ms    (Highest priority)
Queue 1: RR, q=16ms
Queue 2: FCFS         (Lowest priority)

Rules:
- New process starts in Queue 0
- If uses full time quantum → demoted to next queue
- If gives up CPU (I/O) → stays in same queue or promoted
```

**Visual:**
```
New Process → Queue 0 (RR, 8ms)
    ↓ (uses full quantum)
Queue 1 (RR, 16ms)
    ↓ (uses full quantum)
Queue 2 (FCFS)
```

---

## Concurrency and Synchronization

### Race Conditions

**Problem**: When multiple threads access shared data simultaneously, results depend on timing.

**Example:**
```python
# Thread 1 and Thread 2 both execute:
counter = counter + 1

# What can happen:
# Thread 1 reads counter = 5
# Thread 2 reads counter = 5
# Thread 1 writes counter = 6
# Thread 2 writes counter = 6
# Result: 6 (should be 7!)
```

**Visual Timeline:**
```
Time →
Thread 1: Read(5) ──────────── Write(6)
Thread 2:        Read(5) ──────────── Write(6)
Result: Lost update!
```

### Critical Section

**Critical Section**: Code that accesses shared resources and must not be executed by more than one thread at a time.

**Requirements:**
1. **Mutual Exclusion**: Only one thread in critical section
2. **Progress**: If no thread in critical section, allow one to enter
3. **Bounded Waiting**: Thread waiting to enter will eventually get chance

### Mutex (Mutual Exclusion)

**Mutex** is like a key to a room. Only one person can have the key at a time.

**How it works:**
```
Thread 1: lock(mutex) → Enter critical section → unlock(mutex)
Thread 2: lock(mutex) → Wait... → Enter critical section → unlock(mutex)
```

**Pseudocode:**
```python
mutex = Lock()

def increment():
    mutex.lock()
    counter = counter + 1  # Critical section
    mutex.unlock()
```

**Properties:**
- **Binary semaphore** (0 or 1)
- **Ownership**: Only thread that locked can unlock
- **Blocking**: If locked, thread waits

### Semaphore

**Semaphore** is like a parking lot with N spaces. N cars can park simultaneously.

**Types:**
- **Binary Semaphore**: Like mutex (0 or 1)
- **Counting Semaphore**: Can have value > 1

**Operations:**
- **wait()** (P): Decrement count, block if count = 0
- **signal()** (V): Increment count, wake waiting thread

**Example - Producer-Consumer:**
```python
empty = Semaphore(10)  # 10 empty slots
full = Semaphore(0)    # 0 full slots
mutex = Semaphore(1)   # For critical section

def producer():
    while True:
        item = produce()
        empty.wait()    # Wait for empty slot
        mutex.wait()
        buffer.add(item)
        mutex.signal()
        full.signal()   # Signal item available

def consumer():
    while True:
        full.wait()     # Wait for item
        mutex.wait()
        item = buffer.remove()
        mutex.signal()
        empty.signal()  # Signal slot empty
        consume(item)
```

### Spinlock

**Spinlock**: Thread continuously checks if lock is available (spins in a loop).

**Example:**
```python
while lock.is_locked():
    # Keep checking (spinning)
    pass
lock.acquire()
```

**Pros:**
- No context switch (fast for short waits)
- Good for multicore systems

**Cons:**
- Wastes CPU cycles
- Bad for long waits

**When to use:**
- Very short critical sections
- Multicore systems
- When context switch cost > spin cost

### Read-Write Locks

**Problem**: Multiple readers can read simultaneously, but only one writer.

**Solution**: Read-Write Lock

**Rules:**
- Multiple readers can hold lock simultaneously
- Writer needs exclusive access
- Writer blocks readers, readers block writer

**Example:**
```
Readers: R1, R2, R3
Writer: W1

Timeline:
R1 acquires read lock ──────────────── releases
R2 acquires read lock ──────────────── releases
R3 acquires read lock ──────────────── releases
W1 waits ──────────────────────────── acquires write lock ─── releases
```

**Use Cases:**
- Database systems (many reads, few writes)
- Caches
- Configuration data

### Deadlock

**Deadlock** occurs when two or more processes are waiting for each other to release resources.

**Example:**
```
Process A: Has Resource 1, needs Resource 2
Process B: Has Resource 2, needs Resource 1

Result: Both wait forever (deadlock)
```

**Visual:**
```
Process A ──(waiting for)──> Resource 2
    ↑                           ↓
    │                      Process B
    │                           ↓
Resource 1 <──(waiting for)─────┘
```

### Deadlock Conditions (Coffman Conditions)

All four must be true for deadlock:

1. **Mutual Exclusion**: Resources can't be shared
2. **Hold and Wait**: Process holds resource while waiting for another
3. **No Preemption**: Can't take resource from process
4. **Circular Wait**: Circular chain of processes waiting

### Deadlock Prevention

**Prevent one of the four conditions:**

1. **Prevent Mutual Exclusion**: Make resources shareable (not always possible)

2. **Prevent Hold and Wait**: 
   - Request all resources at once
   - Release all before requesting new ones

3. **Allow Preemption**: Take resource from process if needed

4. **Prevent Circular Wait**: 
   - Order resources (always request in same order)
   - Example: Always request Resource 1 before Resource 2

### Deadlock Avoidance

**Banker's Algorithm**: System checks if granting resource would lead to deadlock.

**Concept:**
- System knows maximum resources each process needs
- Before granting, checks if safe state exists
- Only grants if safe

**Safe State**: System can allocate resources to all processes in some order without deadlock.

### Deadlock Detection and Recovery

**Detection:**
- Build resource allocation graph
- Check for cycles
- If cycle exists → deadlock

**Recovery:**
1. **Process Termination**: Kill one or more processes
2. **Resource Preemption**: Take resource from process, give to another

---

## Memory Management

### Virtual Memory

**Problem**: Programs need more memory than physically available.

**Solution**: **Virtual Memory** - Illusion of more memory than physically exists.

**How it works:**
- Each process sees its own virtual address space
- OS maps virtual addresses to physical addresses
- Not all virtual memory needs to be in physical RAM

**Analogy**: Like a library catalog. You see all books (virtual), but not all are on shelves (physical). Some are in storage (disk).

### Virtual Address Space

**32-bit System:**
```
0x00000000 ──────────── 0xFFFFFFFF
    ↓                        ↓
  4 GB virtual address space
```

**Layout:**
```
High Address
┌─────────────┐
│   Stack     │ (grows downward)
├─────────────┤
│     ↓       │
│   (free)    │
│     ↑       │
├─────────────┤
│    Heap     │ (grows upward)
├─────────────┤
│    Data     │ (global variables)
├─────────────┤
│    Code     │ (program code)
Low Address
```

### Paging

**Paging** divides memory into fixed-size blocks called **pages**.

**Concept:**
- Virtual memory divided into pages (typically 4KB)
- Physical memory divided into frames (same size as pages)
- Page table maps virtual pages to physical frames

**Example:**
```
Virtual Address: 0x12345678
Page Number: 0x12345
Offset: 0x678

Page Table:
Page 0x12345 → Frame 0xABCD

Physical Address: 0xABCD678
```

**Visual:**
```
Virtual Memory          Physical Memory
┌──────────┐           ┌──────────┐
│ Page 0   │ ────────> │ Frame 5  │
│ Page 1   │ ────────> │ Frame 2  │
│ Page 2   │           │          │ (not in memory)
│ Page 3   │ ────────> │ Frame 8  │
└──────────┘           └──────────┘
```

### Page Fault

**Page Fault**: Process tries to access page not in physical memory.

**What happens:**
1. CPU generates page fault interrupt
2. OS checks if page is valid
3. If valid but not in memory:
   - Find free frame (or evict one)
   - Load page from disk
   - Update page table
   - Resume process

**Types:**
- **Minor Page Fault**: Page in memory but not in page table (shared memory)
- **Major Page Fault**: Page not in memory, need to load from disk

### Page Replacement Algorithms

When memory is full, which page to evict?

**1. FIFO (First In First Out)**
- Evict oldest page
- Simple but not optimal

**2. LRU (Least Recently Used)**
- Evict page not used for longest time
- Good performance, but expensive to implement

**3. Optimal**
- Evict page that won't be used for longest time
- Optimal but requires future knowledge (not practical)

**4. Clock Algorithm**
- Approximation of LRU
- More efficient than LRU

### Shared Memory

**Question**: Can 2 processes map to same physical address?

**Answer**: Yes! Through shared memory.

**How:**
- Processes map different virtual addresses to same physical frame
- Changes by one process visible to others
- Used for IPC (Inter-Process Communication)

**Example:**
```
Process A Virtual: 0x1000 → Physical: 0x5000
Process B Virtual: 0x2000 → Physical: 0x5000 (same!)

Both see same data.
```

### Heap and Stack

**Stack:**
- Stores local variables, function parameters, return addresses
- Automatic allocation/deallocation
- Fast (just move stack pointer)
- Limited size
- **LIFO** (Last In First Out)

**Heap:**
- Stores dynamically allocated memory
- Manual allocation/deallocation (malloc/free)
- Slower (need to find free block)
- Larger size
- Can fragment

**Visual:**
```
Memory Layout:
High Address
┌─────────────┐
│   Stack     │ ← Grows downward
│     ↓       │
│   (free)    │
│     ↑       │
│    Heap     │ ← Grows upward
└─────────────┘
Low Address
```

**Why stack for function calls?**
- Fast (just increment/decrement pointer)
- Automatic cleanup (when function returns)
- Supports recursion naturally

**Stack Frame:**
```
Function Call Stack:
┌─────────────────┐
│ main()          │
│  - local vars   │
│  - return addr  │
├─────────────────┤
│ function1()     │
│  - parameters   │
│  - local vars   │
│  - return addr  │
├─────────────────┤
│ function2()     │
│  - parameters   │
│  - local vars   │
│  - return addr  │
└─────────────────┘
```

### Stack Overflow

**Stack Overflow**: Stack grows too large, exceeds available memory.

**Causes:**
- Deep recursion
- Large local arrays
- Infinite recursion

**Example:**
```python
def infinite_recursion():
    infinite_recursion()  # Calls itself forever
    # Stack keeps growing → Stack Overflow
```

### Dynamic Memory Allocation

**Heap Allocation Process:**

1. **Request memory**: `malloc(size)`
2. **OS finds free block** (first fit, best fit, worst fit)
3. **Allocate block**, mark as used
4. **Return pointer** to allocated memory

**Deallocation:**
1. **Free memory**: `free(pointer)`
2. **Mark block as free**
3. **Merge adjacent free blocks** (coalescing)

**Problems:**
- **Fragmentation**: Free blocks scattered, can't use for large allocation
- **Memory Leaks**: Forgot to free memory

### Garbage Collection

**Garbage Collection**: Automatic memory management.

**How it works:**
1. **Mark**: Mark all reachable objects
2. **Sweep**: Free unmarked objects
3. **Compact**: Move objects to reduce fragmentation

**When triggered:**
- When heap is full
- Periodically
- On allocation request

**Types:**
- **Mark and Sweep**: Mark reachable, sweep unreachable
- **Copying**: Copy live objects to new space
- **Generational**: Different strategies for old/new objects

**Pros:**
- No memory leaks
- No manual memory management

**Cons:**
- Overhead (pauses, CPU usage)
- Less control

### Pointers

**Pointer**: Variable that stores memory address.

**Example:**
```c
int x = 10;
int *ptr = &x;  // ptr stores address of x

*ptr = 20;      // Change value at address
// Now x = 20
```

**Pass by Value vs Pass by Reference:**

**Pass by Value:**
```c
void increment(int x) {
    x = x + 1;  // Only local copy changes
}

int a = 5;
increment(a);
// a is still 5
```

**Pass by Reference:**
```c
void increment(int *x) {
    *x = *x + 1;  // Changes original
}

int a = 5;
increment(&a);
// a is now 6
```

### Global Variables

**Where are global variables stored?**

- **Initialized globals**: Data segment
- **Uninitialized globals**: BSS (Block Started by Symbol) segment
- **Constants**: Code segment or read-only data segment

---

## File Systems

### "Everything is a File" in Linux

**Philosophy**: In Linux, many things are represented as files:
- Regular files
- Directories
- Devices (mouse, keyboard, monitor)
- Sockets
- Pipes

**Why?**
- Unified interface
- Simple abstraction
- Easy to work with

### Device Communication

**How devices communicate:**

```
Application → System Call → Kernel → Device Driver → Hardware
```

**Example - Keyboard:**
```
You press key
    ↓
Keyboard hardware sends interrupt
    ↓
Kernel receives interrupt
    ↓
Device driver reads key code
    ↓
Available at /dev/input/event0 (as file)
    ↓
Application reads from file
```

### File Descriptors

**File Descriptor**: Integer that represents open file/device.

**Standard File Descriptors:**
- **0**: stdin (standard input)
- **1**: stdout (standard output)
- **2**: stderr (standard error)

**Example:**
```c
int fd = open("file.txt", O_RDONLY);
// fd is a file descriptor (e.g., 3)

read(fd, buffer, size);
close(fd);
```

**Visual:**
```
Process
├── File Descriptor Table
│   ├── 0 → stdin
│   ├── 1 → stdout
│   ├── 2 → stderr
│   └── 3 → file.txt
│
└── Open File Table (kernel)
    └── file.txt metadata
```

### Buffering

**Buffer**: Temporary storage area.

**Why needed?**
- **Speed mismatch**: Disk slow, CPU fast
- **Efficiency**: Write in chunks, not byte-by-byte
- **Reduce system calls**: Batch operations

**Types:**
- **Unbuffered**: Write immediately
- **Line buffered**: Buffer until newline
- **Fully buffered**: Buffer until full

**Example:**
```c
printf("Hello");  // Buffered, not printed yet
fflush(stdout);   // Force write buffer
// Now "Hello" appears
```

### Concurrent File Access

**What happens if 2 processes read/write same file?**

**Scenario 1: Both Reading**
- ✅ Safe, no problem

**Scenario 2: One Reading, One Writing**
- ⚠️ Reader might get inconsistent data
- Solution: File locking

**Scenario 3: Both Writing**
- ❌ Data corruption
- Solution: File locking, atomic operations

**File Locking:**
- **Shared Lock (Read Lock)**: Multiple readers allowed
- **Exclusive Lock (Write Lock)**: Only one writer, no readers

---

## System Calls

### What is a System Call?

**System Call (syscall)**: Interface between user programs and OS kernel.

**Why needed?**
- User programs can't directly access hardware
- Need kernel's help for privileged operations

**Common System Calls:**
- `open()`, `read()`, `write()`, `close()` - File operations
- `fork()`, `exec()`, `wait()` - Process management
- `socket()`, `bind()`, `listen()` - Networking
- `malloc()`, `brk()` - Memory management

### How to Make a System Call

**Steps:**
1. User program calls library function (e.g., `printf()`)
2. Library function prepares arguments
3. Library function triggers system call (special instruction)
4. CPU switches to kernel mode
5. Kernel executes system call
6. Return to user mode
7. Return value to user program

**Example:**
```c
// User code
printf("Hello");

// Library code (simplified)
write(1, "Hello", 5);  // System call

// Kernel executes
// Returns to user space
```

### CPU Mode Switch

**User Space vs Kernel Space:**

**User Space:**
- Normal program execution
- Limited privileges
- Can't access hardware directly

**Kernel Space:**
- OS code execution
- Full privileges
- Can access hardware

**Mode Switch:**
```
User Mode → System Call → Kernel Mode
              ↓
         Execute syscall
              ↓
Kernel Mode → Return → User Mode
```

**Cost:**
- Context switch overhead
- Mode switch overhead
- Cache/TLB misses

---

## Caching

### In-Memory Cache

**Cache**: Fast storage for frequently accessed data.

**Why use cache?**
- **Speed**: Memory (RAM) much faster than disk
- **Reduce load**: Less database queries
- **Cost**: RAM more expensive than disk

**Examples:**
- **Memcached**: Simple key-value cache
- **Redis**: Advanced data structures, persistence

### LRU Cache Implementation

**LRU (Least Recently Used)**: Evict least recently used item when cache full.

**Data Structures:**
- **HashMap**: O(1) lookup
- **Doubly Linked List**: O(1) insertion/deletion, maintains order

**Operations:**
- **Get**: Move to front (most recently used)
- **Put**: Add to front, evict from back if full

**Visual:**
```
Cache (size 3):
[Most Recent] → [Item 2] → [Item 1] → [Least Recent]
     ↑                              ↑
   Head                           Tail

After accessing Item 1:
[Most Recent] → [Item 1] → [Item 2] → [Item 3]
```

**Thread-Safe LRU:**
- Use locks (mutex) around operations
- Or use concurrent data structures

### Cache Stampede

**Problem**: Many requests for same missing cache key → all query database.

**Solutions:**

**1. Lock per Key**
```python
lock = Lock(key)
if not cache.has(key):
    with lock:
        if not cache.has(key):  # Double check
            value = db.query()
            cache.set(key, value)
return cache.get(key)
```

**2. Probabilistic Early Expiration**
- Refresh cache before expiration
- Reduces simultaneous misses

**3. Background Refresh**
- Refresh in background before expiration

---

## Sorting Algorithms

### Quicksort vs Merge Sort

**Quicksort:**
- **Average**: O(n log n)
- **Worst**: O(n²) - if pivot is always smallest/largest
- **Best**: O(n log n)
- **Space**: O(log n) - recursion stack
- **In-place**: Yes
- **Stable**: No

**Merge Sort:**
- **Average**: O(n log n)
- **Worst**: O(n log n)
- **Best**: O(n log n)
- **Space**: O(n) - needs extra array
- **In-place**: No
- **Stable**: Yes

### Which is Faster?

**Quicksort** is generally faster in practice because:
- Better cache locality
- Less overhead
- In-place (no extra memory)

**But**:
- Merge sort has guaranteed O(n log n)
- Merge sort is stable
- Merge sort better for linked lists

### Real-World Usage

**Quicksort:**
- C++ `std::sort()`
- Java `Arrays.sort()` (for primitives)
- General-purpose sorting

**Merge Sort:**
- Java `Collections.sort()` (stable sort needed)
- External sorting (sorting data that doesn't fit in memory)
- Stable sort requirements

**Hybrid Approaches:**
- Many implementations use hybrid (e.g., Timsort in Python)
- Use quicksort for large arrays
- Use insertion sort for small subarrays

---

## Summary

Operating systems manage resources (CPU, memory, I/O) and provide abstractions for applications. Understanding processes, threads, scheduling, memory management, and synchronization is crucial for building efficient backend systems.

**Key Takeaways:**
- Processes are isolated, threads share memory
- Schedulers decide which process runs
- Synchronization prevents race conditions
- Virtual memory provides illusion of large memory
- System calls bridge user and kernel space
- Caching improves performance
- Choose sorting algorithm based on requirements

