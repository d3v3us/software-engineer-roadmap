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

#### Security

- [Encryption and Hashing](./security/01-ENCRYPTION-HASHING-DEEP-DIVE.md) - Symmetric/asymmetric encryption, hash functions, password hashing, digital signatures, key management
- [SSL/TLS](./security/02-SSL-TLS-DEEP-DIVE.md) - TLS handshake, certificates, certificate authorities, cipher suites, perfect forward secrecy, performance optimization
- [Credential Storage](./security/03-CREDENTIAL-STORAGE-DEEP-DIVE.md) - Password hashing, database credentials, API keys, secret management services, key rotation, best practices
- [DDoS Defense](./security/04-DDOS-DEFENSE-DEEP-DIVE.md) - DDoS attack types, detection, defense strategies, rate limiting, protection services, incident response
- [API Authentication and Authorization](./security/05-API-AUTHENTICATION-AUTHORIZATION-DEEP-DIVE.md) - Authentication vs authorization, authentication methods (API keys, Basic Auth, JWT, OAuth 2.0, Sessions), OAuth 2.0 flows, OpenID Connect, authorization models (RBAC, ABAC), best practices, common security issues

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
