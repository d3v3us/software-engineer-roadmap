# Анализ вопросов из fabinhojorge/Back-End-Developer-Interview-Questions-1

## Источник
https://github.com/fabinhojorge/Back-End-Developer-Interview-Questions-1

## Категории вопросов

### 1. General Questions

#### ✅ Уже покрыто:
- Functional Programming → `programming/02-FUNCTIONAL-PROGRAMMING-DEEP-DIVE.md`
- Design vs Architecture → частично в `programming/04-CODE-DESIGN-PRINCIPLES-DEEP-DIVE.md`
- TCP vs HTTP → `networking/01-TCP-IP-DEEP-DIVE.md`, `networking/02-HTTP-HTTPS-DEEP-DIVE.md`
- Client-side vs Server-side rendering → `networking/08-CLIENT-SERVER-RENDERING-DEEP-DIVE.md`
- Reliable communication protocol → `networking/09-RELIABLE-COMMUNICATION-PROTOCOLS-DEEP-DIVE.md`
- Mutable vs Immutable → `programming/08-MUTABLE-VS-IMMUTABLE-DEEP-DIVE.md`
- Object-Relational impedance mismatch → `database/06-OBJECT-RELATIONAL-IMPEDANCE-MISMATCH-DEEP-DIVE.md`
- Cache sizing → `operating-system/12-CACHE-SIZING-DEEP-DIVE.md`

#### ❌ Отсутствует:
1. **Functional Programming Importance** - детальное объяснение важности функционального программирования
2. **Design vs Architecture vs Functionality vs Aesthetic** - детальное сравнение
3. **Browser Profit Models** - как компании зарабатывают на браузерах (не backend, но может быть интересно)
4. **TCP Socket Overhead** - детальное объяснение overhead открытия TCP сокета
5. **Encapsulation Importance** - важность инкапсуляции
6. **Real-Time Systems and Heap Memory** - связь между real-time системами и heap memory
7. **Null Reference Removal** - как убрать null references и последствия

---

### 2. Open Questions

#### ✅ Уже покрыто:
- Fault tolerance → частично в `software-development/16-DISTRIBUTED-SYSTEMS-PATTERNS-DEEP-DIVE.md`
- Failures in Distributed Systems → `software-development/19-DISTRIBUTED-SYSTEMS-FUNDAMENTALS-DEEP-DIVE.md`
- Network partitions → `database/11-CAP-THEOREM-BASE-DEEP-DIVE.md`
- Request/Reply vs Publish/Subscribe → `networking/24-MESSAGE-QUEUES-DEEP-DIVE.md`, `networking/56-EVENT-STREAMING-PUB-SUB-DEEP-DIVE.md`

#### ❌ Отсутствует:
1. **Fault Tolerance in Web Applications** - детально про fault tolerance в веб-приложениях
2. **Fault Tolerance in Desktop Applications** - fault tolerance в desktop приложениях
3. **Reconciliation after Network Partitions** - детально про reconciliation после network partitions
4. **Fallacies of Distributed Computing** - 8 fallacies of distributed computing

---

### 3. Questions about Software Lifecycle and Team Management

#### ❌ Не добавляем - это не технические вопросы (soft skills, management)

---

### 4. Questions about logic and algorithms

#### ✅ Уже покрыто:
- FIFO/LIFO → `go/29-GO-STACK-QUEUE-DEEP-DIVE.md`
- Stack overflow → частично в `operating-system/10-MEMORY-LEAKS-DEEP-DIVE.md`
- Memory leak → `operating-system/10-MEMORY-LEAKS-DEEP-DIVE.md`
- Garbage collection → `operating-system/14-GARBAGE-COLLECTION-DEEP-DIVE.md`
- Message broker → `networking/24-MESSAGE-QUEUES-DEEP-DIVE.md`
- Web server → частично в `go/21-GO-HTTP-DEEP-DIVE.md`
- Sorting large files → частично в `operating-system/08-SORTING-ALGORITHMS-DEEP-DIVE.md`

#### ❌ Отсутствует (специфичные алгоритмические задачи):
1. **Tail-Recursive Factorial** - tail recursion (частично покрыто в functional programming)
2. **REPL and RPN Calculator** - REPL и RPN калькулятор (не критично для backend)
3. **Defragger Utility Design** - дизайн утилиты дефрагментации (не критично)
4. **Random Mazes Generation** - генерация случайных лабиринтов (не критично)
5. **Unique Random Numbers** - генерация уникальных случайных чисел (не критично)
6. **Sorting 10GB/10TB Files** - сортировка очень больших файлов (может быть полезно)
7. **rnd() Function Implementation** - реализация функции rnd() (не критично)

