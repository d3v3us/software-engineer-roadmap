# Backend Software Engineer Interview Knowledge Base

A comprehensive, detailed knowledge base covering all essential topics for backend software engineering interviews. Each topic is thoroughly explained with analogies, visual descriptions, and practical examples.

This knowledge base is based on the [backend-swe-interview-questions](https://github.com/tamhoang1412/backend-swe-interview-questions) repository, expanded with detailed explanations, analogies, and visual descriptions to create a valuable learning resource.

## Table of Contents

### Comprehensive Overview Files

Quick reference guides covering all topics:

1. [Networking Overview](./01-NETWORKING.md) - TCP/IP, HTTP/HTTPS, DNS, Load Balancing, CDN, WebSockets, API Design
2. [Operating System Overview](./02-OPERATING-SYSTEM.md) - Processes, Threads, Scheduling, Memory Management, Concurrency, System Calls
3. [Database Overview](./03-DATABASE.md) - SQL vs NoSQL, Indexing, Query Optimization, Replication, Sharding, Transactions
4. [Security Overview](./04-SECURITY.md) - Encryption, Hashing, SSL/TLS, Certificates, Credential Storage, DDoS Defense
5. [Programming Paradigm Overview](./05-PROGRAMMING-PARADIGM.md) - OOP, Functional Programming, Design Patterns
6. [Software Development Process Overview](./06-SOFTWARE-DEVELOPMENT-PROCESS.md) - DDD, TDD, BDD

### Deep Dive Files

In-depth, comprehensive guides for deep understanding:

#### Networking

- [TCP/IP](./networking/01-TCP-IP-DEEP-DIVE.md) - Complete understanding of TCP: handshakes, flow control, congestion control, reliability mechanisms
- [HTTP/HTTPS](./networking/02-HTTP-HTTPS-DEEP-DIVE.md) - HTTP fundamentals, methods, status codes, headers, HTTPS/TLS, caching, RESTful design
- [DNS](./networking/03-DNS-DEEP-DIVE.md) - DNS hierarchy, resolution process, record types, caching, security, troubleshooting
- [Load Balancing](./networking/04-LOAD-BALANCING-DEEP-DIVE.md) - Load balancing algorithms, layer 4 vs layer 7, health checks, session persistence, high availability
- [WebSockets](./networking/05-WEBSOCKETS-DEEP-DIVE.md) - WebSocket protocol, handshake, frames, real-time communication, scaling, use cases
- [REST vs GraphQL vs gRPC](./networking/06-REST-GRAPHQL-GRPC-DEEP-DIVE.md) - Detailed comparison of API styles, when to use each, performance, flexibility, implementation patterns
- [CDN](./networking/07-CDN-DEEP-DIVE.md) - Content delivery networks: edge locations, caching strategies, cache invalidation, performance optimization, security

#### Operating System

- [Memory Management](./operating-system/01-MEMORY-MANAGEMENT-DEEP-DIVE.md) - Virtual memory, paging, page tables, heap/stack, memory allocation, garbage collection
- [Concurrency](./operating-system/02-CONCURRENCY-DEEP-DIVE.md) - Race conditions, mutex, semaphore, deadlock, condition variables, atomic operations
- [Processes and Threads](./operating-system/03-PROCESSES-THREADS-DEEP-DIVE.md) - Process isolation, thread sharing, IPC, context switching, when to use each
- [Scheduling Algorithms](./operating-system/04-SCHEDULING-ALGORITHMS-DEEP-DIVE.md) - FCFS, SJF, Round Robin, Priority, Multilevel queues, real-time scheduling
- [File Systems](./operating-system/05-FILE-SYSTEMS-DEEP-DIVE.md) - File system structure, inodes, directory organization, file operations, journaling, permissions
- [System Calls](./operating-system/06-SYSTEM-CALLS-DEEP-DIVE.md) - User/kernel space, syscall mechanism, common system calls, performance, security
- [Caching](./operating-system/07-CACHING-DEEP-DIVE.md) - Cache hierarchy, LRU/LFU policies, in-memory caching (Redis/Memcached), cache stampede, distributed caching
- [Sorting Algorithms](./operating-system/08-SORTING-ALGORITHMS-DEEP-DIVE.md) - Quicksort, Merge sort, Heap sort, complexity analysis, real-world usage, choosing algorithms

#### Database

- [Database Indexing](./database/01-DATABASE-INDEXING-DEEP-DIVE.md) - B-Trees, index types, composite indexes, query optimization, best practices
- [Database Transactions](./database/02-DATABASE-TRANSACTIONS-DEEP-DIVE.md) - ACID properties, isolation levels, concurrency problems, locking, MVCC, distributed transactions
- [Database Replication](./database/03-DATABASE-REPLICATION-DEEP-DIVE.md) - Master-slave architecture, binary logs, replication process, replication lag, failover
- [Database Sharding](./database/04-DATABASE-SHARDING-DEEP-DIVE.md) - Sharding strategies, shard key selection, querying across shards, rebalancing
- [Query Optimization](./database/05-QUERY-OPTIMIZATION-DEEP-DIVE.md) - Query execution, EXPLAIN plans, join optimization, aggregation, subquery optimization, monitoring

#### Security

- [Encryption and Hashing](./security/01-ENCRYPTION-HASHING-DEEP-DIVE.md) - Symmetric/asymmetric encryption, hash functions, password hashing, digital signatures, key management
- [SSL/TLS](./security/02-SSL-TLS-DEEP-DIVE.md) - TLS handshake, certificates, certificate authorities, cipher suites, perfect forward secrecy, performance optimization
- [Credential Storage](./security/03-CREDENTIAL-STORAGE-DEEP-DIVE.md) - Password hashing, database credentials, API keys, secret management services, key rotation, best practices
- [DDoS Defense](./security/04-DDOS-DEFENSE-DEEP-DIVE.md) - DDoS attack types, detection, defense strategies, rate limiting, protection services, incident response

#### Programming Paradigm

- [OOP](./programming/01-OOP-DEEP-DIVE.md) - Classes and objects, inheritance, polymorphism, encapsulation, abstraction, composition, design patterns, SOLID principles
- [Functional Programming](./programming/02-FUNCTIONAL-PROGRAMMING-DEEP-DIVE.md) - Pure functions, immutability, higher-order functions, function composition, recursion, lazy evaluation

#### Software Development Process

- [DDD](./software-development/01-DDD-DEEP-DIVE.md) - Domain-Driven Design: ubiquitous language, bounded contexts, tactical patterns, aggregates, domain events, implementation guide
- [TDD](./software-development/02-TDD-DEEP-DIVE.md) - Test-Driven Development: Red-Green-Refactor cycle, writing good tests, test types, mocking, TDD patterns and practices
- [BDD](./software-development/03-BDD-DEEP-DIVE.md) - Behavior-Driven Development: Given-When-Then, Gherkin syntax, BDD tools, step definitions, collaboration, living documentation

---

## How to Use This Knowledge Base

This knowledge base is organized in two levels:

### Level 1: Overview Files

Start with the comprehensive overview files (`01-NETWORKING.md`, `02-OPERATING-SYSTEM.md`, etc.) for a broad understanding of all topics.

### Level 2: Deep Dive Files

For topics you need to master deeply, read the corresponding deep dive files in the topic-specific directories. These provide:

- **Extensive explanations** with multiple analogies
- **Step-by-step breakdowns** of complex concepts
- **Visual diagrams** and flowcharts
- **Real-world examples** and use cases
- **Common pitfalls** and how to avoid them
- **Best practices** and optimization strategies

**For Interview Preparation:**

- Start with overview files for breadth
- Deep dive into topics you're weak on
- Understand the concepts, not just memorize
- Practice explaining concepts in your own words
- Work through examples
- Connect concepts across sections

**For Learning:**

- Read deep dive files for topics you're studying
- Take notes as you read
- Try implementing examples
- Experiment with code
- Build projects applying these concepts

---

## Contributing

This knowledge base is designed to be comprehensive and detailed. If you find areas that need expansion or clarification, please contribute improvements.

---

## License

This knowledge base is created for educational purposes. Original questions and structure based on [backend-swe-interview-questions](https://github.com/tamhoang1412/backend-swe-interview-questions).
