# Memory Management Deep Dive - Complete Understanding

## Table of Contents
1. [The Memory Problem - Why We Need Memory Management](#the-memory-problem---why-we-need-memory-management)
2. [Physical vs Virtual Memory - The Illusion](#physical-vs-virtual-memory---the-illusion)
3. [Virtual Memory - How It Works](#virtual-memory---how-it-works)
4. [Paging - Dividing Memory into Pages](#paging---dividing-memory-into-pages)
5. [Page Tables - Mapping Virtual to Physical](#page-tables---mapping-virtual-to-physical)
6. [Page Faults - When Memory is Missing](#page-faults---when-memory-is-missing)
7. [Page Replacement Algorithms - Which Page to Evict](#page-replacement-algorithms---which-page-to-evict)
8. [Heap and Stack - Two Types of Memory](#heap-and-stack---two-types-of-memory)
9. [Memory Allocation - How Programs Get Memory](#memory-allocation---how-programs-get-memory)
10. [Garbage Collection - Automatic Memory Management](#garbage-collection---automatic-memory-management)
11. [Memory Fragmentation - The Space Problem](#memory-fragmentation---the-space-problem)
12. [Shared Memory - Processes Sharing Data](#shared-memory---processes-sharing-data)

---

## The Memory Problem - Why We Need Memory Management

### The Fundamental Challenge

**Problem:**
- Programs need memory to run
- Physical RAM is limited (e.g., 16 GB)
- Multiple programs run simultaneously
- Each program thinks it has unlimited memory
- Programs must be isolated from each other

**Real-World Analogy:**
Imagine a library with 1000 books (physical memory), but:
- 10 people want to read (10 programs)
- Each person wants to read 500 books (each program needs 500 MB)
- Total need: 5000 books, but only 1000 available
- **Solution**: Virtual library system - each person gets a catalog showing all books, but actual books are shared and swapped

### What Memory Management Solves

**1. Isolation:**
```
Program A can't access Program B's memory
Prevents crashes and security issues
```

**2. Abstraction:**
```
Programs see virtual memory addresses
OS maps to physical addresses
Programs don't need to know physical layout
```

**3. Efficiency:**
```
Multiple programs share physical memory
Not all programs need all memory at once
Swap unused memory to disk
```

**4. Protection:**
```
Prevent programs from accessing kernel memory
Prevent unauthorized access
```

---

## Physical vs Virtual Memory - The Illusion

### Physical Memory (RAM)

**What it is:**
- Actual hardware (RAM chips)
- Limited size (e.g., 8 GB, 16 GB, 32 GB)
- Fast access (nanoseconds)
- Volatile (lost when power off)

**Characteristics:**
- **Fixed size**: Can't exceed installed RAM
- **Direct addressing**: Physical address = actual location
- **Shared**: All programs use same physical memory

**Visual:**
```
Physical Memory (16 GB):
[0x00000000] ──────────────────── [0x3FFFFFFF]
    ↓                                ↓
Program A data    Program B data    Program C data
```

### Virtual Memory (The Illusion)

**What it is:**
- Each process sees its own address space
- Appears to have more memory than physically available
- OS maps virtual addresses to physical addresses
- Can be larger than physical RAM

**Characteristics:**
- **Appears unlimited**: Each process sees full address space
- **Isolated**: Each process has its own virtual memory
- **Protected**: Can't access other processes' memory

**Visual:**
```
Process A Virtual Memory (4 GB):
[0x00000000] ──────────────────── [0xFFFFFFFF]
    ↓                                ↓
Process A thinks it has 4 GB

Process B Virtual Memory (4 GB):
[0x00000000] ──────────────────── [0xFFFFFFFF]
    ↓                                ↓
Process B thinks it has 4 GB

Physical Memory (16 GB):
[Actual 16 GB shared by all processes]
```

### Why Virtual Memory?

**Benefits:**

**1. Program Simplicity:**
```
Programs don't need to manage physical memory
Just use virtual addresses
OS handles mapping
```

**2. Security:**
```
Process A can't access Process B's memory
Virtual addresses are isolated
```

**3. Flexibility:**
```
Memory can be moved in physical RAM
Program doesn't know or care
OS can optimize layout
```

**4. Overcommitment:**
```
10 programs, each needs 4 GB
Total: 40 GB virtual
Physical: 16 GB
→ Works! (Not all memory used at once)
```

### The Mapping

**How Virtual → Physical:**
```
Program uses: Virtual address 0x1000
OS translates: → Physical address 0x5000
CPU accesses: Physical address 0x5000
```

**Process:**
1. Program accesses virtual address
2. CPU generates page fault (if not in memory) or uses TLB
3. OS looks up in page table
4. OS translates to physical address
5. CPU accesses physical memory

---

## Virtual Memory - How It Works

### Virtual Address Space Layout

**32-bit System (4 GB virtual address space):**

```
High Address (0xFFFFFFFF)
┌─────────────────────────┐
│   Kernel Space          │  (1 GB, protected)
│   (OS code, drivers)    │
├─────────────────────────┤
│   Stack                 │  (grows downward)
│   (local variables,     │
│    function calls)      │
│         ↓               │
│   (free space)          │
│         ↑               │
│   Heap                  │  (grows upward)
│   (dynamic memory)      │
├─────────────────────────┤
│   Data Segment          │  (global variables)
├─────────────────────────┤
│   Code Segment          │  (program code)
Low Address (0x00000000)
```

**64-bit System (Much larger virtual address space):**
- Typically 48-bit virtual addresses (256 TB)
- Even more space for programs

### Why This Layout?

**Stack at Top, Growing Down:**
- **Historical**: Easier to implement
- **Collision detection**: Stack and heap grow toward each other
- **Efficient**: Simple pointer arithmetic

**Heap at Bottom, Growing Up:**
- **Dynamic allocation**: Can grow as needed
- **Flexible**: Allocate blocks of any size

**Code at Bottom:**
- **Fixed**: Code doesn't change size
- **Read-only**: Usually protected from modification

### Address Space Example

**C Program:**
```c
int global_var = 10;        // Data segment
static int static_var = 20; // Data segment

int main() {
    int local_var = 30;     // Stack
    int *ptr = malloc(100); // Heap
    return 0;
}
```

**Memory Layout:**
```
High Address
┌─────────────┐
│   Stack     │
│   local_var │ ← 30
│   ptr       │ ← pointer to heap
├─────────────┤
│   Heap      │
│   [100 bytes]│ ← malloc'd memory
├─────────────┤
│   Data      │
│   global_var│ ← 10
│   static_var│ ← 20
├─────────────┤
│   Code      │
│   main()    │
Low Address
```

---

## Paging - Dividing Memory into Pages

### What is Paging?

**Paging** divides memory into fixed-size blocks called **pages** (virtual) and **frames** (physical).

**Analogy:**
- **Book**: Memory
- **Pages**: Fixed-size blocks (e.g., 4 KB each)
- **Page numbers**: Virtual page numbers
- **Physical location**: Which shelf the page is on

### Why Fixed-Size Pages?

**Benefits:**
- **Simple management**: All pages same size
- **Easy allocation**: Just find free frame
- **Efficient**: No external fragmentation
- **Hardware support**: CPU has page size (usually 4 KB)

**Problems with Variable Sizes:**
- **Fragmentation**: Small gaps between blocks
- **Complex allocation**: Must find exact size
- **Inefficient**: Hard to manage

### Page Size

**Common Sizes:**
- **4 KB**: Most common (x86, x86-64)
- **8 KB**: Some systems
- **2 MB**: Large pages (for performance)
- **1 GB**: Huge pages (for very large data)

**Trade-offs:**

**Small Pages (4 KB):**
- **Pros**: Less wasted space, fine-grained control
- **Cons**: More pages to manage, larger page tables

**Large Pages (2 MB):**
- **Pros**: Fewer pages, smaller page tables, better TLB coverage
- **Cons**: More wasted space if not fully used

### How Paging Works

**Step 1: Divide Virtual Memory into Pages**
```
Virtual Address Space (4 GB):
Page 0:  0x00000000 - 0x00000FFF (4 KB)
Page 1:  0x00001000 - 0x00001FFF (4 KB)
Page 2:  0x00002000 - 0x00002FFF (4 KB)
...
Page 1048575: 0xFFFFF000 - 0xFFFFFFFF (4 KB)
```

**Step 2: Divide Physical Memory into Frames**
```
Physical Memory (16 GB):
Frame 0:  0x00000000 - 0x00000FFF (4 KB)
Frame 1:  0x00001000 - 0x00001FFF (4 KB)
...
Frame 4194303: 0x3FFFFFFF - 0x3FFFFFFF (4 KB)
```

**Step 3: Map Pages to Frames**
```
Page 0 → Frame 100
Page 1 → Frame 250
Page 2 → Not in memory (on disk)
Page 3 → Frame 50
...
```

### Address Translation

**Virtual Address Structure:**
```
Virtual Address (32 bits):
[Page Number (20 bits)] [Offset (12 bits)]
     ↓                        ↓
  Which page            Position in page
```

**Example:**
```
Virtual Address: 0x00123456

Breakdown:
  0x00123 = Page number (0x123 = 291)
  0x456 = Offset (1110 bytes into page)

Translation:
  Page 291 → Frame 500 (from page table)
  Physical Address = Frame 500 + Offset
                  = 0x00050000 + 0x456
                  = 0x00050456
```

**Visual:**
```
Virtual Address: 0x00123456
    │
    ├─ Page Number: 0x123 (291)
    │     │
    │     └─→ Page Table
    │           │
    │           └─→ Frame: 500
    │
    └─ Offset: 0x456
          │
          └─→ Physical Address: 0x00050456
```

---

## Page Tables - Mapping Virtual to Physical

### What is a Page Table?

**Page Table**: Data structure that maps virtual pages to physical frames.

**Structure:**
```
Page Table Entry (PTE):
┌─────────────────────────────────────┐
│ Frame Number │ Flags │ Other Info   │
└─────────────────────────────────────┘
```

**Example:**
```
Page Table:
Page 0 → Frame 100, Present=1, Read/Write=1
Page 1 → Frame 250, Present=1, Read/Write=0
Page 2 → (Not in memory), Present=0
Page 3 → Frame 50, Present=1, Read/Write=1
```

### Page Table Entry (PTE) Flags

**Common Flags:**

**1. Present Bit:**
- **1**: Page is in physical memory
- **0**: Page is on disk (page fault)

**2. Read/Write Bit:**
- **1**: Page can be written
- **0**: Page is read-only

**3. User/Supervisor Bit:**
- **1**: User mode can access
- **0**: Only kernel can access

**4. Accessed Bit:**
- **1**: Page has been accessed
- **0**: Page not accessed recently
- Used for page replacement

**5. Dirty Bit:**
- **1**: Page has been modified
- **0**: Page is unchanged
- Used to decide if page needs to be written to disk

### Page Table Size Problem

**32-bit System:**
```
Virtual address space: 4 GB
Page size: 4 KB
Number of pages: 4 GB / 4 KB = 1,048,576 pages

Each PTE: 4 bytes
Page table size: 1,048,576 × 4 bytes = 4 MB per process!

10 processes = 40 MB just for page tables!
```

**Problem:**
- Each process needs its own page table
- Page tables are large
- Wastes memory

**Solutions:**

**1. Multi-Level Page Tables:**
```
Instead of one huge table, use hierarchy:
Level 1: Page directory (points to page tables)
Level 2: Page tables (points to frames)
```

**2. Inverted Page Tables:**
```
One table for all processes
Entry per physical frame
Slower lookup, but smaller
```

### Multi-Level Page Tables

**Two-Level Example (x86):**

**Structure:**
```
Page Directory (1024 entries)
    ↓
Page Tables (1024 entries each)
    ↓
Physical Frames
```

**Address Translation:**
```
Virtual Address: 0x00123456
    │
    ├─ Page Directory Index: 0x0 (bits 22-31)
    │     │
    │     └─→ Page Directory Entry
    │           │
    │           └─→ Points to Page Table
    │
    ├─ Page Table Index: 0x123 (bits 12-21)
    │     │
    │     └─→ Page Table Entry
    │           │
    │           └─→ Frame Number: 500
    │
    └─ Offset: 0x456 (bits 0-11)
          │
          └─→ Physical Address: Frame 500 + 0x456
```

**Benefits:**
- **Sparse**: Only allocate page tables for used pages
- **Smaller**: Most processes don't use all 4 GB
- **Efficient**: Only 2 memory accesses (directory + table)

**Example:**
```
Process uses 100 MB:
  Without multi-level: 4 MB page table
  With multi-level: ~100 KB (only used pages)
```

### Translation Lookaside Buffer (TLB)

**Problem:**
- Page table lookup requires memory access
- Every memory access needs translation
- **Slow!**

**Solution: TLB (Cache for Page Table)**
- **Hardware cache** of recent translations
- **Very fast**: Accessed in 1 CPU cycle
- **Small**: 64-512 entries

**How it works:**
```
1. CPU needs virtual address
2. Check TLB first
3. If found (TLB hit) → Use cached translation
4. If not found (TLB miss) → Look up page table, update TLB
```

**TLB Hit Rate:**
- **Typical**: 95-99% hit rate
- **Very important**: TLB misses are expensive

**Visual:**
```
Memory Access:
Virtual Address
    │
    ├─→ TLB (fast, ~1 cycle)
    │     │
    │     ├─ Hit (95-99%) → Physical Address (fast!)
    │     │
    │     └─ Miss (1-5%) → Page Table (slow, ~100 cycles)
    │                           │
    │                           └─→ Physical Address
    │
    └─→ Access Physical Memory
```

---

## Page Faults - When Memory is Missing

### What is a Page Fault?

**Page Fault**: CPU exception when program accesses virtual page that's not in physical memory.

**Causes:**
1. **Page not loaded**: Page is on disk
2. **Invalid access**: Page doesn't exist
3. **Permission violation**: Trying to write read-only page

### Page Fault Types

**1. Minor Page Fault (Soft Fault):**
- Page is in memory, but not in page table
- **Common**: Shared memory, copy-on-write
- **Fast**: Just update page table

**2. Major Page Fault (Hard Fault):**
- Page is not in memory (on disk)
- **Slow**: Must load from disk
- **Expensive**: Disk I/O is slow (milliseconds)

**3. Invalid Page Fault:**
- Page doesn't exist
- **Error**: Segmentation fault
- **Result**: Program crashes

### Page Fault Handling Process

**Step-by-Step:**

**1. CPU Generates Page Fault:**
```
Program accesses virtual address 0x00123456
CPU looks up in page table
Page table entry: Present=0 (not in memory)
CPU generates page fault interrupt
```

**2. OS Page Fault Handler:**
```
OS receives interrupt
Saves process state
Checks page table entry
```

**3. Determine Fault Type:**
```
If Present=0 and Valid:
  → Major page fault (load from disk)
If Present=0 and Invalid:
  → Invalid access (kill process)
If Present=1 but permission error:
  → Permission violation (kill process)
```

**4. Handle Major Page Fault:**
```
a. Find free frame (or evict one)
b. Load page from disk into frame
c. Update page table (Present=1, Frame=X)
d. Resume process
```

**5. Resume Execution:**
```
Process continues from same instruction
Now page is in memory
Access succeeds
```

### Page Fault Example

**Scenario:**
```
Process accesses: 0x00123456
Page table: Page 291, Present=0
```

**What Happens:**
```
1. CPU: "Page 291 not in memory" → Page fault
2. OS: "Handle page fault"
3. OS: "Find free frame" → Frame 500
4. OS: "Load page 291 from disk to frame 500"
5. OS: "Update page table: Page 291 → Frame 500, Present=1"
6. OS: "Resume process"
7. CPU: "Retry access" → Success!
```

**Timeline:**
```
Time →
Access 0x00123456
    ↓
Page Fault (interrupt)
    ↓
OS Handler (save state)
    ↓
Load from Disk (~10 ms)
    ↓
Update Page Table
    ↓
Resume Process
    ↓
Access Succeeds
```

### Demand Paging

**Concept:**
- **Don't load all pages at once**
- **Load pages on demand** (when accessed)
- **Lazy loading**: Only load what's needed

**Benefits:**
- **Faster startup**: Program starts immediately
- **Less memory**: Only used pages in memory
- **Efficient**: Don't waste memory on unused code

**Example:**
```
Program size: 100 MB
Program uses: 10 MB initially
Memory used: 10 MB (not 100 MB)
→ Much more efficient!
```

---

## Page Replacement Algorithms - Which Page to Evict

### The Problem

**Scenario:**
- Physical memory is full
- New page needs to be loaded
- **Which page to evict?**

**Goal:**
- Evict page that won't be needed soon
- Minimize page faults
- Keep frequently used pages

### Optimal Algorithm (Theoretical)

**Strategy:**
- Evict page that will be used **furthest in the future**
- **Optimal**: Minimizes page faults
- **Problem**: Requires knowing the future (not practical)

**Use:**
- **Benchmark**: Compare other algorithms to optimal
- **Reference**: Best possible performance

### FIFO (First In First Out)

**Strategy:**
- Evict **oldest** page (first loaded)

**Implementation:**
```
Queue of pages in order loaded:
[Page A] → [Page B] → [Page C] → [Page D]

Evict: Page A (oldest)
```

**Pros:**
- Simple to implement
- Fair (every page gets same time)

**Cons:**
- **Belady's Anomaly**: More frames can cause more faults
- Doesn't consider usage

**Example:**
```
Pages accessed: A, B, C, D, A, B, E
Frames: 3

FIFO:
Load A → [A]
Load B → [A, B]
Load C → [A, B, C]
Load D → [B, C, D] (evict A)
Access A → Fault! Load A → [C, D, A] (evict B)
Access B → Fault! Load B → [D, A, B] (evict C)
Load E → [A, B, E] (evict D)

Total faults: 7
```

### LRU (Least Recently Used)

**Strategy:**
- Evict page **not used for longest time**

**Implementation:**
```
Track last access time for each page:
Page A: accessed 10ms ago
Page B: accessed 50ms ago
Page C: accessed 100ms ago

Evict: Page C (least recently used)
```

**Pros:**
- **Good performance**: Close to optimal
- **Intuitive**: Recent pages likely to be used again

**Cons:**
- **Expensive**: Must track access times
- **Hardware support**: Needs special hardware

**Example:**
```
Pages accessed: A, B, C, D, A, B, E
Frames: 3

LRU:
Load A → [A]
Load B → [A, B]
Load C → [A, B, C]
Load D → [B, C, D] (evict A - least recently used)
Access A → Fault! Load A → [C, D, A] (evict B)
Access B → Fault! Load B → [D, A, B] (evict C)
Load E → [A, B, E] (evict D)

Total faults: 7
```

**Implementation Options:**

**1. Timestamp:**
```
Each page: Last access time
On access: Update timestamp
On evict: Find oldest timestamp
```

**2. Linked List:**
```
Move accessed page to front
Evict page from back
```

**3. Counter:**
```
Global counter increments
Page stores counter value on access
Evict page with smallest counter
```

### Clock Algorithm (Approximation of LRU)

**Strategy:**
- **Approximate LRU** without full tracking
- **Circular buffer** with reference bit

**How it works:**
```
Pages in circular list:
[A] → [B] → [C] → [D] → [A] → ...

Each page has reference bit:
  1 = Recently accessed
  0 = Not recently accessed

Clock hand moves:
  If reference bit = 1:
    Set to 0, move to next
  If reference bit = 0:
    Evict this page
```

**Example:**
```
Pages: [A(1)] → [B(1)] → [C(0)] → [D(1)]
Clock hand at C

Need to evict:
  C has bit=0 → Evict C
```

**Benefits:**
- **Cheaper than LRU**: Only one bit per page
- **Good performance**: Close to LRU
- **Hardware friendly**: Simple to implement

### Working Set Algorithm

**Strategy:**
- Keep pages that were accessed in **recent time window**
- Evict pages outside working set

**Concept:**
```
Working Set = Pages accessed in last T seconds
Example: T = 100ms

Pages accessed in last 100ms: A, B, C
→ Keep A, B, C
→ Evict others
```

**Benefits:**
- **Adaptive**: Adjusts to program behavior
- **Efficient**: Keeps actively used pages

---

## Heap and Stack - Two Types of Memory

### Stack - Automatic Memory

**What is Stack?**
- **LIFO** (Last In First Out) data structure
- Stores **local variables**, function parameters, return addresses
- **Automatic**: Allocated/deallocated automatically
- **Fast**: Just move stack pointer

**Characteristics:**
- **Fixed size**: Usually 1-8 MB per thread
- **Fast allocation**: O(1) - just increment pointer
- **Automatic cleanup**: When function returns
- **Limited scope**: Only accessible in current function

**Visual:**
```
Stack (grows downward):
High Address
┌─────────────┐
│ function3() │ ← Top of stack
│   local_var │
│   param     │
├─────────────┤
│ function2() │
│   local_var │
│   param     │
├─────────────┤
│ function1() │
│   local_var │
│   param     │
├─────────────┤
│   main()    │
Low Address
```

### Stack Frame Structure

**What's in a Stack Frame:**
```
┌─────────────────────┐
│ Return Address      │ ← Where to return
├─────────────────────┤
│ Previous Frame Ptr │ ← Link to previous frame
├─────────────────────┤
│ Local Variables     │ ← Function's local data
├─────────────────────┤
│ Parameters          │ ← Function arguments
└─────────────────────┘
```

**Example:**
```c
int add(int a, int b) {
    int sum = a + b;
    return sum;
}

int main() {
    int result = add(3, 5);
    return 0;
}
```

**Stack Frames:**
```
main():
┌─────────────┐
│ result      │
└─────────────┘

add(3, 5):
┌─────────────┐
│ sum         │
├─────────────┤
│ a = 3       │
│ b = 5       │
├─────────────┤
│ return addr │ → back to main
└─────────────┘
```

### Why Stack for Function Calls?

**Benefits:**

**1. Automatic Cleanup:**
```
Function returns → Stack frame popped
All local variables automatically freed
No memory leaks
```

**2. Supports Recursion:**
```
Each recursive call gets its own stack frame
Natural nesting
```

**3. Fast:**
```
Allocation: Just move pointer
Deallocation: Just move pointer back
O(1) operations
```

**4. Cache Friendly:**
```
Stack is contiguous
Good for CPU cache
```

### Stack Overflow

**What is it?**
- Stack grows too large
- Exceeds stack size limit
- **Crashes program**

**Causes:**

**1. Deep Recursion:**
```c
void infinite_recursion() {
    infinite_recursion();  // Calls itself forever
    // Stack keeps growing → Stack overflow!
}
```

**2. Large Local Arrays:**
```c
void function() {
    int huge_array[1000000];  // 4 MB on stack
    // Might exceed stack size
}
```

**3. Infinite Function Calls:**
```
Function A calls B
Function B calls A
Function A calls B
... (infinite loop)
Stack keeps growing
```

**Prevention:**
- Use heap for large data
- Limit recursion depth
- Increase stack size (if possible)

### Heap - Dynamic Memory

**What is Heap?**
- **Dynamic memory** allocation
- **Manual**: Programmer allocates/deallocates
- **Flexible**: Any size, any time
- **Slower**: Must find free block

**Characteristics:**
- **Large size**: Can use most of virtual memory
- **Flexible**: Allocate any size
- **Manual management**: Must free memory
- **Can fragment**: Free blocks scattered

**Visual:**
```
Heap (grows upward):
Low Address
┌─────────────┐
│   Allocated │
├─────────────┤
│   Free      │
├─────────────┤
│   Allocated │
├─────────────┤
│   Free      │
├─────────────┤
│   Allocated │
High Address
```

### Heap Allocation Process

**Request Memory:**
```c
int *ptr = malloc(100);  // Allocate 100 bytes
```

**What Happens:**
```
1. OS finds free block ≥ 100 bytes
2. Allocate block
3. Mark as used
4. Return pointer
```

**Allocation Strategies:**

**1. First Fit:**
```
Find first free block that fits
Fast, but can fragment
```

**2. Best Fit:**
```
Find smallest free block that fits
Less fragmentation, but slower
```

**3. Worst Fit:**
```
Find largest free block
Leave large free blocks
Rarely used
```

### Heap Deallocation

**Free Memory:**
```c
free(ptr);  // Deallocate
```

**What Happens:**
```
1. Mark block as free
2. Merge adjacent free blocks (coalescing)
3. Add to free list
```

**Coalescing:**
```
Before:
[Used][Free][Free][Used]

After free:
[Used][Free (merged)][Used]
```

### Heap vs Stack Comparison

| Aspect | Stack | Heap |
|--------|-------|------|
| **Size** | Fixed (1-8 MB) | Large (GBs) |
| **Speed** | Very fast | Slower |
| **Management** | Automatic | Manual |
| **Scope** | Function | Global |
| **Fragmentation** | No | Yes |
| **Allocation** | O(1) | O(n) worst case |
| **Use Case** | Local variables | Dynamic data |

### When to Use Stack vs Heap

**Use Stack:**
- Small, fixed-size data
- Local variables
- Function parameters
- Short-lived data

**Use Heap:**
- Large data
- Dynamic size
- Long-lived data
- Shared between functions

---

## Memory Allocation - How Programs Get Memory

### Static Allocation

**What it is:**
- Memory allocated at **compile time**
- Size known before program runs
- **Fixed location**

**Examples:**
```c
int global_var = 10;           // Static (data segment)
static int static_var = 20;    // Static (data segment)
int array[100];                // Static (data segment)
```

**Characteristics:**
- **Fast**: No runtime allocation
- **Fixed size**: Can't change
- **Persistent**: Exists for program lifetime

### Dynamic Allocation

**What it is:**
- Memory allocated at **runtime**
- Size determined when program runs
- **Flexible**

**C Example:**
```c
int *ptr = malloc(100 * sizeof(int));  // Allocate 100 integers
// Use ptr
free(ptr);  // Deallocate
```

**C++ Example:**
```cpp
int *ptr = new int[100];  // Allocate
delete[] ptr;              // Deallocate
```

**Characteristics:**
- **Flexible**: Size determined at runtime
- **Manual**: Must free memory
- **Slower**: Runtime allocation overhead

### Memory Allocation Functions

**C:**
- `malloc(size)`: Allocate memory
- `calloc(n, size)`: Allocate and zero
- `realloc(ptr, size)`: Resize allocation
- `free(ptr)`: Deallocate

**C++:**
- `new`: Allocate (calls constructor)
- `delete`: Deallocate (calls destructor)
- `new[]`: Allocate array
- `delete[]`: Deallocate array

### Memory Leaks

**What is Memory Leak?**
- Allocate memory but never free it
- Memory becomes unusable
- **Program uses more and more memory**

**Example:**
```c
void function() {
    int *ptr = malloc(100);
    // Forgot to free(ptr)
    // Memory leak!
}
```

**Consequences:**
- **Memory exhaustion**: Program runs out of memory
- **Slowdown**: System becomes slow
- **Crash**: Program or system crashes

**Prevention:**
- Always free allocated memory
- Use smart pointers (C++)
- Use garbage collection (Java, Python)
- Use memory profilers

---

## Garbage Collection - Automatic Memory Management

### What is Garbage Collection?

**Garbage Collection**: Automatic memory management - system automatically frees memory that's no longer used.

**Languages with GC:**
- Java
- Python
- JavaScript
- Go
- C# (.NET)

**Languages without GC:**
- C
- C++ (but can use smart pointers)

### Why Garbage Collection?

**Benefits:**
- **No memory leaks**: Automatic cleanup
- **Simpler code**: Don't worry about freeing
- **Fewer bugs**: Can't forget to free

**Costs:**
- **Overhead**: GC uses CPU time
- **Pauses**: GC can pause program
- **Less control**: Can't control when memory is freed

### How Garbage Collection Works

**Basic Concept:**
```
1. Find all objects that are still reachable
2. Everything else is garbage
3. Free garbage
```

**Reachability:**
```
Root objects (global variables, stack variables)
    ↓
Point to other objects
    ↓
Those objects point to more objects
    ↓
All reachable objects = Keep
All unreachable objects = Garbage
```

### Mark and Sweep Algorithm

**Phase 1: Mark**
```
1. Start from root objects
2. Mark all reachable objects
3. Traverse all pointers
```

**Phase 2: Sweep**
```
1. Scan all objects
2. If not marked → Free it
3. Clear marks
```

**Visual:**
```
Before GC:
Root → A → B → C
      ↓
      D (unreachable - garbage)

Mark Phase:
Root → A✓ → B✓ → C✓
      ↓
      D (not marked)

Sweep Phase:
Free D
Keep A, B, C
```

### Copying Garbage Collection

**Concept:**
- Divide heap into two spaces
- Copy live objects to other space
- Free entire old space

**Process:**
```
From Space: [A][B][C][D] (garbage)
To Space:   [empty]

Copy live objects:
To Space:   [A][B][C]

Free From Space entirely
```

**Benefits:**
- **No fragmentation**: Objects compacted
- **Fast allocation**: Just increment pointer

**Costs:**
- **Memory overhead**: Need 2x memory
- **Copying cost**: Must copy live objects

### Generational Garbage Collection

**Observation:**
- **Most objects die young**
- **Old objects likely to stay**

**Strategy:**
- **Young generation**: New objects, collect frequently
- **Old generation**: Survived objects, collect rarely

**Process:**
```
New objects → Young generation
If survive collection → Promote to old generation
Collect young generation frequently
Collect old generation rarely
```

**Benefits:**
- **Efficient**: Focus on young objects
- **Fast**: Most collections are small

### GC Pauses

**Problem:**
- GC must stop program to collect
- **Pause time**: Program freezes
- **Latency**: Affects response time

**Solutions:**

**1. Incremental GC:**
```
Collect in small increments
Shorter pauses
More frequent pauses
```

**2. Concurrent GC:**
```
GC runs in parallel with program
Minimal pauses
More complex
```

**3. Tuning:**
```
Adjust GC parameters
Balance pause time vs throughput
```

---

## Memory Fragmentation - The Space Problem

### What is Fragmentation?

**Fragmentation**: Free memory is broken into small, non-contiguous pieces.

**Types:**

**1. External Fragmentation:**
```
Free memory exists, but not contiguous
Can't allocate large block
```

**Example:**
```
Memory: [Used][Free 100][Used][Free 200][Used][Free 50]
Need: 150 bytes
Available: 350 bytes total, but not contiguous
→ Can't allocate!
```

**2. Internal Fragmentation:**
```
Allocated block larger than needed
Wasted space inside block
```

**Example:**
```
Need: 10 bytes
Allocate: 16 bytes (minimum block size)
Waste: 6 bytes
```

### Fragmentation Example

**Scenario:**
```
Allocate: 100, 200, 100, 200 bytes
Free: 200 (middle), 100 (first)
```

**Result:**
```
[Free 100][Used 200][Free 200][Used 100][Used 200]
```

**Problem:**
- Need 250 bytes
- Have 300 bytes free, but not contiguous
- **Can't allocate!**

### Solutions to Fragmentation

**1. Compaction:**
```
Move objects to make free space contiguous
[Free 100][Used 200][Free 200][Used 100]
    ↓
[Used 200][Used 100][Free 300]
```

**2. Buddy System:**
```
Divide memory into power-of-2 sizes
Merge adjacent blocks of same size
```

**3. Slab Allocation:**
```
Pre-allocate blocks of fixed sizes
No fragmentation for those sizes
```

---

## Shared Memory - Processes Sharing Data

### Can Two Processes Map to Same Physical Address?

**Answer: Yes! Through shared memory.**

### How Shared Memory Works

**Process:**
```
1. Process A requests shared memory
2. OS allocates physical frame
3. Process A maps virtual page to physical frame
4. Process B maps different virtual page to same physical frame
5. Both processes see same data
```

**Visual:**
```
Process A Virtual: 0x1000 → Physical: 0x5000
Process B Virtual: 0x2000 → Physical: 0x5000 (same!)

Both see same data at 0x5000
```

### Use Cases

**1. Inter-Process Communication (IPC):**
```
Process A writes data
Process B reads data
Fast communication
```

**2. Shared Libraries:**
```
Multiple processes use same library code
Share physical memory
Save memory
```

**3. Database Shared Buffers:**
```
Multiple database connections
Share buffer cache
Efficient
```

### Copy-on-Write (COW)

**Concept:**
- Initially, processes share same physical page
- If one process writes, **copy page** before writing
- Other process still uses original

**Benefits:**
- **Save memory**: Share until modified
- **Fast**: No copy until needed

**Example:**
```
Process A and B share page
Process A reads → No copy (still shared)
Process B writes → Copy page, then write
Now: Separate pages
```

---

## Summary

Memory management is fundamental to understanding how computers work. Virtual memory, paging, heap, stack, and garbage collection are essential concepts for backend engineers.

**Key Takeaways:**
- Virtual memory provides isolation and abstraction
- Paging divides memory into fixed-size pages
- Page tables map virtual to physical addresses
- Page faults load pages from disk
- Stack is fast, automatic, limited size
- Heap is flexible, manual, larger size
- Garbage collection automates memory management
- Fragmentation is a common problem
- Shared memory allows processes to share data

**Next Steps:**
- Understand your language's memory model
- Learn to use memory profilers
- Practice with manual memory management (C/C++)
- Understand GC tuning (Java, Go, etc.)
- Monitor memory usage in production

