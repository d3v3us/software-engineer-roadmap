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
- [Client-Side vs Server-Side Rendering](./networking/08-CLIENT-SERVER-RENDERING-DEEP-DIVE.md) - SSR vs CSR, trade-offs (performance, SEO, UX), hybrid approaches (SSG, ISR, hydration), when to use each
- [Reliable Communication Protocols](./networking/09-RELIABLE-COMMUNICATION-PROTOCOLS-DEEP-DIVE.md) - Building reliability on unreliable channels, sequence numbers, ACKs, checksums, retransmission, TCP example, designing reliable protocols
- [API Gateway](./networking/10-API-GATEWAY-DEEP-DIVE.md) - Single entry point, routing, authentication, rate limiting, caching, load balancing, API versioning, gateway patterns (BFF, aggregation, orchestration)
- [Message Queues](./networking/11-MESSAGE-QUEUES-DEEP-DIVE.md) - Asynchronous messaging, decoupling, message queue patterns (point-to-point, pub-sub, request-reply, work queue), delivery guarantees, popular systems (RabbitMQ, Kafka, SQS)
- [Circuit Breaker and Rate Limiting](./networking/12-CIRCUIT-BREAKER-RATE-LIMITING-DEEP-DIVE.md) - Circuit breaker pattern (prevent cascading failures), rate limiting algorithms (fixed window, sliding window, token bucket), throttling, implementation
- [Service Mesh](./networking/13-SERVICE-MESH-DEEP-DIVE.md) - Service-to-service communication infrastructure, sidecar pattern, control plane vs data plane, features (discovery, load balancing, security, observability), Istio, Linkerd
- [API Design](./networking/14-API-DESIGN-DEEP-DIVE.md) - RESTful API principles, URL design, HTTP methods, status codes, API versioning, error handling, API documentation, common mistakes
- [What Happens When You Type "google.com"](./networking/15-WHAT-HAPPENS-WHEN-YOU-TYPE-GOOGLE-COM-DEEP-DIVE.md) - Complete journey from URL input to page rendering: DNS resolution, TCP connection, TLS handshake, HTTP request/response, browser rendering, performance optimizations
- [Retry and Timeout Patterns](./networking/16-RETRY-TIMEOUT-PATTERNS-DEEP-DIVE.md) - Retry patterns (simple, exponential backoff, jitter), timeout patterns (connection, read, total), when to retry vs when not to, combining retry and timeout, best practices, common mistakes
- [Service Discovery](./networking/17-SERVICE-DISCOVERY-DEEP-DIVE.md) - What is service discovery, client-side vs server-side discovery, service registry, service registration patterns, health checking, implementations (Consul, Eureka, etcd, Zookeeper), DNS-based discovery, service mesh integration, best practices
- [Webhooks](./networking/18-WEBHOOKS-DEEP-DIVE.md) - What are webhooks, webhooks vs polling, how webhooks work, webhook security (signatures, HTTPS), webhook delivery, retries with exponential backoff, idempotency, webhook signatures, best practices, testing, troubleshooting
- [Bulkhead Pattern](./networking/19-BULKHEAD-PATTERN-DEEP-DIVE.md) - What is bulkhead pattern, why it's needed (prevent cascading failures), types of bulkheads (thread pools, connection pools, processes, databases), implementation examples, best practices, common mistakes
- [API Versioning](./networking/20-API-VERSIONING-DEEP-DIVE.md) - What is API versioning, why version APIs, versioning strategies (URL, header, query parameter, content negotiation), semantic versioning, best practices, deprecation strategy, migration between versions, common mistakes
- [API Pagination](./networking/21-API-PAGINATION-DEEP-DIVE.md) - What is API pagination, why it's needed, pagination strategies (offset-based, cursor-based, keyset, page-based), comparison of methods, best practices, pagination in different scenarios, common mistakes
- [API Caching](./networking/22-API-CACHING-DEEP-DIVE.md) - What is API caching, why it's needed, types of caching (client-side, CDN, reverse proxy, application, database), HTTP caching (Cache-Control, ETag, Last-Modified), cache strategies (cache-aside, write-through, write-behind, refresh-ahead), cache invalidation, best practices, common mistakes
- [Load Balancing](./networking/23-LOAD-BALANCING-DEEP-DIVE.md) - What is load balancing, why it's needed, load balancing algorithms (round robin, weighted round robin, least connections, least response time, IP hash), Layer 4 vs Layer 7 load balancing, load balancing architectures, health checks, session persistence (sticky sessions), load balancing in cloud (AWS ELB, GCP, Azure), best practices, common challenges
- [Message Queues](./networking/24-MESSAGE-QUEUES-DEEP-DIVE.md) - What are message queues, why they're needed, message queue patterns (point-to-point, pub/sub, request-reply), queue vs pub/sub, message queue architectures, message delivery guarantees (at-most-once, at-least-once, exactly-once), message ordering, dead letter queues, message queue technologies (RabbitMQ, Kafka, SQS, Redis, Pulsar), best practices, common challenges
- [API Gateway](./networking/25-API-GATEWAY-DEEP-DIVE.md) - What is API Gateway, why it's needed, API Gateway functions (routing, authentication, rate limiting, transformation, aggregation), API Gateway vs Load Balancer, API Gateway patterns (single, multiple, BFF), routing and aggregation, authentication and authorization, rate limiting and throttling, request/response transformation, API Gateway technologies (Kong, AWS API Gateway, Azure API Management, NGINX), best practices, common challenges
- [API Documentation](./networking/26-API-DOCUMENTATION-DEEP-DIVE.md) - What is API documentation, why it matters, types of API documentation (reference, getting started, tutorials), API documentation formats (OpenAPI/Swagger, RAML, API Blueprint, Markdown), OpenAPI/Swagger, API documentation best practices, documentation structure, code examples, error documentation, versioning documentation, interactive documentation, documentation tools (Swagger, Postman, Stoplight, ReadMe), common mistakes
- [REST API Principles](./networking/27-REST-API-PRINCIPLES-DEEP-DIVE.md) - What is REST, REST principles (stateless, client-server, uniform interface, resource-based), RESTful resources (identification, naming), HTTP methods (GET, POST, PUT, PATCH, DELETE), status codes (2xx, 4xx, 5xx), REST constraints (stateless, cacheable, layered system), REST vs RPC, REST best practices, common mistakes
- [HTTP Status Codes](./networking/28-HTTP-STATUS-CODES-DEEP-DIVE.md) - What are HTTP status codes, status code categories (1xx informational, 2xx success, 3xx redirection, 4xx client error, 5xx server error), 1xx informational (100 Continue, 101 Switching Protocols), 2xx success (200 OK, 201 Created, 204 No Content, 202 Accepted), 3xx redirection (301 Moved, 302 Found, 304 Not Modified), 4xx client error (400 Bad Request, 401 Unauthorized, 403 Forbidden, 404 Not Found, 409 Conflict, 429 Too Many Requests), 5xx server error (500 Internal Server Error, 502 Bad Gateway, 503 Service Unavailable, 504 Gateway Timeout), status code best practices, common mistakes
- [HTTP Headers](./networking/29-HTTP-HEADERS-DEEP-DIVE.md) - What are HTTP headers, request headers (User-Agent, Accept, Authorization, Content-Type, Content-Length, Host, Accept-Encoding, If-None-Match, If-Modified-Since), response headers (Content-Type, Content-Length, Cache-Control, ETag, Last-Modified, Location, Set-Cookie, Server), general headers (Connection, Date, Transfer-Encoding, Upgrade), entity headers (Content-Encoding, Content-Language, Content-Location, Content-MD5), security headers (Authorization, WWW-Authenticate, X-Content-Type-Options, X-Frame-Options, X-XSS-Protection, Strict-Transport-Security, Content-Security-Policy), caching headers (Cache-Control, ETag, Last-Modified), best practices, common headers