**Примечание**: Большинство этих вопросов - это алгоритмические задачи, которые не требуют отдельного deep-dive файла. Они больше подходят для практических упражнений.

---

### 5. Questions about Software Architecture

#### ✅ Уже покрыто:
- Cache usefulness → `operating-system/07-CACHING-DEEP-DIVE.md`
- Event-Driven Architecture → `software-development/14-EVENT-DRIVEN-ARCHITECTURE-DEEP-DIVE.md`
- Code readability → частично в `programming/09-CODE-REFACTORING-DEEP-DIVE.md`
- Scale out vs scale up → `software-development/29-SOFTWARE-SCALABILITY-DEEP-DIVE.md`
- Failover and sessions → частично в `networking/43-SESSION-MANAGEMENT-DEEP-DIVE.md`
- CQRS → `database/07-EVENT-SOURCING-CQRS-DEEP-DIVE.md`
- Three-Tier architecture → частично в `software-development/17-SOFTWARE-ARCHITECTURE-PATTERNS-DEEP-DIVE.md`
- Scalability design → `software-development/29-SOFTWARE-SCALABILITY-DEEP-DIVE.md`
- Publish-Subscribe disadvantages → частично в `networking/24-MESSAGE-QUEUES-DEEP-DIVE.md`
- Performance vs Scalability → частично в `software-development/29-SOFTWARE-SCALABILITY-DEEP-DIVE.md`
- Cloud Ready characteristics → частично в `software-development/21-CLOUD-COMPUTING-FUNDAMENTALS-DEEP-DIVE.md`
- SOA vs Microservices → `software-development/62-SOA-SERVICE-ORIENTED-ARCHITECTURE-DEEP-DIVE.md`

#### ❌ Отсутствует:
1. **C10k Problem** - проблема C10k и стратегии решения
2. **Decentralized P2P System Design** - дизайн децентрализованной P2P системы
3. **CGI Scaling Issues** - почему CGI не масштабируется
4. **Vendor Lock-in Defense** - защита от vendor lock-in
5. **CPUs Since 80s and Programming Impact** - изменения в CPU и влияние на программирование
6. **Performance in Lifecycle** - когда учитывать performance в lifecycle
7. **Denial of Service from Design** - DoS из-за дизайна/архитектуры
8. **Tight Coupling When OK** - когда допустимо tight coupling
9. **Unity of Design** - единство дизайна и архитектура

---

### 6. Questions about Service Oriented Architecture and Microservices

#### ✅ Уже покрыто:
- SOA long-lived transactions → `software-development/48-SAGA-PATTERN-DEEP-DIVE.md`
- SOA vs Microservices → `software-development/62-SOA-SERVICE-ORIENTED-ARCHITECTURE-DEEP-DIVE.md`
- Web services versioning → `networking/20-API-VERSIONING-DEEP-DIVE.md`
- Transaction vs compensation → `software-development/48-SAGA-PATTERN-DEEP-DIVE.md`
- Microservices pros/cons → `software-development/05-MONOLITH-MICROSERVICES-DEEP-DIVE.md`

#### ❌ Отсутствует:
1. **Microservice Too Micro** - когда microservice слишком маленький

---

### 7. Questions about Security

#### ✅ Уже покрыто:
- Two Factor Authentication → частично в `security/05-API-AUTHENTICATION-AUTHORIZATION-DEEP-DIVE.md`

#### ❌ Отсутствует:
1. **Two Factor Authentication Implementation** - детальная реализация 2FA

---

### 8. Questions about Design Patterns

#### ✅ Уже покрыто:
- Design Patterns → `programming/03-DESIGN-PATTERNS-DEEP-DIVE.md`, `programming/13-DESIGN-PATTERNS-ADVANCED-DEEP-DIVE.md`

---

### 9. Questions about Code Design

#### ✅ Уже покрыто:
- Code Design → `programming/04-CODE-DESIGN-PRINCIPLES-DEEP-DIVE.md`

---

### 10. Questions about Languages

#### ❌ Не добавляем - это вопросы про конкретные языки, не общие концепции

---

### 11. Web Questions

#### ✅ Уже покрыто:
- Большинство web вопросов покрыто в networking секции

---

### 12. Databases Questions

#### ✅ Уже покрыто:
- Большинство database вопросов покрыто в database секции

---

### 13. NoSQL Questions

#### ✅ Уже покрыто:
- NoSQL → `database/30-NOSQL-DATABASES-DEEP-DIVE.md`

