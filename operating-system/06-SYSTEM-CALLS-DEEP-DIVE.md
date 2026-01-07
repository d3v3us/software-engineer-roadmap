# System Calls Deep Dive - Complete Understanding

## Table of Contents
1. [What are System Calls?](#what-are-system-calls)
2. [Why Do We Need System Calls?](#why-do-we-need-system-calls)
3. [User Space vs Kernel Space](#user-space-vs-kernel-space)
4. [How System Calls Work - The Mechanism](#how-system-calls-work---the-mechanism)
5. [Common System Calls](#common-system-calls)
6. [System Call Interface - The API](#system-call-interface---the-api)
7. [System Call Overhead and Performance](#system-call-overhead-and-performance)
8. [System Call Tracing and Debugging](#system-call-tracing-and-debugging)
9. [Security and System Calls](#security-and-system-calls)

---

## What are System Calls?

### Definition

**System Call (syscall)**: Interface between user programs and operating system kernel.

**Purpose:**
- **Access kernel services**: User programs can't directly access hardware
- **Privileged operations**: Need kernel's help
- **Abstraction**: Hide hardware complexity

### Real-World Analogy

**System Call = Service Request:**
- **User program**: Customer
- **System call**: Request to service desk
- **Kernel**: Service desk (has privileges)
- **Hardware**: Resources customer needs

**Example:**
```
Customer: "I need to print"
Service Desk: "I'll handle that" (has access to printer)
Customer: Can't access printer directly
```

---

## Why Do We Need System Calls?

### The Protection Problem

**Problem:**
```
User programs can't be trusted
Might crash system
Might access other programs' data
Might damage hardware
```

**Solution: Privilege Levels**
```
User mode: Limited privileges
Kernel mode: Full privileges
System calls: Switch to kernel mode
```

### What Requires System Calls?

**Operations that need kernel:**
- **File operations**: Read, write files
- **Process management**: Create, kill processes
- **Memory management**: Allocate memory
- **Network operations**: Send, receive data
- **Device access**: Access hardware

---

## User Space vs Kernel Space

### User Space

**What it is:**
- **Normal program execution**
- **Limited privileges**
- **Can't access hardware directly**
- **Protected memory**

**Limitations:**
- **Can't access kernel memory**
- **Can't execute privileged instructions**
- **Can't access hardware directly**

### Kernel Space

**What it is:**
- **OS code execution**
- **Full privileges**
- **Can access hardware**
- **Can access all memory**

**Capabilities:**
- **Direct hardware access**
- **Manage memory**
- **Schedule processes**
- **Handle interrupts**

### The Boundary

**Visual:**
```
┌─────────────────────┐
│   User Space         │ ← User programs
│   (Limited)          │
├─────────────────────┤
│   System Call        │ ← Interface
│   Boundary           │
├─────────────────────┤
│   Kernel Space       │ ← OS kernel
│   (Privileged)       │
└─────────────────────┘
```

---

## How System Calls Work - The Mechanism

### Step-by-Step Process

**1. User Program Calls Library Function:**
```c
printf("Hello");
```

**2. Library Function Prepares System Call:**
```c
// Inside printf (simplified)
write(1, "Hello", 5);  // System call
```

**3. Trap to Kernel:**
```
Special instruction (syscall, int 0x80)
CPU switches to kernel mode
Jumps to kernel code
```

**4. Kernel Executes:**
```
Kernel receives system call number
Looks up in system call table
Executes kernel function
```

**5. Return to User Space:**
```
Kernel finishes
Returns result
CPU switches back to user mode
Returns to user program
```

### System Call Number

**Each system call has number:**
```
1 = exit
2 = fork
3 = read
4 = write
...
```

**Process:**
```
1. User program: syscall(4, ...)  // write
2. Kernel: Looks up number 4
3. Kernel: Calls sys_write()
4. Kernel: Returns result
```

---

## Common System Calls

### File Operations

**open:**
```c
int fd = open("file.txt", O_RDONLY);
```
- **Opens file**: Returns file descriptor
- **Modes**: Read, write, append

**read:**
```c
read(fd, buffer, size);
```
- **Reads data**: From file descriptor
- **Returns**: Bytes read

**write:**
```c
write(fd, buffer, size);
```
- **Writes data**: To file descriptor
- **Returns**: Bytes written

**close:**
```c
close(fd);
```
- **Closes file**: Releases file descriptor

### Process Management

**fork:**
```c
pid_t pid = fork();
```
- **Creates process**: Child process
- **Returns**: PID in parent, 0 in child

**exec:**
```c
execve("/bin/ls", args, env);
```
- **Replaces process**: Loads new program
- **No return**: Process image replaced

**wait:**
```c
wait(&status);
```
- **Waits for child**: Blocks until child exits
- **Returns**: Child's exit status

**exit:**
```c
exit(0);
```
- **Terminates process**: Returns exit code

### Memory Management

**brk/sbrk:**
```c
void *ptr = sbrk(1024);
```
- **Changes heap size**: Allocates memory
- **Used by**: malloc internally

**mmap:**
```c
void *ptr = mmap(NULL, size, PROT_READ|PROT_WRITE, MAP_PRIVATE, fd, 0);
```
- **Maps file to memory**: Memory-mapped I/O
- **Efficient**: Direct memory access

### Network Operations

**socket:**
```c
int sock = socket(AF_INET, SOCK_STREAM, 0);
```
- **Creates socket**: Network endpoint

**bind:**
```c
bind(sock, &addr, sizeof(addr));
```
- **Binds socket**: To address

**listen:**
```c
listen(sock, backlog);
```
- **Listens**: For connections

**accept:**
```c
int client = accept(sock, &addr, &len);
```
- **Accepts connection**: Returns new socket

**connect:**
```c
connect(sock, &addr, sizeof(addr));
```
- **Connects**: To remote address

---

## System Call Interface - The API

### POSIX System Calls

**Standard Interface:**
```
Portable Operating System Interface
Standard across Unix-like systems
```

### Wrapper Functions

**Library Wrappers:**
```
C library provides wrappers
Easier to use
Error handling
```

**Example:**
```c
// System call directly (hard)
syscall(SYS_write, 1, "Hello", 5);

// Library wrapper (easy)
write(1, "Hello", 5);
```

---

## System Call Overhead and Performance

### Overhead Components

**1. Mode Switch:**
```
User mode → Kernel mode
Kernel mode → User mode
Takes time
```

**2. Context Save/Restore:**
```
Save user context
Restore kernel context
Later: Save kernel, restore user
```

**3. Validation:**
```
Kernel validates parameters
Security checks
Takes time
```

**4. Cache/TLB Effects:**
```
Different memory space
Cache misses
TLB flushes
```

### Reducing Overhead

**1. Batch Operations:**
```
Multiple operations in one call
readv, writev: Vector I/O
```

**2. Avoid Unnecessary Calls:**
```
Cache results
Don't call repeatedly
```

**3. Use Efficient Alternatives:**
```
mmap instead of read/write
Event-driven instead of polling
```

---

## System Call Tracing and Debugging

### strace (Linux)

**Trace system calls:**
```bash
strace ls
```

**Output:**
```
openat(AT_FDCWD, ".", O_RDONLY|O_NONBLOCK|O_CLOEXEC|O_DIRECTORY) = 3
getdents64(3, /* 10 entries */, 32768) = 280
close(3) = 0
```

**Use:**
- **Debug**: See what system calls program makes
- **Performance**: Identify slow calls
- **Security**: See file access

### Other Tools

**dtrace (Solaris, macOS):**
```
Dynamic tracing
More advanced
```

**perf (Linux):**
```
Performance analysis
System-wide
```

---

## Security and System Calls

### System Call Filtering

**seccomp (Linux):**
```
Restrict system calls
Sandbox applications
Security
```

**Example:**
```
Only allow read, write, exit
Block all other system calls
Prevents malicious code
```

### System Call Attacks

**1. System Call Injection:**
```
Inject malicious system calls
Bypass security
```

**2. Race Conditions:**
```
TOCTOU (Time-of-check-time-of-use)
Check then use (time passes, changes)
```

**Defense:**
- **Atomic operations**: Check and use atomically
- **Validation**: Validate in kernel
- **Sandboxing**: Restrict system calls

---

## Summary

System calls are the interface between user programs and the operating system. Understanding how they work, their overhead, and security implications is essential for backend engineers.

**Key Takeaways:**
- System calls bridge user and kernel space
- Required for privileged operations
- Mode switch has overhead
- Many common operations (file, process, network)
- Can be traced and debugged
- Security considerations important
- Optimize to reduce overhead

**Next Steps:**
- Understand your language's system call usage
- Learn to trace system calls
- Optimize system call usage
- Understand security implications
- Monitor system call performance

