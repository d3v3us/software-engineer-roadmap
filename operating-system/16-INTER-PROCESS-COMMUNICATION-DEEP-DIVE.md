# Inter-Process Communication (IPC) Deep Dive - Complete Understanding

## Table of Contents
1. [What is IPC?](#what-is-ipc)
2. [Why Do We Need IPC?](#why-do-we-need-ipc)
3. [IPC Methods Overview](#ipc-methods-overview)
4. [Pipes - Simple One-Way Communication](#pipes---simple-one-way-communication)
5. [Named Pipes (FIFOs) - Persistent Pipes](#named-pipes-fifos---persistent-pipes)
6. [Message Queues - Structured Messages](#message-queues---structured-messages)
7. [Shared Memory - Fastest IPC](#shared-memory---fastest-ipc)
8. [Semaphores - Synchronization for IPC](#semaphores---synchronization-for-ipc)
9. [Sockets - Network IPC](#sockets---network-ipc)
10. [Signals - Asynchronous Notifications](#signals---asynchronous-notifications)
11. [Memory-Mapped Files - File-Based IPC](#memory-mapped-files---file-based-ipc)
12. [IPC Performance Comparison](#ipc-performance-comparison)
13. [Choosing the Right IPC Method](#choosing-the-right-ipc-method)
14. [IPC Best Practices](#ipc-best-practices)

---

## What is IPC?

### Definition

**Inter-Process Communication (IPC)**: Mechanisms that allow processes to communicate and synchronize with each other.

**Problem:**
```
Process A (isolated memory)
Process B (isolated memory)

How do they communicate?
```

**Solution: IPC**
```
Process A ←─IPC─→ Process B
```

### Why Processes Can't Directly Communicate

**Process Isolation:**
- **Separate memory**: Each process has own memory space
- **Protected**: OS prevents direct access
- **Security**: Isolation provides security
- **Stability**: Crash doesn't affect others

**Need for IPC:**
- **Cooperation**: Processes need to work together
- **Data sharing**: Need to share data
- **Coordination**: Need to coordinate actions
- **Synchronization**: Need to synchronize

---

## Why Do We Need IPC?

### Use Cases

**1. Client-Server Architecture:**
```
Client Process → IPC → Server Process
  Request              Response
```

**2. Producer-Consumer:**
```
Producer Process → IPC → Consumer Process
  Produce data          Consume data
```

**3. Microservices:**
```
Service A → IPC → Service B → IPC → Service C
```

**4. Data Processing Pipeline:**
```
Process 1 → IPC → Process 2 → IPC → Process 3
  Stage 1          Stage 2          Stage 3
```

### Benefits

**1. Modularity:**
- **Separate processes**: Different functionality
- **Independent**: Can develop independently
- **Maintainable**: Easier to maintain

**2. Scalability:**
- **Distribute**: Distribute across machines
- **Scale**: Scale independently
- **Load balance**: Load balance across processes

**3. Fault Isolation:**
- **Isolated failures**: Failure in one doesn't crash others
- **Recovery**: Can recover individual processes
- **Resilience**: More resilient system

---

## IPC Methods Overview

### IPC Methods

**1. Pipes:**
- **Type**: One-way, byte stream
- **Scope**: Related processes (parent-child)
- **Speed**: Medium
- **Use**: Simple communication

**2. Named Pipes (FIFOs):**
- **Type**: One-way, byte stream
- **Scope**: Unrelated processes
- **Speed**: Medium
- **Use**: Persistent communication

**3. Message Queues:**
- **Type**: Structured messages
- **Scope**: Unrelated processes
- **Speed**: Medium
- **Use**: Structured communication

**4. Shared Memory:**
- **Type**: Shared memory region
- **Scope**: Unrelated processes
- **Speed**: Fastest
- **Use**: High-performance communication

**5. Sockets:**
- **Type**: Network or local
- **Scope**: Any processes (even remote)
- **Speed**: Slower (network overhead)
- **Use**: Distributed communication

**6. Signals:**
- **Type**: Asynchronous notifications
- **Scope**: Related processes
- **Speed**: Fast
- **Use**: Notifications, interrupts

**7. Memory-Mapped Files:**
- **Type**: File mapped to memory
- **Scope**: Unrelated processes
- **Speed**: Fast
- **Use**: Large data sharing

---

## Pipes - Simple One-Way Communication

### What are Pipes?

**Pipe**: Unidirectional communication channel between processes.

**Characteristics:**
- **One-way**: Data flows in one direction
- **Byte stream**: Stream of bytes
- **Related processes**: Usually parent-child
- **Temporary**: Exists only while processes run

### How Pipes Work

**Creation:**
```c
int pipe_fd[2];
pipe(pipe_fd);
// pipe_fd[0]: Read end
// pipe_fd[1]: Write end
```

**Visual:**
```
Process A (Parent)          Process B (Child)
     │                            │
     │───pipe_fd[1] (write)───────│
     │                            │
     │───pipe_fd[0] (read)───────│
     │                            │
```

**Usage:**
```c
// Parent creates pipe
int pipe_fd[2];
pipe(pipe_fd);

// Fork child
pid_t pid = fork();

if (pid == 0) {
    // Child: Write to pipe
    close(pipe_fd[0]);  // Close read end
    write(pipe_fd[1], "Hello", 5);
    close(pipe_fd[1]);
} else {
    // Parent: Read from pipe
    close(pipe_fd[1]);  // Close write end
    char buffer[100];
    read(pipe_fd[0], buffer, 100);
    close(pipe_fd[0]);
}
```

### Pipe Characteristics

**1. Unidirectional:**
```
Data flows: Writer → Reader
Cannot: Reader → Writer (need another pipe)
```

**2. Byte Stream:**
```
No message boundaries
Just stream of bytes
Reader must know message format
```

**3. Blocking:**
```
Read blocks if pipe empty
Write blocks if pipe full
```

**4. Limited Size:**
```
Pipe has buffer (typically 64KB)
Write blocks if buffer full
```

### Bidirectional Communication

**Two Pipes:**
```c
int pipe1[2], pipe2[2];
pipe(pipe1);  // Parent → Child
pipe(pipe2);  // Child → Parent

// Parent writes to pipe1[1], reads from pipe2[0]
// Child writes to pipe2[1], reads from pipe1[0]
```

---

## Named Pipes (FIFOs) - Persistent Pipes

### What are Named Pipes?

**Named Pipe (FIFO)**: Pipe with a name in filesystem.

**Difference from Regular Pipes:**
- **Regular pipe**: Anonymous, temporary
- **Named pipe**: Has name, persistent

### Creating Named Pipes

**Command Line:**
```bash
mkfifo mypipe
# Creates named pipe "mypipe"
```

**Program:**
```c
mkfifo("mypipe", 0666);
// Creates named pipe "mypipe"
```

### Using Named Pipes

**Process 1 (Writer):**
```c
int fd = open("mypipe", O_WRONLY);
write(fd, "Hello", 5);
close(fd);
```

**Process 2 (Reader):**
```c
int fd = open("mypipe", O_RDONLY);
char buffer[100];
read(fd, buffer, 100);
close(fd);
```

### Characteristics

**1. Persistent:**
- **Exists in filesystem**: Like a file
- **Survives processes**: Exists after processes end
- **Can be reused**: Multiple processes can use

**2. Unrelated Processes:**
- **No parent-child**: Unrelated processes can use
- **File-based**: Access like a file
- **Flexible**: More flexible than regular pipes

**3. Blocking:**
- **Open blocks**: Open blocks until other end opens
- **Read blocks**: Read blocks if empty
- **Write blocks**: Write blocks if full

---

## Message Queues - Structured Messages

### What are Message Queues?

**Message Queue**: IPC mechanism that allows processes to exchange structured messages.

**Characteristics:**
- **Structured**: Messages have structure
- **Persistent**: Messages persist until read
- **Priority**: Can have priorities
- **Multiple readers**: Multiple processes can read

### Message Queue Operations

**1. Create/Open:**
```c
key_t key = ftok("/tmp", 'A');
int msgid = msgget(key, 0666 | IPC_CREAT);
```

**2. Send Message:**
```c
struct message {
    long mtype;      // Message type
    char mtext[100]; // Message data
};

struct message msg;
msg.mtype = 1;
strcpy(msg.mtext, "Hello");
msgsnd(msgid, &msg, sizeof(msg.mtext), 0);
```

**3. Receive Message:**
```c
struct message msg;
msgrcv(msgid, &msg, sizeof(msg.mtext), 1, 0);
// Receive message of type 1
```

**4. Remove Queue:**
```c
msgctl(msgid, IPC_RMID, NULL);
```

### Message Types

**Priority-Based:**
```c
// Send high priority
msg.mtype = 1;  // High priority
msgsnd(msgid, &msg, size, 0);

// Send low priority
msg.mtype = 10;  // Low priority
msgsnd(msgid, &msg, size, 0);

// Receive: Gets highest priority first
```

**Type-Based:**
```c
// Send different message types
msg.mtype = TYPE_REQUEST;
msgsnd(msgid, &msg, size, 0);

msg.mtype = TYPE_RESPONSE;
msgsnd(msgid, &msg, size, 0);

// Receive specific type
msgrcv(msgid, &msg, size, TYPE_REQUEST, 0);
```

### Benefits

**1. Structured:**
- **Message format**: Defined message format
- **Type safety**: Can type-check messages
- **Clear**: Clear message boundaries

**2. Persistent:**
- **Survives processes**: Messages persist
- **Reliable**: Don't lose messages
- **Queue**: Acts like a queue

**3. Priority:**
- **Priority support**: Can prioritize messages
- **Ordering**: Control message order
- **Urgent**: Handle urgent messages first

---

## Shared Memory - Fastest IPC

### What is Shared Memory?

**Shared Memory**: Memory region shared between processes.

**Characteristics:**
- **Fastest**: Fastest IPC method
- **Direct access**: Direct memory access
- **No copying**: No data copying
- **Synchronization needed**: Must synchronize access

### How Shared Memory Works

**1. Create Shared Memory:**
```c
key_t key = ftok("/tmp", 'A');
int shmid = shmget(key, 1024, 0666 | IPC_CREAT);
```

**2. Attach to Process:**
```c
void* shm = shmat(shmid, NULL, 0);
// shm points to shared memory
```

**3. Use Shared Memory:**
```c
// Process 1: Write
strcpy((char*)shm, "Hello");

// Process 2: Read
printf("%s\n", (char*)shm);
```

**4. Detach:**
```c
shmdt(shm);
```

**5. Remove:**
```c
shmctl(shmid, IPC_RMID, NULL);
```

### Memory Mapping

**Visual:**
```
Process A Virtual Memory    Physical Memory    Process B Virtual Memory
0x1000 ──────────────────→ 0x5000  ←────────────────── 0x2000
(Shared memory region)     (Same physical)    (Shared memory region)
```

**Both processes see same physical memory:**
- **Process A**: Writes to 0x1000
- **Physical**: Stored at 0x5000
- **Process B**: Reads from 0x2000 (maps to 0x5000)
- **Result**: Process B sees what Process A wrote

### Synchronization

**Problem:**
```
Process A: Read value (5)
Process B: Read value (5)
Process A: Write value (6)
Process B: Write value (6)
Result: 6 (should be 7!)
```

**Solution: Semaphores or Mutexes:**
```c
// Use semaphore to synchronize
sem_wait(sem);  // Lock
// Access shared memory
*value = *value + 1;
sem_post(sem);  // Unlock
```

### Benefits

**1. Performance:**
- **Fastest**: Fastest IPC method
- **No copying**: No data copying overhead
- **Direct access**: Direct memory access

**2. Efficiency:**
- **Low overhead**: Minimal overhead
- **Large data**: Efficient for large data
- **High throughput**: High throughput

**3. Flexibility:**
- **Any data**: Can share any data structure
- **Complex structures**: Can share complex structures
- **Direct access**: Direct memory access

### Drawbacks

**1. Synchronization:**
- **Must synchronize**: Must use locks/semaphores
- **Complex**: More complex than other methods
- **Race conditions**: Risk of race conditions

**2. No Persistence:**
- **Volatile**: Lost when processes end
- **No durability**: Not persistent
- **Temporary**: Temporary only

---

## Semaphores - Synchronization for IPC

### What are Semaphores?

**Semaphore**: Synchronization primitive for coordinating access to shared resources.

**Purpose:**
- **Synchronize**: Synchronize access to shared memory
- **Coordinate**: Coordinate between processes
- **Prevent race conditions**: Prevent race conditions

### Binary Semaphore (Mutex)

**Binary Semaphore**: Semaphore with value 0 or 1.

**Operations:**
- **wait() (P)**: Decrement, block if 0
- **signal() (V)**: Increment, wake waiting process

**Example:**
```c
sem_t mutex;
sem_init(&mutex, 0, 1);  // Initial value 1

// Process 1
sem_wait(&mutex);  // Lock
// Critical section
access_shared_memory();
sem_post(&mutex);  // Unlock

// Process 2
sem_wait(&mutex);  // Wait if locked
// Critical section
access_shared_memory();
sem_post(&mutex);  // Unlock
```

### Counting Semaphore

**Counting Semaphore**: Semaphore with value >= 0.

**Use Case:**
- **Resource pool**: Pool of resources
- **Limit access**: Limit number of concurrent accesses

**Example:**
```c
sem_t pool;
sem_init(&pool, 0, 10);  // 10 resources

// Process: Get resource
sem_wait(&pool);  // Decrement (9 left)
use_resource();
sem_post(&pool);  // Increment (10 left)
```

### Producer-Consumer with Semaphores

**Problem:**
```
Producer: Produces items
Consumer: Consumes items
Buffer: Shared buffer
Need: Synchronize access
```

**Solution:**
```c
sem_t empty;   // Empty slots
sem_t full;    // Full slots
sem_t mutex;   // Mutual exclusion

sem_init(&empty, 0, 10);  // 10 empty slots
sem_init(&full, 0, 0);    // 0 full slots
sem_init(&mutex, 0, 1);   // Mutex

// Producer
void producer() {
    while (1) {
        item = produce();
        sem_wait(&empty);  // Wait for empty slot
        sem_wait(&mutex);
        buffer[in] = item;
        in = (in + 1) % SIZE;
        sem_post(&mutex);
        sem_post(&full);   // Signal item available
    }
}

// Consumer
void consumer() {
    while (1) {
        sem_wait(&full);   // Wait for item
        sem_wait(&mutex);
        item = buffer[out];
        out = (out + 1) % SIZE;
        sem_post(&mutex);
        sem_post(&empty);  // Signal slot empty
        consume(item);
    }
}
```

---

## Sockets - Network IPC

### What are Sockets?

**Socket**: Endpoint for communication, can be local or network.

**Types:**
- **Unix domain sockets**: Local communication
- **Internet sockets**: Network communication

### Unix Domain Sockets

**Local Communication:**
```c
// Server
int server_fd = socket(AF_UNIX, SOCK_STREAM, 0);
struct sockaddr_un addr;
addr.sun_family = AF_UNIX;
strcpy(addr.sun_path, "/tmp/socket");
bind(server_fd, (struct sockaddr*)&addr, sizeof(addr));
listen(server_fd, 5);

int client_fd = accept(server_fd, NULL, NULL);
send(client_fd, "Hello", 5, 0);

// Client
int sock = socket(AF_UNIX, SOCK_STREAM, 0);
struct sockaddr_un addr;
addr.sun_family = AF_UNIX;
strcpy(addr.sun_path, "/tmp/socket");
connect(sock, (struct sockaddr*)&addr, sizeof(addr));
recv(sock, buffer, 100, 0);
```

**Benefits:**
- **Faster**: Faster than network sockets
- **Local**: Only local communication
- **Secure**: More secure (local only)

### Internet Sockets

**Network Communication:**
```c
// Server
int server_fd = socket(AF_INET, SOCK_STREAM, 0);
struct sockaddr_in addr;
addr.sin_family = AF_INET;
addr.sin_port = htons(8080);
addr.sin_addr.s_addr = INADDR_ANY;
bind(server_fd, (struct sockaddr*)&addr, sizeof(addr));
listen(server_fd, 5);

// Client
int sock = socket(AF_INET, SOCK_STREAM, 0);
struct sockaddr_in addr;
addr.sin_family = AF_INET;
addr.sin_port = htons(8080);
inet_pton(AF_INET, "127.0.0.1", &addr.sin_addr);
connect(sock, (struct sockaddr*)&addr, sizeof(addr));
```

**Benefits:**
- **Distributed**: Can communicate across network
- **Flexible**: Very flexible
- **Standard**: Standard protocol

---

## Signals - Asynchronous Notifications

### What are Signals?

**Signal**: Asynchronous notification sent to a process.

**Purpose:**
- **Notifications**: Notify process of events
- **Interrupts**: Interrupt process execution
- **Control**: Control process behavior

### Common Signals

**1. SIGTERM:**
- **Purpose**: Request termination
- **Action**: Process can handle or ignore
- **Use**: Graceful shutdown

**2. SIGKILL:**
- **Purpose**: Force termination
- **Action**: Cannot be caught or ignored
- **Use**: Force kill

**3. SIGINT:**
- **Purpose**: Interrupt (Ctrl+C)
- **Action**: Process can handle
- **Use**: User interrupt

**4. SIGUSR1, SIGUSR2:**
- **Purpose**: User-defined signals
- **Action**: Process-defined
- **Use**: Custom notifications

### Sending Signals

**Command Line:**
```bash
kill -SIGTERM <pid>  # Send SIGTERM
kill -9 <pid>        # Send SIGKILL
```

**Program:**
```c
kill(pid, SIGTERM);  // Send signal to process
```

### Handling Signals

**Signal Handler:**
```c
void signal_handler(int sig) {
    printf("Received signal %d\n", sig);
    // Handle signal
}

signal(SIGTERM, signal_handler);
// Now SIGTERM calls signal_handler
```

**Benefits:**
- **Asynchronous**: Asynchronous notification
- **Fast**: Fast delivery
- **Simple**: Simple mechanism

**Limitations:**
- **Limited data**: Can't send data (just notification)
- **Unreliable**: May be lost
- **Race conditions**: Can have race conditions

---

## Memory-Mapped Files - File-Based IPC

### What are Memory-Mapped Files?

**Memory-Mapped File**: File mapped into process memory space.

**How it works:**
- **Map file**: Map file to memory
- **Access like memory**: Access like regular memory
- **OS handles**: OS handles file I/O
- **Shared**: Multiple processes can map same file

### Creating Memory-Mapped Files

**Process 1 (Writer):**
```c
int fd = open("shared_file", O_RDWR | O_CREAT, 0666);
ftruncate(fd, 1024);  // Set file size

void* map = mmap(NULL, 1024, PROT_READ | PROT_WRITE, 
                 MAP_SHARED, fd, 0);
strcpy((char*)map, "Hello");
munmap(map, 1024);
close(fd);
```

**Process 2 (Reader):**
```c
int fd = open("shared_file", O_RDONLY);
void* map = mmap(NULL, 1024, PROT_READ, 
                 MAP_SHARED, fd, 0);
printf("%s\n", (char*)map);
munmap(map, 1024);
close(fd);
```

### Benefits

**1. Persistent:**
- **File-based**: Stored in file
- **Survives processes**: Persists after processes end
- **Durable**: Durable storage

**2. Large Data:**
- **Efficient**: Efficient for large data
- **OS handles**: OS handles paging
- **Virtual memory**: Uses virtual memory

**3. Shared:**
- **Multiple processes**: Multiple processes can map
- **Automatic sync**: OS handles synchronization
- **Efficient**: Efficient sharing

---

## IPC Performance Comparison

### Speed Comparison

**Fastest to Slowest:**
```
1. Shared Memory:    ~1-10 ns (fastest)
2. Memory-Mapped:    ~10-100 ns
3. Pipes:            ~1-10 μs
4. Named Pipes:      ~1-10 μs
5. Message Queues:   ~1-10 μs
6. Unix Sockets:     ~10-100 μs
7. Internet Sockets: ~100-1000 μs (slowest, network overhead)
```

### Bandwidth Comparison

**Highest to Lowest:**
```
1. Shared Memory:    ~GB/s (highest)
2. Memory-Mapped:    ~GB/s
3. Pipes:            ~MB/s
4. Named Pipes:      ~MB/s
5. Message Queues:   ~MB/s
6. Unix Sockets:     ~MB/s
7. Internet Sockets: ~MB/s (depends on network)
```

### Latency Comparison

**Lowest to Highest:**
```
1. Shared Memory:    Lowest latency
2. Memory-Mapped:    Low latency
3. Signals:          Low latency (but limited)
4. Pipes:            Medium latency
5. Named Pipes:      Medium latency
6. Message Queues:   Medium latency
7. Sockets:          Higher latency
```

---

## Choosing the Right IPC Method

### Decision Factors

**1. Performance:**
- **High performance**: Shared memory, memory-mapped files
- **Medium performance**: Pipes, message queues
- **Lower performance**: Sockets (network overhead)

**2. Data Size:**
- **Small data**: Pipes, message queues, signals
- **Large data**: Shared memory, memory-mapped files
- **Very large**: Memory-mapped files

**3. Process Relationship:**
- **Parent-child**: Pipes
- **Unrelated (local)**: Named pipes, message queues, shared memory
- **Remote**: Sockets

**4. Persistence:**
- **Temporary**: Pipes, shared memory
- **Persistent**: Named pipes, message queues, memory-mapped files

**5. Synchronization:**
- **Built-in**: Message queues (queue semantics)
- **Manual**: Shared memory (need semaphores)
- **Automatic**: Pipes (blocking I/O)

### Decision Matrix

| Need | Best Choice |
|------|-------------|
| **Fastest speed** | Shared memory |
| **Simple** | Pipes |
| **Persistent** | Named pipes, message queues |
| **Large data** | Shared memory, memory-mapped files |
| **Remote** | Sockets |
| **Structured** | Message queues |
| **Notifications** | Signals |

---

## IPC Best Practices

### 1. Choose Right Method

**Consider:**
- **Performance needs**: How fast?
- **Data size**: How much data?
- **Process relationship**: Related or unrelated?
- **Persistence**: Need persistence?
- **Complexity**: How complex?

### 2. Handle Errors

**Always check:**
```c
int fd = open("pipe", O_WRONLY);
if (fd == -1) {
    perror("open failed");
    exit(1);
}
```

### 3. Synchronize Access

**For shared memory:**
```c
// Always use synchronization
sem_wait(mutex);
// Access shared memory
sem_post(mutex);
```

### 4. Clean Up Resources

**Always clean up:**
```c
// Close file descriptors
close(fd);

// Detach shared memory
shmdt(shm);

// Remove IPC objects
shmctl(shmid, IPC_RMID, NULL);
```

### 5. Handle Blocking

**Non-blocking I/O:**
```c
// Set non-blocking
int flags = fcntl(fd, F_GETFL);
fcntl(fd, F_SETFL, flags | O_NONBLOCK);

// Now read/write won't block
```

### 6. Use Timeouts

**Timeout on operations:**
```c
// Use select/poll with timeout
struct timeval timeout;
timeout.tv_sec = 5;
timeout.tv_usec = 0;

fd_set read_fds;
FD_SET(fd, &read_fds);
select(fd + 1, &read_fds, NULL, NULL, &timeout);
```

---

## Summary

Inter-Process Communication enables processes to work together, share data, and coordinate actions while maintaining process isolation.

**Key Takeaways:**
- **IPC enables communication**: Between isolated processes
- **Multiple methods**: Pipes, message queues, shared memory, sockets, signals
- **Performance varies**: Shared memory fastest, sockets slowest
- **Choose wisely**: Based on needs (speed, size, relationship, persistence)
- **Synchronize**: Must synchronize shared memory access
- **Handle errors**: Always handle errors
- **Clean up**: Always clean up resources

**IPC Methods:**
- **Pipes**: Simple, one-way, parent-child
- **Named pipes**: Persistent pipes, unrelated processes
- **Message queues**: Structured, persistent, priority
- **Shared memory**: Fastest, direct access, needs synchronization
- **Sockets**: Network or local, flexible, standard
- **Signals**: Asynchronous notifications, limited data
- **Memory-mapped files**: File-based, persistent, large data

**Performance:**
- **Fastest**: Shared memory
- **Fast**: Memory-mapped files
- **Medium**: Pipes, message queues
- **Slower**: Sockets (network overhead)

**Next Steps:**
- Practice with different IPC methods
- Understand synchronization needs
- Learn performance characteristics
- Apply to real projects