#### Operating System

- [Memory Management](./operating-system/01-MEMORY-MANAGEMENT-DEEP-DIVE.md) - Virtual memory, paging, page tables, heap/stack, memory allocation, garbage collection
- [Concurrency](./operating-system/02-CONCURRENCY-DEEP-DIVE.md) - Race conditions, mutex, semaphore, deadlock, condition variables, atomic operations
- [Processes and Threads](./operating-system/03-PROCESSES-THREADS-DEEP-DIVE.md) - Process isolation, thread sharing, IPC, context switching, when to use each
- [Scheduling Algorithms](./operating-system/04-SCHEDULING-ALGORITHMS-DEEP-DIVE.md) - FCFS, SJF, Round Robin, Priority, Multilevel queues, real-time scheduling
- [File Systems](./operating-system/05-FILE-SYSTEMS-DEEP-DIVE.md) - File system structure, inodes, directory organization, file operations, journaling, permissions
- [System Calls](./operating-system/06-SYSTEM-CALLS-DEEP-DIVE.md) - User/kernel space, syscall mechanism, common system calls, performance, security
- [Caching](./operating-system/07-CACHING-DEEP-DIVE.md) - Cache hierarchy, LRU/LFU policies, in-memory caching (Redis/Memcached), cache stampede, distributed caching
- [Sorting Algorithms](./operating-system/08-SORTING-ALGORITHMS-DEEP-DIVE.md) - Quicksort, Merge sort, Heap sort, complexity analysis, real-world usage, choosing algorithms
- [Real-Time Systems](./operating-system/09-REAL-TIME-SYSTEMS-DEEP-DIVE.md) - Hard vs soft real-time, deadlines, real-time scheduling (RMS, EDF), memory management, real-time languages, design principles
- [Memory Leaks](./operating-system/10-MEMORY-LEAKS-DEEP-DIVE.md) - What are memory leaks, common causes (forgotten references, event listeners, circular references, unlimited caches), detection, prevention, fixing
- [Observability](./operating-system/11-OBSERVABILITY-DEEP-DIVE.md) - Logs, metrics, traces (three pillars), distributed tracing, observability vs monitoring, implementation, best practices
- [Cache Sizing](./operating-system/12-CACHE-SIZING-DEEP-DIVE.md) - Principles for determining cache size, working set analysis, access patterns, memory constraints, hit rate targets, monitoring and adjustment
- [System Idle and Background Processes](./operating-system/13-SYSTEM-IDLE-AND-BACKGROUND-PROCESSES-DEEP-DIVE.md) - What OS does when idle, interrupts, daemons, polling vs event-driven, event handling, power management, cron jobs
- [Garbage Collection](./operating-system/14-GARBAGE-COLLECTION-DEEP-DIVE.md) - Automatic memory management, GC algorithms (mark-and-sweep, copying, generational), incremental and concurrent GC, GC in different languages, performance tuning, real-time GC challenges
- [Heap Memory Allocation](./operating-system/15-HEAP-MEMORY-ALLOCATION-DEEP-DIVE.md) - Dynamic memory allocation, heap vs stack, allocation algorithms (first fit, best fit, worst fit), memory fragmentation, memory pools, heap in different languages, performance and security considerations
- [Inter-Process Communication (IPC)](./operating-system/16-INTER-PROCESS-COMMUNICATION-DEEP-DIVE.md) - IPC methods (pipes, named pipes, message queues, shared memory, sockets, signals, memory-mapped files), performance comparison, choosing the right method, synchronization, best practices
- [Deadlock](./operating-system/17-DEADLOCK-DEEP-DIVE.md) - What is deadlock, four necessary conditions (mutual exclusion, hold and wait, no preemption, circular wait), deadlock detection, prevention strategies, deadlock avoidance (Banker's algorithm), deadlock recovery, best practices
- [Context Switching](./operating-system/18-CONTEXT-SWITCHING-DEEP-DIVE.md) - What is context switching, why it's needed, what is context (PCB), context switch process, context switch cost and overhead, minimizing context switches, user vs kernel context switches, performance impact, best practices
- [Process, Thread, and Scheduling](./operating-system/19-PROCESS-THREAD-SCHEDULING-DEEP-DIVE.md) - What are processes and threads, process vs thread, process lifecycle (new, ready, running, waiting, terminated), thread lifecycle, CPU scheduling, scheduling algorithms (FCFS, SJF, Round Robin, Priority, Multilevel), process states, thread states, multiprocessing vs multithreading, synchronization (locks, semaphores, mutex, condition variables), best practices
- [Memory Management](./operating-system/20-MEMORY-MANAGEMENT-DEEP-DIVE.md) - What is memory management, memory hierarchy (registers, cache, RAM, disk), virtual memory, memory allocation (static, dynamic), memory deallocation (manual, automatic), memory fragmentation (external, internal), garbage collection (mark and sweep, copying, generational), memory leaks, memory protection, best practices
- [File Systems](./operating-system/21-FILE-SYSTEMS-DEEP-DIVE.md) - What is file system, file system structure (boot block, superblock, inode table, data blocks), file system types (ext4, NTFS, HFS+/APFS, ZFS), inode system (file metadata, structure), directory structure (directory as file, operations), file allocation methods (contiguous, linked, indexed), file system operations (read, write, delete), journaling file systems (journaling modes, benefits), distributed file systems (NFS, CIFS/SMB, HDFS), best practices
- [Operating System Scheduling](./operating-system/22-OPERATING-SYSTEM-SCHEDULING-DEEP-DIVE.md) - What is CPU scheduling, scheduling objectives (maximize throughput, minimize response time, minimize turnaround time, fairness), scheduling algorithms (FCFS, SJF, Round Robin, Priority, Multilevel), preemptive vs non-preemptive scheduling, multi-level scheduling (multi-level queue, multi-level feedback queue), real-time scheduling (hard/soft real-time, Rate Monotonic, EDF), scheduling in different systems (Linux CFS, Windows, macOS), best practices

#### Database

- [Database Indexing](./database/01-DATABASE-INDEXING-DEEP-DIVE.md) - B-Trees, index types, composite indexes, query optimization, best practices
- [Database Transactions](./database/02-DATABASE-TRANSACTIONS-DEEP-DIVE.md) - ACID properties, isolation levels, concurrency problems, locking, MVCC, distributed transactions
- [Database Replication](./database/03-DATABASE-REPLICATION-DEEP-DIVE.md) - Master-slave architecture, binary logs, replication process, replication lag, failover
- [Database Sharding](./database/04-DATABASE-SHARDING-DEEP-DIVE.md) - Sharding strategies, shard key selection, querying across shards, rebalancing
- [Query Optimization](./database/05-QUERY-OPTIMIZATION-DEEP-DIVE.md) - Query execution, EXPLAIN plans, join optimization, aggregation, subquery optimization, monitoring
- [Object-Relational Impedance Mismatch](./database/06-OBJECT-RELATIONAL-IMPEDANCE-MISMATCH-DEEP-DIVE.md) - OOP vs Relational paradigms, mismatches (granularity, inheritance, identity, association), ORM solutions, Active Record vs Data Mapper
- [Event Sourcing and CQRS](./database/07-EVENT-SOURCING-CQRS-DEEP-DIVE.md) - Event Sourcing (store events, derive state), CQRS (separate read/write models), Event Sourcing + CQRS together, Saga pattern (distributed transactions with compensation)
- [Database Connection Pooling](./database/08-DATABASE-CONNECTION-POOLING-DEEP-DIVE.md) - What is connection pooling, why it's needed, how it works, configuration (size, timeouts, validation), connection lifecycle, sizing guidelines, monitoring, common issues and solutions, best practices
- [Database Migrations](./database/09-DATABASE-MIGRATIONS-DEEP-DIVE.md) - What are migrations, why they're needed, migration tools (Rails, Django, Alembic, Flyway, Liquibase), writing migrations, best practices, rollback strategies, zero-downtime migrations, data migrations, testing migrations
- [Distributed Locking](./database/10-DISTRIBUTED-LOCKING-DEEP-DIVE.md) - What is distributed locking, why it's needed, challenges (network partitions, clock skew), algorithms (simple lock, ownership, Redlock), implementations (Redis, Zookeeper, Database, etcd), timeout and deadlocks, reentrancy, best practices, common pitfalls
- [CAP Theorem and BASE](./database/11-CAP-THEOREM-BASE-DEEP-DIVE.md) - CAP theorem (Consistency, Availability, Partition Tolerance), CP vs AP systems, why CA is not practical, PACELC extension, BASE properties (Basically Available, Soft state, Eventual consistency), ACID vs BASE comparison, tunable consistency, real-world examples
- [Database Normalization and Denormalization](./database/12-DATABASE-NORMALIZATION-DENORMALIZATION-DEEP-DIVE.md) - What is normalization, normal forms (1NF, 2NF, 3NF, BCNF), why normalize, what is denormalization, why denormalize, denormalization techniques, when to normalize vs denormalize, best practices
- [Database Backup and Recovery](./database/13-DATABASE-BACKUP-RECOVERY-DEEP-DIVE.md) - What is database backup, why backups are needed, types of backups (full, incremental, differential, continuous), backup strategies, backup storage, recovery strategies, point-in-time recovery (PITR), disaster recovery (RTO, RPO), backup testing, best practices
- [Query Optimization](./database/14-QUERY-OPTIMIZATION-DEEP-DIVE.md) - What is query optimization, query execution process, execution plans, understanding EXPLAIN, common performance issues, optimization techniques, index optimization, join optimization, query rewriting, statistics and cardinality, best practices, common mistakes
- [Database Sharding](./database/15-DATABASE-SHARDING-DEEP-DIVE.md) - What is database sharding, why it's needed, sharding vs replication, sharding strategies (range-based, hash-based, directory-based, geographic), horizontal vs vertical sharding, shard key selection, sharding architectures, shard management, cross-shard queries, shard rebalancing, best practices, common challenges
- [Consistency Models](./database/16-CONSISTENCY-MODELS-DEEP-DIVE.md) - What is consistency, ACID consistency, CAP theorem and consistency, consistency models (strong, eventual, weak, causal, session), read-your-writes consistency, monotonic reads, consistency in distributed systems, trade-offs (consistency vs availability), best practices
- [Database Partitioning](./database/17-DATABASE-PARTITIONING-DEEP-DIVE.md) - What is database partitioning, why it's needed, partitioning vs sharding, types of partitioning (range, hash, list, composite), horizontal vs vertical partitioning, partitioning strategies (date-based, geographic, hash-based), partition key selection, partition pruning, partition maintenance (adding, dropping, merging, splitting), best practices, common challenges
- [Database Connection Management](./database/18-DATABASE-CONNECTION-MANAGEMENT-DEEP-DIVE.md) - What is connection management, connection lifecycle (creation, usage, return, cleanup), connection pooling, connection string configuration, connection timeouts (connection, query, idle), connection health checks, connection leaks (detection, prevention), transaction management, best practices, common issues
- [Database Performance Tuning](./database/19-DATABASE-PERFORMANCE-TUNING-DEEP-DIVE.md) - What is performance tuning, performance metrics (query response time, throughput, resource usage), query performance (slow query identification, optimization), index optimization, database configuration (buffer pool, connection limits, query cache), connection pooling optimization, caching strategies (database-level, application-level), partitioning and sharding, monitoring and profiling, best practices
- [Database Locking Mechanisms](./database/20-DATABASE-LOCKING-MECHANISMS-DEEP-DIVE.md) - What is database locking, why it's needed, types of locks (shared/read, exclusive/write, intent), lock granularity (row-level, page-level, table-level), lock modes (shared, exclusive, update), lock compatibility matrix, deadlocks (detection, resolution), lock escalation, optimistic vs pessimistic locking, best practices, common issues
- [MVCC (Multi-Version Concurrency Control)](./database/21-MVCC-MULTI-VERSION-CONCURRENCY-CONTROL-DEEP-DIVE.md) - What is MVCC, why it's needed, how MVCC works, version management (creation, visibility, cleanup), read consistency (snapshot isolation), write operations, transaction isolation with MVCC, MVCC vs locking (advantages, disadvantages), MVCC implementation (PostgreSQL, MySQL InnoDB), best practices, common issues (version bloat, long transactions, vacuum lag)
- [Database Query Execution](./database/22-DATABASE-QUERY-EXECUTION-DEEP-DIVE.md) - What is query execution, query execution pipeline (parsing, optimization, planning, execution), query parsing (lexical analysis, syntax analysis, validation), query optimization (generate plans, estimate costs, choose best), execution plans (sequential scan, index scan, index only scan, bitmap scan), query execution methods, join algorithms (nested loop, hash, merge), sorting and aggregation, query execution performance (I/O, CPU, memory), best practices
- [Database Storage Engines](./database/23-DATABASE-STORAGE-ENGINES-DEEP-DIVE.md) - What is storage engine, storage engine architecture (storage layer, index layer, transaction layer, buffer layer), common storage engines, InnoDB (MySQL - ACID, row-level locking, foreign keys, crash recovery), MyISAM (MySQL - fast reads, no transactions, table-level locking), PostgreSQL storage (heap storage, MVCC, WAL), MongoDB storage engines (WiredTiger, MMAPv1), storage engine comparison, choosing storage engine, best practices
- [Transaction Logs and WAL](./database/24-TRANSACTION-LOGS-WAL-DEEP-DIVE.md) - What are transaction logs, why they matter (durability, recovery, replication), Write-Ahead Logging (WAL) principle, transaction log structure (transaction ID, operation type, old/new values, LSN), log record types (begin, update, commit, abort), log writing and flushing (buffering, flush strategies), checkpointing (full, incremental), log-based recovery (analysis, redo, undo phases), WAL in different databases (PostgreSQL, MySQL InnoDB, SQL Server), best practices
- [Database Query Caching](./database/25-DATABASE-QUERY-CACHING-DEEP-DIVE.md) - What is query caching, query cache types (result cache, plan cache), query cache architecture (cache key, value, storage), cache invalidation (data changes, schema changes, time-based), query cache configuration (size, type, limits), query cache in different databases (MySQL - deprecated, PostgreSQL - no built-in, SQL Server - plan cache), best practices, common issues (stale data, low hit rate, memory pressure)
- [Database Concurrency Control](./database/26-DATABASE-CONCURRENCY-CONTROL-DEEP-DIVE.md) - What is concurrency control, concurrency problems (lost update, dirty read, non-repeatable read, phantom read), concurrency control methods (locking, timestamp-based, optimistic, MVCC), locking-based (two-phase locking, lock types, granularity), timestamp-based (timestamp ordering, validation rules), optimistic (read, validation, write phases), MVCC (version storage, snapshot isolation), isolation levels and concurrency, best practices
- [Database Buffer Pool](./database/27-DATABASE-BUFFER-POOL-DEEP-DIVE.md) - What is buffer pool, buffer pool architecture (page frames, page table, free list, LRU list), page management (request, miss, hit, eviction), buffer pool algorithms (LRU, Clock, Adaptive), dirty pages (mark dirty, write back, clean), buffer pool configuration (size, instances, page size), buffer pool monitoring (hit rate, I/O, dirty pages), best practices, common issues (low hit rate, memory pressure, checkpoint storms)
- [Database Index Maintenance](./database/28-DATABASE-INDEX-MAINTENANCE-DEEP-DIVE.md) - What is index maintenance, index fragmentation (internal, external, causes, impact), index rebuilding (when to rebuild, online vs offline), index reorganizing (when to reorganize, reorganize vs rebuild), index statistics (what are statistics, why they matter, updating), index monitoring (fragmentation, usage, size, performance), automated maintenance (maintenance jobs, automation benefits), best practices, common issues (high fragmentation, stale statistics, maintenance overhead)

#### Security

- [Encryption and Hashing](./security/01-ENCRYPTION-HASHING-DEEP-DIVE.md) - Symmetric/asymmetric encryption, hash functions, password hashing, digital signatures, key management
- [SSL/TLS](./security/02-SSL-TLS-DEEP-DIVE.md) - TLS handshake, certificates, certificate authorities, cipher suites, perfect forward secrecy, performance optimization
- [Credential Storage](./security/03-CREDENTIAL-STORAGE-DEEP-DIVE.md) - Password hashing, database credentials, API keys, secret management services, key rotation, best practices
- [DDoS Defense](./security/04-DDOS-DEFENSE-DEEP-DIVE.md) - DDoS attack types, detection, defense strategies, rate limiting, protection services, incident response
- [API Authentication and Authorization](./security/05-API-AUTHENTICATION-AUTHORIZATION-DEEP-DIVE.md) - Authentication vs authorization, authentication methods (API keys, Basic Auth, JWT, OAuth 2.0, Sessions), OAuth 2.0 flows, OpenID Connect, authorization models (RBAC, ABAC), best practices, common security issues
- [API Security Best Practices](./security/06-API-SECURITY-BEST-PRACTICES-DEEP-DIVE.md) - What is API security, common API vulnerabilities (OWASP API Top 10), authentication and authorization, input validation, output encoding, rate limiting, HTTPS and TLS, API keys management, secrets management, security headers, error handling security, logging and monitoring, best practices, common mistakes
- [Security Vulnerabilities](./security/07-SECURITY-VULNERABILITIES-DEEP-DIVE.md) - What are security vulnerabilities, OWASP Top 10 (injection, broken authentication, sensitive data exposure, XXE, broken access control, security misconfiguration, XSS, insecure deserialization, known vulnerabilities, insufficient logging), injection attacks (SQL, command, LDAP), authentication vulnerabilities, sensitive data exposure, XML external entities (XXE), broken access control, security misconfiguration, cross-site scripting (XSS), insecure deserialization, using components with known vulnerabilities, insufficient logging and monitoring, prevention strategies (defense in depth, secure by design, regular audits), best practices

#### Programming Paradigm

- [OOP](./programming/01-OOP-DEEP-DIVE.md) - Classes and objects, inheritance, polymorphism, encapsulation, abstraction, composition, design patterns, SOLID principles
- [Functional Programming](./programming/02-FUNCTIONAL-PROGRAMMING-DEEP-DIVE.md) - Pure functions, immutability, higher-order functions, function composition, recursion, lazy evaluation
- [Design Patterns](./programming/03-DESIGN-PATTERNS-DEEP-DIVE.md) - Singleton, Inversion of Control, Law of Demeter, Active Record vs Data Mapper, Inheritance vs Composition, Anti-Corruption Layer, Separation of Concerns, DRY, Dependency Hell, Globals, Null References
- [Code Design Principles](./programming/04-CODE-DESIGN-PRINCIPLES-DEEP-DIVE.md) - High Cohesion, Loose Coupling, DRY, Refactoring, Code Comments, Design vs Architecture, Early Testing, Domain Logic in Stored Procedures
- [Closures and Generics](./programming/05-CLOSURES-GENERICS-DEEP-DIVE.md) - Closures (functions with memory), Generics (type parameters), Type Erasure, practical applications
- [Unicode](./programming/06-UNICODE-DEEP-DIVE.md) - Unicode standard, code points, encoding forms (UTF-8, UTF-16, UTF-32), common issues, normalization, best practices
- [Streaming](./programming/07-STREAMING-DEEP-DIVE.md) - Streaming vs batch processing, streaming patterns (event streaming, pipelines, windowing, backpressure), implementation, technologies
- [Mutable vs Immutable](./programming/08-MUTABLE-VS-IMMUTABLE-DEEP-DIVE.md) - Mutable and immutable data structures, pros and cons, performance comparison, thread safety, when to use each, persistent data structures, real-world examples
- [Code Refactoring](./programming/09-CODE-REFACTORING-DEEP-DIVE.md) - What is refactoring, code smells, refactoring techniques (extract method, extract class, rename, move, replace conditional), refactoring safety, workflow, best practices
- [Data Serialization](./programming/10-DATA-SERIALIZATION-DEEP-DIVE.md) - What is serialization, serialization formats (JSON, XML, Protocol Buffers, Avro, MessagePack, BSON), comparison of formats, choosing the right format, best practices, schema evolution
- [Error Handling](./programming/11-ERROR-HANDLING-DEEP-DIVE.md) - What is error handling, types of errors, error handling strategies (fail fast, fail safe, retry), exception handling, error codes vs exceptions, error propagation, error recovery, error logging, error handling patterns, best practices, common mistakes
- [Algorithms and Data Structures](./programming/12-ALGORITHMS-DATA-STRUCTURES-DEEP-DIVE.md) - What are algorithms and data structures, time complexity (Big O notation), space complexity, common data structures (array, linked list, stack, queue, hash table, binary tree, heap), common algorithms (binary search, merge sort, quick sort, BFS, DFS), sorting algorithms comparison, searching algorithms, graph algorithms (shortest path, MST, topological sort), dynamic programming, best practices
- [Advanced Design Patterns](./programming/13-DESIGN-PATTERNS-ADVANCED-DEEP-DIVE.md) - What are design patterns, creational patterns (Singleton, Factory, Builder), structural patterns (Adapter, Decorator, Facade), behavioral patterns (Observer, Strategy, Command), concurrency patterns (Producer-Consumer, Thread Pool, Lock), architectural patterns (MVC, Repository, Unit of Work), anti-patterns (God Object, Spaghetti Code, Copy-Paste), pattern selection (when to use, don't over-engineer, trade-offs), best practices
- [Code Quality Metrics](./programming/14-CODE-QUALITY-METRICS-DEEP-DIVE.md) - What are code quality metrics, code complexity metrics (cyclomatic complexity, cognitive complexity), code coverage metrics (line, branch, function, statement coverage), maintainability metrics (maintainability index, code duplication), code smell metrics (long methods, large classes, duplicate code), technical debt metrics (debt ratio, debt time, debt cost), performance metrics (execution time, memory, CPU), security metrics (vulnerabilities, security issues, dependencies), best practices

#### Software Development Process

- [DDD](./software-development/01-DDD-DEEP-DIVE.md) - Domain-Driven Design: ubiquitous language, bounded contexts, tactical patterns, aggregates, domain events, implementation guide
- [TDD](./software-development/02-TDD-DEEP-DIVE.md) - Test-Driven Development: Red-Green-Refactor cycle, writing good tests, test types, mocking, TDD patterns and practices
- [BDD](./software-development/03-BDD-DEEP-DIVE.md) - Behavior-Driven Development: Given-When-Then, Gherkin syntax, BDD tools, step definitions, collaboration, living documentation
- [CI/CD](./software-development/04-CICD-DEEP-DIVE.md) - Continuous Integration and Continuous Delivery: CI/CD pipeline, automated testing, deployment strategies, best practices, challenges
- [Monolith vs Microservices](./software-development/05-MONOLITH-MICROSERVICES-DEEP-DIVE.md) - Architecture comparison, when to use each, migration strategies (strangler pattern), challenges (communication, consistency, service discovery)
- [Green Field vs Brown Field](./software-development/06-GREEN-BROWN-FIELD-DEEP-DIVE.md) - Green field (new projects) vs brown field (legacy projects), advantages and challenges, working with legacy code, migration strategies
- [Idempotency](./software-development/07-IDEMPOTENCY-DEEP-DIVE.md) - What is idempotency, why it matters, idempotency in HTTP, implementing idempotency (idempotency keys), patterns, challenges
- [Performance Testing](./software-development/08-PERFORMANCE-TESTING-DEEP-DIVE.md) - Types of performance testing (load, stress, spike, volume, endurance), performance metrics, testing tools (JMeter, Gatling, k6), best practices
- [High Availability and Disaster Recovery](./software-development/09-HIGH-AVAILABILITY-DISASTER-RECOVERY-DEEP-DIVE.md) - High availability patterns (redundancy, active-passive, active-active), disaster recovery strategies, backup strategies (3-2-1 rule), failover mechanisms, measuring availability
- [Git Branching Strategies](./software-development/10-GIT-BRANCHING-STRATEGIES-DEEP-DIVE.md) - Git vs Mercurial branching, Git Flow, GitHub Flow, GitLab Flow, Trunk-Based Development, feature branches, release and hotfix branches, choosing the right strategy, best practices
- [Code Review](./software-development/11-CODE-REVIEW-DEEP-DIVE.md) - What is code review, why it matters, what to review (correctness, design, performance, security, testing), giving and receiving feedback, best practices, automated code review
- [Graceful Shutdown](./software-development/12-GRACEFUL-SHUTDOWN-DEEP-DIVE.md) - What is graceful shutdown, why it's needed, shutdown signals (SIGTERM, SIGINT), shutdown process, connection draining, request completion, resource cleanup, health checks during shutdown, shutdown timeout, implementation patterns, best practices
- [Leader Election](./software-development/13-LEADER-ELECTION-DEEP-DIVE.md) - What is leader election, why it's needed, leader election algorithms (Bully, Ring), implementations (Zookeeper, etcd, Redis, Database, Raft), leader failure and re-election, split-brain problem, best practices, common pitfalls
- [Event-Driven Architecture](./software-development/14-EVENT-DRIVEN-ARCHITECTURE-DEEP-DIVE.md) - What is event-driven architecture, event-driven vs request-response, event-driven patterns, event sourcing, CQRS, event streaming, event bus and message brokers, event ordering and consistency, event versioning, event replay, best practices, common challenges
- [Testing Strategies](./software-development/15-TESTING-STRATEGIES-DEEP-DIVE.md) - What is testing, why it matters, testing pyramid (unit, integration, E2E), unit testing, integration testing, end-to-end testing, test types by purpose (functional, performance, security, regression), TDD (Red-Green-Refactor), BDD (Given-When-Then), testing best practices, test coverage, common testing mistakes
- [Distributed Systems Patterns](./software-development/16-DISTRIBUTED-SYSTEMS-PATTERNS-DEEP-DIVE.md) - What are distributed systems patterns, communication patterns (request-response, pub/sub, message queue, request-reply), consistency patterns (2PC, Saga, Event Sourcing), reliability patterns (circuit breaker, retry, bulkhead, timeout), scalability patterns (sharding, replication, caching, load balancing), data patterns (CQRS, materialized views, database per service), coordination patterns (leader election, distributed locking, service discovery), best practices
- [Software Architecture Patterns](./software-development/17-SOFTWARE-ARCHITECTURE-PATTERNS-DEEP-DIVE.md) - What are architecture patterns, layered architecture, microservices architecture, monolithic architecture, event-driven architecture, serverless architecture, hexagonal architecture (ports and adapters), clean architecture, choosing architecture (factors: team size, complexity, scale, technology), best practices
- [System Design Principles](./software-development/18-SYSTEM-DESIGN-PRINCIPLES-DEEP-DIVE.md) - What is system design, scalability principles (horizontal vs vertical scaling, scalability patterns), reliability principles (fault tolerance, error handling), availability principles (high availability, availability patterns), performance principles (optimization, metrics), security principles (security by design, security patterns), maintainability principles (code quality, architecture quality), system design process (requirements, capacity estimation, architecture, detailed design, bottlenecks, scaling), common patterns (load balancer, read replicas, caching, CDN), best practices
- [Distributed Systems Fundamentals](./software-development/19-DISTRIBUTED-SYSTEMS-FUNDAMENTALS-DEEP-DIVE.md) - What are distributed systems, why distributed systems (scalability, reliability, performance, geographic distribution), characteristics (concurrency, no global clock, independent failures, heterogeneity), challenges (network partitions, partial failures, consistency, latency), distributed system models (client-server, peer-to-peer, microservices), communication (request-response, message passing, RPC, pub/sub), consistency (strong, eventual, weak, CAP theorem), fault tolerance (redundancy, replication, health monitoring, circuit breakers), scalability (horizontal, vertical, load balancing, caching), best practices

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

This knowledge base is created for educational purposes. Original questions and structure based on:

- [backend-swe-interview-questions](https://github.com/tamhoang1412/backend-swe-interview-questions)
- [Back-End-Developer-Interview-Questions](https://github.com/arialdomartini/Back-End-Developer-Interview-Questions)