---

### 14. Code Versioning Questions

#### ✅ Уже покрыто:
- Versioning → `software-development/32-SOFTWARE-VERSIONING-DEEP-DIVE.md`, `software-development/10-GIT-BRANCHING-STRATEGIES-DEEP-DIVE.md`

---

### 15. Concurrency Questions

#### ✅ Уже покрыто:
- Concurrency → `operating-system/02-CONCURRENCY-DEEP-DIVE.md`, `programming/16-CONCURRENCY-PARALLELISM-DEEP-DIVE.md`

---

### 16. Distributed Systems Questions

#### ✅ Уже покрыто:
- Distributed Systems → `software-development/19-DISTRIBUTED-SYSTEMS-FUNDAMENTALS-DEEP-DIVE.md`, `software-development/16-DISTRIBUTED-SYSTEMS-PATTERNS-DEEP-DIVE.md`

---

### 17. Bill Gates Style Questions

#### ❌ Не добавляем - это не технические вопросы

---

### 18. Questions based on snippets of code

#### ❌ Не добавляем - это практические задачи, не концептуальные темы

---

## Итоговый список недостающих тем

### Высокий приоритет (важные для backend):
1. **TCP Socket Overhead** - детальное объяснение overhead
2. **Encapsulation Importance** - важность инкапсуляции
3. **Real-Time Systems and Heap Memory** - связь real-time и heap memory
4. **Fault Tolerance in Web Applications** - fault tolerance в веб-приложениях
5. **Reconciliation after Network Partitions** - reconciliation после partitions
6. **Fallacies of Distributed Computing** - 8 fallacies
7. **C10k Problem** - проблема C10k и решения
8. **CGI Scaling Issues** - почему CGI не масштабируется
9. **Vendor Lock-in Defense** - защита от vendor lock-in
10. **Performance in Lifecycle** - когда учитывать performance
11. **Denial of Service from Design** - DoS из-за дизайна
12. **Tight Coupling When OK** - когда допустимо tight coupling
13. **Two Factor Authentication Implementation** - детальная реализация 2FA
14. **Microservice Too Micro** - когда microservice слишком маленький

### Средний приоритет:
15. **Functional Programming Importance** - важность функционального программирования
16. **Design vs Architecture vs Functionality vs Aesthetic** - детальное сравнение
17. **Null Reference Removal** - как убрать null и последствия
18. **Fault Tolerance in Desktop Applications** - fault tolerance в desktop
19. **Decentralized P2P System Design** - дизайн P2P системы
20. **CPUs Since 80s and Programming Impact** - изменения CPU и влияние
21. **Unity of Design** - единство дизайна
22. **Sorting Large Files (10GB/10TB)** - сортировка очень больших файлов

### Низкий приоритет (специфичные или не критичные):
23. **Browser Profit Models** - как браузеры зарабатывают (не backend)
24. **Tail-Recursive Factorial** - tail recursion (частично покрыто)
25. **REPL and RPN Calculator** - не критично для backend
26. **Defragger Utility** - не критично
27. **Random Mazes** - не критично
28. **Unique Random Numbers** - не критично
29. **rnd() Function** - не критично

---

## Рекомендации

### Добавить обязательно (высокий приоритет):
1. TCP Socket Overhead Deep Dive
2. Encapsulation Importance Deep Dive
3. Real-Time Systems and Heap Memory Deep Dive
4. Fault Tolerance in Web Applications Deep Dive
5. Reconciliation after Network Partitions Deep Dive
6. Fallacies of Distributed Computing Deep Dive
7. C10k Problem Deep Dive
8. CGI Scaling Issues Deep Dive
9. Vendor Lock-in Defense Deep Dive
10. Performance in Lifecycle Deep Dive
11. Denial of Service from Design Deep Dive
12. Tight Coupling When OK Deep Dive
13. Two Factor Authentication Implementation Deep Dive
14. Microservice Too Micro Deep Dive

### Добавить по желанию (средний приоритет):
15. Functional Programming Importance Deep Dive
16. Design vs Architecture vs Functionality vs Aesthetic Deep Dive
17. Null Reference Removal Deep Dive
18. Fault Tolerance in Desktop Applications Deep Dive
19. Decentralized P2P System Design Deep Dive
20. CPUs Since 80s and Programming Impact Deep Dive
21. Unity of Design Deep Dive
22. Sorting Large Files Deep Dive

### Можно пропустить:
- Browser profit models (не backend)
- Алгоритмические задачи (практические упражнения)
- Soft skills вопросы
- Code snippets вопросы

